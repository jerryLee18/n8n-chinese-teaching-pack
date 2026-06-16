# n8n + Dify 雙平台實戰：用 n8n 編排流程 + Dify 做 RAG 知識庫

> 作者：Jerry Lee | n8n 自動化工程師
> 難度：⭐⭐⭐☆☆ | 閱讀時間：8 分鐘

---

## 為什麼要搭 n8n + Dify？

中文 AI 圈有個共識：**「n8n 只能做出海，Dify 才是中國市場的王。」** 但這個說法只對了一半。

### 兩個平台各自的強項

| 能力 | n8n | Dify |
|------|:---:|:----:|
| 通用 API 串接（400+ 節點） | ✅ | ❌ |
| 多步驟自動化排程 | ✅ | ❌ |
| RAG 知識庫問答 | ❌（要自己架） | ✅（內建） |
| 向量檢索 + 文件上傳 | ❌ | ✅ |
| LINE / Telegram / Slack 通知 | ✅ | ❌ |
| 電商訂單自動化 | ✅ | ❌ |

**你不需要選邊站。把兩個平台串起來，各做各擅長的事。**

---

## 案例：客服 RAG 系統

假設你有一個場景：

> 客戶透過 LINE 問問題 → 系統查知識庫 → 自動回覆答案

- **Dify 負責：** 知識庫管理 + RAG 檢索（上傳產品手冊、FAQ、內部文件）
- **n8n 負責：** 接收 LINE 訊息 → 呼叫 Dify API 查答案 → 回覆 LINE + 記錄到 Google Sheets

這樣的好處：
- Dify 的 RAG pipeline 不用自己架（省 2-3 天開發）
- n8n 的多渠道接入（LINE、網站客服、Email）不用重複開發
- 兩個平台各自更新，不互相卡

---

## 實戰：n8n 呼叫 Dify API 的完整流程

### 架構圖

```
LINE User → n8n Webhook → Format Dify Request
                                │
                        HTTP Request → Dify API
                        (Chat Messages)   │
                                │    Dify RAG Pipeline
                                │    (向量檢索 + LLM 生成)
                                │         │
                        ← Dify Response ←
                                │
                        Process + Format → Reply to LINE
                                │
                        Log to Google Sheets
```

### 步驟 1：在 Dify 建立知識庫應用

1. 登入 [Dify](https://cloud.dify.ai) → 建立新應用 → 選「聊天助手」
2. 上傳你的知識庫文件（支援 PDF、Word、TXT、Markdown）
3. 在「知識庫」頁面 → 勾選你剛上傳的知識庫
4. 發布應用 → 記下 **API Key**（在「API 存取」頁面）

### 步驟 2：在 n8n 建立 workflow

我們已經幫你做好完整的 workflow JSON：[`n8n-dify-rag-integration.json`](../templates/n8n-dify-rag-integration.json)

直接 import 到你的 n8n，設定兩個環境變數就搞定：

```bash
# .env
DIFY_API_KEY=app-xxxxxxxxxx
DIFY_BASE_URL=https://api.dify.ai
```

節點說明：

| 節點 | 功能 |
|------|------|
| **Webhook** | 接收外部請求（LINE、網站客服、自訂前端） |
| **Parse User Query** | 解析使用者問題，建立 Dify API 所需的 payload |
| **Build Dify Headers** | 設定 API Key 和 Content-Type，從 `$env` 讀取 |
| **Call Dify API** | HTTP Request → Dify `/v1/chat-messages`，支援 conversation_id 續接對話 |
| **Process Dify Response** | 解析 Dify 回傳的答案 + 檢索來源 + conversation_id |
| **Format Response** | 包裝成統一的 JSON 格式，方便下游節點使用 |
| **Respond to Webhook** | 回傳結果給呼叫方 |
| **Error Handler** | Dify API 呼叫失敗時的降級處理 |

### 步驟 3：測試

```bash
curl -X POST http://localhost:5678/webhook/dify-rag \
  -H "Content-Type: application/json" \
  -d '{
    "query": "退換貨政策是什麼？",
    "user": "customer-001",
    "conversation_id": ""
  }'
```

預期回傳：

```json
{
  "success": true,
  "answer": "我們的退換貨政策是：購買後 7 天內可申請退貨...",
  "conversation_id": "abc-123-def",
  "sources": [
    {
      "doc": "退換貨政策.pdf",
      "segment": 5,
      "content": "購買後 7 天內可申請退貨，商品須保持完整包裝..."
    }
  ],
  "source_count": 1,
  "provider": "dify-rag"
}
```

---

## 進階：串接 LINE 即時客服

把 Webhook 換成 LINE Webhook，加上回覆節點：

```
LINE Message → n8n Webhook → Dify RAG → LINE Reply
```

n8n 內建 LINE 節點，不用寫任何 code。流程是：

1. LINE 使用者傳訊息 → n8n LINE Trigger
2. Code 節點取出訊息文字 → 送 Dify
3. Dify 回傳答案 → LINE Reply 節點送回使用者
4. （可選）Google Sheets 記錄每次問答

這樣你就有一個 **12 節點、零程式碼的 LINE + RAG 客服機器人**。

---

## 延伸：n8n + Dify + 你自己的 API

Dify 的 RAG 結果不一定完美。你可以用 n8n 做後處理：

```
Dify RAG → n8n Code 節點過濾 → 二次 LLM 潤飾 → 回覆使用者
```

例如：
- Dify 檢索到 5 個片段 → n8n Code 節點過濾掉不相關的 → 只留 2 個片段餵給 LLM
- Dify 答案太短 → n8n 加一個 OpenAI 節點做擴寫
- 偵測到敏感詞 → n8n 自動路由到人工客服

**n8n 是流程引擎，Dify 是 AI 引擎。兩個加在一起，就是最好的 RAG 客服系統。**

---

## 成本對比

| 方案 | 月費 | RAG 能力 | 通用自動化 | 自訂程度 |
|------|------|----------|-----------|---------|
| Dify Cloud（Pro） | $59/月 | ✅ 內建 | ❌ | 低 |
| n8n Cloud（Pro） | $20/月 | ❌（需自建） | ✅ 400+ 節點 | 高 |
| **n8n + Dify 雙平台** | **$0-79/月**（自託管可 $0） | ✅ | ✅ | **最高** |

自託管方案（Docker）：
- n8n: `docker run n8nio/n8n`（免費，無限制）
- Dify: `docker compose up`（社群版免費）

**總成本：$0**（不含 LLM API 費用和伺服器）

---

## 重點整理

1. **n8n 和 Dify 不是競爭對手，是互補工具**
2. 範例 workflow 已經做好：[`n8n-dify-rag-integration.json`](../templates/n8n-dify-rag-integration.json) — import 即用
3. 最常見的場景：LINE 客服 + Dify RAG + n8n 流程編排
4. 自託管雙平台成本 $0，比任何 SaaS 方案都便宜

---

## 延伸閱讀

- [n8n vs Dify：台灣/中文市場怎麼選？](05-n8n-vs-dify-comparison.md) — 先搞清楚兩個工具的定位
- [n8n AI Agent 實戰：打造你的第一個 AI 客服](02-n8n-ai-agent-customer-service.md) — 如果不用 Dify，純 n8n 怎麼做
- [n8n 萬用 Webhook 接法](04-n8n-webhook-line-telegram.md) — Dify API 的回傳結果要接去哪

---

*Written by Jerry Lee — n8n 自動化工程師。我做 n8n + Dify 雙平台整合，因為中文市場不應該只有 Dify 一個選項。*
