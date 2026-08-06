---
source: personal-learning-workspace
content_type: note
source_id: 12
published_date: 2026-08-06
---

# Footwear Factory Routing Demo

> 分類：ERP  
> Tags：無  
> 建立日期：2026-08-06

# Footwear Factory Routing Demo

## ERP 鞋廠途程管理系統（Routing Learning Demo）

---

# 專案目標

本專案是一套 **ERP Routing（途程表）學習系統**。

目的不是建立完整 ERP，而是透過一個鞋廠案例，完整理解 ERP 中 Routing 的用途，以及它如何與工令、MES、生產進度、工時、成本串接。

希望完成後，能真正理解：

* Routing 是什麼
* Routing 為什麼存在
* Routing 如何建立
* Routing 如何被 ERP 使用
* Routing 如何控制製造流程
* Routing 如何提供 MES 報工依據
* Routing 如何提供成本計算依據

本專案定位：

> ERP Manufacturing Learning Demo

---

# 工廠背景

工廠：

Footwear Factory

產品：

Running Shoes

ERP 使用者：

* 生管
* 生產主管
* MES 操作員
* IE 工程師
* 成本會計

---

# 專案範圍

本專案只專注於 Routing。

暫時不包含：

* 採購
* 銷售
* 財務
* MRP
* APS
* 委外加工
* 庫存管理

後續再逐步擴充。

---

# ERP 資料流

```text
Item Master
      │
      ▼
Operation Master
      │
      ▼
Work Center
      │
      ▼
Routing
      │
      ▼
Work Order
      │
      ▼
Work Order Operation
      │
      ▼
MES Reporting
      │
      ▼
Progress
      │
      ▼
Time Analysis
      │
      ▼
Cost Analysis
```

---

# Demo 操作流程

使用者依照畫面一步一步完成整個 ERP 流程。

```
Step 1
建立產品

↓

Step 2
建立工作中心

↓

Step 3
建立製程

↓

Step 4
建立 Routing

↓

Step 5
建立 Work Order

↓

Step 6
Routing 自動展開

↓

Step 7
MES 報工

↓

Step 8
查看工令進度

↓

Step 9
查看 Routing 成本與工時分析
```

每一步都可以看到 ERP 資料如何流動。

---

# Step 1：產品主檔（Item Master）

目的：

建立產品。

例如：

| Item Code | Item Name     |
| --------- | ------------- |
| SHOE-001  | Running Shoes |

欄位：

* ItemCode
* ItemName
* Category
* BaseUOM

完成後：

產品即可建立 Routing。

---

# Step 2：工作中心（Work Center）

建立工廠工作站。

例如：

| Code  | Name            |
| ----- | --------------- |
| CUT   | Cutting         |
| PRINT | Printing        |
| SEW   | Sewing          |
| GLUE  | Sole Gluing     |
| QC    | Quality Control |
| PACK  | Packing         |

欄位：

* WorkCenterCode
* WorkCenterName
* Labor Rate
* Machine Rate
* Daily Capacity

用途：

Routing 每一道工序都會指定工作中心。

---

# Step 3：製程主檔（Operation Master）

建立標準製程。

例如：

| Code | Name          |
| ---- | ------------- |
| OP10 | Cutting       |
| OP20 | Printing      |
| OP30 | Sewing        |
| OP40 | Sole Gluing   |
| OP50 | Quality Check |
| OP60 | Packing       |

欄位：

* OperationCode
* OperationName
* Description

---

# Step 4：建立 Routing

建立產品製造流程。

Running Shoes：

```text
Cutting
   ↓
Printing
   ↓
Upper Sewing
   ↓
Sole Gluing
   ↓
Quality Check
   ↓
Packing
```

Routing Header：

* Routing Code
* Product
* Version
* Status

Routing Detail：

* Sequence
* Operation
* Work Center
* Setup Time
* Run Time
* Queue Time
* Move Time

