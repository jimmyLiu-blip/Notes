# SQL基礎 + Stored Procedure 實務應用全攻略

## 📑 目錄
- [Part 1: 資料庫基礎操作](#part-1-資料庫基礎操作)
- [Part 2: 資料表管理](#part-2-資料表管理)
- [Part 3: 基本CRUD操作](#part-3-基本crud操作)
- [Part 4: Stored Procedure基礎](#part-4-stored-procedure基礎)
- [Part 5: SP進階功能](#part-5-sp進階功能)
- [Part 6: 交易與錯誤處理](#part-6-交易與錯誤處理)
- [Part 7: 實務應用案例](#part-7-實務應用案例)

---

## Part 1: 資料庫基礎操作

### 1. 切換資料庫
```sql
USE 資料庫名稱;
```
**作用**: 所有後續語句都會在該 DB 執行。

---

## Part 2: 資料表管理

### 2. 建立基本表格
```sql
CREATE TABLE Countries (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Population BIGINT NULL
);
```

### 3. 插入資料
```sql
INSERT INTO Countries(Name, Population) VALUES
('Taiwan', 23500000),
('Japan', 126000000),
('USA', 331000000),
('Estonia', 1330000);
```
**注意**: 欄位順序必須與 VALUES 對應。

### 4. 欄位預設值(DEFAULT)

#### 4.1 建立表格時內建 DEFAULT
```sql
CREATE TABLE Products(
    ProductId INT IDENTITY(1,1) PRIMARY KEY,
    ProductName NVARCHAR(100) NOT NULL,
    Price DECIMAL(10,2) NOT NULL DEFAULT 0,
    Stock INT NOT NULL DEFAULT 0,
    CreateDate DATETIME NOT NULL DEFAULT GETDATE()
);
```

#### 4.2 使用 ALTER 新增 DEFAULT
```sql
ALTER TABLE Products
ADD CONSTRAINT DF_Products_CreateDate 
DEFAULT GETDATE() FOR CreateDate;
```

#### 4.3 修改或刪除 Default Constraint
**新增**:
```sql
ALTER TABLE Products
ADD CONSTRAINT DF_Products_Price 
DEFAULT 2000 FOR Price;
```

**刪除**:
```sql
ALTER TABLE Products
DROP CONSTRAINT DF_Products_Price;
```
⭐ **要改 DEFAULT → 一定要 DROP 再 ADD**

### 5. 補上主鍵/外鍵

#### 補上 Primary Key
```sql
ALTER TABLE Customers
ADD CONSTRAINT PK_Customers PRIMARY KEY (CustomerId);
```

#### 補上 Foreign Key
```sql
ALTER TABLE Orders 
ADD CONSTRAINT FK_Orders_Customers 
FOREIGN KEY (CustomerId)
REFERENCES Customers(CustomerId);
```

### 6. 刪除資料表
```sql
DROP TABLE IF EXISTS Orders;
```

---

## Part 3: 基本CRUD操作

### 7. 刪除資料
```sql
DELETE FROM Products
WHERE ProductName = 'Pixel10';
```

### 8. TOP + ORDER BY 查詢最大/最小值
```sql
SELECT TOP(1) ProductName
FROM Products
ORDER BY Stock DESC;
```

### 9. JOIN + GROUP BY 分析查詢
查每位客戶訂單總額:
```sql
SELECT 
    C.CustomerId, 
    C.CustomerName, 
    SUM(O.Amount) AS TotalAmount
FROM Customers C
JOIN Orders O ON C.CustomerId = O.CustomerId
GROUP BY C.CustomerId, C.CustomerName;
```

---

## Part 4: Stored Procedure基礎

### 10. 建立基本 SP - 新增客戶
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

**執行**:
```sql
EXEC spAddCustomer
    @CustomerName = N'小小兵',
    @Phone = '09123456789';
```

### 11. Update 型 SP
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

### 12. 刪除 Stored Procedure
```sql
DROP PROCEDURE IF EXISTS spAddOrder;
```

---

## Part 5: SP進階功能

### 13. 使用 ALTER 修改既有 SP ⭐重大技能
**用途**: 當 SP 已被建立,但需要補上條件、修正錯誤、加入防呆時。

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

⭐ **重點**:
- CREATE 只能用一次
- 修改 SP 全用 ALTER PROCEDURE
- 企業專案中 SP 會不斷調整 → 一定會用到

### 14. 新增前的檢查(NOT EXISTS)
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

### 15. OUTPUT 參數概念

**用途**: OUTPUT 用來讓 SP 回傳單一值(不是資料表)。

**範例**:
```sql
CREATE PROCEDURE spReturnNumber
    @Result INT OUTPUT
AS
BEGIN
    SET @Result = 168;
END;
GO

-- 呼叫
DECLARE @MyNum INT;
EXEC spReturnNumber @Result = @MyNum OUTPUT;
SELECT @MyNum AS ReturnedValue;
```

⭐ **重點**: OUTPUT 一定要在定義與執行時都寫

### 16. 新增資料後回傳新 Id (OUTPUT + SCOPE_IDENTITY)

#### 新增商品並回傳 Id
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

#### 新增訂單並回傳 OrderId
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

**呼叫方式**:
```sql
DECLARE @Id INT;
EXEC spAddOrderWithId 
    @CustomerId = 5, 
    @Amount = 66666, 
    @NewOrderId = @Id OUTPUT;
SELECT @Id AS NewOrderId;
```

⭐ **重點**:
- SCOPE_IDENTITY() → 取得同一 Scope 的自動編號
- 不要用 @@IDENTITY
- 常用於 Web API 新增資料後回前端

### 17. 查詢型 SP - 透過 JOIN 查詢資料
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

⭐ **重點**:
- 使用 LEFT JOIN → 即使沒訂單也可查到客戶
- 查詢型 SP 是企業系統最常使用的一種 SP
- 通常會搭配條件、排序、分頁

### 18. 查詢前 N 名(TOP(@N) + ORDER BY)
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

⭐ **重點**:
- TOP(@N) 支援變數 → 動態查詢
- 必須搭配 ORDER BY
- SUM + GROUP BY 是基本分析查詢

### 19. 彈性搜尋 - 可選條件(搜尋表單功能)
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

⭐ **重點**(企業常用):
- 彈性搜尋 = 每一欄都可填或不填
- WHERE 條件須寫成 (@Param IS NULL OR …)
- 這種 SP 幾乎每個企業後台都會用

### 20. 部分更新(COALESCE 技巧)
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
        Phone = COALESCE(@Phone, Phone)
    WHERE CustomerId = @CustomerId; 

    PRINT '更新成功'; 
END;
```

⭐ **重點**:
- COALESCE(a, b) → 若 a 不為 NULL 用 a,否則用 b
- 適合用於後台管理介面「只改幾個欄位」

### 21. 刪除前檢查(NOT EXISTS + EXISTS 防呆)
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
        PRINT '顧客仍有訂單,不能刪除';
        RETURN;
    END

    DELETE FROM Customers
    WHERE CustomerId = @CustomerId;

    PRINT '刪除成功';
END;
```

⭐ **重點**:
- 必學實務技巧(避免刪除造成外鍵錯誤)
- 很多公司 SP 都會用這種「三段式防呆」

---

## Part 6: 交易與錯誤處理

### 22. 交易(Transaction)基本概念

**交易** = 多個 SQL 必須一起成功 或 一起失敗。
否則就回復(ROLLBACK)保持資料乾淨。

#### 基本語法
```sql
BEGIN TRAN;
...
COMMIT;  -- 全部成功
```

如果有錯誤:
```sql
ROLLBACK;  -- 全部取消
```

### 23. 例外處理 TRY / CATCH
```sql
BEGIN TRY
    -- 可能會錯的 SQL
END TRY
BEGIN CATCH
    -- 錯誤發生後要做什麼?
END CATCH
```

### 24. TRY + CATCH + TRAN 完整模板(必背!)
```sql
BEGIN TRY
    BEGIN TRAN;

    -- 可能會失敗的 SQL

    COMMIT;  -- 全部成功
END TRY
BEGIN CATCH
    ROLLBACK; -- 發生錯誤 → 全部取消

    PRINT '發生錯誤';
    PRINT ERROR_MESSAGE();
END CATCH;
```

#### 牛刀小試
```sql
BEGIN TRY
    BEGIN TRAN;

    PRINT '➡ 開始新增客戶...';
    INSERT INTO Customers(CustomerName, Phone)
    VALUES (N'測試流程客戶', '0911000222');
    PRINT '✓ 客戶新增成功';

    PRINT '➡ 開始新增訂單...';
    INSERT INTO Orders(CustomerId, Amount)
    VALUES (99999, 50000);  -- 故意放錯,觸發外鍵錯誤
    PRINT '✓ 訂單新增成功';

    COMMIT;
    PRINT '🎉 交易成功,全部完成!';
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT '❌ 發生錯誤,已回復資料';
    PRINT ERROR_MESSAGE();
END CATCH;
```

### 25. 交易實戰 - 新增訂單(含驗證)
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
    PRINT '發生錯誤,已回復資料';
    PRINT ERROR_MESSAGE();
END CATCH;
```

---

## Part 7: 實務應用案例

### 案例1: 扣庫存(Inventory Deduction)

**用途**: 扣庫存是電商、倉儲、超商系統最基礎的商業邏輯。

#### 流程步驟

**① 檢查商品是否存在**
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

**② 讀取目前庫存**
```sql
SELECT @Stock = Stock
FROM Products
WHERE ProductId = @ProductId;
```

**③ 判斷庫存是否足夠**
```sql
IF @Stock < @Quantity
BEGIN
    PRINT '庫存不足';
    ROLLBACK;
    RETURN;
END
```

**④ 計算新庫存**
```sql
SET @NewStock = @Stock - @Quantity;
```

**⑤ 更新庫存**
```sql
UPDATE Products 
SET Stock = @NewStock
WHERE ProductId = @ProductId;
```

#### 完整扣庫存 SQL
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
    PRINT '扣庫存成功!剩餘庫存 = ' + CAST(@NewStock AS NVARCHAR(50))
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT ERROR_MESSAGE();
END CATCH;
```

⭐ **重點整理**:
- ✓ 交易確保庫存不會扣一半
- ✓ NOT EXISTS 必備
- ✓ 讀取庫存必須先存入變數
- ✓ 庫存不足 → 立即中止
- ✓ TRY / CATCH 保護整個扣庫存流程

---

### 案例2: 退貨(Return Inventory)

**用途**: 退貨流程是電商、倉儲、POS中必備的商業邏輯。

#### 完整流程(五步驟)

**① 商品存在檢查**
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

**② 避免不合法退貨(不可 <= 0)**
```sql
IF @Quantity <= 0
BEGIN
    PRINT '商品不可退負數';
    ROLLBACK;
    RETURN;
END
```

**③ 讀取目前庫存**
```sql
SELECT @Stock = Stock
FROM Products
WHERE ProductId = @ProductId;
```

**④ 計算新庫存**
```sql
SET @NewStock = @Stock + @Quantity;
```

**⑤ 更新庫存**
```sql
UPDATE Products
SET Stock = @NewStock
WHERE ProductId = @ProductId;
```

#### 完整 SQL
```sql
DECLARE @ProductId INT = 5;
DECLARE @Quantity INT = 6;
DECLARE @Stock INT;
DECLARE @NewStock INT;

BEGIN TRY
    BEGIN TRAN;

    -- ① 商品存在?
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

    COMMIT;
    PRINT '退貨成功!新的庫存 = ' + CAST(@NewStock AS NVARCHAR(20));
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT ERROR_MESSAGE();
END CATCH;
```

---

### 案例3: 建立訂單+扣庫存+訂單明細

**用途**: 商品 → 扣庫存 → 建立訂單 → 建立訂單明細的完整交易流程。

#### 完整邏輯流程(7個步驟)

**① 檢查商品是否存在**
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

**② 讀取庫存與單價**
```sql
SELECT 
    @Stock = Stock,
    @UnitPrice = Price
FROM Products
WHERE ProductId = @ProductId;
```

**③ 判斷庫存是否足夠**
```sql
IF @Stock < @Quantity
BEGIN 
    PRINT '庫存不足';
    ROLLBACK;
    RETURN;
END
```

**④ 扣庫存**
```sql
SET @NewStock = @Stock - @Quantity;

UPDATE Products
SET Stock = @NewStock 
WHERE ProductId = @ProductId;
```

**⑤ 計算訂單金額**
```sql
SET @OrderAmount = @UnitPrice * @Quantity;
```

**⑥ 建立 Orders 資料並取得 OrderId**
```sql
INSERT INTO Orders(CustomerId, Amount)
VALUES(@CustomerId, @OrderAmount);

SET @OrderId = SCOPE_IDENTITY();
```

**⑦ 建立訂單明細 OrderDetails**
```sql
INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
VALUES (@OrderId, @ProductId, @Quantity, @UnitPrice);
```

#### 完整 SQL
```sql
DECLARE @CustomerId INT = 3;
DECLARE @ProductId INT = 2;
DECLARE @Quantity INT = 4;
DECLARE @Stock INT;
DECLARE @NewStock INT;
DECLARE @UnitPrice DECIMAL(10,2);
DECLARE @OrderAmount DECIMAL(10,2);
DECLARE @OrderId INT;

BEGIN TRY
    BEGIN TRAN;

    -- 1) 確認商品存在
    IF NOT EXISTS(
        SELECT 1
        FROM Products
        WHERE ProductId = @ProductId)
    BEGIN 
        PRINT '商品不存在';
        ROLLBACK;
        RETURN;
    END

    -- 2) 取得庫存與單價
    SELECT 
        @Stock = Stock,
        @UnitPrice = Price
    FROM Products
    WHERE ProductId = @ProductId;

    -- 3) 確認庫存
    IF @Stock < @Quantity
    BEGIN 
        PRINT '庫存不足';
        ROLLBACK;
        RETURN;
    END

    -- 4) 更新庫存
    SET @NewStock = @Stock - @Quantity;
    UPDATE Products
    SET Stock = @NewStock 
    WHERE ProductId = @ProductId;

    -- 5) 建立訂單
    SET @OrderAmount = @UnitPrice * @Quantity;
    INSERT INTO Orders(CustomerId, Amount)
    VALUES(@CustomerId, @OrderAmount);

    -- 6) 取得 OrderId
    SET @OrderId = SCOPE_IDENTITY();

    -- 7) 新增訂單明細
    INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
    VALUES (@OrderId, @ProductId, @Quantity, @UnitPrice);

    COMMIT;
    PRINT '完成!新增訂單成功,OrderId = ' + CAST(@OrderId AS NVARCHAR(20));
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT '發生錯誤,交易已回復';
    PRINT ERROR_MESSAGE();
END CATCH;
```

---

### 案例4: 多商品訂單(一次購買兩項商品)

**用途**: 同一張訂單,購買兩個商品的完整企業級交易流程。

#### 多商品交易流程(總共 9 步)

**① 確認客戶存在**
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

**② & ③ 商品存在驗證**
```sql
IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId1)
BEGIN PRINT '商品1不存在'; ROLLBACK; RETURN; END

IF NOT EXISTS(SELECT 1 FROM Products WHERE ProductId = @ProductId2)
BEGIN PRINT '商品2不存在'; ROLLBACK; RETURN; END
```

**④ 一次讀取庫存 + 單價**
```sql
SELECT @Stock1 = Stock, @UnitPrice1 = Price
FROM Products WHERE ProductId = @ProductId1;

SELECT @Stock2 = Stock, @UnitPrice2 = Price
FROM Products WHERE ProductId = @ProductId2;
```

**⑤ 庫存是否足夠?**
```sql
IF @Stock1 < @Quantity1
BEGIN PRINT '商品1庫存不足'; ROLLBACK; RETURN; END

IF @Stock2 < @Quantity2
BEGIN PRINT '商品2庫存不足'; ROLLBACK; RETURN; END
```

**⑥ 扣庫存**
```sql
UPDATE Products
SET Stock = @Stock1 - @Quantity1
WHERE ProductId = @ProductId1;

UPDATE Products
SET Stock = @Stock2 - @Quantity2
WHERE ProductId = @ProductId2;
```

**⑦ 建立訂單(Orders 只有一筆)**
```sql
SET @OrderAmount = @Quantity1 * @UnitPrice1
                 + @Quantity2 * @UnitPrice2;

INSERT INTO Orders(CustomerId, Amount)
VALUES (@CustomerId, @OrderAmount);

SET @OrderId = SCOPE_IDENTITY();
```

**⑧ & ⑨ 建立訂單明細(兩筆 OrderDetails)**
```sql
INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
VALUES (@OrderId, @ProductId1, @Quantity1, @UnitPrice1);

INSERT INTO OrderDetails(OrderId, ProductId, Quantity, UnitPrice)
VALUES (@OrderId, @ProductId2, @Quantity2, @UnitPrice2);
```

#### 完整 SQL
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
        SELECT 1 FROM Customers
        WHERE CustomerId = @CustomerId)
    BEGIN
        PRINT '此顧客ID不存在';
        ROLLBACK;
        RETURN;
    END

    -- ② 商品1存在?
    IF NOT EXISTS(
        SELECT 1 FROM Products
        WHERE ProductId = @ProductId1)
    BEGIN
        PRINT '商品1不存在';
        ROLLBACK;
        RETURN;
    END

    -- ③ 商品2存在?
    IF NOT EXISTS(
        SELECT 1 FROM Products
        WHERE ProductId = @ProductId2)
    BEGIN
        PRINT '商品2不存在';
        ROLLBACK;
        RETURN;
    END

    -- ④ 取得庫存與單價
    SELECT @Stock1 = Stock, @UnitPrice1 = Price
    FROM Products WHERE ProductId = @ProductId1;

    SELECT @Stock2 = Stock, @UnitPrice2 = Price
    FROM Products WHERE ProductId = @ProductId2;

    -- ⑤ 庫存足夠?
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

    -- ⑦ 建立訂單(1 筆)
    SET @OrderAmount = @Quantity1 * @UnitPrice1
                     + @Quantity2 * @UnitPrice2;

    INSERT INTO Orders(CustomerId, Amount)
    VALUES (@CustomerId, @OrderAmount);

    SET @OrderId = SCOPE_IDENTITY();

