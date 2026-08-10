---
source: personal-learning-workspace
content_type: note
source_id: 14
published_date: 2026-08-10
---

# Flask SECRET_KEY：為什麼 Session 需要簽章？

> 分類：Python  
> Tags：Session / Cookie / SECRET_KEY / Web Security  
> 建立日期：2026-08-10

# Flask SECRET_KEY：為什麼 Session 需要簽章？

> 學習主題：Flask / Session / Cookie / SECRET_KEY / Web Security

今天學習 Flask 的 `SECRET_KEY`。一開始我不太理解：「為什麼一個網站需要一組 SECRET_KEY？」

理解 Session Cookie 的運作方式之後，就會發現它主要是在解決一個非常重要的問題：

> **Client 傳回 Server 的資料不能直接相信。**

---

## 1. SECRET_KEY 是什麼？

Flask 的 `SECRET_KEY` 可以簡單理解成：

> **Server 私藏的一顆防偽印章。**

它不是：

* 使用者登入密碼
* API Key
* 資料庫密碼

它主要用來對 Session Cookie 等資料進行**簽章（Signing）**。

目的不是讓別人看不到資料，而是讓 Server 可以判斷：

> **這份資料是不是我之前簽發的？中間有沒有被修改？**

---

# 2. 先理解 Session

假設使用者登入成功後，我們在 Flask 裡面寫：

```python
session["user_id"] = 25
session["display_name"] = "Honda"
```

這代表 Server 希望記住：

```text
user_id = 25
display_name = Honda
```

這樣使用者下一次 Request 進來時，我們就知道：

```text
這個使用者是誰？
```

---

# 3. Session Cookie 放在哪裡？

Flask 預設的 Session 是以 Cookie 的方式交給瀏覽器保存。

概念上：

```text
Flask Server
     │
     │ 登入成功
     ▼
建立 Session

user_id = 25
display_name = Honda

     │
     ▼
產生 Session Cookie
     │
     ▼
Client Browser
```

之後使用者再次 Request：

```text
Client Browser
     │
     │ 帶著 Cookie
     ▼
Flask Server
```

Server 就可以利用 Session 判斷目前是哪一個使用者。

---

# 4. 問題：Client 可以修改資料

這就是今天最重要的問題。

假設完全沒有任何保護，Client 保存：

```text
user_id = 25
```

Server 收到之後直接相信：

```python
user_id = cookie["user_id"]

records = get_glucose_records(user_id)
```

正常情況：

```text
user_id = 25
      ↓
查詢 user_id = 25
      ↓
看到自己的資料
```

但是 Cookie 是放在 Client 端。

如果有人把：

```text
user_id = 25
```

改成：

```text
user_id = 24
```

而 Server 完全不驗證，Server 可能會認為：

```text
你就是 user_id = 24
```

然後查詢：

```sql
SELECT *
FROM glucose_records
WHERE user_id = 24;
```

這樣就可能看到其他使用者的資料。

---

# 5. 更危險的情況：修改權限

假設 Session 裡還保存：

```text
user_id = 25
role = user
```

攻擊者如果可以任意修改 Cookie，就可能嘗試改成：

```text
user_id = 25
role = admin
```

如果 Server 直接相信 Client：

```python
if session["role"] == "admin":
    allow_admin_page()
```

那就可能變成：

```text
普通使用者

     ↓

修改 Cookie

     ↓

role = admin

     ↓

Server 相信

     ↓

取得不應該擁有的權限
```

所以真正危險的地方不是「Cookie 被修改」這件事本身。

而是：

> **Server 如果相信被修改過的 Cookie，就可能做出錯誤的授權決定。**

---

# 6. SECRET_KEY 開始發揮作用

Flask 不會單純把 Session 資料交給 Client。

它會使用 `SECRET_KEY` 對 Session 產生簽章。

概念上：

```text
Session

user_id = 25
role = user

       +

SECRET_KEY

       ↓

   簽章演算法

       ↓

Signature
```

最後交給 Client 的 Cookie，可以簡化理解成：

```text
資料 + 簽章
```

例如：

```text
user_id = 25
role = user
signature = ABC123
```

注意：

`SECRET_KEY` **不會交給 Client**。

它只存在 Server。

---

# 7. Client 再次 Request 時發生什麼？

Client 不需要重新產生簽章。

它只需要把 Server 原本給它的 Cookie 帶回來：

```text
Client

Cookie:
user_id = 25
role = user
signature = ABC123

        │
        ▼

      Server
```

Server 收到之後，使用自己的：

```text
SECRET_KEY
```

重新計算一次簽章。

例如：

```text
user_id = 25
role = user
     +
SECRET_KEY

     ↓

重新計算

     ↓

ABC123
```

然後比較：

```text
Client 帶回來：

ABC123

Server 自己算：

ABC123
```

兩個相同：

```text
ABC123 == ABC123

✅ 驗證成功
```

代表資料沒有被修改。

---

# 8. 如果 Client 偷改資料呢？

假設原本：

```text
user_id = 25
role = user
signature = ABC123
```

