# n8n 中文教學資源包

> 6 篇完整教學 + 4 個中文化模板 — 專為中文使用者打造的 n8n 入門到進階資源

---

## 為什麼要有這個資源包？

n8n 在繁體/簡體中文圈的教學內容幾近於零。Dify 在中國吃下整個 AI workflow 心智份額，但 n8n 更適合做**通用自動化**（不只是 AI 應用）。

這個資源包就是來填坑的。

## 內容清單

### 📚 6 篇教學文章（`articles/`）

| # | 標題 | 難度 | 字數 |
|---|------|------|------|
| 1 | [n8n 新手 30 分鐘上手：從安裝到第一個自動化](articles/01-n8n-30min-quickstart.md) | ⭐⭐ | ~2500 |
| 2 | [n8n AI Agent 實戰：打造你的第一個 AI 客服](articles/02-n8n-ai-agent-customer-service.md) | ⭐⭐⭐ | ~3000 |
| 3 | [n8n + 電商自動化：Shopify 訂單到 Slack 通知一條龍](articles/03-n8n-shopify-slack-ecommerce.md) | ⭐⭐⭐ | ~2800 |
| 4 | [n8n 萬用 Webhook 接法：LINE / Telegram / 表單全串接](articles/04-n8n-webhook-line-telegram.md) | ⭐⭐⭐ | ~3000 |
| 5 | [n8n vs Dify：台灣/中文市場怎麼選？](articles/05-n8n-vs-dify-comparison.md) | ⭐⭐ | ~3000 |
| 6 | [n8n + Dify 雙平台實戰：n8n 編排流程 + Dify 做 RAG 知識庫](articles/06-n8n-dify-rag-integration.md) | ⭐⭐⭐ | ~3500 |

### 🛠️ 4 個中文化 Workflow 模板（`templates/`）

| 模板 | 用途 | 節點數 |
|------|------|--------|
| [AI 潛在客戶評分 CRM](templates/lead-qualification-crm-zh.json) | Webhook 接收表單 → AI 評分 → Slack 通知 → Sheets 記錄 | 9 |
| [代理商任務接收與路由](templates/agency-task-intake-zh.json) | 客戶需求 → AI 分類(開發/設計/行銷/客服) → 自動路由 Slack | 12 |
| [AI 影片製作管線](templates/ai-video-pipeline-zh.json) | 影片需求 → AI 腳本 → 並行 TTS + DALL-E → 組合回覆 | 9 |
| [n8n × Dify RAG 整合](templates/n8n-dify-rag-integration.json) | n8n HTTP Request → Dify API → RAG 知識庫問答 → 回覆 | 10 |

所有模板的 prompt、欄位標籤、錯誤訊息均已中文化。

## 如何匯入模板

1. 開啟你的 n8n 實例
2. 右上角 **"+ "** → **Import from File**
3. 選擇 `.json` 檔案
4. 設定所需憑證（OpenAI、Slack、Google Sheets）
5. 設為 **Active**

## 發布目標平台

| 平台 | 網址 | 策略 |
|------|------|------|
| CSDN | csdn.net | 技術文章流量最大 |
| 思否 SegmentFault | segmentfault.com | 開發者社群 |
| 知乎 | zhihu.com | 建立「n8n 自動化」專欄 |
| 騰訊雲開發者 | cloud.tencent.com/developer | 企業開發者 |
| 楓葉網 | mapleleaf.com | 已有 n8n 文章瀏覽基礎 |

## 帳號資訊

- 註冊信箱：richmomo1515@gmail.com
- 作者名：Jerry Lee（或「程式自動化筆記」）

## 發布策略

1. **CSDN + 思否 + 知乎** 同步發布五篇系列文（每週一篇，連續五週）
2. 每篇文末導流到 GitHub 下載模板
3. 第五篇（n8n vs Dify）作為導流主力 — 踩關鍵字紅利
4. 累積流量後可出付費進階教學（50–100 元/課）

## 授權

MIT — 可自由轉載、修改、商用。只要求保留作者署名。

---

**作者：Jerry Lee** | [GitHub](https://github.com/jerryLee18) | richmomo1515@gmail.com

---

## 💰 支持這個專案
這個資源包是免費開源的。如果對你有幫助：

- ⭐ Star [我們的 GitHub](https://github.com/jerryLee18/ai-automation-portfolio)
- 🛒 [贊助 $29](https://momospark273.gumroad.com/l/lpamft) — 支持我們繼續做更多中文教學
- 📩 richmomo1515@gmail.com — 需要客製化 n8n/Dify 整合方案？
