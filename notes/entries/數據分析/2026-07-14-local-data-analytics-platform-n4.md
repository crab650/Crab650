---
source: personal-learning-workspace
content_type: note
source_id: 4
published_date: 2026-07-14
---

# Local Data Analytics Platform

> 分類：數據分析  
> Tags：無  
> 建立日期：2026-07-14

# 新資料集操作學習指南

這份文件記錄如何把一份陌生 CSV 接入 Local Data Analytics Platform。範例使用
`Sample - Superstore.csv`，但流程可以重複套用到其他資料集。

## 核心觀念

框架不修改原始 CSV。每套資料集都放在獨立目錄：

```text
datasets/<dataset_name>/
  source.csv
  dataset_config.json
  cleaning_rules.py
  analysis_views.sql
  charts.py                 # 選用：資料集專用圖表
```

資料依序經過：

```text
CSV → SQLite Raw → SQLite Clean → DuckDB Fact/Dimensions
    → DuckDB Views → CSV/Excel/Markdown/PNG
```

## 步驟 1：保存原始資料

先把收到的 CSV 放在專案根目錄供檢查，不要使用 Excel 另存或手動修正內容，
以免改變編碼、日期或數值。

本次原始檔：

```text
Sample - Superstore.csv
```

正式接入時複製為：

```text
datasets/superstore/source.csv
```

## 步驟 2：初步檢查

至少確認：

- 檔案編碼
- 列數與欄數
- 欄位名稱
- 缺值與重複列
- 每一列代表的業務事件
- 日期與數值是否能解析

Superstore 的檢查結果：

- 編碼：`cp1252`
- 9,994 列、21 欄
- 每列是一張訂單中的一項商品
- 完全重複列為 0
- 日期範圍為 2014-01-03 至 2018-01-05

## 步驟 3：選擇主鍵

主鍵必須非空且唯一。不要因為欄位名稱看起來像 ID 就直接採用。

Superstore：

- `Row ID`：9,994 個唯一值，適合作為主鍵
- `Order ID`：只有 5,009 個唯一值，一張訂單可有多個商品，因此不適合作為列主鍵
- `Order ID + Product ID` 仍有重複，不能取代 `Row ID`

清理後，`Row ID` 會改名為 SQL 友善的 `row_id`。

## 步驟 4：建立 dataset_config.json

重要設定：

```json
{
  "schema_version": 1,
  "name": "superstore",
  "source_file": "source.csv",
  "source_encoding": "cp1252",
  "primary_key": "row_id"
}
```

- `schema_version`：目前固定為 1
- `name`：必須與資料集目錄名稱一致
- `source_encoding`：必須使用實際 CSV 編碼
- `required_columns`：來源 CSV 必須包含的欄位
- `primary_key`：清理後資料使用的唯一鍵

## 步驟 5：只驗證，不匯入

先執行：

```powershell
python .\run_dataset.py superstore --validate-only
```

這會驗證設定、路徑、編碼、CSV 表頭與必要欄位，不會建立分析資料表。

查看可用資料集：

```powershell
python .\run_dataset.py --list
```

## 步驟 6：匯入 Raw table

驗證成功後才執行：

```powershell
python .\run_dataset.py superstore --step import_csv
```

結果會寫入：

```text
database/superstore.db → SuperstoreRaw
```

Raw table 應保留來源內容，只額外加入 `imported_at`。

## 步驟 7：清理資料

`cleaning_rules.py` 負責資料集特有的轉換。本例會：

- 將欄名轉為 snake_case
- 解析下單與出貨日期
- 計算 `shipping_days`
- 將 Sales、Quantity、Discount、Profit 轉為數值

執行：

```powershell
python .\run_dataset.py superstore --step clean_data
```

結果會寫入 `SuperstoreClean`，並產生資料品質報告。清理不代表刪除不理想的
業務結果，例如負利潤可能是重要分析訊號。

## 步驟 8：建立資料倉庫

確認 Clean table 後執行：

```powershell
python .\run_dataset.py superstore --step build_dw
```

結果會寫入：

```text
database/superstore_dw.duckdb
```

先從 Fact table 開始，確定需求後再增加 dimensions，避免尚未理解資料就過度建模。

## 步驟 9：定義分析問題

先寫業務問題，再寫 SQL。例如：

1. 總銷售額、總利潤與訂單數是多少？
2. 每月銷售與利潤如何變化？
3. 哪些類別、地區或商品正在虧損？
4. 折扣與利潤之間有什麼關係？

每一項分析在 `analysis_views.sql` 建立 view，再於 `exports` 設定輸出查詢。

## 步驟 10：建立 Views 與報表

