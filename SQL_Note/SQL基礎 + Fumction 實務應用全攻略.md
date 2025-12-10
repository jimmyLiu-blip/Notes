# 📘 SQL Functions 完整筆記（含常錯點 + 弱點補強）

## 1️⃣ Function 與 Stored Procedure 的差異

### ✔ Function（函數）
- 一定要**回傳值**
- 不能做資料修改（不能 INSERT / UPDATE / DELETE）
- 可以用在 SELECT / WHERE / JOIN
- 適合：計算、格式化、資料驗證、封裝邏輯

### ✔ Stored Procedure（SP）
- 可以有回傳值、也可以沒有
- 可以修改資料
- 不能放在 SELECT 裡
- 適合：商業流程、整段流程邏輯、交易（TRAN）

---

## 2️⃣ Function 的種類

| 類型 | 回傳 | 用途 | 企業使用度 |
|------|------|------|------------|
| Scalar Function | 單一值（如 INT、NVARCHAR） | 計算或格式化 | ⭐⭐⭐⭐⭐（最多） |
| ITVF（Inline Table-Valued Function） | 一張表（SELECT） | 包裝查詢 | ⭐⭐⭐⭐⭐（非常重要） |
| MSTVF（Multi-Statement Table-Valued Function） | 一張表（可多步驟運算） | 複雜查詢 | ⭐⭐⭐ |

---

## 3️⃣ Scalar Function 基本語法

```sql
CREATE FUNCTION dbo.函數名稱
(
    @參數1 資料型別,
    @參數2 資料型別
)
RETURNS 回傳型別
AS
BEGIN
    RETURN ...
END;
GO
```

### 重點：
- 一定要寫 `RETURNS` 型別
- 一定要 `RETURN`
- 可用在 SELECT：
```sql
SELECT dbo.fnCalcDiscount(100, 0.1);
```

---

## 4️⃣ 常見工具函數（Scalar Function）

### ✔ Email 標準化
- 轉小寫
- TRIM 清空白
- NULL 就回傳 NULL

```sql
RETURN LOWER(TRIM(@Email));
```

### ✔ Safe 字串處理（避免 NULL / 空字串）
```sql
IF @Input IS NULL OR TRIM(@Input) = ''
    RETURN @DefaultValue;
RETURN TRIM(@Input);
```

### ✔ 手機格式化（重要）

#### 🌟 必須理解的字串函數：

**1. REPLACE**
- 移除特定字元：
```sql
SET @Clean = REPLACE(@Clean, '-', '');
```

- 多個清洗動作：
```sql
SET @Clean = REPLACE(REPLACE(REPLACE(@Phone, '-', ''), ' ', ''), ')', '');
```

**2. TRANSLATE（SQL Server 2017+）**
- 一次取代多個字元：
```sql
SET @Clean = TRANSLATE(@Phone, '()- ', '    ');
```
- `'()- '` → 這些字元會被轉成 `' '` 也就是空白
- 再 `REPLACE(@Clean, ' ', '')` 就能把全部清掉

**3. 清洗後的驗證**
- 使用字串正規化：
```sql
@Clean LIKE '%[^0-9]%'
```
- `[^0-9]` = 非數字
- 有任何非數字 → 無效 → 回傳 NULL

### ✔ REPLICATE
```sql
REPLICATE('*', 次數)
```

例：
- `REPLICATE('*', 3)` → `***`
- `REPLICATE('*', LEN(@Name)-2)`

**常用於：**
- 隱碼（姓名遮罩）
- 自動產生重複字串
- 格式模板填空

### ✔ 姓名遮罩
```sql
LEFT(@Name, 1) 
+ REPLICATE('*', LEN(@Name) - 2) 
+ RIGHT(@Name, 1);
```

---

## 5️⃣ 日期函數（需要強化的重點）

### ❗(1) DATEDIFF（弱點：第一次看到）

**格式：**
```sql
DATEDIFF(單位, start, end)
```

**例如：**
```sql
DATEDIFF(YEAR, @BirthDate, GETDATE())
```

**⚠ 但它只比較年份差，不等於「實歲」！**

