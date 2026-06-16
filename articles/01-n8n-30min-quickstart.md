# n8n 新手 30 分鐘上手：從安裝到第一個自動化

> 作者：Jerry Lee | n8n 自動化工程師
> 難度：⭐⭐☆☆☆（初學者友善）

---

你聽過 n8n，但打開介面後不知道從何開始？這篇文章帶你 30 分鐘內完成安裝、建立第一個 workflow，並且實際跑起來。

## n8n 是什麼？

n8n（發音：n-eight-n）是一個開源的 **workflow automation** 工具。你可以想像它是 Zapier / Make 的開源替代品，但功能更強大：

- **免費自託管**（self-host）：用自己的伺服器，無限制執行次數
- **400+ 節點**：Slack、Google Sheets、OpenAI、LINE、Telegram、資料庫⋯⋯
- **程式碼節點**：可以用 JavaScript / Python 寫自訂邏輯
- **AI 原生整合**：內建 OpenAI、Anthropic 節點，做 AI agent 超簡單

跟市面上的競爭者比：

| 特性 | n8n | Zapier | Make | Dify |
|------|-----|--------|------|------|
| 自託管 | ✅ 免費 | ❌ 雲端限定 | ❌ 雲端限定 | ✅ |
| 節點數量 | 400+ | 5000+ | 1500+ | 30+ |
| AI Agent | ✅ 原生 | ✅ 原生 | ❌ | ✅（主打） |
| 學習曲線 | 中低 | 低 | 中 | 中 |
| 適合場景 | 通用自動化 | 簡單串接 | 視覺化流程 | AI 應用 |

**一句話總結：n8n 是做「所有自動化」的瑞士刀，Dify 是專注做「AI 應用」的工具。**

## 第一步：安裝 n8n

### 方法一：Docker（推薦，最簡單）

```bash
docker run -d --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e N8N_SECURE_COOKIE=false \
  n8nio/n8n
```

打開瀏覽器 → `http://localhost:5678`，設定帳號密碼，完成。

### 方法二：npm（開發用）

```bash
npm install n8n -g
n8n start
```

### 方法三：Railway / Render（雲端免費部署）

1. 到 [Railway](https://railway.app) 或 [Render](https://render.com)
2. 選擇 Docker 部署
3. Image: `n8nio/n8n`
4. Port: `5678`
5. 五分鐘上線

## 第二步：建立第一個 Workflow

我們來做一個最簡單的自動化：**每天早上的天氣通知 → Telegram**。

### 步驟 1：Schedule Trigger（定時觸發）

1. 點右上角 **"+"** → 新增 Workflow
2. 搜尋 `Schedule` → 點擊 **Schedule Trigger**
3. 設定每天 7:00 AM 觸發

### 步驟 2：HTTP Request（抓天氣 API）

1. 點 **"+"** → 搜尋 `HTTP Request`
2. URL：`https://api.open-meteo.com/v1/forecast?latitude=25.03&longitude=121.56&current_weather=true`
3. Method: GET

這是免費的天氣 API，不需要註冊。上面的座標是台北，你可以改成自己的城市。

### 步驟 3：Code Node（整理資料）

1. 點 **"+"** → 搜尋 `Code`
2. Language: JavaScript
3. 貼上這段程式碼：

```javascript
const weather = $input.first().json.current_weather;
const message = `
☀️ 早安！今日天氣：
🌡️ 溫度：${weather.temperature}°C
💨 風速：${weather.windspeed} km/h
🕐 更新時間：${weather.time}
`;
return { message };
```

### 步驟 4：Telegram 發送

1. 點 **"+"** → 搜尋 `Telegram`
2. 你需要先設定 Telegram Bot：
   - 在 Telegram 搜尋 `@BotFather` → `/newbot`
   - 取得 Bot Token
3. 在 n8n 新增 Telegram 憑證，貼上 Token
4. Chat ID：你的 Telegram ID（可以用 `@userinfobot` 查詢）
5. Text：`{{ $json.message }}`

### 步驟 5：執行！

點右上角 **"Execute Workflow"** → 你的 Telegram 就會收到天氣通知了！

## 第三步：設為自動執行

Workflow 右上角開關 → **Active**（綠色 = 啟用中）。它現在每天早上 7 點會自動跑。

## 常見問題

**Q: n8n 免費嗎？**
A: 自託管完全免費。n8n Cloud 有免費額度（20 次/月），付費方案 $20/月起。

**Q: 跟 Make.com 比哪個好？**
A: Make 視覺化更直覺、學習曲線更平。n8n 更靈活（Code node 可以寫 JS/Python）、節點更多、自託管完全免費。如果你會寫一點 code → n8n。完全非技術 → Make。

**Q: 可以接 LINE 嗎？**
A: 可以。用 Webhook + LINE Messaging API。後續文章會專門講。

## 下一步？

完成第一個 workflow 後，試試看：
1. 加入 IF 節點：下雨時才提醒帶傘
2. 接 OpenAI：讓它用自然語言寫天氣摘要
3. 存到 Google Sheets：記錄每日天氣歷史

下一篇我們來做 **n8n + OpenAI AI Agent** — 打造你的第一個 AI 客服！

---

*本文發表於 CSDN / 思否 / 知乎「n8n 自動化」專欄。完整教學資源：[GitHub](https://github.com/jerryLee18) | [n8n 官方文件](https://docs.n8n.io)*
