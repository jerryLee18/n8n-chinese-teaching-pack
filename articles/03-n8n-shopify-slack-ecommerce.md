# n8n + 電商自動化：Shopify 訂單到 Slack 通知一條龍

> 作者：Jerry Lee | n8n 自動化工程師
> 難度：⭐⭐⭐☆☆

---

電商賣家每天都要檢查訂單、通知倉庫出貨、追蹤物流。手動做這些事情，一天至少浪費 2 小時。這篇文章教你用 n8n 把 Shopify 訂單自動化：從收到訂單 → 分析資料 → Slack 通知 → Google Sheets 記錄，全部自動完成。

## 你會學到什麼

- 如何串接 Shopify Webhook
- 如何用 n8n 解析訂單資料
- 如何發送客製化的 Slack 通知
- 如何自動記錄到 Google Sheets

## 架構圖

```
Shopify 新訂單 → Webhook → Code（解析訂單）→ IF（訂單金額 > 門檻）
                                                    ├── 高 → Slack 特急通知
                                                    └── 一般 → Google Sheets 紀錄
```

## Step 1：在 Shopify 設定 Webhook

1. Shopify 管理後台 → 設定 → 通知
2. 新增 Webhook → 選擇 `Order creation`
3. URL: `https://你的-n8n-網址/webhook/shopify-order`
4. Format: JSON

Shopify 會在每次有新訂單時 POST 到這個 URL。

## Step 2：n8n Webhook 接收訂單

新增 **Webhook** 節點：
- Method: `POST`
- Path: `shopify-order`
- Raw Body: 開啟（Shopify 送來的 body 是 JSON 字串，需要手動 parse）

## Step 3：Code Node 解析訂單

```javascript
const body = typeof $json.body === 'string' ? JSON.parse($json.body) : $json.body;

const order = {
  id: body.id,
  order_number: body.order_number || body.name,
  email: body.email || body.customer?.email || '',
  customer_name: `${body.customer?.first_name || ''} ${body.customer?.last_name || ''}`.trim(),
  total: parseFloat(body.total_price || 0),
  currency: body.currency || 'TWD',
  items: (body.line_items || []).map(item => ({
    name: item.title || item.name,
    qty: item.quantity,
    price: item.price
  })),
  shipping_method: body.shipping_lines?.[0]?.title || '未知',
  payment_status: body.financial_status || 'unknown',
  created_at: body.created_at
};

// 計算商品總數
order.total_items = order.items.reduce((sum, item) => sum + item.qty, 0);

// 組裝 Slack 訊息
const itemsList = order.items
  .map(i => `  • ${i.name} x${i.qty} — $${i.price}`)
  .join('\n');

order.slack_message = `
🛒 *新訂單 #${order.order_number}*
👤 ${order.customer_name} (${order.email})
📦 ${order.total_items} 件商品
${itemsList}
💰 總金額：$${order.total} ${order.currency}
🚚 運送方式：${order.shipping_method}
💳 付款狀態：${order.payment_status}
`;

return { json: order };
```

## Step 4：IF 節點 — 高單價判斷

新增 **IF** 節點：
- 條件：`total > 5000`（大於 5000 = 高單價）

這樣高單價訂單可以發送特別通知、一般訂單走標準流程。

## Step 5：Slack 通知

新增 **Slack** 節點 → Channel → Send Message

高金額訂單分支的訊息可以加 `<!channel>` 來通知所有人：

```
🚨 <!channel> 大單來了！
{{ $json.slack_message }}
```

一般訂單則走普通訊息。

## Step 6：Google Sheets 記錄

新增 **Google Sheets** → Append Row

欄位對應：
- A: 訂單編號
- B: 客戶名稱
- C: Email
- D: 商品數量
- E: 總金額
- F: 運送方式
- G: 付款狀態
- H: 建立時間

這樣就可以在 Sheets 裡做月報表、分析營收趨勢。

## 進階玩法

這個基礎架構可以擴充成完整的電商自動化系統：

### 1. 自動標籤客戶
用 OpenAI 分類客戶：新客/回頭客/批發商 → 對應不同的 Slack 通知語氣

### 2. 庫存警示
訂單商品數量接近庫存下限 → 自動發 Slack 叫補貨

### 3. 物流追蹤
接到 GreenTomato / 新竹物流的 webhook → 更新 Google Sheets 的物流狀態欄位

### 4. 棄單挽回
Shopify 的 `checkout/create` webhook → 15 分鐘後自動發折扣碼 email

### 5. 多店舖整合
一個 n8n workflow 接多家 Shopify 店鋪 → 統一到同一個 Slack 頻道 + Sheets

## 完整架構總結

```
Shopify A ─┐
Shopify B ─┤──→ Webhook → 解析 → 分類 → Slack
Shopify C ─┘                              └→ Sheets
                                          └→ Email(大單)
                                          └→ CRM(客戶建檔)
```

## 下一步？

下一篇我們來探討 **n8n 萬用 Webhook 接法：LINE / Telegram / 表單全串接**，把你的客服管道全部打通。

---

*完整 workflow JSON 可於 [GitHub](https://github.com/jerryLee18) 下載。*
