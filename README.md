# Taiwan Stock Analyzer (tw-stock-analyzer.skill)

A Claude Skill designed for analyzing Taiwan stock financial reports, corporate data, and market trends, generating interactive HTML reports with built-in visualizations and data export functionality.

## Features

- **Automated Symbol Extraction**: Supports single or multiple Taiwan stock tickers, stock names, or mixed queries. Automatically filters out invalid symbols.
- **Comprehensive Financial Data Gathering**: Fetches data for the current year to the current month from trusted sources including MOPS (Market Observation Post System) and TWSE (Taiwan Stock Exchange).
- **Interactive HTML Reports**: Generates standalone HTML artifacts containing basic company profiles, financial metrics, MOPS announcements, interactive charts, and financial news summaries.
- **Chart Visualizations & Data Export**: Built-in interactive charts for institutional investor trading, stock price trends, monthly revenue, and quarterly revenue, complete with CSV export buttons.
- **Objective & Unbiased Analysis**: Presents balanced metrics, positive and negative factors, and valuation indicators without offering direct buy/sell investment advice.

## Data Sources

- **MOPS (公開資訊觀測站)**: Financial statements, material announcements, basic company profiles.
- **TWSE (臺灣證券交易所)**: Institutional trading activity, stock prices, market data.
- **Financial News**: Recent relevant financial news and updates.

## Installation & Setup

1. Clone or download this repository.
2. Ensure your workspace folder name follows standard conventions (alphanumeric, underscores, hyphens only, e.g., `tw-stock-analyzer`).
3. Copy the contents of `tw-stock-analyzer.skill` or upload it to your Claude environment (**Settings > Customize > Skills** in the Claude web interface).
4. Ensure the required capability (Code execution and file creation) is enabled in Claude Settings.

## Usage

Simply enter one or more Taiwan stock symbols (e.g., `2330`, `1815`, `2330 台積電`) into the conversation. Claude will automatically trigger the skill, gather the relevant financial data, and render an interactive HTML report artifact.

## Directory Structure

```text
tw-stock-analyzer/
├── tw-stock-analyzer.skill          # Main skill instruction file
└── files/                            # Test cases, sample inputs, and reference files
```


# 台股財報分析技能 (tw-stock-analyzer.skill)

此為適用於 Claude 的 Skill 模組，專門用於抓取並分析台灣股市個股財報、公司基本面與市場動態，並生成包含互動圖表與 CSV 下載功能的獨立 HTML 報告。

## 功能特點

- **自動代號辨識**：支援單一或多個台股代號、公司名稱或混合輸入，自動過濾無效格式。
- **完整財報數據抓取**：自動檢索當年度至最新月份之公開資訊（資料來源包含公開資訊觀測站 MOPS 與證券交易所 TWSE）。
- **互動式 HTML 報告**：自動生成包含基本資訊、財務指標、重大訊息、互動式圖表及新聞摘要的獨立 HTML Artifacts。
- **數據圖表化與匯出**：內建三大法人買賣超、股價走勢、月營收與季營收等動態圖表，並提供一鍵下載 CSV 功能。
- **客觀數據分析**：提供正反面因素與估值指標，保持中立客觀，不提供任何買賣建議。

## 資料來源

- **公開資訊觀測站 (MOPS)**：財務報表、重大訊息公告、公司基本資料。
- **臺灣證券交易所 (TWSE)**：三大法人買賣超、股價歷史數據、市場相關資訊。
- **財經新聞來源**：近期與該個股相關之重大財務新聞。

## 安裝與設定方式

1. 下載或複製本專案資料庫。
2. 請確保專案根目錄名稱符合規範（僅包含英數字、底線與連字號，例如 `tw-stock-analyzer`）。
3. 複製 `tw-stock-analyzer.skill` 的內容，或將其上傳至 Claude 網頁版的 **Settings > Customize > Skills**（設定 > 客製化 > 技能）。
4. 請確認 Claude 設定中的相關功能（Code execution and file creation）已開啟。

## 使用方式

在對話中直接輸入一個或多個台股代號（例如 `2330`、`1815` 或 `2330 台積電`），Claude 將會自動觸發該技能，即時檢索最新數據並生成專屬的 HTML 財報分析報告。

## 目錄結構

```text
tw-stock-analyzer/
├── tw-stock-analyzer.skill          # 主要 Skill 指令與設定檔
└── files/                            # 測試範例、範本與參考資料夾
```
