🟦 Part 1：資料庫操作（Database Basics）
1. 切換資料庫
USE 資料庫名稱;


作用：所有後續語句都會在該 DB 執行。

---

🟦 Part 2：建立資料表（CREATE TABLE）
2. 建立基本表格
```sql
CREATE TABLE Countries
(
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Population BIGINT NULL
);
```

3. 插入資料（INSERT INTO ... VALUES）
```sql
INSERT INTO Countries(Name, Population) VALUES
('Taiwan', 23500000),
('Japan', 126000000),
('USA', 331000000),
('Estonia', 1330000);
```

注意：欄位順序必須與 VALUES 對應。

---

🟦 Part 3：欄位預設值（DEFAULT Constraint）

4. 使用 ALTER 新增 DEFAULT
```sql
ALTER TABLE Products
ADD CONSTRAINT DF_Products_CreateDate DEFAULT GETDATE() FOR CreateDate;
```

4.1 建立表格時內建 DEFAULT
```sql
CREATE TABLE Products(
    ProductId INT IDENTITY(1,1) PRIMARY KEY,
    ProductName NVARCHAR(100) NOT NULL,
    Price DECIMAL(10,2) NOT NULL DEFAULT 0,
    Stock INT NOT NULL DEFAULT 0,
    CreateDate DATETIME NOT NULL DEFAULT GETDATE()
);
```
---

🟦 Part 4：補上 PK / FK（關聯設定）
4.5 補上 Primary Key
```sql
ALTER TABLE Customers
ADD CONSTRAINT PK_Customers PRIMARY KEY (CustomerId);
```
補上 Foreign Key
```sql
ALTER TABLE Orders 
ADD CONSTRAINT FK_Orders_Customers 
FOREIGN KEY (CustomerId)
REFERENCES Customers(CustomerId);
```
---

🟦 Part 5：CRUD（新增 / 查詢 / 更新 / 刪除）
5. 刪除資料
```sql
DELETE FROM Products
WHERE ProductName = 'Pixel10';
```

6. TOP + ORDER BY 查詢最大／最小值
```sql
SELECT TOP(1) ProductName
FROM Products
ORDER BY Stock DESC;
```
---

🟦 Part 6：Stored Procedure（SP）基礎 CRUD
7. Update 型 SP
```sql
CREATE PROCEDURE spUpdatePrice
    @ProductId INT,
    @NewPrice DECIMAL(10,2)
AS
BEGIN
    UPDATE Products
    SET Price = @NewPrice
    WHERE ProductId = @ProductId;
END;
```
---

🟦 Part 7：新增資料後回傳新 Id（OUTPUT + SCOPE_IDENTITY）
8. 新增商品並回傳新 Id
```sql
CREATE PROCEDURE spAddProductWithId
    @ProductName NVARCHAR(100),
    @Price DECIMAL(10,2),
    @Stock INT,
    @NewId INT OUTPUT
AS
BEGIN
    INSERT INTO Products(ProductName, Price, Stock)
    VALUES(@ProductName, @Price, @Stock);

    SET @NewId = SCOPE_IDENTITY();
END;
```
---

🟦 Part 8：基本管理 — 刪除資料表、刪除 SP
```sql
9. 刪除表格
DROP TABLE IF EXISTS Orders;
```

13. 刪除 Stored Procedure
```sql
DROP PROCEDURE IF EXISTS spAddOrder;
```
---

🟦 Part 9：JOIN + GROUP BY 實務（分析類查詢）
10. 查每位客戶訂單總額
```sql
SELECT C.CustomerId, C.CustomerName, SUM(O.Amount) AS TotalAmount
FROM Customers C
JOIN Orders O ON C.CustomerId = O.CustomerId
GROUP BY C.CustomerId, C.CustomerName;
```
---

🟦 Part 10：基本 CRUD Stored Procedure 實戰
11. 新增客戶 SP
```sql
CREATE PROCEDURE spAddCustomer 
    @CustomerName NVARCHAR(100), 
    @Phone NVARCHAR(20) 
AS 
BEGIN 
    INSERT INTO Customers(CustomerName, Phone)
    VALUES(@CustomerName, @Phone);
END;
```
執行
```sql
EXEC spAddCustomer
    @CustomerName = N'小小兵',
    @Phone = '09123456789';
```
---

🟦 Part 11：新增資料前的檢查（NOT EXISTS）
12. 新增訂單但需先確認客戶存在
```sql
IF NOT EXISTS(
    SELECT 1 
    FROM Customers 
    WHERE CustomerId = @CustomerId
)
BEGIN
    PRINT 'CustomerId 不存在';
    RETURN;
END
```
---


🟦 Part 12：使用 ALTER 修改既有 Stored Procedure（SP 重大技能）
📌 用途

當 SP 已被建立，但需要補上條件、修正錯誤、加入防呆時，用 ALTER PROCEDURE。

📌 範例：改善 spAddOrder 的版本
```sql
ALTER PROCEDURE spAddOrder
    @CustomerId INT,
    @Amount DECIMAL(10,2)
AS
BEGIN
    IF NOT EXISTS (
        SELECT 1 
        FROM Customers 
        WHERE CustomerId = @CustomerId
    )
    BEGIN
        PRINT 'Customer 不存在';
        RETURN;
    END

    INSERT INTO Orders(CustomerId, Amount)
    VALUES(@CustomerId, @Amount);
END;
```
⭐ 重點

CREATE 只能用一次

修改 SP 全用 ALTER PROCEDURE

企業專案中 SP 會不斷調整 → 一定會用到

---

🟦 Part 13：查詢型 SP — 透過 JOIN 查詢資料
📌 用途

給定 CustomerId，查詢其完整訂單資料（含名稱、金額、日期）。