Client 偷偷修改：

```text
user_id = 25
role = admin
signature = ABC123
```

但是他不知道 `SECRET_KEY`，所以只能保留原本的簽章。

Server 收到後重新計算：

```text
user_id = 25
role = admin
     +
SECRET_KEY

     ↓

重新計算

     ↓

XYZ789
```

比較：

```text
Client：

ABC123

Server：

XYZ789
```

結果：

```text
ABC123 != XYZ789

❌ 驗證失敗
```

Server 就知道：

> **這份 Session 被修改過。**

---

# 9. 為什麼攻擊者不能自己產生新的簽章？

因為產生合法簽章需要：

```text
資料
+
SECRET_KEY
```

攻擊者雖然可能知道：

```text
user_id = 25
role = admin
```

甚至知道網站使用 Flask。

但是他不知道：

```text
SECRET_KEY
```

所以無法產生 Server 能接受的合法簽章。

這就是為什麼 `SECRET_KEY` 必須保密。

---

# 10. SECRET_KEY 不是在保護整個資料庫

這也是今天很重要的一個觀念。

假設 SQLite 有：

```text
SQLite
│
├── users
│   ├── user_id
│   ├── username
│   ├── password_hash
│   └── email
│
└── glucose_records
    ├── user_id
    ├── glucose
    └── measured_at
```

`SECRET_KEY` 並不是把 SQLite 裡面的資料全部加密或簽章。

它主要保護的是 Flask Session 等需要簽章的資料。

例如：

```python
session["user_id"] = 25
session["display_name"] = "Honda"
```

---

# 11. 簽章不等於加密

這是今天最容易搞混的地方。

### 加密 Encryption

目的：

> 不讓別人看懂內容。

概念：

```text
Honda
  ↓
Encryption
  ↓
x8F92k...
```

沒有正確的 Key，就無法還原原始內容。

---

### 簽章 Signing

目的：

> 防止別人偷偷修改內容。

概念：

```text
user_id = 25
      +
SECRET_KEY
      ↓
Signature
```

資料本身不一定是秘密。

真正重要的是：

```text
資料只要被修改
      ↓
簽章就會對不上
```

因此 Flask Session Cookie 的核心概念是：

> **Integrity（完整性 / 防竄改）**

而不是單純：

> **Confidentiality（機密性 / 不讓人看到）**

---

# 12. 完整登入流程

把今天學習的內容全部串起來：

```text
① 使用者登入

Client
   │
   │ username + password
   ▼
Server
   │
   │ 驗證帳號密碼
   ▼
登入成功
   │
   ▼
建立 Session

user_id = 25

   │
   │ SECRET_KEY
   ▼
產生簽章
   │
   ▼
Session Cookie
   │
   ▼
Client 保存
```

下一次 Request：

```text
Client
   │
   │ Cookie
   ▼
Server
   │
   │ SECRET_KEY
   ▼
重新計算簽章
   │
   ▼
比較簽章
   │
   ├── 相同 → ✅ 接受
   │
   └── 不同 → ❌ 不信任
```

---

# 13. Client 到底負責什麼？

Client 不需要知道 `SECRET_KEY`。

也不需要自己產生新的簽章。

Client 基本上只是：

```text
Server 給我 Cookie
        ↓
瀏覽器保存
        ↓
下次 Request
        ↓
原封不動帶回 Server
```

而：

```text
產生簽章
驗證簽章
保存 SECRET_KEY
```

都是 Server 的責任。

可以用一句話記：

> **Server 負責蓋章與驗章，Client 負責保管並帶回。**

---

# 14. 為什麼 SECRET_KEY 不能放 GitHub？

如果攻擊者拿到 `SECRET_KEY`，安全模型就被破壞了。

因為他可能有能力針對簽章資料產生 Server 可以接受的有效簽章。

所以真正的：

```text
SECRET_KEY
```

不應該放在：

```text
GitHub
README
.env.example
前端 JavaScript
HTML
公開截圖
Log
```

正式環境應該把它放在 Server 的安全設定或環境變數中。

---

# 15. 今天最重要的觀念

今天最大的收穫其實不是記住 Flask 的語法，而是理解 Web Security 的一個基本原則：

> **永遠不要因為資料是 Client 傳回來的，就直接相信它。**

Flask 的 `SECRET_KEY` 與 Session 簽章機制，就是讓 Server 可以確認：

```text
「這是不是我之前認證過的資料？」

以及

「資料離開 Server 之後，有沒有被修改？」
```

最後可以把整個概念濃縮成：

```text
登入成功
   ↓
建立 Session
   ↓
SECRET_KEY 簽章
   ↓
Cookie 給 Client
   ↓
Client 下次帶回
   ↓
Server 使用 SECRET_KEY 驗章
   ↓
確認 Session 沒被竄改
   ↓
信任 Session 中的登入狀態
```

## 一句話總結

> **SECRET_KEY 就像 Flask Server 的私人印章：Client 可以拿著 Server 蓋過章的文件回來，但不能自己修改文件後再偽造一個合法的章。**
