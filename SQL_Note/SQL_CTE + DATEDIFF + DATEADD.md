# SQL進階技巧 - CTE + 日期函數完整指南

## 📑 目錄
- [Part 1: CTE 基礎概念](#part-1-cte-基礎概念)
- [Part 2: CTE 進階應用](#part-2-cte-進階應用)
- [Part 3: 日期函數](#part-3-日期函數)
- [Part 4: JOIN 類型比較](#part-4-join-類型比較)
- [Part 5: Window Function 與 CTE](#part-5-window-function-與-cte)
- [Part 6: 常見錯誤與盲點](#part-6-常見錯誤與盲點)
- [Part 7: 學習路線圖](#part-7-學習路線圖)

---

## Part 1: CTE 基礎概念

### 什麼是 CTE?

**CTE (Common Table Expression)** 是一段可以重複使用的暫時資料集。

### 基本語法

```sql
WITH 別名 AS (
    SELECT ...
)
SELECT ...
FROM 別名;
```

### CTE 的優點

✅ 讓 SQL 變清楚  
✅ 方便分步驟拆解邏輯  
✅ 可以多段堆疊（企業最常用）  
✅ 程式碼可讀性高  
✅ 易於維護和除錯

### 單段 CTE 範例

```sql
-- 計算平均訂單金額
WITH AvgAmount AS (
    SELECT AVG(Amount) AS Average
    FROM Orders
)
SELECT 
    O.OrderId,
    O.Amount,
    A.Average,
    O.Amount - A.Average AS Difference
FROM Orders O
CROSS JOIN AvgAmount A;
```

⭐ **使用時機**: 
- 需要先計算某個統計值（平均、最大、最小）
- 該統計值會在主查詢中多次使用

---

## Part 2: CTE 進階應用

### 多段 CTE（企業最常見）

```sql
WITH Step1 AS (
    -- 第一步：篩選資料
    SELECT 
        CustomerId,
        OrderDate,
        Amount
    FROM Orders
    WHERE OrderDate >= DATEADD(MONTH, -3, GETDATE())
),
Step2 AS (
    -- 第二步：聚合計算
    SELECT 
        CustomerId,
        SUM(Amount) AS TotalSpent,
        COUNT(*) AS OrderCount
    FROM Step1
    GROUP BY CustomerId
),
Step3 AS (
    -- 第三步：排名
    SELECT 
        CustomerId,
        TotalSpent,
        OrderCount,
        DENSE_RANK() OVER (ORDER BY TotalSpent DESC) AS Ranking
    FROM Step2
)
-- 最後查詢
SELECT *
FROM Step3
WHERE Ranking <= 10;
```

### 多段 CTE 的思維方式

```
原始資料
   ↓
Step1: 篩選條件（WHERE）
   ↓
Step2: 聚合計算（GROUP BY）
   ↓
Step3: 排名或進階處理（Window Function）
   ↓
最終結果
```

⭐ **企業級實戰技巧**:
1. 每個 CTE 只做一件事
2. 命名要清楚（FilteredOrders, CustomerTotal, RankedCustomers）
3. 複雜邏輯先在紙上畫流程圖
4. 可以逐步測試每個 CTE

### CTE 類型總覽

| CTE 類型 | 用途 | 難度 |
|---------|------|------|
| 單段 CTE | 提前算平均、最大值 | ⭐ 基礎 |
| 雙段 CTE | 先篩選 → 再聚合 | ⭐⭐ 進階 |
| 多段 CTE | 先篩選 → 再聚合 → 再排名 | ⭐⭐⭐ 企業級 |
| CTE + Window Function | 排名、累計、移動平均 | ⭐⭐⭐ 企業級 |
| CTE + CROSS JOIN | 全域資料加入每列 | ⭐⭐⭐ 企業級 |
| Recursive CTE | 階層、遞迴查詢 | ⭐⭐⭐⭐ 高級 |

---

## Part 3: 日期函數

### DATEDIFF - 計算日期差異

#### 基本語法

```sql
DATEDIFF(時間單位, 起始日, 結束日)
```

#### 時間單位選項

- `YEAR` - 年
- `MONTH` - 月
- `DAY` - 日
- `HOUR` - 小時
- `MINUTE` - 分鐘
- `SECOND` - 秒

#### 範例

```sql
-- 計算天數差異
SELECT DATEDIFF(DAY, '2025-01-01', '2025-01-05') AS DaysDiff;
-- 結果：4

-- 計算月份差異
SELECT DATEDIFF(MONTH, '2024-01-01', '2025-01-01') AS MonthsDiff;
-- 結果：12

-- 計算年份差異
SELECT DATEDIFF(YEAR, '2020-01-01', '2025-01-01') AS YearsDiff;
-- 結果：5
```

#### ⚠️ 重要觀念

| 表達式 | 意義 |
|-------|------|
| `DATEDIFF(DAY, w1, w2) = 1` | w2 是 w1 的隔天 |
| `DATEDIFF(DAY, w2, w1) = 1` | w1 是 w2 的隔天 |
| `DATEDIFF(DAY, w1, w2) = -1` | w1 是 w2 的隔天（順序顛倒）|

⭐ **順序很重要**: 起始日與結束日順序顛倒會得到相反的結果

#### 實務應用

```sql
-- 計算訂單到出貨的天數
SELECT 
    OrderId,
    OrderDate,
    ShipDate,
    DATEDIFF(DAY, OrderDate, ShipDate) AS ProcessDays
FROM Orders;

-- 找出超過 7 天未出貨的訂單
SELECT *
FROM Orders
WHERE DATEDIFF(DAY, OrderDate, GETDATE()) > 7
  AND ShipDate IS NULL;
```

---

### DATEADD - 日期加減

#### 基本語法

```sql
DATEADD(時間單位, 數字, 日期)
```

#### 範例

```sql
-- 加一天
SELECT DATEADD(DAY, 1, '2025-01-01') AS NextDay;
-- 結果：2025-01-02

-- 減三個月
SELECT DATEADD(MONTH, -3, GETDATE()) AS ThreeMonthsAgo;

-- 加一年
SELECT DATEADD(YEAR, 1, '2025-01-01') AS NextYear;
-- 結果：2026-01-01

-- 減 7 天
SELECT DATEADD(DAY, -7, GETDATE()) AS LastWeek;
```

#### 實務應用 - 篩選最近 N 個月的資料

```sql
-- 最近三個月的訂單
SELECT *
FROM Orders
WHERE OrderDate >= DATEADD(MONTH, -3, GETDATE());

-- 最近 30 天的訂單
SELECT *
FROM Orders
WHERE OrderDate >= DATEADD(DAY, -30, GETDATE());

-- 最近一年的訂單
SELECT *
FROM Orders
WHERE OrderDate >= DATEADD(YEAR, -1, GETDATE());
```

⭐ **為什麼 `DATEADD(MONTH, -3, GETDATE())` 是最近三個月?**

因為 SQL 的時間函數都是以 `GETDATE()` (今天) 為基準:
- `GETDATE()` = 今天
- `DATEADD(MONTH, -3, GETDATE())` = 今天往前推 3 個月
- 所以 `WHERE OrderDate >= DATEADD(MONTH, -3, GETDATE())` 就是「從三個月前到現在」

#### 組合使用 DATEADD 和 DATEDIFF

```sql
-- 計算每筆訂單的預計到貨日（下單後 3 天）
SELECT 
    OrderId,
    OrderDate,
    DATEADD(DAY, 3, OrderDate) AS EstimatedDelivery,
    DATEDIFF(DAY, OrderDate, GETDATE()) AS DaysSinceOrder
FROM Orders;
```

---

### 進階日期函數（待學習）

#### EOMONTH - 取得月底日期

```sql
-- 取得當月最後一天
SELECT EOMONTH(GETDATE()) AS EndOfMonth;

-- 取得下個月最後一天
SELECT EOMONTH(GETDATE(), 1) AS EndOfNextMonth;
```

#### DATEFROMPARTS - 組合日期

```sql
-- 從年月日組合日期
SELECT DATEFROMPARTS(2025, 12, 25) AS ChristmasDay;
-- 結果：2025-12-25
```

#### DATEPART - 取得日期部分

```sql
-- 取得年份
SELECT DATEPART(YEAR, GETDATE()) AS CurrentYear;

-- 取得月份
SELECT DATEPART(MONTH, '2025-12-25') AS Month;
-- 結果：12

-- 取得星期幾（1=週日, 7=週六）
SELECT DATEPART(WEEKDAY, GETDATE()) AS DayOfWeek;
```

---

## Part 4: JOIN 類型比較

### JOIN vs CROSS JOIN

| 類型 | 作用 | 配對方式 | 用途 |
|-----|------|---------|------|
| **INNER JOIN** | 兩表依條件配對 | 需要 ON 條件 | 99% 的情況 |
| **LEFT JOIN** | 保留左表所有資料 | 需要 ON 條件 | 即使沒配對也顯示 |
| **CROSS JOIN** | 無條件組合（笛卡兒積）| 不需要 ON | 把一筆資料加到所有筆 |

### INNER JOIN 範例

```sql
-- 客戶與訂單配對
SELECT 
    C.CustomerName,
    O.OrderId,
    O.Amount
FROM Customers C
INNER JOIN Orders O 
    ON C.CustomerId = O.CustomerId;
```

### LEFT JOIN 範例

```sql
-- 顯示所有客戶,包括沒有訂單的
SELECT 
    C.CustomerName,
    O.OrderId,
    O.Amount
FROM Customers C
LEFT JOIN Orders O 
    ON C.CustomerId = O.CustomerId;
```

### CROSS JOIN 範例

```sql
-- 計算每筆訂單佔總額的比例
WITH TotalAmount AS (
    SELECT SUM(Amount) AS Total
    FROM Orders
)
SELECT 
    O.OrderId,
    O.Amount,
    T.Total,
    O.Amount * 100.0 / T.Total AS Percentage
FROM Orders O
CROSS JOIN TotalAmount T;
```

⭐ **CROSS JOIN 使用時機**:
- 需要把一個統計值（平均、總和、稅率）加到每一筆資料
- 該統計值是「全域的」,不需要配對條件
- 統計值通常只有一筆資料

### ⚠️ 常見錯誤

```sql
-- ❌ 錯誤：使用 JOIN 會影響資料筆數
SELECT O.*, A.Average
FROM Orders O
JOIN AvgAmount A;  -- 這會報錯或產生錯誤結果

-- ✅ 正確：使用 CROSS JOIN
SELECT O.*, A.Average
FROM Orders O
CROSS JOIN AvgAmount A;
```

---

## Part 5: Window Function 與 CTE

### Window Function 基礎

Window Function 可以在不改變資料筆數的情況下進行計算。

#### 三大排名函數比較

| 函數 | 相同值處理 | 排名連續性 | 使用場景 |
|------|-----------|-----------|---------|
| **ROW_NUMBER()** | 給不同排名 | 連續 | 需要唯一排名 |
| **RANK()** | 給相同排名 | 不連續 | 允許並列，跳號 |
| **DENSE_RANK()** | 給相同排名 | 連續 | 允許並列，不跳號 |

#### 範例比較

假設有以下分數：95, 95, 90, 85

| 分數 | ROW_NUMBER() | RANK() | DENSE_RANK() |
|-----|--------------|--------|--------------|
| 95  | 1            | 1      | 1            |
| 95  | 2            | 1      | 1            |
| 90  | 3            | 3      | 2            |
| 85  | 4            | 4      | 3            |

### DENSE_RANK() 詳解

```sql
-- 基本用法：依消費金額排名
SELECT 
    CustomerId,
    TotalSpent,
    DENSE_RANK() OVER (ORDER BY TotalSpent DESC) AS Ranking
FROM CustomerSummary;
```

#### 加入 PARTITION BY - 分組排名

```sql
-- 每個部門內的薪資排名
SELECT 
    EmployeeName,
    Department,
    Salary,
    DENSE_RANK() OVER (
        PARTITION BY Department
        ORDER BY Salary DESC
    ) AS DeptRank
FROM Employees;
```

⭐ **PARTITION BY 的作用**:
- 類似 GROUP BY，但不會減少資料筆數
- 在每個分組內獨立排名
- 排名會在每個新分組重新開始為 1

### CTE + Window Function 實戰範例

#### 範例1: 找出每個客戶的排名

```sql
WITH CustomerTotal AS (
    SELECT 
        CustomerId,
        SUM(Amount) AS TotalSpent
    FROM Orders
    GROUP BY CustomerId
)
SELECT 
    CustomerId,
    TotalSpent,
    DENSE_RANK() OVER (ORDER BY TotalSpent DESC) AS Ranking
FROM CustomerTotal;
```

#### 範例2: 找出每個部門薪資前3名

```sql
WITH RankedEmployees AS (
    SELECT 
        EmployeeName,
        Department,
        Salary,
        DENSE_RANK() OVER (
            PARTITION BY Department
            ORDER BY Salary DESC
        ) AS DeptRank
    FROM Employees
)
SELECT *
FROM RankedEmployees
WHERE DeptRank <= 3;
```

#### 範例3: 企業級報表 - 客戶消費分析

```sql
WITH RecentOrders AS (
    -- Step1: 篩選最近三個月
    SELECT 
        CustomerId,
        Amount,
        OrderDate
    FROM Orders
    WHERE OrderDate >= DATEADD(MONTH, -3, GETDATE())
),
CustomerSummary AS (
    -- Step2: 計算每位客戶總消費
    SELECT 
        CustomerId,
        SUM(Amount) AS TotalSpent,
        COUNT(*) AS OrderCount
    FROM RecentOrders
    GROUP BY CustomerId
),
OverallAverage AS (
    -- Step3: 計算平均消費
    SELECT AVG(TotalSpent) AS AvgSpent
    FROM CustomerSummary
)
-- Step4: 最終報表
SELECT 
    CS.CustomerId,
    CS.TotalSpent,
    CS.OrderCount,
    OA.AvgSpent,
    CS.TotalSpent - OA.AvgSpent AS DiffFromAvg,
    DENSE_RANK() OVER (ORDER BY CS.TotalSpent DESC) AS Ranking
FROM CustomerSummary CS
CROSS JOIN OverallAverage OA
ORDER BY Ranking;
```

---

## Part 6: 常見錯誤與盲點

### ❌ 錯誤 1: CTE + JOIN 使用錯誤

**錯誤寫法**:
```sql
WITH AvgAmount AS (
    SELECT AVG(Amount) AS Average
    FROM Orders
)
SELECT O.*, A.Average
FROM Orders O
JOIN AvgAmount A;  -- ❌ 錯誤！缺少 ON 條件
```

**正確寫法**:
```sql
WITH AvgAmount AS (
    SELECT AVG(Amount) AS Average
    FROM Orders
)
SELECT O.*, A.Average
FROM Orders O
CROSS JOIN AvgAmount A;  -- ✅ 正確！
```

⭐ **記憶口訣**:
- JOIN = 需要配對（需要 ON）
- CROSS JOIN = 不需要配對（全域資料）

---

### ❌ 錯誤 2: 忘記 PARTITION BY 的作用

**問題**: 想要每個部門內排名，但寫成全域排名

**錯誤寫法**:
```sql
SELECT 
    EmployeeName,
    Department,
    Salary,
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS Rank
    -- ❌ 這是全公司排名，不是部門內排名
FROM Employees;
```

**正確寫法**:
```sql
SELECT 
    EmployeeName,
    Department,
    Salary,
    DENSE_RANK() OVER (
        PARTITION BY Department  -- ✅ 加上這行
        ORDER BY Salary DESC
    ) AS DeptRank
FROM Employees;
```

---

### ❌ 錯誤 3: 日期邏輯方向搞反

**問題**: 不確定 `DATEADD(MONTH, -3, GETDATE())` 的含義

**理解方式**:
```sql
GETDATE()                    = 今天 (2025-12-12)
DATEADD(MONTH, -3, GETDATE()) = 三個月前 (2025-09-12)

-- 所以這個條件的意思是：
WHERE OrderDate >= DATEADD(MONTH, -3, GETDATE())
-- 「訂單日期 >= 三個月前」= 「最近三個月的訂單」
```

⭐ **記憶方式**:
- 正數 = 未來
- 負數 = 過去
- `>=` 過去某個日期 = 「從那天到現在」

---

### ❌ 錯誤 4: CTE 最後必須從 CTE 查詢

**錯誤寫法**:
```sql
WITH FilteredData AS (
    SELECT * FROM Orders
    WHERE Amount > 1000
)
SELECT *
FROM Orders;  -- ❌ 錯誤！沒有使用 CTE
```

**正確寫法**:
```sql
WITH FilteredData AS (
    SELECT * FROM Orders
    WHERE Amount > 1000
)
SELECT *
FROM FilteredData;  -- ✅ 正確！從 CTE 查詢
```

---

### ❌ 錯誤 5: DATEDIFF 參數順序搞混

```sql
-- 這兩個結果相反！
DATEDIFF(DAY, '2025-01-01', '2025-01-05')  -- 結果: 4
DATEDIFF(DAY, '2025-01-05', '2025-01-01')  -- 結果: -4
```

⭐ **記憶方式**: DATEDIFF(單位, **起點**, **終點**)
- 終點 > 起點 → 正數
- 起點 > 終點 → 負數

---

## Part 7: 學習路線圖

### ✅ 已掌握技能（目前位置）

| 內容 | 狀態 | 難度 |
|------|------|------|
| CTE 基礎語法 | ✅ 完成 | ⭐ |
| 單段 CTE | ✅ 完成 | ⭐ |
| CTE + GROUP BY | ✅ 完成 | ⭐⭐ |
| 多段 CTE（2-3段）| ✅ 完成 | ⭐⭐⭐ |
| CTE + CROSS JOIN | ✅ 完成 | ⭐⭐⭐ |
| CTE + DENSE_RANK | ✅ 完成 | ⭐⭐⭐ |
| 企業級報表 CTE | ✅ 完成 | ⭐⭐⭐ |
| DATEADD 基礎 | ✅ 完成 | ⭐⭐ |
| DATEDIFF 基礎 | ✅ 完成 | ⭐⭐ |

**🎉 你已經比多數新人強！你現在的程度約在「中高階」水平。**

---

### 🔥 待學習技能（接下來要學什麼）

#### 🔸 階段 3: Window Function 進階（中級）

**預計學習時間**: 2-3 週

| 主題 | 內容 |
|------|------|
| ROW_NUMBER | 唯一排名，分頁功能 |
| RANK vs DENSE_RANK | 三種排名函數的差異 |
| LAG / LEAD | 取前一筆/後一筆資料 |
| Running Total | 累計總和 |
| Moving Average | 移動平均 |

**典型題目**:
```sql
-- 每位客戶最近一次訂單
WITH RankedOrders AS (
    SELECT 
        CustomerId,
        OrderId,
        OrderDate,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerId
            ORDER BY OrderDate DESC
        ) AS RowNum
    FROM Orders
)
SELECT *
FROM RankedOrders
WHERE RowNum = 1;
```

---

#### 🔸 階段 4: 多表 JOIN + CTE（企業實戰）

**預計學習時間**: 3-4 週

| 主題 | 內容 |
|------|------|
| 3+ 表 JOIN | 訂單 + 明細 + 商品 + 客戶 |
| CTE + 多重 JOIN | 複雜報表 |
| 客戶分群 (RFM) | 最近購買、頻率、金額 |
| 月報表生成 | 每月統計 + 同比 |

**典型題目**:
```sql
-- 客戶 RFM 分析
WITH CustomerRFM AS (
    SELECT 
        CustomerId,
        DATEDIFF(DAY, MAX(OrderDate), GETDATE()) AS Recency,
        COUNT(*) AS Frequency,
        SUM(Amount) AS Monetary
    FROM Orders
    WHERE OrderDate >= DATEADD(YEAR, -1, GETDATE())
    GROUP BY CustomerId
)
SELECT 
    CustomerId,
    Recency,
    Frequency,
    Monetary,
    CASE 
        WHEN Recency <= 30 AND Frequency >= 5 THEN 'VIP'
        WHEN Recency <= 90 AND Frequency >= 3 THEN 'Active'
        ELSE 'At Risk'
    END AS CustomerSegment
FROM CustomerRFM;
```

---

#### 🔸 階段 5: 進階日期處理

**預計學習時間**: 1-2 週

| 主題 | 內容 |
|------|------|
| EOMONTH | 月底計算 |
| DATEFROMPARTS | 組合日期 |
| DATEPART | 提取年月日週 |
| 週次計算 | 第幾週統計 |
| 連續日期 | 找出連續登入天數 |

**典型題目**:
```sql
-- 每月最後一天的訂單統計
SELECT 
    EOMONTH(OrderDate) AS MonthEnd,
    COUNT(*) AS OrderCount,
    SUM(Amount) AS TotalAmount
FROM Orders
GROUP BY EOMONTH(OrderDate)
ORDER BY MonthEnd;
```

---

#### 🔸 階段 6: Recursive CTE（高級）

**預計學習時間**: 2-3 週

**⚠️ 這是最難的部分！**

| 主題 | 內容 |
|------|------|
| 組織階層 | 主管 → 部門 → 員工 |
| 分類樹 | 大分類 → 中分類 → 小分類 |
| 日期展開 | 起迄日期展開成每日 |
| 連續數列 | 找連續登入、連續缺席 |

**典型題目**:
```sql
-- 展開員工階層（找出所有下屬）
WITH EmployeeHierarchy AS (
    -- 起始點：某個主管
    SELECT 
        EmployeeId,
        EmployeeName,
        ManagerId,
        1 AS Level
    FROM Employees
    WHERE EmployeeId = 1  -- 從老闆開始
    
    UNION ALL
    
    -- 遞迴：找下一層
    SELECT 
        E.EmployeeId,
        E.EmployeeName,
        E.ManagerId,
        EH.Level + 1
    FROM Employees E
    INNER JOIN EmployeeHierarchy EH
        ON E.ManagerId = EH.EmployeeId
)
SELECT *
FROM EmployeeHierarchy
ORDER BY Level, EmployeeId;
```

---

### 📊 完整學習進度圖

```
✅ 已完成（70%）
├── CTE 基礎
├── CTE 多段拆解
├── CTE + CROSS JOIN
├── DENSE_RANK 排名
├── DATEADD / DATEDIFF
└── 企業級報表 CTE

🔥 進行中
└── Window Function 進階

⏳ 待學習（30%）
├── ROW_NUMBER / LAG / LEAD
├── 多表複雜 JOIN
├── 進階日期函數
└── Recursive CTE（最難）
```

---

### 🎯 學習建議

#### 短期目標（1個月內）
1. ✅ 熟練 ROW_NUMBER
2. ✅ 理解 LAG / LEAD
3. ✅ 能寫出「每個客戶最近一次訂單」
4. ✅ 掌握 Running Total

#### 中期目標（2-3個月內）
1. ✅ 能處理 3+ 表的複雜 JOIN
2. ✅ 獨立寫出月報表
3. ✅ 理解並應用 RFM 模型
4. ✅ 掌握進階日期函數

#### 長期目標（6個月內）
1. ✅ 完全理解 Recursive CTE
2. ✅ 能處理階層資料
3. ✅ 能優化複雜查詢效能
4. ✅ 達到企業資深工程師水平

---

### 💡 練習資源推薦

1. **LeetCode SQL**
   - 184. Department Highest Salary
   - 185. Department Top Three Salaries
   - 601. Human Traffic of Stadium（連續問題）

2. **HackerRank SQL**
   - Advanced Join
   - Window Functions

3. **實際專案練習**
   - 建立自己的電商資料庫
   - 定期生成銷售報表
   - 模擬真實業務場景

---

## 📝 總結

### 你的優勢
✅ CTE 基礎扎實  
✅ 理解企業級多段 CTE  
✅ 掌握 CROSS JOIN 使用時機  
✅ 能結合日期函數做時間篩選  
✅ 具備基本 Window Function 能力

### 需要加強的部分
⚠️ ROW_NUMBER / RANK 還不熟  
⚠️ LAG / LEAD 未接觸  
⚠️ 複雜多表 JOIN 經驗少  
⚠️ 進階日期函數待學習  
⚠️ Recursive CTE 完全未學

### 下一步行動
1. 完成 Window Function 進階章節
2. 練習 LeetCode SQL 184, 185
3. 寫出「每個客戶最近一次訂單」
4. 學習 LAG / LEAD 函數
5. 挑戰一個複雜的企業級報表

**加油！你已經走了 70% 的路程，