📌 範例
```sql
ALTER PROCEDURE spGetOrdersByCustomer
    @CustomerId INT
AS
BEGIN
    SELECT 
        C.CustomerName, 
        O.OrderId, 
        O.Amount, 
        O.OrderDate
    FROM Customers C
    LEFT JOIN Orders O 
        ON C.CustomerId = O.CustomerId
    WHERE C.CustomerId = @CustomerId;
END;
```

⭐ 重點

使用 LEFT JOIN → 即使沒訂單也可查到客戶

查詢型 SP 是企業系統最常使用的一種 SP

通常會搭配條件、排序、分頁

---

Part 14：Stored Procedure 的 OUTPUT 參數概念（基礎但重要）
📌 用途

OUTPUT 用來讓 SP 回傳單一值（不是資料表）。

例如：

計算值

新建立資料的 Id

結果狀態

📌 範例
```sql
CREATE PROCEDURE spReturnNumber
    @Result INT OUTPUT
AS
BEGIN
    SET @Result = 168;
END;
GO

DECLARE @MyNum INT;
EXEC spReturnNumber @Result = @MyNum OUTPUT;
SELECT @MyNum AS ReturnedValue;
```

⭐ 重點

OUTPUT 一定要在定義與執行時都寫

回傳單一值用 OUTPUT，比 SELECT 更精準

---
    
🟦 Part 15：新增訂單並回傳 OrderId（OUTPUT + SCOPE_IDENTITY）
📌 用途

新增資料後，取得新資料列的 Id（常用於前端立即顯示新項目）。

📌 完整 SP
```sql
ALTER PROCEDURE spAddOrderWithId 
    @CustomerId INT, 
    @Amount DECIMAL(10,2), 
    @NewOrderId INT OUTPUT 
AS 
BEGIN 
    IF NOT EXISTS (
        SELECT 1 
        FROM Customers 
        WHERE CustomerId = @CustomerId
    ) 
    BEGIN 
        PRINT '顧客不存在'; 
        RETURN; 
    END 

    INSERT INTO Orders(CustomerId, Amount) 
    VALUES (@CustomerId, @Amount); 

    SET @NewOrderId = SCOPE_IDENTITY(); 
END;
```
📌 呼叫方式
```sql
DECLARE @Id INT;

EXEC spAddOrderWithId 
    @CustomerId = 5, 
    @Amount = 66666, 
    @NewOrderId = @Id OUTPUT;

SELECT @Id AS NewOrderId;
```
⭐ 重點

SCOPE_IDENTITY() → 取得同一 Scope 的自動編號

不要用 @@IDENTITY

常用於 Web API 新增資料後回前端

---

🟦 Part 16：查詢前 N 名（TOP(@N) + ORDER BY）
📌 用途

查詢前 N 位消費金額最高的顧客（排行榜功能）。

📌 SP
```sql
ALTER PROCEDURE spTopCustomersByAmount
    @TopN INT
AS
BEGIN
    SELECT TOP(@TopN) 
        C.CustomerId,
        C.CustomerName, 
        SUM(O.Amount) AS Total_Amount
    FROM Customers C
    JOIN Orders O 
        ON C.CustomerId = O.CustomerId
    GROUP BY C.CustomerId, C.CustomerName
    ORDER BY Total_Amount DESC;
END;
```
⭐ 重點

TOP(@N) 支援變數 → 動態查詢

必須搭配 ORDER BY

SUM + GROUP BY 是基本分析查詢

---

🟦 Part 17：進階 WHERE — 可選條件搜尋（搜尋表單功能）
📌 用途

建立搜尋頁面 API / SP → 允許使用者輸入任意條件：

✔ 名稱（可以不填）
✔ 最低金額（可不填）
✔ 最高金額（可不填）

📌 SP
```sql
ALTER PROCEDURE spSearchOrders
    @CustomerName NVARCHAR(100) = NULL,
    @MinAmount DECIMAL(10,2) = NULL,
    @MaxAmount DECIMAL(10,2) = NULL
AS
BEGIN
    SELECT 
        C.CustomerName, 
        O.OrderId, 
        O.Amount, 
        O.OrderDate
    FROM Customers C
    JOIN Orders O 
        ON C.CustomerId = O.CustomerId
    WHERE 
        (@CustomerName IS NULL OR C.CustomerName LIKE '%' + @CustomerName + '%')
        AND (@MinAmount IS NULL OR O.Amount >= @MinAmount)
        AND (@MaxAmount IS NULL OR O.Amount <= @MaxAmount);
END;
```
⭐ 重點（企業常用）

彈性搜尋 = 每一欄都可填或不填

WHERE 條件須寫成 (@Param IS NULL OR …)

這種 SP 幾乎每個企業後台都會用

---

🟦 Part 18：部分更新（COALESCE 技巧）
📌 用途

更新資料時，不強迫使用者提供所有欄位，只更新提供的欄位。

📌 SP
```sql
ALTER PROCEDURE spUpdateCustomerInfo 
    @CustomerId INT, 
    @CustomerName NVARCHAR(100) = NULL, 
    @Phone NVARCHAR(20) = NULL 
AS 
BEGIN 
    IF NOT EXISTS (
        SELECT 1 
        FROM Customers 
        WHERE CustomerId = @CustomerId
    )
    BEGIN 
        PRINT '客戶不存在'; 
        RETURN; 
    END 

    IF @CustomerName IS NULL AND @Phone IS NULL 
    BEGIN 
        PRINT '沒有資料可以更新'; 
        RETURN; 
    END 

    UPDATE Customers 
    SET 
        CustomerName = COALESCE(@CustomerName, CustomerName), 
        Phone        = COALESCE(@Phone, Phone)
    WHERE CustomerId = @CustomerId; 

    PRINT '更新成功'; 
END;
```
⭐ 重點

