# 📘 SQLZOO 學習筆記整理  
> 日期：2025-10-21  
> 主題：SQL 基礎語法與實務邏輯練習

---

## 🧩 一、基本查詢語法

### SELECT 與 WHERE
```sql
SELECT name, population
FROM world
WHERE population = 64000;

SELECT：選取要顯示的欄位

FROM：指定資料表

WHERE：設定篩選條件
```
--------


## 二、邏輯與條件判斷

### 比較與邏輯運算
```sql
SELECT name, area, population
FROM world
WHERE area > 3000000 OR population > 250000000;


AND：同時成立

OR：任一成立

XOR：僅一個成立（排除同時成立）

NOT：否定條件
```
--------

## 🔢 三、數值與單位運算
```sql
SELECT name, area * 2 AS double_area
FROM world;


AS：為輸出欄位取別名

可搭配 /、*、+、- 進行運算

```
--------

## 📋 四、篩選多值或範圍

### IN / BETWEEN
```sql
SELECT name, population
FROM world
WHERE name IN ('France', 'Germany', 'Italy');

SELECT name
FROM world
WHERE population BETWEEN 1000000 AND 9999999;


IN：在特定清單內，要搭配多值

BETWEEN：介於兩數之間（含上下限）

```
--------

## 🔍 五、字串處理

### LIKE 模糊搜尋
```sql
SELECT name
FROM world
WHERE name LIKE '%United%';


%：任意長度字串

_：單一字元

用於搜尋包含特定字的資料

CONCAT 拼接：
SELECT name
FROM world
WHERE capital = CONCAT(name, ' City');


將多個字串組合成一個

REPLACE 字串替換：
SELECT name, REPLACE(capital, name, '') AS extension
FROM world
WHERE capital LIKE CONCAT(name, '%');


移除國家名稱以取得延伸部分（例："Monaco-Ville" → "-Ville"）

LEFT / MID 函數：
SELECT LEFT(capital, 1) AS first_letter
FROM world;


LEFT(str, n)：取字串左邊 n 個字

MID(str, start, length)：從指定位置取字串
ex：
SELECT MID('Mexico City', 1, 6);
Mexico

LEFT(str, n)	從左邊取 n 個字元
RIGHT(str, n)	從右邊取 n 個字元
MID(str, start, length)	從任意位置開始取字串
LENGTH(str)	回傳字串長度（幾個字）

```
--------

## 💰 六、ROUND 四捨五入與單位轉換

```sql
SELECT name,
       ROUND(population / 1000000, 2) AS population_in_millions,
       ROUND(gdp / 1000000000, 2) AS gdp_in_billions
FROM world
WHERE continent = 'South America';


ROUND(value, 2)：保留 2 位小數

ROUND(value, -3)：取整到千位

搭配 /1000000、/1000000000 做單位轉換（百萬 / 十億）

```
--------

## 🧠 七、排除雙重條件（XOR）

```sql
SELECT name, population, area
FROM world
WHERE (area > 3000000) XOR (population > 250000000);


僅符合其中之一的條件會被選出

可用於「只要面積大或人口多，但不能同時成立」的情境

```
--------
## 🧾 八、綜合應用（人均 GDP）
```sql
SELECT name,
       ROUND(gdp / population, -3) AS per_capita_gdp
FROM world
WHERE gdp >= 1000000000000;


gdp / population：人均 GDP

ROUND(..., -3)：四捨五入至千位（最接近 $1000）
```
--------

## 九、布林排序（Boolean Sorting）
```sql
SELECT winner, subject
FROM nobel
WHERE yr = 1984
ORDER BY (subject IN ('Physics','Chemistry')), subject, winner;

(subject IN ('Physics','Chemistry')) → 對於 Chemistry / Physics 回傳 1，其他科目為 0

ORDER BY 會先依 0 → 1 排序，因此「非 Chemistry / Physics」會先出現

可視為「自訂排序優先權」的方式

```
--------

