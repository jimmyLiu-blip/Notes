# 📘 SQLZOO 學習筆記整理  
> 日期：2025-10-23  
> 主題：The JOIN Operation — 資料表關聯與多表查詢

---

## 🧩 一、JOIN 基本觀念

### 🔹 JOIN 是什麼？
JOIN 用於**將多張資料表依共同欄位關聯起來**，  
使我們能在同一查詢中取得不同表的資料。

常見的關聯方式：
| JOIN 類型 | 說明 |
|------------|------|
| **INNER JOIN** | 只取兩表中皆有對應的資料（交集） |
| **LEFT JOIN (LEFT OUTER JOIN)** | 左表全部保留，右表無對應以 NULL 補齊 |
| **RIGHT JOIN (RIGHT OUTER JOIN)** | 右表全部保留，左表無對應以 NULL 補齊 |
| **FULL JOIN (FULL OUTER JOIN)** | 左右表皆保留，沒有對應的地方以 NULL 補齊（部分資料庫不支援） |

---

## 🧱 二、基本語法結構

```sql
SELECT 欄位
FROM 表A
JOIN 表B ON 表A.欄位 = 表B.欄位;

📘 ON 用來指定兩表關聯的欄位。
常見範例：

SELECT matchid, player
FROM goal
JOIN game ON goal.matchid = game.id;
```
---
## ⚙️ 三、JOIN 搭配篩選條件
```sql
SELECT player, teamid, stadium, mdate
FROM game
JOIN goal ON game.id = goal.matchid
WHERE teamid = 'GER';

💡 說明：

JOIN 負責「關聯資料」

WHERE 負責「篩選條件」
```
---

## 🧾 四、使用別名 (AS) 增加可讀性
```sql
SELECT g.player, g.teamid, t.coach
FROM goal AS g
JOIN eteam AS t ON g.teamid = t.id;

💡 AS 為表取暱稱，方便撰寫與辨識。
```
---

## 🔗 五、多重 JOIN（同時連接三張表）
```sql
SELECT player, teamname, stadium, mdate
FROM goal
JOIN game ON goal.matchid = game.id
JOIN eteam ON goal.teamid = eteam.id;


💡 常用於：

結合「球員進球紀錄」＋「比賽資訊」＋「隊伍名稱」

統整成完整報表
```
---

## 🔍 六、LEFT JOIN（外連接）
```sql
SELECT t.name, g.matchid
FROM eteam AS t
LEFT JOIN goal AS g ON t.id = g.teamid;


💡 說明：

顯示所有隊伍，即使該隊沒有進球也會顯示（以 NULL 補齊）。

若改用 INNER JOIN，則只會顯示有進球的隊伍。
```
---
## 📊 七、結合聚合函數的 JOIN
```sql
SELECT teamname, COUNT(player) AS goals
FROM goal
JOIN eteam ON goal.teamid = eteam.id
GROUP BY teamname
ORDER BY goals DESC;


💡 說明：

JOIN 連接資料表

COUNT() 統計進球數

GROUP BY 按隊伍分組

ORDER BY 依進球數排序（由多到少）
```
---

## 🧠 八、JOIN 與 WHERE 的執行順序
```sql
執行階段	說明
1️⃣ FROM	指定資料來源
2️⃣ JOIN	關聯資料表
3️⃣ ON	定義關聯條件
4️⃣ WHERE	篩選資料（連接後）
5️⃣ GROUP BY	分組統計
6️⃣ HAVING	分組後篩選
7️⃣ SELECT	選擇輸出欄位
8️⃣ ORDER BY	最後排序結果
```
---

## 🧷 九、JOIN + DISTINCT（避免重複資料）
```sql
SELECT DISTINCT player
FROM game
JOIN goal ON matchid = id
WHERE teamid = 'ITA';

💡 說明：
JOIN 後若一個球員在多場比賽進球，會重複出現。
使用 DISTINCT 去除重複列，取得唯一名單。

```
---