COALESCE(a, b) → 若 a 不為 NULL 用 a，否則用 b

適合用於後台管理介面「只改幾個欄位」

---

🟦 Part 19：刪除前檢查（NOT EXISTS + EXISTS 防呆）
📌 用途

企業系統中，刪除資料前一定會「檢查關聯性」。

📌 SP：刪除顧客的完整流程
```sql
ALTER PROCEDURE spDeleteCustomer
    @CustomerId INT
AS
BEGIN
    IF NOT EXISTS(
        SELECT 1
        FROM Customers
        WHERE CustomerId = @CustomerId
    )
    BEGIN
        PRINT '顧客不存在';
        RETURN;
    END

    IF EXISTS(
        SELECT 1
        FROM Orders
        WHERE CustomerId = @CustomerId
    )
    BEGIN
        PRINT '顧客仍有訂單，不能刪除';
        RETURN;
    END

    DELETE FROM Customers
    WHERE CustomerId = @CustomerId;

    PRINT '刪除成功';
END;
```
⭐ 重點

必學實務技巧（避免刪除造成外鍵錯誤）

很多公司 SP 都會用這種「三段式防呆」

---

🟦 Part 20：修改或刪除 Default Constraint
📌 用途

若欄位預設值（DEFAULT）寫錯或需調整，需要 ADD 或 DROP。

📌 新增 Default Constraint
```sql
ALTER TABLE Products
    ADD CONSTRAINT DF_Products_Price 
    DEFAULT 2000 FOR Price;
```
📌 刪除 Default Constraint
```sql
ALTER TABLE Products
    DROP CONSTRAINT DF_Products_Price;
```
⭐ 重點

要改 DEFAULT → 一定要 DROP 再 ADD

DEFAULT 是維護活動很常調整的項目

---

交易 Transaction

交易 = 多個 SQL 必須一起成功 或 一起失敗。
否則就回復（ROLLBACK）保持資料乾淨。

🧱 交易的基本語法
```sql
BEGIN TRAN;
...
COMMIT;  -- 全部成功
```

如果有錯誤：
```sql
ROLLBACK;  -- 全部取消
```

---
🌟 第二步：例外處理 TRY / CATCH

SQL Server 的錯誤處理方式：
```sql
BEGIN TRY
    -- 可能會錯的 SQL
END TRY
BEGIN CATCH
    -- 錯誤發生後要做什麼？
END CATCH
```
---

🔥 TRY + CATCH + TRAN 的完整模板（必背！）
```sql
BEGIN TRY
    BEGIN TRAN;

    -- 可能會失敗的 SQL

    COMMIT;  -- 全部成功
END TRY
BEGIN CATCH
    ROLLBACK; -- 發生錯誤 → 全部取消

    PRINT '發生錯誤';
END CATCH;
```

牛刀小試

```sql
BEGIN TRY
	BEGIN TRAN;

	PRINT '➡ 開始新增客戶...';

	INSERT INTO Customers(CustomerName, Phone)
	VALUES (N'測試流程客戶', '0911000222');

    PRINT '✔ 客戶新增成功';

    PRINT '➡ 開始新增訂單...';

	INSERT INTO Orders(CustomerId, Amount)
    VALUES (99999, 50000);  -- 這裡故意放錯，觸發外鍵錯誤

    PRINT '✔ 訂單新增成功';

    COMMIT;
    PRINT '🎉 交易成功，全部完成！';
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT '❌ 發生錯誤，已回復資料';
    PRINT ERROR_MESSAGE();
END CATCH;
```
這是我們之後寫所有「重要 SP」的標準格式。

---
🟦 Part 21：交易（TRANSACTION）+ TRY / CATCH

🔥 標準模板（必背）
```sql
BEGIN TRY
    BEGIN TRAN;

    -- 可能失敗的 SQL

    COMMIT;   -- 全部成功
END TRY
BEGIN CATCH
    ROLLBACK; -- 發生錯誤 → 回復
    PRINT ERROR_MESSAGE();
END CATCH;
```
新增訂單（含交易 + 驗證客戶存在 + SCOPE_IDENTITY）
```sql
BEGIN TRY
	BEGIN TRAN;

	DECLARE @CustomerId INT = 5;
	DECLARE @NewOrderId INT;

	IF NOT EXISTS(
		SELECT 1 
		FROM Customers
		WHERE CustomerId = @CustomerId
	)
	BEGIN
		PRINT '顧客不存在';
		ROLLBACK;
		RETURN;
	END

	INSERT INTO Orders(CustomerId, Amount)
	VALUES(@CustomerId, 12345)

	SET @NewOrderId = SCOPE_IDENTITY();

	PRINT '新訂單的 OrderId = ' + CAST(@NewOrderId AS NVARCHAR(20));

	COMMIT;
	PRINT '交易完成!';
END TRY
BEGIN CATCH
	ROLLBACK;
	PRINT '發生錯誤，已回復資料';
	PRINT ERROR_MESSAGE();
END CATCH;
```

---

🟦 Part 22：扣庫存（Inventory Deduction）— 交易 + 驗證流程（
📌 用途

扣庫存（減少庫存數量）是電商、倉儲、超商系統最基礎的商業邏輯。
此流程必須保證：

商品存在

庫存足夠

庫存更新正確

交易不能錯扣（全部成功或全部失敗）

因此必須搭配：
✔ BEGIN TRAN / COMMIT / ROLLBACK
✔ TRY / CATCH
✔ EXISTS / NOT EXISTS
✔ 變數儲存庫存值

