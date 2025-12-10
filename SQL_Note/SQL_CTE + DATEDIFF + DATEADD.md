# 📘 SQL 筆記總整理（CTE + DATEDIFF + DATEADD + 盲點修正）

## 目錄

1. [CTE 基礎概念](#cte-basic)
2. [CTE 多段拆解（企業常用）](#cte-multi)
3. [CTE 常見錯誤與盲點](#cte-wrong)
4. [日期函數 DATEDIFF](#datediff)
5. [日期函數 DATEADD](#dateadd)
6. [CROSS JOIN 與 JOIN 差異](#crossjoin)
7. [今日未熟 / 需補強項目](#weakness)
8. [CTE 學習路線：目前位置、剩下多少內容？](#progress)

---

## 1️⃣ CTE 基礎概念 {#cte-basic}

**CTE（Common Table Expression）** 是一段可以重複使用的暫時資料集。

### 語法

```sql
WITH 別名 AS (
    SELECT ...
)
SELECT ...
FROM 別名;
```

### CTE 的目的

- 讓 SQL 變清楚
- 方便分步驟拆解邏輯
- 可以多段堆疊（企業最常用）

---

## 2️⃣ CTE 多段拆解（企業最常見） {#cte-multi}

```sql
WITH Step1 AS (
    SELECT ...
),
Step2 AS (
    SELECT ...
    FROM Step1
),
Step3 AS (
    SELECT ...
    FROM Step2
)
SELECT *
FROM Step3;
```

### ✔ 今日你成功練習的 CTE 類型

| CTE 類型 | 用途 | 你是否掌握 |
|---------|------|-----------|
| 單段 CTE | 提前算平均、最大值 | ✔ |
| 多段 CTE | 先篩選 → 再聚合 → 再排名 | ✔（企業級）|
| 企業報表級 CTE | Window Function + Ranking | ✔ |
| 多 CTE + CROSS JOIN | 全域資料加入每列 | ✔ |
| 過濾最近 X 月 | 用 DATEADD | ✔ |

**你現在已經進到 CTE 中高階。**

---

## 3️⃣ CTE 常見錯誤與盲點（今天你遇到的） {#cte-wrong}

### ❌ 盲點 1：CTE + CROSS JOIN 使用錯誤

**你曾把：**
```sql
CROSS JOIN CustomerTotal
```

**寫成：**
```sql
JOIN CustomerTotal
```

👉 **JOIN** 會影響資料筆數  
👉 **CROSS JOIN** 是把一筆的全域資料加入每一列（如稅率、平均值）

---

### ❌ 盲點 2：排名時忘記 DENSE_RANK 分組方式

**正確使用：**
```sql
DENSE_RANK() OVER (ORDER BY TotalSpent DESC)
```

**如果要依部門、月份排名，要：**
```sql
DENSE_RANK() OVER (
    PARTITION BY 部門
    ORDER BY Salary DESC
)
```

---

### ❌ 盲點 3：篩選最近三個月邏輯不熟

**你不知道：**
```sql
WHERE OrderDate >= DATEADD(MONTH, -3, GETDATE())
```

**含義是：** 以今天為基準往前推三個月。

---

### ❌ 盲點 4：CTE 最後 SELECT 必須從 CTE 拿資料

**例如寫成：**
```sql
FROM Orders o
```

**而不是：**
```sql
FROM Step3
```

---

## 4️⃣ DATEDIFF（日期差異） {#datediff}

### 格式

```sql
DATEDIFF(時間單位, 起始日, 結束日)
```

### 範例

```sql
DATEDIFF(DAY, '2025-01-01', '2025-01-05')
-- 結果：4
```

### 今日你常問的問題

| 表達式 | 意義 |
|-------|------|
| `DATEDIFF(DAY, w1, w2) = 1` | w2 是 w1 的隔天 |
| `DATEDIFF(DAY, w2, w1) = 1` | w1 是 w2 的隔天 |
| 順序顛倒會影響結果 | ✔ |

---

## 5️⃣ DATEADD（加減時間） {#dateadd}

### 格式

```sql
DATEADD(時間單位, 數字, 日期)
```

### 例子

```sql
DATEADD(DAY, 1, '2025-01-01') → 2025-01-02
DATEADD(MONTH, -3, GETDATE()) → 三個月前
```

### 今日你搞不懂的地方

**為什麼 `DATEADD(MONTH, -3, GETDATE())` 是最近三個月？**

因為 SQL 的時間函數都是以 `GETDATE()` 為基準。

---

## 6️⃣ JOIN vs CROSS JOIN 差異 {#crossjoin}

| 類型 | 作用 | 用途 |
|-----|------|------|
| JOIN | 兩表依條件配對 | 99% 情況 |
| CROSS JOIN | 無條件組合（笛卡兒積）| 把一筆資料加到所有筆（例如稅率、平均值）|

### 今日你理解正確

- ✔ 用 CROSS JOIN 加平均值
- ✔ 用 JOIN 配對客戶與訂單

---

## 7️⃣ 今日未熟 / 需補強項目（已整理） {#weakness}

### 🔶 (1) CTE + Window Function 更進階用法

**你目前會：**
- `DENSE_RANK()`
- `ORDER BY DESC`

**但還不會：**
- `ROW_NUMBER()`
- `RANK()`
- `PARTITION BY` 多欄位
- Window Frame (`ROWS BETWEEN...`)

---

### 🔶 (2) 多 CTE + 多 JOIN 同時使用

你今天做得很好，但真正複雜報表會：
- CTE × 4 個
- JOIN × 3 個
- Window Function × 2 個

→ 還沒完全接觸

---

### 🔶 (3) 日期邏輯進階

**你還沒學：**
- `EOMONTH()` — 取得月底
- `DATEFROMPARTS()` — 組日期
- 月份排名
- 週次計算

---

### 🔶 (4) 類似 LeetCode 184 / 185 的難題

**像：**
- 每部門薪資最高
- 每客戶最近一次訂單
- 找出連續紀錄

這些你還沒練。

---

### 🔶 (5) Recursive CTE（遞迴 CTE）

**未來必學。**

**企業常用於：**
- 部門階層 Tree（主管 → 小組 → 成員）
- 分類階層
- 展開日期區間

---

## 8️⃣ CTE 學習路線（你目前在哪？還剩多少？） {#progress}

### ✔ 你已完成

| 內容 | 狀態 |
|------|------|
| CTE 基礎 | ✔ |
| CTE + GROUP BY | ✔ |
| CTE + CROSS JOIN | ✔ |
| CTE + Window Function（基本）| ✔ |
| 企業級報表 CTE | ✔ |

**你已經比多數新人強。**

---

### 🔥 你還沒學（CTE 第 3 ～ 5 章）

#### 🔸 第 3 章：CTE + 多重 Window Function（中級）

- `ROW_NUMBER` vs `RANK` vs `DENSE_RANK`
- 分組 Top N
- 每位客戶「最近一次訂單」
- 每位員工「最高薪水的紀錄」

---

#### 🔸 第 4 章：CTE + 多表 JOIN（真實企業報表）

- 訂單明細 × 商品 × 客戶 × 會員等級
- 每月排行榜
- 客戶復購模型（RFM 模型）

---

#### 🔸 第 5 章：Recursive CTE（高級）

- 主管 → 部門 → 員工階層
- 展開日期範圍成每日一列
- 計算連續天數