# 📘 SQL 學習筆記整理  
> 日期：2025-11-05
> 主題：SQL Procedure

---

## 1.最基本 Stored Procedure：查詢部門員工 
```sql
### ➤ 建立
CREATE PROCEDURE dbo.usp_GetEmployeesByDept
    @DeptID INT  -- 這是輸入參數
AS
BEGIN
    SELECT 
        e.ID         AS EmployeeID,
        e.Name       AS EmployeeName,
        e.Salary,
        d.Name       AS DepartmentName
    FROM dbo.Employees e
    JOIN dbo.Departments d
      ON e.DepartmentID = d.ID
    WHERE e.DepartmentID = @DeptID;
END
GO

### ➤ 測試執行
EXEC dbo.usp_GetEmployeesByDept @DeptID = 1;

✅ 結果
會列出部門 ID = 1（RD）的員工。
你可以改成 @DeptID = 3 來查 Sales 部門。

補充：usp = User Stored Procedure 的縮寫
```
--------
## 2.如何修正已創建的預存程序名稱？
```sql
1.使用系統程序 sp_rename
EXEC sp_rename 'usp_GetEmployeesByDepy', 'usp_GetEmployeesByDept';

2.先刪除，再重新創建
-- 步驟 1: 刪除舊的程序
DROP PROCEDURE dbo.usp_GetEmployeesByDepy;
GO

-- 步驟 2: 使用正確的名稱重新創建
CREATE PROCEDURE dbo.usp_GetEmployeesByDept
    @DeptID INT
AS
BEGIN
    -- 您的 SELECT 查詢邏輯
    SELECT ...
END
GO
```
--------

## 3. 加入多條件與錯誤處理
```sql
ALTER PROCEDURE dbo.usp_GetEmployeesByDept
	@DeptID INT = NULL,
	@MinSalary INT = NULL
AS
BEGIN
	SET NOCOUNT ON; --避免回傳「x rows affected」不回傳影響幾列的訊息
	SELECT 
		E.ID	 AS EmployeeID,
		E.NAME   AS EmployeeName,
		E.Salary, 
		D.Name	 AS DepartmentName
	FROM dbo.Employees	 AS E
	JOIN dbo.Departments AS D
	ON E.DepartmentID = D.ID
	WHERE
		(@DeptID IS NULL OR e.DepartmentID = @DeptID)
		AND (@MinSalary IS NULL OR e.Salary >= @MinSalary)
	ORDER BY e.Salary DESC;
END
GO

```
--------
## Stored Procedure with OUTPUT 參數
```sql
1.➤ 建立一個回傳部門平均薪資的版本
CREATE PROCEDURE dbo.usp_GetDeptAvgSalary
    @DeptID INT,
    @AvgSalary DECIMAL(10,2) OUTPUT  -- 輸出參數
AS
BEGIN
    SELECT @AvgSalary = AVG(CAST(Salary AS DECIMAL(10,2)))
    FROM dbo.Employees
    WHERE DepartmentID = @DeptID;
END
GO

2.➤ 執行（取得輸出）
DECLARE @Result DECIMAL(10,2);
EXEC dbo.usp_GetDeptAvgSalary @DeptID = 1, @AvgSalary = @Result OUTPUT;
SELECT @Result AS AvgSalary;

```
## 用 TRY...CATCH 捕捉錯誤（交易 / 錯誤處理基礎）
```sql
1.➤ 建立一個安全更新薪資的範例
CREATE PROCEDURE dbo.usp_UpdateEmployeeSalary
	@EmpID INT,
	@NewSalary INT
AS
BEGIN
	BEGIN TRY
		BEGIN TRANSACTION; --開始交易

		UPDATE dbo.Employees
		SET	Salary = @NewSalary
		WHERE ID = @EmpID;

		IF @@ROWCOUNT = 0 --上一個操作的影響列如果 = 0
			THROW 50001, '找不到員工 ID', 1; --THROW 錯誤編號, 錯誤訊息, 狀態碼;

		COMMIT TRANSACTION; --正常就提交
		PRINT '薪資更新成功'
	END TRY
	BEGIN CATCH
		ROLLBACK TRANSACTION; --出錯就回滾 = 取消雙方交易，回到交易前
		PRINT '發生錯誤：' + ERROR_MESSAGE();
	END CATCH
END
GO

2.➤ 測試
EXEC dbo.usp_UpdateEmployeeSalary @EmpID = 1, @NewSalary = 80000;
EXEC dbo.usp_UpdateEmployeeSalary @EmpID = 99, @NewSalary = 60000;  -- 測試錯誤


```
--------
## 刪除 / 修改 Stored Procedure
```sql
1.-- 修改
ALTER PROCEDURE dbo.usp_GetEmployeesByDept AS ...

2.-- 刪除
DROP PROCEDURE dbo.usp_GetEmployeesByDept;

```
--------
## 寫一個 SP：usp_AddEmployee → 新增一筆員工資料，成功後用 OUTPUT 回傳新員工 ID
```sql
1. 新增一筆員工資料，成功後用 OUTPUT 回傳新員工 I
ALTER PROCEDURE dbo.usp_AddEmployee

	@NewEmpID INT OUTPUT,

	@EmptName NVARCHAR(20),

	@EmptSalary INT,

	@EmptDeptID INT

AS

BEGIN

	SET NOCOUNT ON;

	INSERT INTO dbo.Employees(Name, Salary, DepartmentID)

	VALUES (@EmptName, @EmptSalary, @EmptDeptID);

	SELECT @NewEmpID = SCOPE_IDENTITY();

	IF @NewEmpID IS NULL 
        THROW 50002, '新增員工失敗，未取得新ID', 1;
	PRINT '新員工 ' + @EmptName + ' 資料新增成功。';
END
GO

2.OUTPUT 回傳新員工 ID
DECLARE @Result INT;

EXEC dbo.usp_AddEmployee

@NewEmpID = @Result OUTPUT,

@EmptName = N'Xuan',

@EmptSalary = 50000,

@EmptDeptID = 1

SELECT @Result AS NewEmployeeID;

