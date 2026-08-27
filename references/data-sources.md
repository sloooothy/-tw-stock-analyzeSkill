# Data sources reference

Two separate official, keyless, JSON/CSV data gateways cover the Taiwan market. There is no single unified API — always check both when resolving a code, and use the matching one for everything downstream.

| Market | Gateway | Notes |
|---|---|---|
| 上市 (TWSE-listed) | `https://openapi.twse.com.tw/v1/...` | Chinese field names (e.g. `公司代號`, `董事長`) |
| 上櫃 (TPEx-listed) | `https://www.tpex.org.tw/openapi/...` | Field naming differs from TWSE; some endpoints use English keys |

**Fetching pattern (important):** `web_fetch` in this environment refuses URLs that haven't appeared in a prior `web_search`/`web_fetch` result. So for every endpoint below: run `web_search` with the literal endpoint URL (or close to it) as the query first — it reliably surfaces the URL as an indexed result — then `web_fetch` that URL. Do this once per endpoint per session; each endpoint returns the *entire market* in one response, so don't refetch it per stock code.

If a path below has drifted (returns 404 or unexpected shape), fetch the swagger docs to find the current path rather than guessing:
- TWSE swagger: `https://openapi.twse.com.tw/v1/swagger.json` (confirmed reliably fetchable)
- TPEx swagger: `https://www.tpex.org.tw/openapi/swagger.json` (confirmed to sometimes trigger bot-detection blocks on direct fetch — see fallback below)

**Known limitation — TPEx (上櫃) bot detection:** `tpex.org.tw` endpoints occasionally reject `web_fetch` outright with a bot-detection error, even though the same URL works fine in a browser. This is intermittent, not universal — always try the direct fetch first. When it's blocked:
1. Retry once (transient blocks sometimes clear).
2. Fall back to `mops.twse.com.tw` (公開資訊觀測站) via `web_search` + `web_fetch` — MOPS covers **both** 上市 and 上櫃 companies for company profile, material announcements, and monthly revenue, just through a different (HTML, ajax-based) interface than the clean OpenAPI JSON. Search for the specific MOPS query page for the data type needed (e.g. "mops.twse.com.tw 個股 基本資料" or "mops.twse.com.tw 重大訊息").
3. If both fail for a given data point, say so plainly in the report ("此項資料因上櫃資料來源暫時無法存取，未能取得") rather than silently leaving a blank or guessing — this is more useful to the user than a fabricated number.

This limitation does not affect 上市 (TWSE) stocks, whose OpenAPI has been reliable in testing.

## Step 1: Resolve & validate a code — company profile

- TWSE: `https://openapi.twse.com.tw/v1/opendata/t187ap03_L`
- TPEx: search for the TPEx equivalent via the swagger doc above (field/endpoint names on TPEx shift more often than TWSE's) — look for a company-profile / 基本資料 endpoint.

Both return one JSON array covering the whole market. Match on the company code field (TWSE: `公司代號`). Useful fields from the TWSE payload:
- `公司名稱` (full legal name), `公司簡稱` (short name — use this in filenames)
- `成立日期`, `上市日期` (ROC-calendar `YYYYMMDD` where YYY is years since 1911 — e.g. `19620209` here is actually a **Gregorian** date already for pre-1990 fields on this particular endpoint; TWSE dates on t187ap03_L are Gregorian `YYYYMMDD`, but some *other* TWSE endpoints (like monthly revenue) use ROC calendar `YYYMMDD` — always sanity-check the digit count: 8 digits = Gregorian, 7 digits = ROC (add 1911 to the leading 3 digits))
- `董事長`, `總經理`, `產業別` (industry code), `營業項目`/business description if present, `網址`

A code found in neither payload is invalid — record and skip.

## Step 2: Latest price

TWSE has same-day and recent quote endpoints (see swagger for exact path names, e.g. under 證券交易/exchangeReport tag — `BWIBBU_ALL` gives P/E, dividend yield, P/B by code; other endpoints give daily OHLC). TPEx has equivalent daily close-quote endpoints under its swagger doc. These typically only return the *current/latest* trading day — there's no long history in one call, so for a "price over the period" chart, accumulate by checking whether a historical daily-quotes endpoint exists (check swagger), or fall back to a monthly average endpoint if daily history isn't available via OpenAPI.

## Step 3: Revenue (monthly & quarterly)

- TWSE monthly revenue: `https://openapi.twse.com.tw/v1/opendata/t187ap05_L` — whole-market snapshot of the **latest** month's revenue with prior-month and year-ago comparisons. To build a multi-month trend within the reporting period, you generally only get the current snapshot per call; if no historical-series endpoint exists, note in the report that monthly revenue reflects the latest disclosed figures rather than a full historical pull, and lean on MOPS's own monthly-revenue history page (queried via `web_search` + `web_fetch`) to backfill earlier months of the current year if needed.
- TPEx: equivalent monthly revenue endpoint — check swagger for exact path (naming convention differs from TWSE's `t187apXX_L`).
- Quarterly figures: derive from quarterly financial statement endpoints (TWSE swagger lists several under 財務報表, e.g. `t187ap06_L_ci` for income statement) or by summing three months of monthly revenue as an approximation — label clearly which method was used.

Date fields on these endpoints are typically **ROC calendar** (e.g. `1150730` = ROC year 115, i.e. 2026, July 30). Always convert before charting or displaying: Gregorian year = ROC year + 1911.

## Step 4: Material announcements (重大訊息)

- TWSE: `https://openapi.twse.com.tw/v1/opendata/t187ap04_L` — whole-market daily material announcements. Filter by company code, keep everything within the reporting period, sort most-recent-first. Watch for disclosure titles mentioning re-filing/correction of financial reports (公告更正/重新公告財務資訊) — these are worth flagging explicitly per the user's request to catch fast-rising stocks getting flagged for extra disclosure.
- TPEx: equivalent announcements endpoint via its swagger doc, or fall back to searching MOPS directly (`mops.twse.com.tw` covers both markets for announcements) via `web_search` + `web_fetch`.

## Step 5: Institutional trading (三大法人買賣超)

- TWSE: swagger lists endpoints under 證券交易 for three-institutional-investors buy/sell by stock (foreign investors 外資, investment trusts 投信, dealers 自營商). Search the swagger doc for "三大法人" to find the exact current path.
- TPEx: equivalent under its own swagger doc.
- These are typically daily snapshots too — accumulate across the reporting period the same way as price data, or note if only the latest day is available.

## Step 6: News

Use `web_search` for recent news, favoring the company's short name + code as the query (e.g. "台積電 2330 營收"), and prefer results from the last 1-3 months. Standard copyright rules apply: paraphrase, don't reproduce more than a short quote (<15 words), one quote per source maximum.

## A note on Goodinfo / Yahoo奇摩股市 / 鉅亨網

These sites render tables via client-side JavaScript. `web_fetch` on them typically returns an empty shell page (confirmed: fetching `goodinfo.tw/tw/StockDetail.asp?STOCK_ID=<any code>` returns essentially nothing useful regardless of whether the code is valid). Use them only as a `web_search` source for news/context/qualitative commentary, never as a structured-data source — the official OpenAPI endpoints above are more reliable and are safe to cite.