```powershell
python .\run_dataset.py superstore --step create_views
python .\run_dataset.py superstore --step export_reports
```

完整執行則使用：

```powershell
python .\run_dataset.py superstore
```

輸出位置：

```text
output/superstore/csv/
output/superstore/excel/
output/superstore/reports/
output/superstore/charts/
```

## 常用除錯指令

```powershell
python .\run_dataset.py superstore --validate-only
python .\run_dataset.py superstore --from-step clean_data
python .\run_dataset.py superstore --verbose
python .\run_dataset.py superstore --log-file .\output\superstore.log
python -m pytest
```

## 圖形化查看 DuckDB

雙擊專案根目錄的：

```text
開啟Superstore_DuckDB檢視器.bat
```

工具會以唯讀模式開啟 `database/superstore_dw.duckdb`。可以選擇資料表、預覽
前 200 筆、查看欄位資訊，或輸入一個 `SELECT` 查詢。為避免誤改框架產物，
工具不允許 UPDATE、DELETE、DROP 等寫入操作。

## 使用框架畫圖

框架的 `framework/charts.py` 提供常用函式：

```text
line_chart              折線圖
bar_chart               長條圖
horizontal_bar_chart    水平長條圖
stacked_bar_chart       堆疊長條圖
area_chart              面積圖
scatter_chart           散點圖
histogram               直方圖
box_chart               箱型圖
pie_chart               圓餅圖
heatmap                 熱力圖
```

每套資料集可建立 `charts.py` 並定義 `generate_charts(frames, output_dir)`。
執行 `export_reports` 時，框架會自動呼叫它。本例的 Monthly Trend：

```python
line_chart(
    frames["monthly_trend"],
    x="month_end",
    y=["total_sales", "total_profit"],
    title="Monthly Sales & Profit Trend",
    output_path=output_dir / "monthly_sales_profit_trend.png",
)
```

## 本次學習進度

- [x] 放入並保存原始 CSV
- [x] 檢查編碼、欄位、筆數和樣本
- [x] 確認資料粒度與主鍵
- [x] 建立 Superstore 資料集骨架
- [x] 建立初版設定與清理規則
- [x] 執行並理解 validate-only
- [x] 匯入並檢查 Raw table
- [x] 清理並檢查 Clean table
- [x] 建立並檢查 DuckDB Fact table
- [x] 定義第一組分析問題
- [x] 建立 Views 與輸出報表

### 第一個分析：Monthly Trend

使用 `vw_monthly_trend` 將 pandas 的月底分組概念轉為 DuckDB SQL：

```sql
SELECT
    LAST_DAY(CAST(order_date_clean AS DATE)) AS month_end,
    SUM(sales) AS total_sales,
    SUM(profit) AS total_profit
FROM FactSuperstore
GROUP BY month_end
ORDER BY month_end;
```

其中 `LAST_DAY` 對應 pandas 的 `freq="ME"`，每列代表一個月份。

### 第二個分析：Sub-Category Summary

使用加總而不是 `pivot_table` 預設的平均值，依 Category、Sub-Category 計算
筆數、總銷售、總利潤與利潤率。輸出兩張水平長條圖，分別比較 Sales 與
Profit，避免兩者量級不同而讓 Profit 線條不明顯。

## 目前實作結果

已建立 DuckDB relations：

```text
FactSuperstore
vw_monthly_trend
vw_subcategory_summary
```

已建立圖表：

```text
output/superstore/charts/monthly_sales_profit_trend.png
output/superstore/charts/subcategory_sales.png
output/superstore/charts/subcategory_profit.png
```

Sub-Category 分析包含 17 個子類別。Tables、Bookcases、Supplies 的總利潤為負，
這些是後續值得深入分析的業務訊號。

## Windows 檔案鎖定注意事項

執行 `build_dw`、`create_views` 或 `export_reports` 前，先關閉正在查看框架產物的
程式或預覽頁面，包括：

- DuckDB Viewer
- CSV/Excel 編輯器或預覽
- PNG 圖片預覽
- VS Code CSV Preview

Windows 可能禁止框架替換正在開啟的 `.duckdb`、`.csv`、`.xlsx` 或 `.png`。
這不是資料分析錯誤；關閉檔案後重跑對應步驟即可。

## 下次續作

目前 Monthly PNG 預覽仍可能鎖住正式輸出。關閉所有 Superstore 輸出預覽後執行：

```powershell
python .\run_dataset.py superstore --step export_reports
```

成功後確認：

```text
output/superstore/csv/subcategory_summary.csv
output/superstore/excel/superstore_analysis.xlsx
output/superstore/reports/superstore_summary.md
```

更完整的當前狀態與續作資訊請見 `續作交接紀錄.md`。
