# n8n AI Agent 實戰：打造你的第一個 AI 客服

> 作者：Jerry Lee | n8n 自動化工程師
> 難度：⭐⭐⭐☆☆（建議先讀完上一篇）

---

2026 年了，AI Agent 已經不是 buzzword，而是真正能省下 80% 客服時間的實用工具。這篇文章帶你在 n8n 裡從零打造一個 **AI 客服 Agent**：能理解客戶問題、查詢內部資料庫、回答問題，不懂的還會轉接真人。

## 什麼是 AI Agent？

跟傳統 chatbot（關鍵字觸發固定回覆）不同，AI Agent 有四個核心能力：

1. **理解意圖**：客戶說「多少錢」跟「運費怎麼算」是不同的意思
2. **查詢資料**：從產品目錄、FAQ 庫撈出相關資訊
3. **推理決策**：判斷這個問題可以自己回答，還是要轉接真人
4. **執行行動**：發送 Slack 通知、建立 CRM ticket

n8n 的 AI Agent 節點把這些能力原生整合好了。不用架 LangChain，不用寫複雜的 agent loop。

## 架構圖

```
客戶訊息 → Webhook → OpenAI 意圖分類 → Code Node 查詢產品目錄
                                              ↓
                           ┌─────────────────┴──────────────┐
                           ↓                                ↓
                    產品存在 → 組合報價回覆              產品不存在 → Slack 轉接真人
```

## Step 1：建立 Webhook 接收訊息

1. 新增 **Webhook** 節點
2. Method: `POST`
3. Path: `/ai-customer-service`
4. Response Mode: `Response Node`

這個 webhook 會接收來自網站表單、LINE、或 Telegram 的訊息。先用 Postman 或 curl 測試：

```bash
curl -X POST http://localhost:5678/webhook/ai-customer-service \
  -H "Content-Type: application/json" \
  -d '{"name":"張先生","message":"你們的 M5 螺絲 1000 顆多少錢？"}'
```

## Step 2：OpenAI 意圖分類

新增 **OpenAI** 節點 → 選擇 `gpt-4o-mini`（便宜又快）。

System Prompt（系統提示詞）：

```
你是一個客服意圖分類 AI。分析客戶訊息，回傳純 JSON：

{
  "intent": "price_inquiry|shipping|warranty|complaint|greeting|other",
  "keywords": ["產品關鍵字"],
  "quantity": 數量或 null,
  "sentiment": "positive|neutral|negative",
  "language": "zh|en"
}

規則：
- 詢問價格 → price_inquiry
- 詢問運送 → shipping
- 抱怨 → complaint
- 打招呼 → greeting
```

User Message: `{{ $json.message }}`

**重要設定：** Options → Response Format → JSON（這樣 AI 才不會回多餘的 markdown）

## Step 3：Code Node 查詢產品目錄

新增 **IF** 節點判斷 `intent === "price_inquiry"` → 進入 Code Node。

Code Node 內建一個產品目錄：

```javascript
const catalog = [
  { id: "M5-001", name: "M5 不鏽鋼螺絲", unit: "顆", pricing: [
    { qty_min: 1, qty_max: 99, price: 2 },
    { qty_min: 100, qty_max: 999, price: 1.5 },
    { qty_min: 1000, qty_max: null, price: 1.1 }
  ]},
  { id: "NUT-M5", name: "M5 六角螺帽", unit: "顆", pricing: [
    { qty_min: 1, qty_max: 99, price: 1 },
    { qty_min: 100, qty_max: 999, price: 0.7 },
    { qty_min: 1000, qty_max: null, price: 0.5 }
  ]}
];

const aiResult = $input.first().json;  // OpenAI 回傳的分類結果
const keywords = (aiResult.keywords || []).map(k => k.toLowerCase());

// 模糊比對產品
const matches = catalog.filter(p => {
  const searchText = (p.id + p.name).toLowerCase();
  return keywords.some(kw => searchText.includes(kw));
});

if (matches.length === 0) {
  return { found: false, reply: "抱歉，我找不到相關產品。請提供更多資訊。" };
}

const product = matches[0];
const qty = aiResult.quantity || 100;
// 找對應的價格級距
const tier = product.pricing.find(t => qty >= t.qty_min && (t.qty_max === null || qty <= t.qty_max));
const total = tier ? tier.price * qty : null;

return {
  found: true,
  reply: `您好 ${$json.name || "客戶"}，\n\n${product.name}：\n• 數量：${qty} ${product.unit}\n• 單價：$${tier.price}\n• 總價：$${total}\n• 交期：約 7-10 工作天\n\n需要正式報價單請回覆「報價」！`
};
```

## Step 4：IF 路由

- 找到產品 → **Respond to Webhook**（回覆給客戶）
- 找不到 → **Slack** 節點（發送通知給真人客服：「客戶問了『XX』但目錄中無匹配，請協助」）

## Step 5：Respond to Webhook

最後加一個 **Respond to Webhook** 節點：
- Respond With: JSON
- Response Body: `{{ $json.reply }}`

## 完整測試

```bash
# 測試價格查詢
curl -X POST http://localhost:5678/webhook/ai-customer-service \
  -H "Content-Type: application/json" \
  -d '{"name":"阿明","message":"M5 螺絲 500 顆報價"}'

# 你會收到：
# 您好 阿明，
#
# M5 不鏽鋼螺絲：
# • 數量：500 顆
# • 單價：$1.5
# • 總價：$750
# • 交期：約 7-10 工作天
```

## 進階：接上實際客服管道

| 管道 | n8n 節點 | 說明 |
|------|---------|------|
| LINE | Webhook（LINE Messaging API） | 需要 LINE Developers 帳號 |
| Telegram | Telegram Trigger | 直接用 Token + Chat ID |
| 網站表單 | Webhook | 你網站上的聯絡表單 |
| WhatsApp | WhatsApp Cloud API（商業帳號） | Meta Business 審核約 3 天 |

## 下一步？

這個基礎版已經可以省下 80% 的報價查詢時間。進階可以加：
1. **記憶對話**：n8n Chat node 的 session key（記住上下文）
2. **Airtable CRM 整合**：自動建立客戶 + 報價紀錄
3. **多語言**：AI 自動偵測語言並用相同語言回覆

下一篇教你 **n8n + Shopify 電商自動化**，從訂單到 Slack 通知一條龍！

---

*完整 workflow JSON 可於 [GitHub](https://github.com/jerryLee18) 下載。*
