🟦 SQL CASE WHEN 全攻略筆記

---

📘 1. CASE WHEN 基礎語法
```sql
CASE 是 SQL 中的「條件判斷」，效果等同 IF / ELSE IF。
```
✔ 基本語法
```sql
CASE
    WHEN 條件1 THEN 結果1
    WHEN 條件2 THEN 結果2
    ELSE 結果3
END
```

CASE 一定要搭配 END 結束。

---

📘 2. SELECT 中使用 CASE（最常用）
📝 用訂單金額分類會員等級
```sql
SELECT 
	Amount,
	CASE
		WHEN Amount >= 30000 THEN 'VIP'
		WHEN Amount >= 10000 THEN 'GOLD'
		ELSE 'Normal'
	END AS CustomerLevel
FROM Orders;
```

Output 會顯示金額與等級對應。

---

📘 3. CASE 在 UPDATE 裡使用
📝 根據金額自動給折扣（寫回資料庫）
```sql
UPDATE Orders
SET DiscountRate =
	CASE
		WHEN Amount >= 30000 THEN 0.90
		WHEN Amount >= 10000 THEN 0.95
		ELSE 1.00
	END;
```

✔ 可直接用 CASE 設定欄位值
✔ 無需再用多個 UPDATE 語句，效率更高

---

📘 4. CASE WHEN 可搭配數值運算

例如：折扣後金額計算
```sql
SELECT 
	Amount,
	CASE
		WHEN Amount >= 20000 THEN Amount * 0.9
		WHEN Amount >= 10000 THEN Amount * 0.95
		ELSE Amount
	END AS FinalAmount
FROM Orders;
```

✔ CASE 的 THEN 可以是數值運算
✔ CASE 的結果也能再命名（AS FinalAmount）

---

📘 5. CASE WHEN 可以用在 ORDER BY（排序規則）

例如：VIP > GOLD > Normal 排序：
```sql
SELECT CustomerName, Amount
FROM Orders
ORDER BY
	CASE
		WHEN Amount >= 30000 THEN 1
		WHEN Amount >= 10000 THEN 2
		ELSE 3
	END;
```

✔ SQL 無法直接依文字排序（VIP → GOLD → Normal）
✔ 但可以用 CASE 手動指定排序優先級

---

📘 6. CASE WHEN 巢狀結構（少用但可讀性高）

你也可以在 THEN 內再寫 CASE：

```sql
CASE 
	WHEN Amount >= 20000 THEN 
		CASE 
			WHEN Amount >= 50000 THEN '超VIP'
			ELSE 'VIP'
		END
	WHEN Amount >= 10000 THEN 'GOLD'
	ELSE 'NORMAL'
END
```

但這種寫法可讀性較差，不推薦日常使用。

📘 7. CASE WHEN 實務應用案例列表（企業最常用）

以下都是後端工程師會用到的情境：

✔ 案例 1：客戶等級分類
```sql
CASE 
	WHEN Amount >= 30000 THEN 'VIP'
	WHEN Amount >= 10000 THEN 'GOLD'
	ELSE 'Normal'
END
```
✔ 案例 2：庫存狀態顯示（紅黃綠燈）
```sql
CASE
	WHEN Stock = 0 THEN 'Out of Stock'
	WHEN Stock < 10 THEN 'Low'
	ELSE 'In Stock'
END
```

✔ 案例 3：訂單狀態（0 = 處理中，1 = 完成）
```sql
CASE Status
	WHEN 0 THEN 'Processing'
	WHEN 1 THEN 'Finished'
	ELSE 'Unknown'
END
```

✔ 案例 4：滿額折扣（你寫過的）
```sql
CASE
	WHEN OrderAmount >= 20000 THEN OrderAmount * 0.90
	WHEN OrderAmount >= 10000 THEN OrderAmount * 0.95
	ELSE OrderAmount
END
```

✔ 案例 5：分類統計（SQL 分析常用）
```sql
SELECT 
	SUM(CASE WHEN Amount >= 30000 THEN 1 ELSE 0 END) AS VIPCount,
	SUM(CASE WHEN Amount >= 10000 THEN 1 ELSE 0 END) AS GoldCount
FROM Orders;
```

✔ CASE 可以放在 SUM / COUNT 內
✔ 這種寫法是「企業 BI 報表」會用的

📘 8. CASE WHEN 寫法注意事項（重要）
項目	說明
```sql
CASE 會逐條執行	第一個符合的條件就結束
順序很重要	越大的條件越要往前放
THEN 的值必須型別一致	不可：文字 + 數字混用
ELSE 建議一定寫	避免回傳 NULL
```

📘 9. CASE WHEN 使用技巧（讓你更像專業工程師）
✔ 技巧 1：條件由大到小寫

避免小條件先命中：
```sql

WHEN Amount >= 30000  → 要放前面
WHEN Amount >= 10000  → 放後面
```

✔ 技巧 2：用 CASE 通常比 IF 寫在 SP 裡更好
```sql
CASE 可以用在 SELECT、UPDATE、ORDER BY、SUM 中
IF 不行。
```

✔ 技巧 3：可與變數 SET 搭配（你已經在用）
```sql
SET @Final =
	CASE ...
```

🎯 小結 — 你需要記住的核心句子

CASE WHEN = SQL 的 if / else，使用最廣，最靈活，也是後端必會語法。