例如：
- `2007-12-31` → `2025-01-01`
- `DATEDIFF(YEAR)` = 18（但事實上還沒滿 18）

👉 **企業作法要加補正：**
```sql
- CASE WHEN 還沒過生日 THEN 1 ELSE 0 END
```

### ❗(2) DATEPART（需注意 DATEFIRST）
```sql
DATEPART(WEEKDAY, @Date)
```

- 回傳值會受到 `SET DATEFIRST` 影響
- 不同 SQL Server 會得到不同星期數字

👉 因此**不推薦直接用它來判斷星期**

### ⭐(3) FORMAT
```sql
FORMAT(@Date, N'dddd', 'zh-TW')
```

- ✔ `N'dddd'` 是「Unicode 格式字串」
  - `dddd` → 顯示完整星期（星期一、星期二…）
- ✔ `'zh-TW'` 是「文化語系設定」，不是時區
  - `zh-TW` → 台灣正體中文格式
  - `zh-CN` → 中國簡體中文
  - `en-US` → 美國英文
  - `ja-JP` → 日文顯示方式

---

## 6️⃣ ITVF（Inline Table-Valued Function）完整說明

### 📌 ITVF 是什麼？

ITVF 是**回傳一張資料表的函數**，語法與視圖（VIEW）類似，但可以：

✔ 接收參數  
✔ 像資料表一樣被 JOIN / APPLY  
✔ 只能包含**一個 SELECT**（不能多段處理 → 多段要用 MSTVF）

**特點：**
- 非常快（效能最好的一種 Function）
- 用於封裝複雜查詢邏輯
- 企業最常用的 Function 形式

### 📌 ITVF 基本語法

```sql
CREATE FUNCTION dbo.函數名稱
(
    @參數 資料型別
)
RETURNS TABLE
AS
RETURN
(
    SELECT ...
    FROM ...
    WHERE ...
);
```

**注意：**
- 一定要 `RETURN ( SELECT … )`
- 不能 `DECLARE` 變數
- 不能 `UPDATE` / `INSERT` / `DELETE`
- 像資料表一樣被 SELECT

### 📌 使用 ITVF 的兩種方式

#### ✔ (1) 像資料表一樣 SELECT
```sql
SELECT *
FROM dbo.fnGetCustomerOrders(5);
```

#### ✔ (2) 與其他表 JOIN / APPLY
- 若 ITVF 每個輸入只有一筆 → JOIN 可以用（但很少這樣設計）
- 若 ITVF 每個輸入回傳多筆 → 必須使用 **APPLY**

### 📌 CROSS APPLY 與 OUTER APPLY（非常重要）

#### ✔ CROSS APPLY（像 INNER JOIN）
- 左側資料每列都會呼叫函數
- 若函數回傳空集合 → 該列不會出現

```sql
FROM Customers C
CROSS APPLY fnGetCustomerOrders(C.CustomerId) O
```

#### ✔ OUTER APPLY（像 LEFT JOIN）
- 左側資料一定出現
- 若函數無資料 → O 的欄位為 NULL

```sql
FROM Customers C
OUTER APPLY fnGetCustomerOrders(C.CustomerId) O
```

### 📌 ITVF 實戰範例

#### ✔ (1) 查詢客戶的所有訂單
```sql
CREATE FUNCTION dbo.fnGetCustomerOrders(@CustomerId INT)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        O.OrderId,
        O.OrderDate,
        O.Amount
    FROM Orders O
    WHERE O.CustomerId = @CustomerId
);
```

#### ✔ (2) 查詢每位客戶本月訂單統計
```sql
CREATE FUNCTION dbo.fnGetMonthlyStats
(
    @CustomerId INT,
    @Year INT,
    @Month INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        COUNT(O.OrderId) AS OrderCount, 
        SUM(O.Amount) AS TotalAmount,
        AVG(O.Amount) AS AvgAmount
    FROM Orders O
    WHERE O.CustomerId = @CustomerId
    AND YEAR(O.OrderDate) = @Year
    AND MONTH(O.OrderDate) = @Month
);
```