① 檢查商品是否存在（NOT EXISTS 防呆）
```sql
IF NOT EXISTS(
	SELECT 1 
	FROM Products
	WHERE ProductId = @ProductId)
BEGIN
	PRINT '商品不存在';
	ROLLBACK;
	RETURN;
END
```
防止扣不存在的商品

最常見的邏輯錯誤 → 必須先檢查

② 讀取目前庫存
```sql
SELECT @Stock = Stock
FROM Products
WHERE ProductId = @ProductId;
```

透過 SELECT @Variable = 欄位 讀出庫存

若商品存在，變數 @Stock 會被填入

③ 判斷庫存是否足夠
```sql
IF @Stock < @Quantity
BEGIN
	PRINT '庫存不足';
	ROLLBACK;
	RETURN;
END
```

若庫存不夠 → 必須中止交易

保證不會扣到負庫存（企業超常用）

④ 計算新庫存
```sql
SET @NewStock = @Stock - @Quantity;
```

計算結果先存入變數

有利於後續印出訊息與更新

⑤ 更新庫存（正式扣庫存）
```sql
UPDATE Products 
SET Stock = @NewStock
WHERE ProductId = @ProductId;
```

🔥 完整扣庫存 SQL（含 TRY / CATCH + TRANSACTION）
```sql
DECLARE @ProductId INT = 1;
DECLARE @Quantity INT = 5;
DECLARE @Stock INT;
DECLARE @NewStock INT;

BEGIN TRY
	BEGIN TRAN

	-- ① 商品存在
	IF NOT EXISTS(
		SELECT 1 
		FROM Products
		WHERE ProductId = @ProductId)
	BEGIN
		PRINT '商品不存在';
		ROLLBACK;
		RETURN;
	END

	-- ② 讀取庫存
	SELECT @Stock = Stock
	FROM Products
	WHERE ProductId = @ProductId;

	-- ③ 庫存不足判斷
	IF @Stock < @Quantity
	BEGIN
		PRINT '庫存不足';
		ROLLBACK;
		RETURN;
	END

	-- ④ 計算新庫存
	SET @NewStock = @Stock - @Quantity;

	-- ⑤ 更新庫存
	UPDATE Products 
	SET Stock = @NewStock
	WHERE ProductId = @ProductId;

	COMMIT;
	PRINT '扣庫存成功！剩餘庫存 = ' + CAST(@NewStock AS NVARCHAR(50))
END TRY
BEGIN CATCH
	ROLLBACK;
	PRINT ERROR_MESSAGE();
END CATCH;
```
⭐ 重點整理（務必理解）
✔ 交易（TRANSACTION）確保庫存不會扣一半

有任何錯誤 → ROLLBACK

扣庫存必須「要嘛成功、要嘛不做」

✔ NOT EXISTS 必備

避免扣不存在的商品。

✔ 讀取庫存必須先存入變數

避免更新後讀不到舊數值。

✔ 庫存不足 → 立即中止

避免負庫存（企業大忌）。

✔ TRY / CATCH 保護整個扣庫存流程

避免資料剩一半、造成系統不一致。

---

🟦 Part 23：退貨（Return Inventory）— 庫存增加流程（Transaction + 驗證邏輯）
📌 用途

退貨流程是電商、倉儲、POS（收銀系統）中必備的商業邏輯。

與扣庫存相反，它需要：

商品存在

退貨數量合法（不可為 0 或負數）

正確計算新庫存

交易成功後更新資料

失敗則回復資料（ROLLBACK）

這題結合 交易（TRAN）、錯誤處理 TRY/CATCH、驗證流程、變數運算、UPDATE，
屬於 中階 SQL 實務題。

📘 完整流程（五步驟）
① 商品存在檢查（NOT EXISTS 防呆）
```sql
IF NOT EXISTS(
	SELECT 1
	FROM Products
	WHERE ProductId = @ProductId)
BEGIN
	PRINT '商品不存在';
	ROLLBACK;
	RETURN;
END
```

避免退一個不存在的商品。

② 避免不合法退貨（不可 <= 0）
```sql
IF @Quantity <= 0
BEGIN
	PRINT '商品不可退負數';
	ROLLBACK;
	RETURN;
END
```

原因：

@Quantity = 0 → 無意義

@Quantity < 0 → 等於扣庫存，邏輯錯誤

③ 讀取目前庫存
```sql
SELECT @Stock = Stock
FROM Products
WHERE ProductId = @ProductId;
```

用變數將庫存抓出來，是後續計算所需。

④ 計算新庫存（庫存 + 退貨量）
```sql
SET @NewStock = @Stock + @Quantity;
```

計算邏輯：

庫存增加（退貨 = 庫存回補）

⑤ 更新庫存
```sql
UPDATE Products
SET Stock = @NewStock
WHERE ProductId = @ProductId;
```

正式將新庫存寫回資料庫。

🔥 完整 SQL（含 TRY / CATCH + TRAN）
```sql
DECLARE @ProductId INT = 5;
DECLARE @Quantity INT = 6;
DECLARE @Stock INT;
DECLARE @NewStock INT;

BEGIN TRY
	BEGIN TRAN;

	-- ① 商品存在？
	IF NOT EXISTS(
		SELECT 1
		FROM Products
		WHERE ProductId = @ProductId)
	BEGIN
		PRINT '商品不存在';
		ROLLBACK;
		RETURN;
	END

	-- ② 避免退貨數量 <= 0
	IF @Quantity <= 0
	BEGIN
		PRINT '商品不可退負數';
		ROLLBACK;
		RETURN;
	END

	-- ③ 讀取目前庫存
	SELECT @Stock = Stock
	FROM Products
	WHERE ProductId = @ProductId;

	-- ④ 計算新庫存
	SET @NewStock = @Stock + @Quantity;

	-- ⑤ 更新庫存
	UPDATE Products
	SET Stock = @NewStock
	WHERE ProductId = @ProductId;

	-- 完成交易
	COMMIT;
	PRINT '退貨成功！新的庫存 = ' + CAST(@NewStock AS NVARCHAR(20));
END TRY
BEGIN CATCH
	ROLLBACK;
	PRINT ERROR_MESSAGE();
END CATCH;
```
⭐ 重點整理（務必理解）
概念	說明
NOT EXISTS	防止退不存在的商品
@Quantity <= 0 檢查	防止不合理退貨
SELECT @Stock = Stock	把庫存抓進變數
SET @NewStock = @Stock + @Quantity	計算新庫存
TRY / CATCH	捕捉例外、確保不會更新一半
TRANSACTION	要嘛全部成功、要嘛全部失敗

