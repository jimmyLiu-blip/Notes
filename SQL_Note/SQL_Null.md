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
