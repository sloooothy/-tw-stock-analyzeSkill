# HTML report template

Before writing the report file, skim `/mnt/skills/public/frontend-design/SKILL.md` for this environment's styling conventions (fonts, spacing, color tokens) — apply them here even though this is a single static file, not a component.

The report is one self-contained `.html` file: inline `<style>`, inline `<script>`, no external requests (it must still work if the user opens it later while offline). Use plain SVG or `<canvas>` + vanilla JS for charts — no CDN chart libraries, since the file has to stand alone.

## Required sections, in order

```markdown
# [公司簡稱]（[代號]）財報分析
分析期間：[起始年月] – [結束年月]　資料擷取：[今天日期]

## 公司基本資料
- 成立日期 / 上市或上櫃日期
- 主要業務／產品
- 董事長 / 總經理
- （若為本年度新上市，明確註明資料期間較短的原因）

## 最新股價與財務數據
- 最新收盤價、漲跌、成交量（含資料時間點）
- 期間累計營收、營業利益、稅後淨利（若可取得）

## 重大訊息
- 依日期新到舊列出本年度重大訊息公告
- 特別標註任何「更正/重新公告財務資訊」類公告

## 圖表與原始數據
- 三大法人買賣超比較
- 股價變化
- 月營收
- 季營收
- 每張圖下方放一個「下載 CSV」按鈕（見下方實作方式），以及一行文字資料來源

## 近期新聞
- 3-6 則近期、與財務表現相關的新聞，各一兩句話摘要 + 來源連結
- 依標準著作權規則改寫，不逐字引用超過約 15 字

## 數據總結
- 正面因素／負面因素並陳（條列）
- 相關估值指標（本益比、殖利率、股價淨值比等，若可取得）
- 明確聲明：本節僅整理現有資訊，不構成買進、持有或賣出建議
```

## Self-contained CSV download pattern

Embed the CSV text as a JS string literal and trigger a download via Blob — no server, no external file reference:

```html
<button onclick="downloadCsv()">下載 CSV</button>
<script>
function downloadCsv() {
  const csv = `日期,收盤價,漲跌\n2026-01-02,650,+5\n2026-01-03,648,-2\n`; // escape embedded quotes/newlines correctly
  const blob = new Blob(["\uFEFF" + csv], { type: "text/csv;charset=utf-8;" }); // BOM so Excel on Windows reads UTF-8 correctly
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "monthly_revenue.csv";
  a.click();
  URL.revokeObjectURL(url);
}
</script>
```

One button per chart (or one combined button for all four datasets, whichever is cleaner given how much data there is) — the person asked for the ability to choose whether to download, so the button must be optional/inert until clicked, never auto-download on page load.

## Chart rendering without external libraries

A simple approach that keeps the file self-contained: render each chart as an inline SVG built from the data at file-generation time (compute point coordinates in Python/JS ahead of time and write static SVG paths), or use `<canvas>` with vanilla JS that runs on page load from an embedded data array. Either is fine — prioritize the numbers being readable (axis labels, units, a legend for the three institutional-investor series) over visual polish.

## Filenames

`/mnt/user-data/outputs/<code>_<公司簡稱>_report.html` and `/mnt/user-data/outputs/<code>_<公司簡稱>_data.csv`, e.g. `1815_富喬_report.html`.