例如：

| Seq | Operation     | Work Center | Std Time |
| --- | ------------- | ----------- | -------- |
| 10  | Cutting       | CUT         | 5 min    |
| 20  | Printing      | PRINT       | 3 min    |
| 30  | Upper Sewing  | SEW         | 20 min   |
| 40  | Sole Gluing   | GLUE        | 15 min   |
| 50  | Quality Check | QC          | 8 min    |
| 60  | Packing       | PACK        | 5 min    |

畫面功能：

* 新增工序
* 修改工序
* 刪除工序
* 上移
* 下移
* 顯示流程圖
* 顯示總標準工時

---

# Step 5：建立 Work Order

例如：

工令：

WO-000001

產品：

Running Shoes

數量：

1000 雙

建立工令時：

ERP 自動讀取 Routing。

---

# Step 6：Routing 自動展開

建立工令後：

ERP 自動產生：

Work Order Operation

例如：

| Seq | Operation     | Status  |
| --- | ------------- | ------- |
| 10  | Cutting       | Waiting |
| 20  | Printing      | Waiting |
| 30  | Upper Sewing  | Waiting |
| 40  | Sole Gluing   | Waiting |
| 50  | Quality Check | Waiting |
| 60  | Packing       | Waiting |

此時即可展示：

> Routing 如何真正變成工令工序。

---

# Step 7：MES 報工

MES 操作：

開始工序

↓

完成工序

↓

輸入：

* Good Qty
* Reject Qty
* Start Time
* End Time

例如：

Cutting：

Good Qty：

1000

Reject：

5

完成後：

下一站：

Printing

即可開始。

讓使用者理解：

Routing 控制整個工序流程。

---

# Step 8：工令進度

畫面：

| Operation    | Good | Reject | Status    |
| ------------ | ---- | ------ | --------- |
| Cutting      | 1000 | 5      | Completed |
| Printing     | 995  | 0      | Running   |
| Upper Sewing | 500  | 0      | Running   |

畫面應顯示：

* 已完成工序
* 生產中工序
* 尚未開始工序
* 完成率
* 良率

讓使用者知道：

ERP 如何追蹤目前做到哪一道工序。

---

# Step 9：Routing 工時與成本分析

比較：

標準工時

VS

MES 實際工時

例如：

| Operation | Std   | Actual |
| --------- | ----- | ------ |
| Cutting   | 5 hr  | 6 hr   |
| Sewing    | 20 hr | 19 hr  |

人工成本：

人工工時 × 人工費率

設備成本：

設備工時 × 設備費率

最後顯示：

* 每道工序成本
* 總加工成本
* 標準工時差異
* 實際工時差異
* 效率分析

---

# Demo 畫面

* Dashboard
* Product Master
* Work Center
* Operation Master
* Routing
* Work Order
* Work Order Operation
* MES Reporting
* Production Progress
* Time Analysis
* Cost Analysis

---

# 使用技術

Backend：

* Python
* Flask
* SQLAlchemy
* SQLite

Frontend：

* Bootstrap 5
* Jinja2
* Chart.js

---

# 本專案希望學到

完成後，希望能真正理解：

* Routing（途程）
* Operation（工序）
* Work Center（工作中心）
* Work Order（工令）
* Work Order Operation（工令工序）
* MES Reporting（報工）
* Standard Time（標準工時）
* Actual Time（實際工時）
* Manufacturing Flow（製造流程）
* Routing Cost（加工成本）

而不是只停留在 ERP 名詞。

---

# 下一階段規劃

完成 Routing Demo 後，逐步擴充：

Phase 2

* BOM Demo

Phase 3

* Material Issue（領料）

Phase 4

* Inventory（庫存）

Phase 5

* MRP

Phase 6

* Cost Rollup

Phase 7

* Full Manufacturing ERP

最終形成一套完整的 ERP 製造系統學習平台。
