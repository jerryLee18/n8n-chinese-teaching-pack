# n8n 萬用 Webhook 接法：LINE / Telegram / 表單全串接

> 作者：Jerry Lee | n8n 自動化工程師
> 難度：⭐⭐⭐☆☆

---

「我的客戶用 LINE、供應商用 Telegram、官網有聯絡表單⋯⋯全部都要手動回也太累了吧？」

這是台灣中小企業最常見的自動化需求。好消息是：**n8n 的 Webhook 可以當作統一收件匣**，把所有管道的訊息集中處理。這篇文章教你怎麼接。

## 統一收件架構

```
LINE 訊息 ─────────┐
Telegram 私訊 ────┤
網站聯絡表單 ─────┼──→ n8n Webhook → 統一處理 → Slack 通知
Google 表單 ──────┤                              └→ Google Sheets
Email (轉發) ─────┘
```

## 接法一：LINE Messaging API

LINE 是台灣覆蓋率最高的通訊軟體。需要先設定 LINE Bot：

### 1. 建立 LINE Bot
1. 到 [LINE Developers Console](https://developers.line.biz/)
2. 建立 Provider → 建立 Channel（Messaging API）
3. 取得 **Channel Secret** 和 **Channel Access Token**

### 2. n8n Webhook 設定

```bash
# Webhook URL（給 LINE 的）
https://你的網域/webhook/line
```

LINE 會用 POST 送來訊息，格式像這樣：

```json
{
  "events": [{
    "type": "message",
    "message": { "type": "text", "text": "你好，想詢價" },
    "source": { "userId": "Uxxx", "type": "user" },
    "replyToken": "xxx"
  }]
}
```

### 3. Code Node 解析 LINE 訊息

```javascript
const events = $input.first().json.events || [];
const messages = events
  .filter(e => e.type === 'message' && e.message?.text)
  .map(e => ({
    source: 'LINE',
    user_id: e.source.userId,
    message: e.message.text,
    reply_token: e.replyToken,
    timestamp: new Date(e.timestamp).toISOString()
  }));

return messages.map(m => ({ json: m }));
```

### 4. 回覆 LINE 訊息

用 HTTP Request 節點呼叫 LINE Reply API：

```
URL: https://api.line.me/v2/bot/message/reply
Method: POST
Headers:
  Authorization: Bearer {{ $env.LINE_CHANNEL_ACCESS_TOKEN }}
  Content-Type: application/json
Body:
  {
    "replyToken": "{{ $json.reply_token }}",
    "messages": [{ "type": "text", "text": "{{ $json.reply_text }}" }]
  }
```

---

## 接法二：Telegram Bot

Telegram 的 Bot API 比 LINE 更簡單，可以直接用 n8n 的 Telegram 節點。

### 1. 建立 Telegram Bot
1. 在 Telegram 搜尋 `@BotFather`
2. `/newbot` → 取名 → 取得 Token

### 2. n8n 設定
直接用 **Telegram Trigger** 節點 → 填入 Token → 自動接收訊息。

或者用 Webhook 模式：
```
https://你的網域/webhook/telegram
```

設定 webhook：
```bash
curl "https://api.telegram.org/bot<TOKEN>/setWebhook?url=https://你的網域/webhook/telegram"
```

### 3. 解析 Telegram 訊息

```javascript
const msg = $input.first().json.message || {};
return {
  source: 'Telegram',
  user_id: msg.from?.id || msg.chat?.id,
  username: msg.from?.username || msg.from?.first_name || '未知',
  message: msg.text || '',
  chat_id: msg.chat?.id,
  timestamp: new Date(msg.date * 1000).toISOString()
};
```

### 4. 回覆 Telegram

用 **Telegram** 節點 → Send Message → Chat ID: `{{ $json.chat_id }}`

---

## 接法三：網站聯絡表單

最簡單。你的網站表單直接 POST 到 n8n Webhook 就行了。

### 前端範例（HTML）

```html
<form action="https://你的網域/webhook/contact-form" method="POST">
  <input name="name" placeholder="姓名">
  <input name="email" placeholder="Email">
  <textarea name="message" placeholder="訊息"></textarea>
  <button>送出</button>
</form>
```

### n8n Webhook 解析

```javascript
const body = typeof $json.body === 'string' ? JSON.parse($json.body) : $json.body;
return {
  source: 'Website Form',
  user_id: body.email,
  name: body.name,
  message: body.message,
  timestamp: new Date().toISOString()
};
```

---

## 接法四：Google 表單 → n8n

如果已經在用 Google 表單收集資料，可以自動轉到 n8n：

1. Google 表單 → 擴充功能 → Apps Script
2. 寫一個簡單的 trigger：

```javascript
function onFormSubmit(e) {
  const data = {
    name: e.values[1],
    email: e.values[2],
    message: e.values[3]
  };
  UrlFetchApp.fetch('https://你的網域/webhook/google-form', {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(data)
  });
}
```

---

## 統一處理：Switch + Code

所有管道都收到同一格式的訊息後，用一個 **Switch** 節點做分派：

```javascript
// 統一的訊息格式
{
  source: 'LINE' | 'Telegram' | 'Website' | 'GoogleForm',
  user_id: '...',
  message: '...',
  timestamp: '...'
}
```

Switch 根據 `source` 欄位分歧：
- LINE → 用 LINE API 回覆
- Telegram → 用 Telegram 節點回覆
- Website → 回覆 email
- GoogleForm → 記錄到 Sheets

---

## 安全提醒

1. **LINE 的 Signature 驗證**：LINE 會在 header `x-line-signature` 送簽名，務必驗證以免被偽造訊息
2. **Webhook URL 不要公開**：路徑用 UUID 而不是 `/webhook`（例如 `/webhook/a1b2c3d4`）
3. **加上 API Key 驗證**：非 LINE/Telegram 的 webhook 加一個 shared secret

---

掌握了這些，你就可以用一個 n8n 把所有訊息管道串在一起。下一篇我們來聊聊 **n8n vs Dify：台灣/中文市場怎麼選？**

---

*完整 workflow JSON 可於 [GitHub](https://github.com/jerryLee18) 下載。*