---

🟦 Part 24：建立訂單（Order）＋扣庫存＋訂單明細（OrderDetails）

⚡（商品 → 扣庫存 → 建立訂單 → 建立訂單明細）完整交易流程

這題是企業級後端邏輯，牽涉多個資料表：

Products（商品）

Orders（訂單主檔）

OrderDetails（訂單明細）

Customers（客戶）

並且包含：

✔ 商品存在檢查
✔ 庫存檢查
✔ 計算訂單金額
✔ 交易保護（TRAN）
✔ SCOPE_IDENTITY 取得 OrderId
✔ INSERT 兩張表（訂單＋明細）
✔ 更新庫存
✔ TRY / CATCH

屬於 中高階 SQL 實戰題（後端工程師必備）。

📘 完整邏輯流程（7 個步驟）
① 檢查商品是否存在（NOT EXISTS）

避免建立一張莫名其妙、沒有商品的訂單。
```sql
IF NOT EXISTS(
	SELECT 1
	FROM Products
	WHERE ProductId = @ProductId)
BEGIN 
	PRINT '商品不存在';
	ROLLBACK;
	RETURN;
END
```

② 讀取庫存與單價（複數欄位一次查）
```sql
SELECT 
	@Stock = Stock,
	@UnitPrice = Price
FROM Products
WHERE ProductId = @ProductId;
```
③ 判斷庫存是否足夠
```sql
IF @Stock - @Quantity < 0
BEGIN 
	PRINT '庫存不足';
	ROLLBACK;
	RETURN;
END
```

④ 扣庫存（Stock - Quantity）
```sql
SET @NewStock = @Stock - @Quantity;

UPDATE Products
SET Stock = @NewStock
WHERE ProductId = @ProductId;
```
⑤ 計算訂單金額（單價 × 數量）
```sql
SET @OrderAmount = @UnitPrice * @Quantity;
```
⑥ 建立 Orders 資料並取得 OrderId
```sql
INSERT INTO Orders(CustomerId, Amount)
VALUES(@CustomerId, @OrderAmount);

SET @OrderId = SCOPE_IDENTITY();
```
⑦ 建立訂單明細 OrderDetails
```sql
INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
VALUES (@OrderId, @ProductId, @Quantity, @UnitPrice);
```
🔥 完整 SQL（含 TRAN + TRY/CATCH）
```sql
DECLARE @CustomerId INT = 3;
DECLARE @ProductId INT = 2;
DECLARE @Quantity INT = 4;

DECLARE @Stock INT;
DECLARE @NewStock INT;

-- 缺少的 DECLARE
DECLARE @UnitPrice DECIMAL(10,2);
DECLARE @OrderAmount DECIMAL(10,2);
DECLARE @OrderId INT;

BEGIN TRY
	BEGIN TRAN;

	-- 1）確認商品存在
	IF NOT EXISTS(
		SELECT 1
		FROM Products
		WHERE ProductId = @ProductId)
	BEGIN 
		PRINT '商品不存在';
		ROLLBACK;
		RETURN;
	END

	-- 2）取得庫存與單價
	SELECT 
		@Stock = Stock,
		@UnitPrice = Price
	FROM Products
	WHERE ProductId = @ProductId;

	-- 3）確認庫存
	IF @Stock < @Quantity
	BEGIN 
		PRINT '庫存不足';
		ROLLBACK;
		RETURN;
	END

	-- 4）更新庫存
	SET @NewStock = @Stock - @Quantity;

	UPDATE Products
	SET Stock = @NewStock 
	WHERE ProductId = @ProductId;

	-- 5）建立訂單
	SET @OrderAmount = @UnitPrice * @Quantity;

	INSERT INTO Orders(CustomerId, Amount)
	VALUES(@CustomerId, @OrderAmount);

	-- 6）取得 OrderId
	SET @OrderId = SCOPE_IDENTITY();

	-- 7）新增訂單明細
	INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
	VALUES (@OrderId, @ProductId, @Quantity, @UnitPrice);

	COMMIT;
	PRINT 'TR3 完成！新增訂單成功，OrderId = ' + CAST(@OrderId AS NVARCHAR(20));
END TRY
BEGIN CATCH
	ROLLBACK;
	PRINT '發生錯誤，交易已回復';
	PRINT ERROR_MESSAGE();
END CATCH;
```

---

🟦 Part 25：多商品訂單（一次購買兩項商品）
⚡（多商品 → 扣庫存 → 建立訂單 → 多筆明細）完整企業級交易流程

這題是在「單一商品訂單」的基礎上提升難度，變成：

同一張訂單，購買兩個商品

每個商品要：

檢查存在

檢查庫存足夠

抓出庫存與單價

扣除庫存

訂單主檔（Orders）只有一筆

訂單明細（OrderDetails）要寫兩筆

全程必須使用 Transaction → 全部成功 or 全部失敗

這是企業後端最常用的邏輯之一，非常接近你之後做 RF 案件排程系統時會用到的複雜流程。

