# 📘 SQL 學習筆記整理  
> 日期：2025-11-03
> 主題：SQL 建立與理解索引（Index）

---

## 🧩 一、什麼是索引（Index）

### SELECT 與 WHERE
```sql
想像你有一本電話簿 📖：

沒有索引 → 你要找「王小明」，只能一頁一頁翻。

有索引（依姓名排序） → 你可以直接跳到「王」開頭那頁。

👉 資料庫的索引也是一樣，它在某個欄位上建立快速搜尋樹結構（B-Tree）。

查詢時 SQL Server 可以直接跳到指定位置，而不必全表掃描。
```
--------

## 🧩 二、索引的兩種主要型態

### Clustered Index（聚集索引）
* 實際資料的物理排序方式。每個表只能有一個。
    * Primary Key 預設是聚集索引
        * 用於「主要排序／查詢依據」欄位

### Non-Clustered Index（非聚集索引）
* 類似「目錄」，指向資料所在頁面。可建立多個。
    * 在 Salary 上建立
        * 常用於 WHERE 條件或 JOIN 條件欄位
查詢時 SQL Server 可以直接跳到指定位置，而不必全表掃描。
--------

## 🧩 三、實作練習

### 在薪水欄位上建立非聚集索引
```sql
CREATE NONCLUSTERED INDEX IX_Employees_Salary
ON Employees (Salary);

📘 說明：

* IX_Employees_Salary 是索引名稱（慣例 IX_開頭）

* (Salary) 是要建立索引的欄位

* 用來加速例如： SELECT * FROM Employees WHERE Salary > 50000;
```
--------
### 刪除索引
```sql
DROP INDEX IX_Employees_Salary ON Employees;
```
--------
### 多欄位索引（進階）
```sql
建立一個同時以部門與薪水排序的索引：

CREATE NONCLUSTERED INDEX IX_Employees_Dept_Salary
ON Employees (DepartmentID, Salary);

這樣在查詢：(前提是資料筆數要夠大)
SELECT * FROM Employees
WHERE DepartmentID = 1 AND Salary > 50000;
時會特別快。
```
--------
## ⚠️ 注意：索引的副作用
* 插入／更新／刪除會變慢（因為索引也要更新）

* 太多索引會浪費磁碟與記憶體

* 要針對「查詢頻繁、資料多」的欄位建立