#### ✔ (3) 查詢最新訂單（1 筆）
```sql
CREATE FUNCTION dbo.fnGetLatestOrder(@CustomerId INT)
RETURNS TABLE
AS
RETURN
(
    SELECT TOP(1)
        O.OrderId,
        O.OrderDate,
        O.Amount
    FROM Orders O
    WHERE O.CustomerId = @CustomerId
    ORDER BY O.OrderDate DESC
);
```

#### ✔ (4) 查詢最高金額訂單（1 筆）
```sql
CREATE FUNCTION dbo.fnGetTopOrder(@CustomerId INT)
RETURNS TABLE
AS
RETURN
(
    SELECT TOP(1)
        O.OrderId,
        O.OrderDate,
        O.Amount
    FROM Orders O
    WHERE O.CustomerId = @CustomerId
    ORDER BY O.Amount DESC
);
```

#### ✔ (5) 查詢最近 3 筆訂單 + 累積金額（RunningTotal）
```sql
CREATE FUNCTION dbo.fnGetTop3OrderWithRunningTotal
(
    @CustomerId INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT TOP(3)
        O.OrderId,
        O.OrderDate,
        O.Amount,
        SUM(O.Amount) OVER(ORDER BY O.OrderDate DESC) AS RunningTotal
    FROM Orders O
    WHERE O.CustomerId = @CustomerId
    ORDER BY O.OrderDate DESC
);
```

### 📌 ITVF 常見錯誤

| 錯誤 | 原因 |
|------|------|
| Invalid column name | 外層 SELECT 欄位名稱與函數不同 |
| Must declare scalar variable | 在 ITVF 內用 DECLARE |
| Incorrect syntax near RETURN | 沒有括號 `( SELECT )` |
| Cannot perform this operation on a function | 嘗試 UPDATE/INSERT/DELETE |

### 📌 ITVF vs MSTVF

| 特性 | ITVF | MSTVF |
|------|------|-------|
| 效能 | ⭐ 最快 | ⭐⭐ 次快 |
| 可否多步驟 | ❌ 不行 | ✔ 可以 |
| 可否宣告變數 | ❌ 不行 | ✔ 可以 |
| 可否多次 INSERT | ❌ 不行 | ✔ 可以 |
| 可否 UPDATE 中間結果 | ❌ 不行 | ✔ 可以 |
| 使用情境 | 查詢封裝 | 資料加工、摘要、統整 |

---

## 7️⃣ Function 常見錯誤（必須學會）

### ❌ 1. 用 `= NULL` 判斷
**必須是：**
```sql
@Value IS NULL
```

### ❌ 2. IF 不加 BEGIN…END（初學者常犯）
```sql
IF 條件
    SET A=1
    SET B=2
```
其實只有第一行屬於 IF，其他會**永遠執行**。

### ❌ 3. LENGTH 判斷錯誤
已經習慣用 `LEN()`。✔

### ❌ 4. 清洗字串時，未使用變數接續改寫

**錯誤寫法：**
```sql
SET @Clean = REPLACE(@Phone, '-', '');
SET @Clean = REPLACE(@Phone, ' ', '');  -- 錯，應該用 @Clean，而不是 @Phone
```

**正確寫法（連續作用在同一變數）：**
```sql
SET @Clean = REPLACE(@Clean, '-', '');
SET @Clean = REPLACE(@Clean, ' ', '');
```

### ❌ 5. Table Function（ITVF）內不能使用變數UPDATE
（進入 MSTVF 後會說明常見陷阱）

---

## 8️⃣ 你的 Function 學習狀態總結

### ✅ 已完全掌握：
- ✔ Scalar Function 架構與語法
- ✔ 字串處理函數（REPLACE、REPLICATE、LEFT、RIGHT）
- ✔ ITVF 架構完整理解
- ✔ CROSS APPLY / OUTER APPLY 絕對熟練
- ✔ Window Function（RunningTotal）實作能力
- ✔ 封裝查詢邏輯能力
- ✔ Debug SQL 的能力
- ✔ 正確命名與欄位習慣

### 🔶 需要補強：
- ⚠ 日期函數深入應用（DATEDIFF 實歲計算）
- ⚠ MSTVF（下一階段）
- ⚠ TRANSLATE 函數實戰練習