## 十、CASE WHEN 條件分類
```sql
SELECT name,
       CASE
           WHEN population > 100000000 THEN 'Large'
           WHEN population > 1000000 THEN 'Medium'
           ELSE 'Small'
       END AS size_category
FROM world;

💡說明：

CASE WHEN 可根據條件輸出不同分類

結尾必須加上 END

可搭配 AS 為新欄位命名

📌 常用於：

分級顯示（人口、面積、GDP）

客製化標籤分類

ORDER BY 自訂排序優先級
```
--------


## 🔤 十一、字串引號處理（Apostrophe Escaping）

文字中若含有 '（單引號），需用兩個連續單引號 ' '
```sql
SELECT *
FROM nobel
WHERE winner = 'EUGENE O''NEILL';

💡說明：

'EUGENE O''NEILL' → SQL 會正確解析為文字 EUGENE O’NEILL

第一個 ' 表示字串開始，第二個 ' 是轉義用，不會被當成結束符號。
```
--------

## 十二、模糊搜尋與字首比對（Sir 範例）
```sql
SELECT winner, yr, subject
FROM nobel
WHERE winner LIKE 'Sir%'
ORDER BY yr DESC, winner ASC;

💡說明：

LIKE 'Sir%' → 找出以 Sir 開頭的名字

% 表示任意長度字串（零個或多個字元）

ORDER BY yr DESC → 由新到舊

, winner ASC → 同一年依名字升冪排列
```
--------

## 十三、組合條件查詢（不同年份/主題）
```sql
SELECT yr, subject, winner
FROM nobel
WHERE (subject='Physics' AND yr=1980)
   OR (subject='Chemistry' AND yr=1984);

   💡說明：

AND 與 OR 可以搭配括號組合多組條件

適用於「年份＋科目」組合條件查詢

常見錯誤：誤用 AND 導致無法同時滿足不同組合條件
```
--------

## 🔡 十四、字首與字尾搜尋
```sql
SELECT winner
FROM nobel
WHERE winner LIKE 'John%';   -- 以 John 開頭

💡說明：

%：任意字串（可為空）

_：任意單一字元

若要匹配結尾，使用 LIKE '%John'
```
--------

## 🧩 十五、IN 與 NOT IN 條件用法延伸
```sql
SELECT name
FROM world
WHERE continent NOT IN ('Europe', 'Asia');
💡說明：

IN：多值匹配

NOT IN：排除特定集合
```
--------

## 十六、布林條件作為數值運算（延伸概念）
```sql
SELECT subject,
       (subject IN ('Physics','Chemistry')) AS is_science
FROM nobel;

結果：

subject	is_science
Physics	1
Chemistry	1
Literature	0

在 SQL 中布林值可直接被當作 0（FALSE）/ 1（TRUE） 使用
可應用於排序、統計、條件分類等場景。
```
--------

## 🧾 十七、綜合題實例（進階應用）
🧪 題目：列出 1984 年所有得主，並讓 Chemistry、Physics 排最後
```sql
SELECT winner, subject
FROM nobel
WHERE yr = 1984
ORDER BY (subject IN ('Physics','Chemistry')),
         subject, winner;

🏆 題目：列出名字中包含單引號 ' 的得主（如 Eugene O’Neill）
SELECT *
FROM nobel
WHERE winner = 'EUGENE O''NEILL';

🧑‍🎓 題目：列出名字以 Sir 開頭的得主，依年份新到舊排列
SELECT winner, yr, subject
FROM nobel
WHERE winner LIKE 'Sir%'
ORDER BY yr DESC, winner;
```
--------

🔍 十八、SQL 關鍵字快速回顧
類別	指令	功能
查詢	SELECT	選擇要顯示的欄位
篩選	WHERE	設定條件
排序	ORDER BY	資料排序，可搭配 ASC / DESC
多值	IN / NOT IN	檢查是否包含於清單內
模糊	LIKE	模糊比對（% 任意字串、_ 單一字）
範圍	BETWEEN	範圍篩選（含上下限）
替換	REPLACE()	字串替換
拼接	CONCAT()	合併字串
四捨五入	ROUND()	取指定位數小數
條件分類	CASE WHEN	條件式輸出對應值
特殊字元	''（兩個單引號）	在字串中表示 ' 符號
---

