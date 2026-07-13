---
source: personal-learning-workspace
content_type: note
source_id: 3
published_date: 2026-07-14
---

# Files-to-Prompt 學習筆記

> 分類：AI  
> Tags：AI  
> 建立日期：2026-07-14

# Files-to-Prompt 學習筆記

## 專案資訊

* **專案名稱：** files-to-prompt
* **作者：** Simon Willison
* **GitHub：** https://github.com/simonw/files-to-prompt
* **語言：** Python
* **類型：** CLI (Command Line Interface) 工具

---

# 一、專案介紹

`files-to-prompt` 是一個專門為大型語言模型（LLM）設計的工具。

它可以將一個或多個資料夾中的程式碼、文件、設定檔等，自動整理成一份完整的 Prompt，方便交給 ChatGPT、Claude、Gemini、Codex 等 AI 模型分析。

它並不是程式分析工具，而是一個 **Context Builder（上下文整理工具）**。

一句話來說：

> **把整個專案整理成 AI 最容易閱讀的格式。**

---

# 二、解決了什麼問題？

以前詢問 AI：

```
請幫我修改我的 ASP.NET Core 專案。
```

AI：

```
請貼程式碼。
```

如果專案有：

* 300 個 C# 檔案
* 50 個 Vue
* 20 個 SQL

根本不可能一個一個貼。

files-to-prompt 可以直接：

```
整個專案
        ↓
自動整理
        ↓
一份 Prompt
        ↓
交給 AI
```

---

# 三、工作流程

```
專案資料夾

        │

        ▼

掃描所有檔案

        │

        ▼

依副檔名篩選

        │

        ▼

排除不需要的資料夾

        │

        ▼

讀取所有文字內容

        │

        ▼

整理成 Prompt

        │

        ▼

輸出 TXT / Markdown / XML
```

---

# 四、主要功能

## 1. 合併多個檔案

例如：

```
Controllers/
Models/
Services/
README.md
```

輸出：

```
Controllers/UserController.cs
------------------

內容

------------------

Models/User.cs

------------------

內容
```

AI 可以一次閱讀整個專案。

---

## 2. 指定副檔名

例如：

```
-e cs
-e vue
-e sql
```

只輸出：

```
*.cs
*.vue
*.sql
```

不需要圖片、DLL、EXE。

---

## 3. Ignore 功能

可以忽略：

```
bin/
obj/
node_modules/
.git/
dist/
```

也支援：

```
*.log
*.tmp
```

---

## 4. Markdown 輸出

可以直接變成：

````text
app.py

```python
...
```
````

方便閱讀。

---

## 5. Claude XML

Claude 官方推薦的 Context 格式。

適合超大型 Prompt。

---

## 6. Line Number

可加入：

```
120
121
122
123
```

AI 可以回答：

> 第 123 行需要修改。

---

# 五、適合哪些 AI？

* ChatGPT
* Claude
* Gemini
* Codex
* Cursor
* Windsurf
* Continue
* OpenHands
* 其他 LLM

---

# 六、適合哪些專案？

Python

```
.py
```

ASP.NET Core

```
.cs
.csproj
```

Vue

```
.vue
.ts
```

React

```
.tsx
.jsx
```

Node.js

```
.ts
.js
```

SQL

```
.sql
```

Markdown

```
.md
```

JSON

```
.json
```

---

# 七、建議的 C# 使用方式

```
files-to-prompt . \
-e cs \
-e csproj \
-e json \
-e vue \
-e ts \
-e sql \
-e md \
--ignore bin \
--ignore obj \
--ignore node_modules \
--ignore dist \
-o project.txt
```

---

# 八、專案架構

```
CLI

↓

掃描資料夾

↓

Ignore Filter

↓

Extension Filter

↓

Read File

↓

Format Prompt

↓

Output
```

每個模組只做一件事情。

符合 Simon Willison 一貫的設計風格。

---

# 九、優點

## 架構簡單

只有一件事情：

> 將專案整理給 AI。

沒有：

* Database
* API
* Web Server

所以非常容易維護。

---

## 高可攜性

任何程式語言都能使用。

例如：

* Python
* C#
* Java
* Go
* Rust

---

## 容易整合

可以搭配：

* GitHub Actions
* Shell Script
* Python
* CI/CD

自動產生 Prompt。

---

# 十、缺點

目前沒有：

* Token 分割
* 專案摘要
* Dependency Analysis
* 呼叫關係分析
* 類別關聯圖
* UML

它只是：

> **整理檔案，不分析內容。**

---

# 十一、我可以如何應用？

## 1. ERP 專案分析

例如：

```
SIF ERP

Controllers

Services

Repositories

SQL
```

全部交給 AI。

詢問：

```
請分析：

1. 系統架構
2. API 設計
3. Repository Pattern
4. 可改善地方
```

---

## 2. 新同事教育

把整個專案整理後：

```
請介紹此系統架構。
```

AI 就能快速說明。

---

## 3. Code Review

直接：

```
請找出：

- Bug
- 重複程式
- 命名問題
- SOLID 問題
```

---

## 4. 文件產生

例如：

```
請根據程式

產生：

README

API 文件

ER Diagram

流程圖
```

---

## 5. AI Coding

例如：

```
新增功能：

Shipping Approval
```

AI 就知道整個專案。

---

# 十二、可以學到哪些設計觀念？

## Single Responsibility Principle

每個模組只做一件事情。

例如：

```
Scanner

↓

Reader

↓

Formatter

↓

Writer
```

---

## CLI Tool 設計

CLI

↓

Core

↓

Output

Simon 很多工具都是這種設計。

例如：

* sqlite-utils
* shot-scraper
* llm
* files-to-prompt

---

## Context Engineering

不是 Prompt Engineering。

而是：

> 如何把資料整理成 AI 最容易理解的格式。

這也是近年 AI Agent 開發的重要能力。

---

# 十三、如果是我，我會升級哪些功能？

新增：

* 專案目錄樹
* Token 自動切割
* C# Dependency Analysis
* Controller → Service → Repository 關聯
* SQL Relationship
* Vue Route 分析
* AI 自動摘要每個檔案
* Mermaid 架構圖
* 自動產生 README
* 專案健康度分析

讓它變成：

> AI Project Analyzer

而不是只有：

> Files to Prompt。

---

# 十四、學習心得

這個專案雖然只有一個簡單的功能，但非常符合 Simon Willison 的開發哲學：

* 一個工具只做好一件事情。
* 保持簡單、可組合、容易維護。
* 優先考慮與其他工具整合，而不是把所有功能都做進同一個專案。

它也讓我了解到，在 AI 時代，「整理上下文（Context Engineering）」和「撰寫 Prompt」同樣重要。當 AI 能看到完整且乾淨的專案內容時，回答品質通常會比只貼幾個檔案高得多。

對我目前開發的 ERP、Flask、ASP.NET Core、Vue、SQL Server 專案而言，這種工具可以大幅提升 AI 協助除錯、重構、產生文件與理解系統架構的效率，也讓我理解 Simon Willison 設計 CLI 工具時追求「小而精、容易組合」的開發思維。