### 📊 你的 SQL 實力評估：
**已超越初級工程師水平，正在邁向中級**
- Scalar Function: ⭐⭐⭐⭐⭐
- ITVF: ⭐⭐⭐⭐⭐
- Window Function: ⭐⭐⭐⭐
- APPLY 運用: ⭐⭐⭐⭐⭐

---

# SQL 日期函數筆記：DATEDIFF 與 DATEADD

## 1. DATEDIFF 的基本語法

```sql
DATEDIFF( datepart, startdate, enddate )
回傳值 = enddate − startdate，以 datepart 指定的單位計算。

常見用法：

sql
複製程式碼
SELECT DATEDIFF(DAY, '2025-01-01', '2025-01-02');  -- 1
SELECT DATEDIFF(DAY, '2025-01-02', '2025-01-01');  -- -1
2. DATEDIFF(DAY, w2.recordDate, w1.recordDate) = 1 與反過來的差異
2.1 寫法一
sql
複製程式碼
DATEDIFF(DAY, w2.recordDate, w1.recordDate) = 1
代表：

w1.recordDate 比 w2.recordDate 晚 1 天

換句話說：w1 = 今天，w2 = 昨天

舉例：

sql
複製程式碼
w2.recordDate = '2025-01-01'
w1.recordDate = '2025-01-02'

DATEDIFF(DAY, w2.recordDate, w1.recordDate) = 1  -- 成立
2.2 寫法二（反過來）
sql
複製程式碼
DATEDIFF(DAY, w1.recordDate, w2.recordDate) = 1
代表：

w2.recordDate 比 w1.recordDate 晚 1 天

換句話說：w1 = 昨天，w2 = 今天

舉例：

sql
複製程式碼
w1.recordDate = '2025-01-01'
w2.recordDate = '2025-01-02'

DATEDIFF(DAY, w1.recordDate, w2.recordDate) = 1  -- 成立
2.3 一句話總結
DATEDIFF(DAY, A, B) = 1 → B 比 A 晚一天

在 Rising Temperature 題目中，如果我們約定：

w1 = 今天（current row）

w2 = 昨天（previous row）

則會寫成：

sql
複製程式碼
DATEDIFF(DAY, w2.recordDate, w1.recordDate) = 1
3. DATEADD 的基本語法
sql
複製程式碼
DATEADD( datepart, number, date )
作用：在某個日期上「加上或減去」一段時間。

範例：

sql
SELECT DATEADD(DAY, 1, '2025-01-01');   -- 2025-01-02
SELECT DATEADD(DAY, -1, '2025-01-02');  -- 2025-01-01
SELECT DATEADD(MONTH, 1, '2025-01-15'); -- 2025-02-15
4. w1.recordDate = DATEADD(DAY, 1, w2.recordDate) 是什麼意思？
sql
w1.recordDate = DATEADD(DAY, 1, w2.recordDate)
代表：

w1 的日期 = w2 的日期 + 1 天

換句話說：w1 比 w2 晚一天

這個語意與下列敘述等價：

sql
DATEDIFF(DAY, w2.recordDate, w1.recordDate) = 1
5. DATEDIFF 寫法與 DATEADD 寫法的對照
DATEDIFF 寫法（比較差距）
sql
DATEDIFF(DAY, w2.recordDate, w1.recordDate) = 1
DATEADD 寫法（直接指定關係）
sql
w1.recordDate = DATEADD(DAY, 1, w2.recordDate)
兩種寫法都表示：

w1 = w2 + 1 天

事實上，很多人覺得 DATEADD 寫法比較直觀，因為：

「今天 = 昨天 + 1 天」

6. 在 Rising Temperature 題目的實戰應用
題目需求：找出「今天比昨天溫度高」的紀錄。

標準 Self Join 寫法：

sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2
  ON w1.recordDate = DATEADD(DAY, 1, w2.recordDate)
WHERE w1.temperature > w2.temperature;
也可以改寫成：

sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2
  ON DATEDIFF(DAY, w2.recordDate, w1.recordDate) = 1
WHERE w1.temperature > w2.temperature;
兩種寫法的 JOIN 條件邏輯是等價的，只是表達方式不同。

