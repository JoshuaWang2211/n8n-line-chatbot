# 🤖 Line ChatBot – 擁有對話記憶功能 + Google 搜尋 + YouTube 搜尋 + OCR 圖片辨識的 GPT 聊天機器人

這是一個建構在 n8n 上的 LINE 聊天機器人工作流程，整合了 Google/YouTube 搜尋、OpenAI GPT、OCR 辨識圖片，以及訊息儲存與對話脈絡處理。

---

## ✨ 功能特色
📚 訊息自動記憶：LINE 使用者訊息會自動儲存至 Google Sheets（每人一分頁），供 AI 提取上下文對話內容，實現「記憶功能」

🔍 智慧語意搜尋：當訊息開頭包含 google、search、搜尋 等關鍵字（不分大小寫），即自動觸發 Google 及 YouTube 搜尋，並回傳整理摘要

🤖 GPT-4o 自然回覆：整合 OpenAI GPT-4o 模型，根據上下文生成自然、貼近人類語氣的回覆，支援多輪互動

📸 圖片敘述能力：支援使用者上傳圖片，透過 GPT 模型進行敘述型辨識，能回覆帶有觀察與語氣的描述內容

🎭 可自訂 AI 性格：可以自訂 AI 的語氣與回應風格（如幽默、知性、開朗、柔和等），打造專屬個性化聊天體驗

🔐 集中管理憑證：所有敏感資訊集中儲存在 Google Sheets 的 Secrets 表單中，不需硬編碼，安全且易於維護與更新

---

🖼️ 使用流程畫面預覽
以下是本專案的 n8n 工作流程總覽，涵蓋 LINE webhook、訊息記錄、AI 回覆、Google 搜尋、OCR 分析等節點：

[![n8n 工作流程總覽](./assets/screenshot.png)](./assets/screenshot.png)

你可以依據自己的需求，增減節點或功能，打造專屬的 LINE AI ChatBot。

---

🔎 觸發即時搜尋的方式：

當使用者傳送的訊息用以下任一關鍵字開頭（不分大小寫），系統將自動進行即時搜尋並回傳整理後的資訊摘要：

"google"、"search"、"搜尋"

範例訊息如：「search GPT-4 是什麼」、「Google 台積電股票」、「搜尋 AI 未來發展」等。

---

## 🚀 快速部署說明

### 1️⃣ 建立 LINE Messaging API channel 並取得 Token

請至 [LINE Developers Console](https://developers.line.biz/console/) 建立一個 Messaging API channel，並取得以下資訊：

- `Channel Access Token`（加入到 Secrets 表中，key 為：`LINE_CHANNEL_TOKEN`）
- （可選）`Channel Secret`（若啟用簽章驗證，可存為 `LINE_CHANNEL_SECRET`）

完成後記得將 webhook URL 加入 LINE 設定，範例如下 👇

---

### 2️⃣ 建立 ngrok 靜態 HTTPS 網址供 webhook 使用

n8n 的 webhook 需要公開網址才能讓 LINE 觸發。

免費版 ngrok 提供一個靜態 HTTPS 網址，可至官網申請。

在電腦上執行：

```bash
ngrok http 5678
```

複製 Forwarding 網址：https://你的子網域.ngrok-free.app

網址後面加上：webhook/line-agent

如下：

```
https://你的子網域.ngrok-free.app/webhook/line-agent
```

請將此網址貼入 LINE Developers Console 的 Webhook URL 設定中。

使用動態 URL 也可以，但每次重開 ngrok 都需要更新 Webhook URL。

也可以使用 Cloudflare Worker 等服務來替代 ngrok，功能相同。

---

### 3️⃣ 建立 Google Sheets：Secrets 表單 + 訊息資料庫

請建立兩份 Google Sheet (勿公開權限)：

#### 第一份：工作表一命名為 Secrets（必要）
用來集中儲存各項憑證資訊，格式如下：

| key                  | value                              |
|----------------------|-------------------------------------|
| LINE_CHANNEL_TOKEN   | xxxxx                              |
| OPENAI_LINE_BOT      | sk-xxxxx                           |
| GOOGLE_SEARCH_API    | AIza...                            |
| GOOGLE_CX_ID         | 1a2b3c4d...                        |
| DATABASE_LINE_BOT_ID | 1lbEeiAhMl-0P5xhtdP...             |

#### 第二份：隨意命名，用作訊息資料庫，將 Sheet ID 存入 Secrets 表單中
"key": DATABASE_LINE_BOT_ID。"value": 貼上此 Sheet 的 ID。
本 workflow 執行時，會自動為每位使用者建立對應的分頁，並儲存他們傳送的訊息與對話紀錄。

---

## 🔐 Secrets 表必要項目："key" (名稱) 及 "value" (API/secret 的值)

| Key 名稱             | 說明                                 |
|----------------------|--------------------------------------|
| LINE_CHANNEL_TOKEN   | LINE Bot 的 Access Token（必要）     |
| OPENAI_LINE_BOT      | OpenAI 的 API 金鑰             |
| GOOGLE_SEARCH_API    | Google Search API / YouTube Data API 金鑰 |
| GOOGLE_CX_ID         | Google Custom Search 引擎 ID         |
| DATABASE_LINE_BOT_ID | 儲存訊息的 Google Sheet 文件 ID      |

---

⚠️ 注意事項：

1. 本專案目前未使用 RAG（Retrieval-Augmented Generation）技術。

儲存在 Sheets 中的對話記憶會全部傳入 GPT 模型中處理 (每名使用者分開)。

若訊息累積過多，可能會導致 API Token 使用量大幅增加，請自行斟酌管理訊息保留長度。

未來版本有可能會整合簡易 RAG 機制以優化成本與效能。

2. 一般對話及圖片辨識功能的預設模型為 GPT-4o，處理即時搜尋結果則用 4o-mini 以節省成本。

若有成本考量，也可將 OpenAI GPT 改為 Google Gemini Flash。

---

## 📬 聯絡作者

本說明檔由 ChatGPT 簡單生成，我只補充了較重要的部分，各項細節需直接看 Workflow。

本專案是 vibe-coding 所生，仍持續開發中，隨時可能更新並增加新功能。

如果有任何問題，或對流程設計有改進建議，歡迎開 issue 或直接聯絡我：

- GitHub：[@JoshuaWang2211](https://github.com/JoshuaWang2211)  
