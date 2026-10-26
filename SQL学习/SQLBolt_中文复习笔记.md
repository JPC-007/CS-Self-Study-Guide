# SQLBolt 学习笔记

按 [SQLBolt](https://sqlbolt.com/) 官网课程顺序整理，以每课的概念、语法和练习内容为主。示例沿用课程中的电影、票房、员工、建筑和城市数据表；练习中的数据只用于学习，不作为现实事实引用。

课程分为三部分：第 1～5 课学习单表查询，第 6～12 课学习关联、表达式和聚合，第 13～18 课学习数据与表结构的修改。以下为中文概括，具体课程可通过每节链接查看。各课的“重难点”是帮助理解课程概念的补充解释，采用对照写法和小型虚构数据说明常见错误。

> 语法以 SQLBolt 教学环境为基础。字符串统一使用单引号；大小写比较、数据类型和修改表结构等行为可能因数据库而异。各节示例独立使用，不是一份从头执行到尾的脚本。

## 目录

- [第 1 课 SELECT 基础查询](#第-1-课-select-基础查询)
- [第 2 课 使用 WHERE 筛选行](#第-2-课-使用-where-筛选行)
  - [重难点 AND 和 OR 混用时的括号](#重难点-and-和-or-混用时的括号)
- [第 3 课 文本条件与 LIKE](#第-3-课-文本条件与-like)
  - [重难点 通配符与引号](#重难点-通配符与引号)
- [第 4 课 去重 排序和限制条数](#第-4-课-去重-排序和限制条数)
  - [重难点 DISTINCT 的范围和稳定排序](#重难点-distinct-的范围和稳定排序)
- [第 5 课 基础查询综合复习](#第-5-课-基础查询综合复习)
- [第 6 课 INNER JOIN 多表查询](#第-6-课-inner-join-多表查询)
  - [重难点 一对多关联为什么会增加行数](#重难点-一对多关联为什么会增加行数)
- [第 7 课 外连接](#第-7-课-外连接)
  - [重难点 LEFT JOIN 中 ON 与 WHERE 的区别](#重难点-left-join-中-on-与-where-的区别)
- [第 8 课 NULL 空值](#第-8-课-null-空值)
  - [重难点 NULL 为什么不能按普通值比较](#重难点-null-为什么不能按普通值比较)
- [第 9 课 表达式与别名](#第-9-课-表达式与别名)
  - [重难点 表达式的括号和数值类型](#重难点-表达式的括号和数值类型)
- [第 10 课 聚合函数与 GROUP BY](#第-10-课-聚合函数与-group-by)
  - [重难点 COUNT 和 AVG 怎样处理 NULL](#重难点-count-和-avg-怎样处理-null)
  - [重难点 LEFT JOIN 后统计员工数](#重难点-left-join-后统计员工数)
  - [重难点 分组后能选择哪些列](#重难点-分组后能选择哪些列)
- [第 11 课 使用 HAVING 筛选分组](#第-11-课-使用-having-筛选分组)
  - [重难点 先筛选员工还是先统计角色](#重难点-先筛选员工还是先统计角色)
- [第 12 课 查询的逻辑执行顺序](#第-12-课-查询的逻辑执行顺序)
  - [重难点 别名在不同子句中是否可用](#重难点-别名在不同子句中是否可用)
- [第 13 课 INSERT 插入数据](#第-13-课-insert-插入数据)
  - [重难点 列清单和值按位置对应](#重难点-列清单和值按位置对应)
- [第 14 课 UPDATE 更新数据](#第-14-课-update-更新数据)
  - [重难点 有 WHERE 也不一定只修改一行](#重难点-有-where-也不一定只修改一行)
- [第 15 课 DELETE 删除数据](#第-15-课-delete-删除数据)
- [第 16 课 CREATE TABLE 创建表](#第-16-课-create-table-创建表)
  - [重难点 默认值 唯一性和非空是不同要求](#重难点-默认值-唯一性和非空是不同要求)
- [第 17 课 ALTER TABLE 修改表结构](#第-17-课-alter-table-修改表结构)
- [第 18 课 DROP TABLE 删除表](#第-18-课-drop-table-删除表)
  - [重难点 区分修改行与修改表](#重难点-区分修改行与修改表)

## 第 1 课 SELECT 基础查询

`SELECT` 指定返回哪些列，`FROM` 指定数据来自哪张表。表中的一行是一条记录，一列是一种属性。

```sql
SELECT title, director
FROM movies;

-- 返回全部列
SELECT *
FROM movies;
```

**练习重点：**从 movies 中分别取出片名、导演、上映年份，以及指定的字段组合。选择列不会自动去掉重复值。[原课程](https://sqlbolt.com/lesson/select_queries_introduction)

## 第 2 课 使用 WHERE 筛选行

`WHERE` 对每行判断条件，只保留符合条件的记录。`AND` 连接需要同时满足的条件，`OR` 连接满足其中之一即可的条件。

| 运算符 | 含义 |
| --- | --- |
| `=`、`!=`、`<>` | 等于、不等于 |
| `<`、`<=`、`>`、`>=` | 大小比较 |
| `BETWEEN a AND b` | 在范围内，包含两端 |
| `NOT BETWEEN a AND b` | 不在该范围内 |
| `IN (...)` | 在给定列表中 |
| `NOT IN (...)` | 不在给定列表中 |

```sql
SELECT title, year
FROM movies
WHERE year BETWEEN 2000 AND 2010;
```

**练习重点：**按 id 定位电影，按年份选择区间内或区间外的电影。注意区间端点是否包含。[原课程](https://sqlbolt.com/lesson/select_queries_with_constraints)

### 重难点 AND 和 OR 混用时的括号

`AND` 的优先级高于 `OR`。下面两种条件表达的意思不同：

```sql
-- 两位导演的电影都必须在 2000 年及以后上映
SELECT title, director, year
FROM movies
WHERE year >= 2000
  AND (director = 'John Lasseter' OR director = 'Andrew Stanton');
```

去掉括号后，条件相当于 `(year >= 2000 AND director = 'John Lasseter') OR director = 'Andrew Stanton'`，第二位导演的电影将不受年份限制。混用逻辑条件时，用括号明确分组。[运算符优先级](https://www.sqlite.org/lang_expr.html#operators)

## 第 3 课 文本条件与 LIKE

文本可以使用等值比较、`IN` 和 `NOT IN`；按文本模式查找时使用 `LIKE` 或 `NOT LIKE`。

| 模式 | 含义 |
| --- | --- |
| `'Toy%'` | 以 Toy 开头 |
| `'%Story%'` | 包含 Story |
| `'WALL-_'` | WALL- 后面恰好一个字符 |

`%` 表示零个或多个字符，`_` 表示一个字符。

```sql
SELECT title, director
FROM movies
WHERE title LIKE 'Toy%';
```

**练习重点：**查找系列电影、指定导演的电影，以及不由该导演执导的电影。官网示例中的大小写比较规则与练习环境有关，不能概括为所有数据库都相同。[原课程](https://sqlbolt.com/lesson/select_queries_with_constraints_pt_2)

### 重难点 通配符与引号

`LIKE 'Toy%'` 也能匹配文本 `Toy`，因为 `%` 可以代表零个字符；`LIKE 'Toy_'` 则要求 Toy 后面恰好还有一个字符。查找包含某段文本时，通常需要在两侧都加 `%`。

文本中的单引号通过连续两个单引号表示：

```sql
SELECT title
FROM movies
WHERE title = 'A Bug''s Life';
```

`*` 在 `SELECT *` 中表示全部列，在 LIKE 模式中不表示任意字符。

## 第 4 课 去重 排序和限制条数

- `DISTINCT`：对选出的结果行去重。选择多列时，对多列组成的整行去重。
- `ORDER BY`：按指定字段排序；`ASC` 升序，`DESC` 降序。
- `LIMIT n`：最多返回 n 行。
- `OFFSET m`：先跳过 m 行。

```sql
SELECT DISTINCT director
FROM movies
ORDER BY director ASC;

SELECT title, year
FROM movies
ORDER BY title ASC
LIMIT 5 OFFSET 5;
```

第二条查询按片名排序，跳过前 5 行，再返回最多 5 行。没有 ORDER BY 时，不能依赖查询结果的顺序。

**练习重点：**导演去重并排序、按上映时间取前几部电影、按片名取第一组与下一组记录。[原课程](https://sqlbolt.com/lesson/filtering_sorting_query_results)

### 重难点 DISTINCT 的范围和稳定排序

假设结果里有三行：`(导演 A, 2000)`、`(导演 A, 2000)`、`(导演 A, 2005)`。`SELECT DISTINCT director, year` 会保留两行，导演 A 仍出现两次；仅选择 director 并去重才剩一行。

分页要先确定顺序。只按 year 排序时，同年电影的相对顺序可能变化，可以再按唯一的 id 排序：

```sql
SELECT id, title, year
FROM movies
ORDER BY year DESC, id ASC
LIMIT 5 OFFSET 10;
```

这是每页 5 条的第 3 页：`OFFSET = (页码 - 1) × 每页条数`。数据不变时，唯一的排序顺序有助于避免同年记录在不同页之间漂移。

## 第 5 课 基础查询综合复习

这一课使用北美城市数据，将前四课组合起来：先确定输出列和数据表，再筛选、排序、截取结果。

```sql
SELECT city, population
FROM north_american_cities
WHERE country = 'United States'
ORDER BY population DESC
LIMIT 2 OFFSET 2;
```

含义：按人口降序排列美国城市，取第 3、4 条记录。

**练习重点：**按国家筛选城市，比较经纬度，按人口取前几名或指定名次范围。北美城市的经度为负数，比较地理位置时要注意数值方向。[原课程](https://sqlbolt.com/lesson/select_queries_review)

## 第 6 课 INNER JOIN 多表查询

相关信息可能分散存放在不同表中。`JOIN` 根据关联条件将它们组合；`INNER JOIN` 仅返回满足条件的匹配行，通常可以简写为 `JOIN`。

```sql
SELECT m.title,
       b.domestic_sales,
       b.international_sales
FROM movies AS m
INNER JOIN boxoffice AS b
  ON m.id = b.movie_id;
```

课程中 movies.id 对应 boxoffice.movie_id。`ON` 描述怎样匹配；`m`、`b` 是表的简短别名。

**练习重点：**把电影名称与票房、评分组合起来，再比较国内外票房或按评分排序。[原课程](https://sqlbolt.com/lesson/select_queries_with_joins)

### 重难点 一对多关联为什么会增加行数

JOIN 按条件生成匹配组合，不保证一条左表记录只出现一次。课程中的电影与票房示例是一对一，不能据此推断所有 JOIN 都如此。

用建筑和员工举例，以下为虚构数据：A 容量为 100，有两名员工；B 容量为 200，没有员工。LEFT JOIN 后为：

| 建筑 | 容量 | 员工 |
| --- | --- | --- |
| A | 100 | Alex |
| A | 100 | Sam |
| B | 200 | NULL |

建筑原始总容量为 300，关联后直接对容量求和会变成 400。问题是同一栋建筑被计算了两次。也不能用 `SUM(DISTINCT capacity)` 一概修补，因为两栋不同建筑可能容量相同。

写关联查询时，先明确结果中的一行代表“电影”“员工”还是“建筑与员工的组合”，再决定如何统计。

## 第 7 课 外连接

内连接只保留匹配行；外连接还可以保留指定一侧没有匹配的记录，缺失的一侧填入 NULL。

| 连接类型 | 保留范围 |
| --- | --- |
| `LEFT JOIN` | 所有左表记录及其匹配结果 |
| `RIGHT JOIN` | 所有右表记录及其匹配结果 |
| `FULL JOIN` | 两侧的匹配与未匹配记录 |

```sql
SELECT DISTINCT b.building_name, e.role
FROM buildings AS b
LEFT JOIN employees AS e
  ON b.building_name = e.building;
```

这里即使某栋建筑没有员工，仍会出现，role 为 NULL。`LEFT OUTER JOIN` 与 `LEFT JOIN` 表达相同的连接类型。

**练习重点：**列出建筑及其中的员工角色，同时保留空建筑。官网本课练习主要使用 LEFT JOIN。[原课程](https://sqlbolt.com/lesson/select_queries_with_outer_joins)

### 重难点 LEFT JOIN 中 ON 与 WHERE 的区别

```sql
-- 保留所有建筑，只匹配其中的工程师
SELECT b.building_name, e.name
FROM buildings AS b
LEFT JOIN employees AS e
  ON b.building_name = e.building
 AND e.role = 'Engineer';
```

```sql
-- 连接后只留下角色为工程师的行
SELECT b.building_name, e.name
FROM buildings AS b
LEFT JOIN employees AS e
  ON b.building_name = e.building
WHERE e.role = 'Engineer';
```

第一种写法中，没有工程师的建筑仍会出现，员工列为 NULL；第二种写法会排除这些行。ON 决定怎样匹配，WHERE 筛选连接后的结果。并非所有右表 WHERE 条件都会排除未匹配行，例如 `WHERE e.building IS NULL` 正是用于保留未匹配行。[外连接与筛选顺序](https://www.sqlite.org/lang_select.html#whereclause)

## 第 8 课 NULL 空值

NULL 用来表示缺失或未知的数据。外连接没有匹配时，也会产生 NULL。

```sql
-- 尚未分配建筑的员工
SELECT name, role
FROM employees
WHERE building IS NULL;

-- 没有员工的建筑
SELECT b.building_name
FROM buildings AS b
LEFT JOIN employees AS e
  ON b.building_name = e.building
WHERE e.building IS NULL;
```

使用 `IS NULL` 或 `IS NOT NULL`，不要使用 `= NULL`。把缺失值随意替换成 0 可能改变统计含义，例如影响平均值。

**练习重点：**识别原表中的空值，以及外连接产生的空值。[原课程](https://sqlbolt.com/lesson/select_queries_with_nulls)

### 重难点 NULL 为什么不能按普通值比较

SQL 条件可能得到 TRUE、FALSE 或 UNKNOWN。NULL 表示未知，`NULL = 1` 与 `NULL <> 1` 都无法确定真假；WHERE 只保留 TRUE。

```sql
-- 不会包含 building 为 NULL 的员工
SELECT name
FROM employees
WHERE building <> '1e';

-- 明确包含未分配建筑的员工
SELECT name
FROM employees
WHERE building <> '1e' OR building IS NULL;
```

**再注意 NOT IN：**`year NOT IN (2000, NULL)` 不能表达“排除 2000 年”。例如 2001 虽然不同于 2000，但与未知值的比较仍未知，整条条件不是 TRUE。这种列表中不应混入用来表示未知的 NULL。[NULL 与 IN 的规则](https://www.sqlite.org/lang_expr.html#in_op)

## 第 9 课 表达式与别名

查询不仅能返回原始列，还能通过算术或函数计算新值。`AS` 给结果列或表起别名，提高可读性。

```sql
SELECT m.title,
       (b.domestic_sales + b.international_sales) / 1000000.0
         AS total_sales_millions
FROM movies AS m
JOIN boxoffice AS b ON m.id = b.movie_id;
```

表达式也可以用于筛选。课程练习中的偶数年份可用 `year % 2 = 0` 判断；具体运算符和函数的支持范围取决于数据库。

**练习重点：**计算总票房并换算成百万单位，将评分转换成百分数，筛选偶数年份上映的电影。[原课程](https://sqlbolt.com/lesson/select_queries_with_expressions)

### 重难点 表达式的括号和数值类型

`(domestic_sales + international_sales) / 1000000.0` 先求总票房再换算单位；去掉括号后，只会对 international_sales 做除法。

整数除法也可能丢失小数。以 SQLite 为例，`5 / 2` 得到 2，`5 / 2.0` 得到 2.5。需要小数结果时，应让操作数具有适当的数值类型。别名只是结果的名称，不会改变计算方式。[算术运算规则](https://www.sqlite.org/lang_expr.html#operators)

## 第 10 课 聚合函数与 GROUP BY

聚合函数将多行信息汇总成统计值。不分组时统计整个结果集；使用 `GROUP BY` 后，分别统计每组。

| 函数 | 作用 |
| --- | --- |
| `COUNT(*)` | 统计行数 |
| `COUNT(column)` | 统计该列非 NULL 的行数 |
| `SUM(column)` | 求和 |
| `AVG(column)` | 求平均值 |
| `MIN(column)` | 求最小值 |
| `MAX(column)` | 求最大值 |

```sql
SELECT role,
       AVG(years_employed) AS average_years
FROM employees
GROUP BY role;
```

每个 role 对应一条平均工龄结果。

**练习重点：**查最大工龄、按角色计算平均工龄、按建筑统计工龄总和。[原课程](https://sqlbolt.com/lesson/select_queries_with_aggregates)

### 重难点 COUNT 和 AVG 怎样处理 NULL

假设三名员工的 years_employed 分别为 `1、NULL、3`：

| 表达式 | 结果 | 解释 |
| --- | --- | --- |
| `COUNT(*)` | 3 | 所有行 |
| `COUNT(years_employed)` | 2 | 有工龄值的行 |
| `SUM(years_employed)` | 4 | 忽略 NULL 后求和 |
| `AVG(years_employed)` | 2 | 4 除以 2 |

把 NULL 改成 0 后，平均值会变为 4 除以 3，统计含义已经改变。对于只返回这些聚合函数、没有 GROUP BY 或 HAVING 的查询，即使输入为零行，也会返回一行：COUNT 为 0，SUM 和 AVG 为 NULL。[聚合函数规则](https://sqlite.org/lang_aggfunc.html)

### 重难点 LEFT JOIN 后统计员工数

```sql
SELECT b.building_name,
       COUNT(e.building) AS employee_count
FROM buildings AS b
LEFT JOIN employees AS e
  ON b.building_name = e.building
GROUP BY b.building_name;
```

空建筑在连接结果中仍有一条右侧补 NULL 的行，所以 `COUNT(*)` 会算成 1。`COUNT(e.building)` 忽略这条 NULL，才能得到 0；这里成功匹配的 e.building 必定非空。

### 重难点 分组后能选择哪些列

按 role 分组后，每一行代表一个角色。直接输出 name 会产生歧义：同一角色有多名员工，到底显示谁？通用写法应让直接输出的普通列出现在 GROUP BY 中，其余列使用有明确意义的聚合表达式。

```sql
SELECT role, building, COUNT(*) AS employee_count
FROM employees
GROUP BY role, building;
```

增加 building 后，每行代表一个“角色＋建筑”组合，统计粒度随之变化。

## 第 11 课 使用 HAVING 筛选分组

`WHERE` 在分组前筛选原始行，`HAVING` 在分组后筛选统计结果。

```sql
SELECT role, COUNT(*) AS employee_count
FROM employees
GROUP BY role
HAVING COUNT(*) > 1;
```

含义：按角色统计人数，只保留人数大于 1 的角色。这里的 HAVING 示例用于说明本课概念。

只想统计某个角色时，可先用 WHERE：

```sql
SELECT COUNT(*) AS artist_count
FROM employees
WHERE role = 'Artist';
```

**练习重点：**统计指定角色的人数、各角色人数和指定角色的工龄总和，分清哪些条件应放在 WHERE。[原课程](https://sqlbolt.com/lesson/select_queries_with_aggregates_pt_2)

### 重难点 先筛选员工还是先统计角色

```sql
SELECT role, COUNT(*) AS employee_count
FROM employees
WHERE years_employed >= 5
GROUP BY role
HAVING COUNT(*) >= 2;
```

这条查询统计的是“每种角色中工龄至少 5 年的员工数”，再保留人数至少为 2 的角色。COUNT 不包含已经被 WHERE 排除的员工。

同层写 `WHERE COUNT(*) >= 2` 是错误的，因为此时还没有分组统计结果。先用一句话明确统计对象，才能判断条件应放在 WHERE 还是 HAVING。

## 第 12 课 查询的逻辑执行顺序

书写时 SELECT 在前，理解结果时按以下逻辑顺序推导：

```text
FROM / JOIN
→ WHERE
→ GROUP BY 与聚合
→ HAVING
→ SELECT
→ DISTINCT
→ ORDER BY
→ LIMIT / OFFSET
```

这是理解查询语义的逻辑模型，不代表数据库实际执行计划必须按同样步骤扫描数据。

```sql
SELECT m.director,
       SUM(b.domestic_sales + b.international_sales) AS total_sales
FROM movies AS m
JOIN boxoffice AS b ON m.id = b.movie_id
GROUP BY m.director
ORDER BY total_sales DESC;
```

先关联电影与票房，再按导演汇总，最后排序。ORDER BY 可以使用输出别名；WHERE 通常不能使用同层 SELECT 新定义的别名。

**练习重点：**统计每位导演的电影数量及电影总票房。[原课程](https://sqlbolt.com/lesson/select_queries_order_of_execution)

### 重难点 别名在不同子句中是否可用

```sql
SELECT role, COUNT(*) AS employee_count
FROM employees
GROUP BY role
HAVING COUNT(*) > 1
ORDER BY employee_count DESC;
```

这段写法在 HAVING 中重复聚合表达式，在 ORDER BY 中使用别名，便于跨数据库理解。不同数据库可能允许在更多位置引用别名，但不要把某个练习环境接受的写法当作统一规则。

DISTINCT 在输出列确定后去重，不能恢复 JOIN 或聚合时已经改变的统计结果。GROUP BY 也不保证输出顺序，需要排序时仍要写 ORDER BY。

## 第 13 课 INSERT 插入数据

表结构规定有哪些字段、各字段允许存放什么类型的数据。`INSERT INTO` 指定目标表和列，`VALUES` 提供对应的值。

```sql
-- 虚构电影，仅演示插入语法
INSERT INTO movies (title, director, year, length_minutes)
VALUES ('Example Movie', 'Example Director', 2020, 95);
```

每组值的数量和顺序需要与列清单一致。显式列出字段，可以清楚表达写入目标；省略的字段必须能使用默认值、自动生成值或允许的空值。上例沿用本课 id 自动增长的设置。

课程也介绍一次提供多组 VALUES，以及在值中使用表达式。

**练习重点：**添加一部电影，再向 boxoffice 添加对应的票房记录，保持 movie_id 与电影 id 对应。[原课程](https://sqlbolt.com/lesson/inserting_rows)

### 重难点 列清单和值按位置对应

`INSERT INTO movies (title, year) VALUES ('Example Movie', 2020)` 中，第一个值写入 title，第二个值写入 year；不是根据值的外观自动选择字段。

省略字段不等于显式写入 NULL。省略的字段可能使用默认值或自动生成值；显式 NULL 是否被接受，则取决于该列的定义和数据库规则。

## 第 14 课 UPDATE 更新数据

`UPDATE` 指定表，`SET` 指定要修改的字段和值，`WHERE` 指定哪些记录需要修改。

```sql
UPDATE movies
SET year = 1999
WHERE title = 'Toy Story 2';
```

多个字段可以用逗号分隔后一起修改。省略 WHERE 会更新所有行；课程建议先用相同条件执行 SELECT，确认目标后再更新。

**练习重点：**修正电影的导演、上映年份，以及同时修正片名和导演。[原课程](https://sqlbolt.com/lesson/updating_rows)

### 重难点 有 WHERE 也不一定只修改一行

如果片名不是唯一的，`WHERE title = 'Toy Story 2'` 可能匹配多行。有 WHERE 只表示限定了范围，不代表范围一定正确。

先用相同条件查询 id 和将要修改的字段，核对目标；如果只要修改某一条记录，可以用已确认的唯一 id 定位。DELETE 同样需要检查条件覆盖了哪些行。

## 第 15 课 DELETE 删除数据

`DELETE FROM` 删除表中的记录，WHERE 决定删除范围。

```sql
DELETE FROM movies
WHERE year < 2005;
```

省略 WHERE 会删除所有行，但表结构仍保留。与更新一样，先用 SELECT 检查筛选范围，有助于避免删错记录。

**练习重点：**按上映年份或导演删除电影。[原课程](https://sqlbolt.com/lesson/deleting_rows)

## 第 16 课 CREATE TABLE 创建表

建表时定义列名、数据类型、约束和默认值。

```sql
CREATE TABLE IF NOT EXISTS movies (
  id INTEGER PRIMARY KEY,
  title TEXT,
  director TEXT,
  year INTEGER,
  length_minutes INTEGER
);
```

`IF NOT EXISTS` 表示同名表存在时跳过创建；它不会修改已有表结构。

| 类型或约束 | 核心含义 |
| --- | --- |
| `INTEGER`、`BOOLEAN` | 整数、布尔值 |
| `FLOAT`、`DOUBLE`、`REAL` | 浮点数 |
| `CHARACTER`、`VARCHAR`、`TEXT` | 文本 |
| `DATE`、`DATETIME` | 日期和时间 |
| `BLOB` | 二进制数据 |
| `PRIMARY KEY` | 唯一标识一行 |
| `AUTOINCREMENT` | 自动生成递增整数，语法和支持因数据库而异 |
| `UNIQUE` | 唯一性约束 |
| `NOT NULL` | 禁止空值 |
| `CHECK` | 检查值是否符合表达式 |
| `FOREIGN KEY` | 维护表之间的引用关系 |
| `DEFAULT` | 指定默认值 |

**练习重点：**为数据库名称、版本和下载次数选择合适的数据类型并建表。[原课程](https://sqlbolt.com/lesson/creating_tables)

### 重难点 默认值 唯一性和非空是不同要求

- `DEFAULT`：省略该字段时采用什么值。
- `NOT NULL`：能否存入 NULL。
- `UNIQUE`：是否允许重复值。
- `PRIMARY KEY`：用哪个字段或字段组合标识记录。

默认值本身不会保证非空或唯一，唯一约束也不等于自动生成 id。不要把这些约束的作用混为一谈。

## 第 17 课 ALTER TABLE 修改表结构

`ALTER TABLE` 修改列、约束或表名。它修改的是结构，与 UPDATE 修改记录不同。

```sql
-- 添加列
ALTER TABLE movies
ADD aspect_ratio FLOAT;

-- 添加带默认值的列
ALTER TABLE movies
ADD language TEXT DEFAULT 'English';

-- 重命名表，独立语法示例
ALTER TABLE movies
RENAME TO films;
```

本课还介绍删除列；具体写法和支持范围因数据库及版本而异。官网练习集中在添加列。

**练习重点：**增加电影画幅比例字段，以及默认值为 English 的语言字段。[原课程](https://sqlbolt.com/lesson/altering_tables)

## 第 18 课 DROP TABLE 删除表

`DROP TABLE` 删除整个表，包括其中的数据和表结构。

```sql
DROP TABLE IF EXISTS movies;
```

`IF EXISTS` 用于避免因表不存在而报错。存在外键等依赖时，还需要处理依赖关系。

| 操作 | 删除内容 | 表结构是否保留 |
| --- | --- | --- |
| `DELETE FROM movies` | 所有数据行 | 保留 |
| `DROP TABLE movies` | 整个表对象及其数据 | 不保留 |

**练习重点：**删除练习中使用的电影表和票房表，理解删除数据与删除表的区别。[原课程](https://sqlbolt.com/lesson/dropping_tables)

### 重难点 区分修改行与修改表

| 目标 | 语句 |
| --- | --- |
| 新增电影记录 | `INSERT INTO movies ...` |
| 修改电影年份 | `UPDATE movies SET ...` |
| 删除部分或全部电影记录 | `DELETE FROM movies ...` |
| 添加电影语言字段 | `ALTER TABLE movies ADD ...` |
| 删除整个电影表 | `DROP TABLE movies` |

DELETE 不会删除某一列；如果只是清空某条记录的字段值，应考虑 UPDATE，并确认该列允许 NULL。
