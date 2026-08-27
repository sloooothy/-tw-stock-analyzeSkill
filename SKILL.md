---
name: tw-stock-analyzer
description: Fetches Taiwan-listed stock (上市/上櫃) financial data, price, institutional trading, revenue, material announcements, and recent news, then produces a single self-contained HTML analysis report per stock. Use this skill whenever the user gives one or more Taiwan stock codes (e.g. "2330", "2330台積電", "幫我看一下1815、2330、2454") and wants a financial report, fundamental analysis, or "值不值得買" style review. Also trigger for phrases like "抓財報", "基本面分析", "台股分析", "這幾檔股票值得買嗎" even if the user doesn't explicitly say "report" or ".html". Do NOT trigger for pure price-quote lookups ("2330現在多少錢") with no analysis intent, or for non-Taiwan tickers (US/HK/China stocks) — this skill only covers TWSE/TPEx-listed securities.
---

# Taiwan Stock Financial Analyzer

## What this skill does

Given one or more Taiwan stock codes, this skill:
1. Validates each code against official TWSE/TPEx data (skipping invalid ones)
2. Pulls this year's financials, price, institutional trading, revenue, material announcements, and recent news — **one stock at a time**, since the user treats each code as a candidate they're considering buying, and different industries aren't comparable side by side anyway
3. Produces one self-contained HTML report per stock, with embedded charts and a "download CSV" button, plus inline chart previews in the conversation

This is a data-gathering and presentation skill, not an investment-advice skill. The final section of the report lays out factors and metrics for the user to weigh themselves — it never issues a buy/hold/sell recommendation. See "Guardrails" below.

## Why this skill exists (read before improvising)

Taiwan stock data lives across several official, unauthenticated JSON/CSV endpoints (TWSE OpenAPI, TPEx OpenAPI) rather than one unified API, and popular finance portals (Goodinfo, Yahoo奇摩股市, 鉅亨網) render their data client-side with JavaScript, so `web_fetch` on those pages usually returns an empty shell — don't rely on them for structured data, only for news/context. Prefer the official OpenAPI endpoints described in `references/data-sources.md`; they return clean JSON/CSV, need no login, and are safe to cite as sources.

`web_fetch` in this environment can only retrieve a URL that has already appeared in a `web_search` or `web_fetch` result in the current conversation. So the pattern for every official endpoint is: **`web_search` the exact endpoint URL (or a query that surfaces it) → then `web_fetch` it.** Once fetched, its JSON payload covers the whole market in one call — cache it mentally and reuse it for every stock code in the user's list rather than re-fetching per stock.

## Step-by-step workflow

### 1. Parse the input into a list of candidate codes

Taiwan stock codes are numeric (4 digits standard, sometimes with a letter suffix for special share classes). The user may mix in company names ("2330台積電") or separate multiple codes with spaces, commas, or newlines. Extract just the numeric code from each entry; if a name is given alongside a code, use it only as a hint — always confirm against the official data, don't trust the user's spelling.

Anything that isn't a plausible numeric stock code (e.g. "a123") is an invalid entry — don't waste a lookup on it, just record it as failed with reason "非有效股票代號格式".

### 2. Resolve and validate each code

Read `references/data-sources.md` for the exact endpoints. In short: fetch the TWSE company-profile snapshot and the TPEx company-profile snapshot (both cover the *entire* market in one call each), then look up each candidate code in both. A code that appears in neither is invalid — record it as failed with reason "查無此股票代號" and move on. Don't let one bad code stop the batch; process every other code in the list.

For each valid code, note whether it's 上市 (TWSE) or 上櫃 (TPEx) — this determines which endpoint family to use for every subsequent step (price, revenue, institutional trading all have separate TWSE/TPEx endpoints with different field names).

### 3. Determine the reporting period

Default window: January 1 of the current year through the most recently completed month. E.g. if today is August 2026, pull January–July 2026 monthly data. If the company listed partway through this window (check 上市日期/上櫃日期 from the profile data), start from its listing month instead, and say so explicitly in the report rather than silently showing a short window — a newly listed company having only 2 months of data is expected, not an error.

### 4. Gather the data

For the resolved stock, collect (see `references/data-sources.md` for endpoints and fields):
- **Basic profile**: company name, incorporation date, listing date, main business/products, chairman, industry category
- **Latest price snapshot**: most recent trading day's close, change, volume
- **Financials for the period**: revenue, gross profit, operating income, net income (from monthly revenue disclosures and/or quarterly financial statements, whichever the endpoints provide for the window)
- **Material announcements (重大訊息)** for the period, most recent first — flag anything about restated/re-disclosed financials, since fast-rising stocks sometimes get asked by MOPS to re-disclose
- **Institutional trading (三大法人買賣超)** for the period — foreign investors, investment trusts, dealers
- **Recent news** relevant to financial performance — use `web_search`, prioritize the last 1-3 months, and paraphrase per standard copyright rules (never quote more than ~15 words, one quote per source max)

If any single data point is unavailable (e.g. a newly-listed company with no quarterly report yet), note it as "無資料" in the report rather than fabricating a number or silently omitting the row.

### 5. Build the four charts

Required charts, per stock, over the reporting period:
1. 三大法人買賣超比較 (foreign / trust / dealer, grouped or stacked)
2. 股價變化 (daily or weekly close over the period)
3. 月營收 (monthly revenue trend)
4. 季營收 (quarterly revenue trend)

Render these two ways:
- **Inline in the conversation**: use the `chart_display_v0` tool for straightforward line/bar charts (price, monthly/quarterly revenue), so the user sees them immediately without opening the file.
- **Embedded in the HTML report**: see `references/report-template.md` for the self-contained chart approach (no external chart library — plain SVG or inline `<canvas>` + vanilla JS, since the report must work as a standalone file with no network access when opened later).

### 6. Assemble the CSV and the HTML report

One CSV per stock with the raw numbers behind the four charts (dates + values, clearly labeled columns, UTF-8 with BOM so it opens correctly in Excel on Windows). Embed this CSV's contents as a JS string inside the HTML report and wire up a "下載 CSV" button using a Blob + `<a download>` — see `references/report-template.md` for the exact pattern. Do not rely on a server; the report must be fully self-contained since the user will download and open it later, possibly offline.

Follow `references/report-template.md` for the full report structure (sections, headings, disclaimer placement). Save the finished file to `/mnt/user-data/outputs/<code>_<公司簡稱>_report.html` and also save the companion `<code>_<公司簡稱>_data.csv` there.

### 7. Present results

Use `present_files` to hand over the HTML (and CSV) for each successfully-processed stock. After all stocks are processed, give a short conversational summary — one or two lines per stock is enough since the report has the detail — followed by a clear list of any codes that failed and why (see step 2).

## Guardrails

- **Never issue a buy/hold/sell recommendation or use persuasive investment language** ("建議買進", "強烈看好", "應該加碼" etc.). The final report section lays out positive and negative factors, valuation metrics, and recent trends side by side, and explicitly leaves the judgment to the reader. If the user directly asks "這檔該買嗎", answer by pointing to the factors in the report and reminding them this skill provides information, not financial advice — you are not a licensed financial advisor.
- **Don't fabricate numbers.** If an endpoint returns nothing for a field, say so ("無資料") rather than estimating or interpolating.
- **One stock per report.** Even when given a list, process and report on each stock independently — don't produce a comparison table across industries the user didn't ask to compare.
- Cite data sources inline in the report (e.g. "資料來源：TWSE OpenAPI t187ap03_L", "資料來源：公開資訊觀測站重大訊息").
