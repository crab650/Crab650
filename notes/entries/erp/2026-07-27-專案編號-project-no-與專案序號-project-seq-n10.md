---
source: personal-learning-workspace
content_type: note
source_id: 10
published_date: 2026-07-27
---

# 專案編號（Project No）與專案序號（Project Seq）

> 分類：ERP  
> Tags：無  
> 建立日期：2026-07-27

````markdown
# ERP 學習筆記：專案編號（Project No）與專案序號（Project Seq）

## 什麼是專案編號（Project No）？

專案編號（Project No）是 ERP 用來**串聯同一個專案或同一筆業務流程**的重要欄位。

它的目的不是代表一張單據，而是代表：

- 同一位客戶
- 同一張銷售訂單
- 同一個產品專案
- 同一個工程案件

只要屬於同一個專案，相關單據都可以使用相同的 Project No。

---

# 為什麼需要 Project No？

一張銷售訂單（Sales Order）往往不會只產生一張採購單。

例如：

客戶訂購 100 台腳踏車。

ERP 發現需要採購：

- 車架
- 輪胎
- 坐墊
- 鏈條

因此會產生多張請購（PR）與多張採購單（PO）。

流程如下：

```text
Sales Order (SO1001)
        │
        ├── PR0001（輪胎）
        ├── PR0002（車架）
        ├── PR0003（坐墊）
        └── PR0004（鏈條）
```

每一張 PR 又可能建立不同的 PO。

例如：

```text
PR0001
    │
    └── PO0001（輪胎供應商）

PR0002
    │
    └── PO0002（車架供應商）

PR0003
    │
    └── PO0003（坐墊供應商）
```

如果沒有 Project No，

ERP 很難知道：

> 這幾張 PO 到底屬於哪一張 Sales Order。

因此所有單據都會填入相同的 Project No。

例如：

Project No：

```text
P202607001
```

---

# Project No 如何串聯整個流程？

所有相關單據都使用相同 Project No。

| 文件 | 單號 | Project No |
|------|------|------------|
| Sales Order | SO1001 | P202607001 |
| Purchase Request | PR0001 | P202607001 |
| Purchase Request | PR0002 | P202607001 |
| Purchase Order | PO0001 | P202607001 |
| Purchase Order | PO0002 | P202607001 |
| 收貨 | GR0001 | P202607001 |
| 應付發票 | AP0001 | P202607001 |

ERP 就知道：

這些資料全部都是同一個專案。

---

# Project Seq（專案序號）是什麼？

有時候：

同一個 Project No 底下，

還會包含很多不同工作。

例如：

Project：

```text
P202607001
```

下面有：

| Project No | Seq | 工作內容 |
|------------|-----|----------|
| P202607001 | 001 | 車架採購 |
| P202607001 | 002 | 輪胎採購 |
| P202607001 | 003 | 坐墊採購 |
| P202607001 | 004 | 包裝材料 |

Project No 相同，

Project Seq 不同。

目的就是：

**方便區分同一專案中的不同工作或不同採購項目。**

---

# 實務案例：腳踏車工廠

客戶：

```text
GIANT
```

下單：

```text
100 台腳踏車
```

ERP 建立：

```text
Sales Order

SO1001

Project No：
BIKE-2026-001
```

系統發現需要採購：

```text
PR001
輪胎

PR002
車架

PR003
坐墊
```

建立 PO：

```text
PO001
輪胎供應商

PO002
車架供應商

PO003
坐墊供應商
```

所有資料都帶：

```text
Project No：
BIKE-2026-001
```

因此 ERP 能一路追蹤：

```text
Sales Order
        │
        ▼
Purchase Request
        │
        ▼
Purchase Order
        │
        ▼
Goods Receipt
        │
        ▼
AP Invoice
        │
        ▼
付款
```

最後：

所有成本都歸屬到：

```text
BIKE-2026-001
```

---

# ERP 可以查詢哪些資訊？

只要輸入：

```text
Project No：
BIKE-2026-001
```

ERP 可以立即查出：

- 有哪些 Sales Order
- 建立了哪些 PR
- 建立了哪些 PO
- 收貨是否完成
- 發票是否已開立
- 是否已付款
- 專案總採購金額
- 專案材料成本
- 專案利潤分析

Project No 就像一個「追蹤碼」，把整個流程串在一起。

---

# 一般採購需要 Project No 嗎？

不一定。

例如：

公司固定採購：

- A4 紙
- 文具
- 清潔用品

通常：

```text
Project No
空白
```

因為不是屬於某一個客戶專案。

---

但是：

如果：

```text
TESLA
新產品開發案
```

所有採購：

```text
Project No
TESLA-001
```

就能知道：

這些採購都是為了 TESLA 專案。

---

# ERP 資料流

```text
                 Sales Order
                     │
                     │
         Project No = BIKE-2026-001
                     │
                     ▼
          Purchase Request (PR)
                     │
                     ▼
          Purchase Order (PO)
                     │
                     ▼
             Goods Receipt
                     │
                     ▼
              AP Invoice
                     │
                     ▼
                  Payment
                     │
                     ▼
          專案成本／利潤分析
```

---

# 與 Project Seq 的關係

```text
Project No
BIKE-2026-001
        │
        ├── Seq 001（車架）
        ├── Seq 002（輪胎）
        ├── Seq 003（坐墊）
        └── Seq 004（包裝）
```

- **Project No**：代表整個專案。
- **Project Seq**：代表專案中的不同工作、材料或採購項目。

---

# 重點整理

## Project No

- 用來串聯整個 ERP 流程。
- 同一專案使用相同 Project No。
- 可跨 Sales、Purchase、Inventory、Accounting 等模組。
- 可追蹤專案成本與利潤。

## Project Seq

- 用來區分同一專案中的不同工作。
- 一個 Project No 可對應多個 Project Seq。
- 常用於大型專案、工程案或多階段採購。

---

# 一句話記住

> **Project No 是整個專案的身分證；Project Seq 是專案裡每一項工作的流水號。**

ERP 利用 **Project No** 將銷售、請購、採購、收貨、發票、付款及成本分析全部串聯起來，再透過 **Project Seq** 區分同一專案中的不同工作內容，使整個專案的進度與成本都能完整追蹤。
````