📘 多商品交易流程（總共 9 步）
① 確認客戶存在
```sql
IF NOT EXISTS(
	SELECT 1
	FROM Customers
	WHERE CustomerId = @CustomerId)
BEGIN
	PRINT '此顧客ID不存在';
	ROLLBACK;
	RETURN;
END
```
② & ③ 商品存在驗證（商品1、商品2）
```sql
IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId1)

IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId2)
```

→ 多商品流程中，每個商品都需要防呆。

④ 一次讀取庫存 + 單價（兩次 SELECT）
```sql
SELECT @Stock1 = Stock, @UnitPrice1 = Price
FROM Products
WHERE ProductId = @ProductId1;

SELECT @Stock2 = Stock, @UnitPrice2 = Price
FROM Products
WHERE ProductId = @ProductId2;
```

注意：每個商品都有不同庫存與單價，不能混用。

⑤ 庫存是否足夠？（兩次檢查）
```sql
IF @Stock1 < @Quantity1

IF @Stock2 < @Quantity2
```

→ 任一商品庫存不足 → 交易取消。

⑥ 扣庫存（更新 Products 兩次）
```sql
UPDATE Products
SET Stock = @Stock1 - @Quantity1
WHERE ProductId = @ProductId1;

UPDATE Products
SET Stock = @Stock2 - @Quantity2
WHERE ProductId = @ProductId2;
```
⑦ 建立訂單（Orders 只有一筆）

訂單金額 = 商品1金額 + 商品2金額
```sql
SET @OrderAmount = @Quantity1 * @UnitPrice1
                 + @Quantity2 * @UnitPrice2;

INSERT INTO Orders(CustomerId, Amount)
VALUES (@CustomerId, @OrderAmount);

SET @OrderId = SCOPE_IDENTITY();
```
⑧ & ⑨ 建立訂單明細（兩筆 OrderDetails）
商品 1：
```sql
INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
VALUES (@OrderId, @ProductId1, @Quantity1, @UnitPrice1);
```
商品 2：
```sql
INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
VALUES (@OrderId, @ProductId2, @Quantity2, @UnitPrice2);
```
🔥 完整 SQL（含 TRAN / TRY / CATCH）
```sql
DECLARE @CustomerId INT = 3;

DECLARE @ProductId1 INT = 2;
DECLARE @Quantity1 INT = 3;
DECLARE @Stock1 INT;
DECLARE @UnitPrice1 DECIMAL(10,2);

DECLARE @ProductId2 INT = 5;
DECLARE @Quantity2 INT = 2;
DECLARE @Stock2 INT;
DECLARE @UnitPrice2 DECIMAL(10,2);

DECLARE @OrderAmount DECIMAL(10,2);
DECLARE @OrderId INT;

BEGIN TRY
	BEGIN TRAN;

	-- ① 檢查顧客
	IF NOT EXISTS(
		SELECT 1
		FROM Customers
		WHERE CustomerId = @CustomerId)
	BEGIN
		PRINT '此顧客ID不存在';
		ROLLBACK;
		RETURN;
	END

	-- ② 商品1存在？
	IF NOT EXISTS(
		SELECT 1
		FROM Products
		WHERE ProductId = @ProductId1)
	BEGIN
		PRINT '商品1不存在';
		ROLLBACK;
		RETURN;
	END

	-- ③ 商品2存在？
	IF NOT EXISTS(
		SELECT 1
		FROM Products
		WHERE ProductId = @ProductId2)
	BEGIN
		PRINT '商品2不存在';
		ROLLBACK;
		RETURN;
	END

	-- ④ 取得庫存與單價
	SELECT @Stock1 = Stock, @UnitPrice1 = Price
	FROM Products
	WHERE ProductId = @ProductId1;

	SELECT @Stock2 = Stock, @UnitPrice2 = Price
	FROM Products
	WHERE ProductId = @ProductId2;

	-- ⑤ 庫存足夠？
	IF @Stock1 < @Quantity1
	BEGIN
		PRINT '商品1庫存不足';
		ROLLBACK;
		RETURN;
	END

	IF @Stock2 < @Quantity2
	BEGIN
		PRINT '商品2庫存不足';
		ROLLBACK;
		RETURN;
	END

	-- ⑥ 扣庫存
	UPDATE Products
	SET Stock = @Stock1 - @Quantity1
	WHERE ProductId = @ProductId1;

	UPDATE Products
	SET Stock = @Stock2 - @Quantity2
	WHERE ProductId = @ProductId2;

	-- ⑦ 建立訂單（1 筆）
	SET @OrderAmount = @Quantity1 * @UnitPrice1
	                 + @Quantity2 * @UnitPrice2;

	INSERT INTO Orders(CustomerId, Amount)
	VALUES (@CustomerId, @OrderAmount);

	SET @OrderId = SCOPE_IDENTITY();

	-- ⑧ 訂單明細 1
	INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
	VALUES (@OrderId, @ProductId1, @Quantity1, @UnitPrice1);

	-- ⑨ 訂單明細 2
	INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
	VALUES (@OrderId, @ProductId2, @Quantity2, @UnitPrice2);

	COMMIT;
	PRINT 'TR3.1 完成，新增訂單成功！OrderId = ' + CAST(@OrderId AS NVARCHAR(20));
END TRY
BEGIN CATCH
	ROLLBACK;
	PRINT '新增失敗，全數回滾';
	PRINT ERROR_MESSAGE();
END CATCH;
```

---

📘 Part 26 — 多商品訂單完整流程筆記（Markdown）
🔹 變數定義（Customer、3 商品、金額變數）

依序建立：

顧客

商品 1、2、3（每個都有：Id、Quantity、Stock、UnitPrice、Subtotal）

