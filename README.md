# tw-stock-analyzer
 
A Claude Skill that fetches Taiwan-listed stock (上市/上櫃) financial data — price, institutional trading, revenue, material announcements, and recent news — and produces a self-contained HTML analysis report per stock.
 
一個 Claude 技能（Skill），可抓取台股（上市／上櫃）財務資料——股價、三大法人買賣、營收、重大訊息、近期新聞——並產出單一自包含的 HTML 分析報告。
 
---
 
## What it does / 功能說明
 
Given one or more Taiwan stock codes, this skill:
 
給定一個或多個台股代號，這個技能會：
 
1. Validates each code against official TWSE/TPEx data, skipping invalid ones.
   依官方資料驗證每個代號，無效代號自動跳過。
2. Pulls this year's financials, price, institutional trading, revenue, material announcements, and recent news — **one stock at a time**.
   抓取該股票本年度至今的財報、股價、法人買賣、營收、重大訊息與新聞——**逐支處理，不跨股票比較**。
3. Produces one self-contained HTML report per stock, with embedded charts (institutional trading, price, monthly revenue, quarterly revenue) and a "download CSV" button.
   為每支股票產出一份自包含 HTML 報告，內嵌四種圖表（三大法人買賣、股價、月營收、季營收）與「下載 CSV」按鈕。
This is a data-gathering and presentation skill, **not an investment-advice skill**. It never issues a buy/hold/sell recommendation — only lays out factors and metrics for the user to judge themselves.
 
這是一個**資料彙整與呈現技能，不是投資建議工具**。報告不會給出買進/持有/賣出建議，只整理正反面因素與指標供使用者自行判斷。
 
---
 
## Data sources / 資料來源
 
Primary: official, keyless OpenAPI endpoints from TWSE (證交所) and TPEx (櫃買中心), plus 公開資訊觀測站 (MOPS) as fallback. See `references/data-sources.md` for full endpoint details, date-format conversion notes (ROC vs. Gregorian calendar), and known limitations (e.g. TPEx bot-detection blocks, incomplete historical series for price/institutional data via public endpoints).
 
主要使用證交所（TWSE）與櫃買中心（TPEx）的官方免金鑰 OpenAPI，並以公開資訊觀測站（MOPS）作為備援。詳細端點、民國/西元曆法轉換、已知限制（例如 TPEx 偶發機器人偵測擋下、股價與法人籌碼公開端點僅提供部分歷史）請見 `references/data-sources.md`。
 
---
 
## Structure / 檔案結構
 
```
tw-stock-analyzer/
├── SKILL.md                       — trigger conditions & step-by-step workflow / 觸發條件與工作流程
├── references/
│   ├── data-sources.md            — API endpoints, date formats, fallbacks / API端點、日期格式、備援機制
│   └── report-template.md         — HTML report structure & chart implementation / 報告結構與圖表實作方式
└── evals/
    └── evals.json                 — test cases / 測試案例
```
 
---
 
## Installation / 安裝方式
 
1. Download `tw-stock-analyzer.skill`.
   下載 `tw-stock-analyzer.skill` 檔案。
2. In Claude, add it via your skills library (Settings → Skills, or the install prompt on the file card).
   在 Claude 的技能庫中新增（設定 → 技能，或直接點擊檔案卡片上的安裝按鈕）。
3. In any conversation, type one or more Taiwan stock codes (e.g. `2330`, `1815 2330 2454`, or `幫我看一下2330台積電的財報`) to trigger it.
   之後在任何對話中輸入一個或多個台股代號（例如 `2330`、`1815 2330 2454`，或「幫我看一下2330台積電的財報」）即可觸發。
---
 
## Known limitations / 已知限制
 
- Public OpenAPI endpoints for price and institutional trading typically return only the latest snapshot, not a full historical series — the skill notes this explicitly in reports rather than fabricating data.
  股價與法人買賣的公開端點通常只回傳最新單一快照，沒有完整歷史序列——技能會在報告中誠實標註，不會捏造數字。
- TPEx (上櫃) endpoints occasionally trigger bot-detection blocks; the skill falls back to MOPS (公開資訊觀測站) in that case.
  TPEx（上櫃）端點偶爾會觸發機器人偵測擋下；此時技能會改用公開資訊觀測站作為備援來源。
- This skill does not provide financial advice. Always verify figures against official sources before making investment decisions.
  本技能不提供投資建議，實際投資決策前請務必以官方資料源再次核實數據。
---
 
## License / 授權
 
Feel free to use, modify, and share. Built collaboratively with Claude (Anthropic).
 
歡迎自由使用、修改、分享。與 Claude（Anthropic）協作開發完成。
 