## 十九、進階查詢語法與邏輯
```sql
📊 一、GROUP BY 分組與聚合函數 (Aggregate Functions)

SELECT continent, MAX(population) AS max_pop, MIN(area) AS min_area
FROM world
GROUP BY continent;

「從 world 這個表格中，選出每一個 continent (洲)，並計算出在該洲內，最高的 population (人口數) 是多少，以及最小的 area (面積) 是多少。」

💡 說明：
函數	功能
MAX(column)	取該組中最大值
MIN(column)	取該組中最小值
SUM(column)	組內總和
AVG(column)	組內平均值
COUNT(*)	計算該組筆數

📌 使用 GROUP BY 時，SELECT 內只能出現：

被分組的欄位

聚合函數 (MAX、MIN、SUM、AVG、COUNT)

🔍 二、HAVING：分組後的篩選條件

SELECT continent
FROM world
GROUP BY continent
HAVING MAX(population) <= 25000000;

💡 說明：

WHERE：分組之前的篩選（針對單筆資料）

HAVING：分組之後的篩選（針對整組結果）

常搭配聚合函數（MAX, AVG, SUM…）

📘 範例：找出「所有國家人口皆小於等於 2500 萬」的洲

SELECT name, continent, population
FROM world
WHERE continent IN (
    SELECT continent
    FROM world
    GROUP BY continent
    HAVING MAX(population) <= 25000000
);

🧩 三、子查詢 (Subquery)
1️⃣ 概念說明

子查詢是「一個 SELECT 包在另一個 SELECT 裡」。
用於比較、篩選或計算某個動態值。

SELECT name, population
FROM world
WHERE population > (
    SELECT population
    FROM world
    WHERE name = 'France'
);


💡 說明：

內層查詢先執行，取得 France 的人口數。

外層查詢再找出所有人口大於 France 的國家。

2️⃣ 巢狀查詢（巢中巢）

可多層嵌套，例如：

SELECT name
FROM world
WHERE population > (
    SELECT AVG(population)
    FROM world
    WHERE continent = (
        SELECT continent
        FROM world
        WHERE name = 'China'
    )
);


👉 找出「中國所在洲」中，人口高於平均的國家。

⚖️ 四、IN、NOT IN、EXISTS、NOT EXISTS 的比較

關鍵字	功能說明	範例

IN	比對是否屬於子查詢清單	
WHERE continent IN (
   SELECT continent 
   FROM world 
   WHERE name='China')

NOT IN	排除子查詢清單	
WHERE continent NOT IN (...)

EXISTS	若子查詢有任何結果則為真	
WHERE EXISTS (
   SELECT 1 
   FROM world w2 
   WHERE w2.continent=w1.continent)

NOT EXISTS	若子查詢無結果則為真	WHERE NOT EXISTS (...)

💡 範例：
找出「整個洲內沒有任何國家人口超過 2500 萬」的洲：

SELECT name, continent
FROM world w1
WHERE NOT EXISTS (
  SELECT 1
  FROM world w2
  WHERE w2.continent = w1.continent
    AND w2.population > 25000000
);

「列出所有國家名稱（name）及其所屬的洲（continent），但僅限於那些『洲內沒有任何一個國家的人口數超過 2,500 萬』的國家。」

🔢 五、比較第二大值與極端條件
1️⃣ 找出每洲人口最多的國家
SELECT name, continent, population
FROM world w1
WHERE population = (
    SELECT MAX(population)
    FROM world w2
    WHERE w2.continent = w1.continent
);

2️⃣ 找出人口最多的國家比同洲第二大多三倍（Three Times Bigger）
SELECT w1.name, w1.continent
FROM world AS w1
WHERE CAST(w1.population AS BIGINT) >
      3 * (
          SELECT MAX(CAST(w2.population AS BIGINT))
          FROM world AS w2
          WHERE w2.continent = w1.continent
            AND w2.name <> w1.name
      );

💡 說明：

CAST(... AS BIGINT)：防止整數溢位。

內層查詢：取「同洲中除自己外最大的人口」＝第二大。

外層：比對是否大於「第二大 × 3」。

🧮 六、自訂排序 (ORDER BY CASE)
SELECT continent, MIN(name) AS first_country
FROM world
GROUP BY continent
ORDER BY CASE continent
           WHEN 'Europe' THEN 1
           WHEN 'Insular Oceania' THEN 2
           WHEN 'North America' THEN 3
           WHEN 'South America' THEN 4
           WHEN 'Africa' THEN 5
           WHEN 'Asia' THEN 6
         END;

💡 說明：

CASE WHEN 可在 ORDER BY 內自訂顯示順序。

SQL 會依回傳值（1, 2, 3, …）由小到大排序。

🧠 七、SQL 執行順序（重要觀念）
步驟	關鍵字	說明
1	FROM	指定資料來源
2	WHERE	篩選資料（分組前）
3	GROUP BY	分組
4	HAVING	篩選組結果（分組後）
5	SELECT	選取欄位或聚合結果
6	ORDER BY	排序顯示結果

🧩 八、實戰題摘要

題目	重點語法	概念
找每洲人口最少的國家	MIN() + GROUP BY	聚合
找每洲中字母順序最前的國家	MIN(name)	字母排序
找每洲所有國家人口都 ≤ 2500 萬	HAVING MAX(population)	分組條件
找人口最多的國家超過第二大 3 倍	子查詢 + 型別轉換	巢狀比較
```
--------