訂單金額與 OrderId

🔹 完整 SQL（含交易）
```sql
DECLARE @CustomerId INT = 5;

DECLARE @ProductId1 INT = 2;
DECLARE @Quantity1 INT = 3;
DECLARE @NewStock1 INT;
DECLARE @Stock1 INT;
DECLARE @UnitPrice1 DECIMAL(10,2);
DECLARE @Subtotal1 DECIMAL(10,2);

DECLARE @ProductId2 INT = 5;
DECLARE @Quantity2 INT = 4;
DECLARE @NewStock2 INT;
DECLARE @Stock2 INT;
DECLARE @UnitPrice2 DECIMAL(10,2);
DECLARE @Subtotal2 DECIMAL(10,2);

DECLARE @ProductId3 INT = 7;
DECLARE @Quantity3 INT = 2;
DECLARE @NewStock3 INT;
DECLARE @Stock3 INT;
DECLARE @UnitPrice3 DECIMAL(10,2);
DECLARE @Subtotal3 DECIMAL(10,2);

DECLARE @OrderAmount DECIMAL(10,2);
DECLARE @FinalAmount DECIMAL(10,2);
DECLARE @OrderId INT;

BEGIN TRY
	BEGIN TRAN;

	-- ① 顧客存在
	IF NOT EXISTS(
		SELECT 1
		FROM Customers
		WHERE CustomerId = @CustomerId)
	BEGIN
		PRINT '顧客不存在';
		ROLLBACK;
		RETURN;
	END

	-- ② 商品存在（3 商品）
	IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId1)
	BEGIN PRINT '商品1不存在'; ROLLBACK; RETURN; END

	IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId2)
	BEGIN PRINT '商品2不存在'; ROLLBACK; RETURN; END

	IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId3)
	BEGIN PRINT '商品3不存在'; ROLLBACK; RETURN; END

	-- ③ 取得庫存與單價（3 商品）
	SELECT @Stock1 = Stock, @UnitPrice1 = Price
	FROM Products WHERE ProductId = @ProductId1;

	SELECT @Stock2 = Stock, @UnitPrice2 = Price
	FROM Products WHERE ProductId = @ProductId2;

	SELECT @Stock3 = Stock, @UnitPrice3 = Price
	FROM Products WHERE ProductId = @ProductId3;

	-- ④ 計算新庫存
	SET @NewStock1 = @Stock1 - @Quantity1;
	SET @NewStock2 = @Stock2 - @Quantity2;
	SET @NewStock3 = @Stock3 - @Quantity3;

	-- ⑤ 庫存不足？
	IF @NewStock1 < 0 BEGIN PRINT '商品1庫存不足'; ROLLBACK; RETURN; END
	IF @NewStock2 < 0 BEGIN PRINT '商品2庫存不足'; ROLLBACK; RETURN; END
	IF @NewStock3 < 0 BEGIN PRINT '商品3庫存不足'; ROLLBACK; RETURN; END

	-- ⑥ 小計 Subtotal
	SET @Subtotal1 = @UnitPrice1 * @Quantity1;
	SET @Subtotal2 = @UnitPrice2 * @Quantity2;
	SET @Subtotal3 = @UnitPrice3 * @Quantity3;

	SET @OrderAmount = @Subtotal1 + @Subtotal2 + @Subtotal3;

	-- ⑦ 折扣邏輯
	SET @FinalAmount =
		CASE
			WHEN @OrderAmount >= 20000 THEN @OrderAmount * 0.9
			WHEN @OrderAmount >= 10000 THEN @OrderAmount * 0.95
			ELSE @OrderAmount
		END;

	-- ⑧ 建立訂單主檔
	INSERT INTO Orders(CustomerId, Amount)
	VALUES(@CustomerId, @FinalAmount);

	SET @OrderId = SCOPE_IDENTITY();

	-- ⑨ 建立三筆訂單明細
	INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice, Subtotal)
	VALUES (@OrderId, @ProductId1, @Quantity1, @UnitPrice1, @Subtotal1);

	INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice, Subtotal)
	VALUES (@OrderId, @ProductId2, @Quantity2, @UnitPrice2, @Subtotal2);

	INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice, Subtotal)
	VALUES (@OrderId, @ProductId3, @Quantity3, @UnitPrice3, @Subtotal3);

	-- ⑩ 更新庫存
	UPDATE Products SET Stock = @NewStock1 WHERE ProductId = @ProductId1;
	UPDATE Products SET Stock = @NewStock2 WHERE ProductId = @ProductId2;
	UPDATE Products SET Stock = @NewStock3 WHERE ProductId = @ProductId3;

	COMMIT;
	PRINT 'TR4 完成，新增訂單成功！OrderId = ' + CAST(@OrderId AS NVARCHAR(20));
END TRY
BEGIN CATCH
	ROLLBACK;
	PRINT 'TR4新增失敗，全數回滾';
	PRINT ERROR_MESSAGE();
END CATCH;
```
⭐ 重點流程圖（簡化記憶）
```sql
顧客存在？
  ↓
3 商品存在？
  ↓
讀取3商品庫存與單價
  ↓
庫存是否足夠？
  ↓
計算 Subtotal（3 商品）
  ↓
計算 OrderAmount
  ↓
折扣運算 FinalAmount
  ↓
新增 Orders（1筆）
  ↓
新增 OrderDetails（3筆）
  ↓
更新庫存（3筆 Update）
  ↓
COMMIT 成功
```

---

🟦 Part 27：Stored Procedure — 三商品訂單建立（spCreateOrderMulti）

此流程是企業級後端常用的訂單建立邏輯，包含：