## 🎯 十、JOIN + 自表關聯（Self Join）
```sql
SELECT a.actor, b.actor, m.title
FROM casting AS c1
JOIN casting AS c2 ON c1.movieid = c2.movieid
JOIN movie AS m ON c1.movieid = m.id
WHERE c1.actorid = (SELECT id FROM actor WHERE name='Art Garfunkel')
  AND c2.actorid <> c1.actorid;

💡 說明：
同一張表（casting）被視為兩個角色（c1、c2）來關聯，
用於找出與特定演員在同一部電影共演的其他演員。
```
---
## 🧮 十一、JOIN + 子查詢（Subquery）
```sql
SELECT m.title
FROM movie AS m
JOIN casting AS c ON m.id = c.movieid
JOIN actor AS a ON c.actorid = a.id
WHERE a.name = 'Harrison Ford'
  AND m.id NOT IN (
    SELECT c.movieid
    FROM casting AS c
    JOIN actor AS a ON c.actorid = a.id
    WHERE a.name = 'Johnny Depp'
  );

💡 說明：
結合 JOIN 與子查詢可實現「有A但沒有B」的條件篩選。
常見於 SQLZOO 第 13~15 題。
```
---
## 🔢 十二、JOIN + COUNT(DISTINCT)
```sql
SELECT teamid, COUNT(DISTINCT player) AS unique_scorers
FROM goal
GROUP BY teamid;

💡 說明：
COUNT(DISTINCT) 結合 JOIN 時，用來避免重複統計同一球員。
例如統計每隊「有進球的球員人數」。
```
---

## 🧩 十三、JOIN 條件可用多欄位
```sql
SELECT *
FROM results AS r
JOIN matches AS m
ON r.team = m.home_team OR r.team = m.away_team;

💡 說明：
有時一張表的關聯條件不只一個欄位（主客場兩欄）。
可用 OR 來建立複合 JOIN 條件。
```
---

## 🧭 十四、JOIN + COALESCE() 處理 NULL
```sql
SELECT teamname, COALESCE(goals, 0) AS total_goals
FROM eteam
LEFT JOIN (
  SELECT teamid, COUNT(*) AS goals
  FROM goal
  GROUP BY teamid
) AS g ON eteam.id = g.teamid;

💡 說明：
在 LEFT JOIN 中常出現 NULL，可用 COALESCE() 將其轉成 0。
實務中常見於報表統計。
```
---

## 🧠 十五、JOIN 結合 HAVING（群組後條件）
```sql
SELECT teamname, COUNT(player) AS goals
FROM goal
JOIN eteam ON goal.teamid = eteam.id
GROUP BY teamname
HAVING COUNT(player) >= 5;

💡 說明：
WHERE 無法篩選聚合結果，要用 HAVING 處理群組後條件
```
---
## 🧮 十六、JOIN + ORDER BY 自訂排序
```sql
SELECT teamname, COUNT(player) AS goals
FROM goal
JOIN eteam ON goal.teamid = eteam.id
GROUP BY teamname
ORDER BY goals DESC, teamname ASC;

💡 說明：
可以依聚合結果（如進球數）排序，再次依名稱次排序。
這種雙層排序在 SQLZOO 常見於最終彙整題。
```
---

## 🧩 十七、JOIN 執行順序與 ON vs WHERE 的差異
```sql
-- ✅ 推薦寫法（JOIN 條件放在 ON）
SELECT *
FROM goal
JOIN game ON goal.matchid = game.id
WHERE game.stadium = 'Warsaw';

-- ⚠️ 錯誤寫法（條件放在 ON 後再誤混篩選）
SELECT *
FROM goal
LEFT JOIN game ON goal.matchid = game.id AND game.stadium = 'Warsaw';

💡 差異：

ON 是「建立關聯條件」

WHERE 是「篩選結果」
在外連接（LEFT JOIN）中放錯位置會導致資料遺失。
```
---

## 📚 十八、JOIN + CASE WHEN 建立分類欄位
```sql
SELECT teamname,
       COUNT(CASE WHEN stadium='Warsaw' THEN 1 END) AS warsaw_goals,
       COUNT(CASE WHEN stadium='Berlin' THEN 1 END) AS berlin_goals
FROM goal
JOIN game ON goal.matchid = game.id
JOIN eteam ON goal.teamid = eteam.id
GROUP BY teamname;

💡 說明：
用 CASE WHEN 在 JOIN 結果中進行條件分類統計。
常見於進階 SQL 報表題。
```
---
## 🧭 十九、JOIN + ALL / ANY（比較運算子）
```sql
SELECT name
FROM world AS w1
WHERE population >= ALL(
  SELECT population
  FROM world AS w2
  WHERE w2.continent = w1.continent
    AND w2.population > 0
);

💡 說明：
ALL / ANY 常與子查詢搭配，用於跨群體比較，
例如找出「某洲人口最多的國家」。
```
---