## 🧮 二十、DISTINCT 唯一值與去重查詢
```sql
📘 基本用法：去除重複列

SELECT DISTINCT continent
FROM world;

💡 說明：

DISTINCT 會讓 SQL 只顯示「唯一不重複」的結果。

若相同資料出現多次，只會顯示一次。

🎯 多欄位搭配 DISTINCT
SELECT DISTINCT continent, region
FROM world;

💡 說明：

若指定多個欄位，SQL 會以整組組合為唯一判斷。

只有當「continent 與 region 的組合」完全相同時才會被視為重複。

📘 範例：

continent	region
Asia	East Asia
Asia	South Asia
Asia	East Asia

→ DISTINCT continent, region 結果：

continent	region
Asia	East Asia
Asia	South Asia

🧩 與 COUNT() 結合：統計唯一項目數
SELECT COUNT(DISTINCT continent)
FROM world;

💡 說明：

計算「唯一」洲別的數量。

若沒有 DISTINCT，COUNT(continent) 會包含重複值。

🔍 與 GROUP BY 的關係
比較項目	DISTINCT	GROUP BY
功能	去除重複列	依欄位分組
結果	唯一值列表	每組一筆結果，可搭配聚合函數
常見用途	簡單去重	統計與彙總
範例	SELECT DISTINCT continent	SELECT continent, COUNT(*) FROM world GROUP BY continent

💡 小技巧：

GROUP BY 通常能達成與 DISTINCT 類似的效果，但更彈性。

若只需要唯一列表（不做統計），用 DISTINCT 效率更高。

🧠 進階應用：搭配子查詢
SELECT DISTINCT continent
FROM world
WHERE population > (
    SELECT AVG(population)
    FROM world
);

💡 說明：

先找出所有人口高於全球平均值的國家，

再列出它們所屬的洲（不重複）。

🧾 小結

DISTINCT 是最簡單的資料去重方式。

只能出現在 SELECT 子句中，不能搭配 WHERE。

適合用於「顯示唯一清單」或「統計唯一值數量」。

若要進一步分組與統計，建議改用 GROUP BY。
```
--------