✔ 多商品存在驗證
✔ 多商品庫存扣除
✔ 小計計算
✔ 訂單總額 + 折扣邏輯（CASE WHEN）
✔ Orders（訂單主檔）
✔ OrderDetails（多筆訂單明細）
✔ TRY / CATCH + TRAN 交易
✔ OUTPUT 回傳新建立 OrderId

1. 參數（輸入＋輸出）
```sql
Stored Procedure 接收：

顧客 Id

三個商品 Id、三個數量

回傳：新訂單 Id（OUTPUT）

CREATE PROCEDURE spCreateOrderMulti
	@CustomerId INT,
	@ProductId1 INT, @Quantity1 INT,
	@ProductId2 INT, @Quantity2 INT,
	@ProductId3 INT, @Quantity3 INT,
	@NewOrderId INT OUTPUT
AS
BEGIN
```
2. 宣告此流程需要的變數

每個商品皆需要：

庫存

單價

小計（單價 × 數量）

新庫存
```sql
DECLARE @Stock1 INT, @Stock2 INT, @Stock3 INT;
DECLARE @NewStock1 INT, @NewStock2 INT, @NewStock3 INT;
DECLARE @UnitPrice1 DECIMAL(10,2), @UnitPrice2 DECIMAL(10,2), @UnitPrice3 DECIMAL(10,2);
DECLARE @Subtotal1 DECIMAL(10,2), @Subtotal2 DECIMAL(10,2), @Subtotal3 DECIMAL(10,2);
DECLARE @OrderAmount DECIMAL(10,2), @FinalAmount DECIMAL(10,2);
```
3. TRY / CATCH + TRAN（交易保護）

任何錯誤 → 全部回滾，不會扣到一半的庫存。
```sql
BEGIN TRY
	BEGIN TRAN;
```
4. 顧客是否存在（NOT EXISTS 防呆）
```sql
IF NOT EXISTS(
	SELECT 1 FROM Customers WHERE CustomerId = @CustomerId)
BEGIN
	PRINT '顧客不存在';
	ROLLBACK;
	RETURN;
END
```
5. 三商品存在性檢查（逐一檢查）
```sql
IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId1)
BEGIN PRINT '商品1不存在'; ROLLBACK; RETURN; END

IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId2)
BEGIN PRINT '商品2不存在'; ROLLBACK; RETURN; END

IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId3)
BEGIN PRINT '商品3不存在'; ROLLBACK; RETURN; END
```
6. 抓取三商品庫存與單價
```sql
SELECT @Stock1 = Stock, @UnitPrice1 = Price
FROM Products WHERE ProductId = @ProductId1;

SELECT @Stock2 = Stock, @UnitPrice2 = Price
FROM Products WHERE ProductId = @ProductId2;

SELECT @Stock3 = Stock, @UnitPrice3 = Price
FROM Products WHERE ProductId = @ProductId3;
```
7. 計算每個商品的新庫存（扣除數量）
```sql
SET @NewStock1 = @Stock1 - @Quantity1;
SET @NewStock2 = @Stock2 - @Quantity2;
SET @NewStock3 = @Stock3 - @Quantity3;
```
8. 庫存不足 → 結束交易
```sql
IF @NewStock1 < 0 BEGIN PRINT '商品1庫存不足'; ROLLBACK; RETURN; END
IF @NewStock2 < 0 BEGIN PRINT '商品2庫存不足'; ROLLBACK; RETURN; END
IF @NewStock3 < 0 BEGIN PRINT '商品3庫存不足'; ROLLBACK; RETURN; END
```
9. 計算小計與訂單總額
```sql
SET @Subtotal1 = @UnitPrice1 * @Quantity1;
SET @Subtotal2 = @UnitPrice2 * @Quantity2;
SET @Subtotal3 = @UnitPrice3 * @Quantity3;

SET @OrderAmount = @Subtotal1 + @Subtotal2 + @Subtotal3;
```
10. 折扣邏輯（CASE WHEN）
```sql
SET @FinalAmount = 
	CASE
		WHEN @OrderAmount >= 30000 THEN @OrderAmount * 0.8
		WHEN @OrderAmount >= 20000 THEN @OrderAmount * 0.9
		WHEN @OrderAmount >= 10000 THEN @OrderAmount * 0.95
		ELSE @OrderAmount
	END;
```
11. 新增訂單主檔（Orders）
```sql
INSERT INTO Orders(CustomerId, Amount)
VALUES(@CustomerId, @FinalAmount);

SET @NewOrderId = SCOPE_IDENTITY
```
12. 新增三筆訂單明細（OrderDetails）
```sql
INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice, Subtotal)
VALUES(@NewOrderId, @ProductId1, @Quantity1, @UnitPrice1, @Subtotal1);

INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice, Subtotal)
VALUES(@NewOrderId, @ProductId2, @Quantity2, @UnitPrice2, @Subtotal2);

INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice, Subtotal)
VALUES(@NewOrderId, @ProductId3, @Quantity3, @UnitPrice3, @Subtotal3);
```
13. 更新三商品庫存
```sql
UPDATE Products SET Stock = @NewStock1 WHERE ProductId = @ProductId1;
UPDATE Products SET Stock = @NewStock2 WHERE ProductId = @ProductId2;
UPDATE Products SET Stock = @NewStock3 WHERE ProductId = @ProductId3;
```
14. COMMIT（成功）或 ROLLBACK（失敗）
```sql
COMMIT;
PRINT 'TR5完成，訂單建立成功！ OrderId = ' 
	+ CAST(@NewOrderId AS NVARCHAR(20));

END TRY
BEGIN CATCH
	ROLLBACK;
	PRINT '新增失敗，全數回滾';
	PRINT ERROR_MESSAGE();
END CATCH;
END
```
