# Mysql学习6-操作表中数据


本文详细介绍 **对表中数据** 的增删查改操作。

<!--more-->

## 1. 查询数据

查询表中数据所涉及的命令是最多的，一个基本的语法格式如下

```mysql
SELECT {* | <字段列名>}
FROM <表 1>, <表 2>…
<join_type> join <join_table> on <join_condition>
WHERE <表达式>
GROUP BY <group by definition>
HAVING <expression> [{<operator> <expression>}…]
ORDER BY <order by definition>
LIMIT [<offset>,] <row count>
```

这里简单解释各条子句的含义，

- `SELECT {*|<字段列名>}` 表示所要查询字段的名称。
- `FROM <表 1>，<表 2>…`，表示查询数据的来源，可以是单个或多个表。
- `WHERE <表达式>` 限定查询数据必须满足的查询条件。
- `GROUP BY< 字段 >` 告诉 MySQL 如何显示查询出来的数据，并按照指定的字段分组。
- `ORDER BY< 字段 >` 告诉 MySQL 按什么样的顺序显示查询出来的数据。
- `LIMIT[，]`，该子句告诉 MySQL 每次显示多少条查询出来的数据。

### 1.1 基本查询

使用 `*` 通配符可以查询整个表的数据

```mysql
SELECT * FROM 表名;
```

下面的表将是后面所有语句执行的基表

```mysql
mysql> SELECT * FROM tb_students_info;
+----+--------+---------+------+------+--------+------------+
| id | name   | dept_id | age  | sex  | height | login_date |
+----+--------+---------+------+------+--------+------------+
|  1 | Dany   |       1 |   25 | F    |    160 | 2015-09-10 |
|  2 | Green  |       3 |   23 | F    |    158 | 2016-10-22 |
|  3 | Henry  |       2 |   23 | M    |    185 | 2015-05-31 |
|  4 | Jane   |       1 |   22 | F    |    162 | 2016-12-20 |
|  5 | Jim    |       1 |   24 | M    |    175 | 2016-01-15 |
|  6 | John   |       2 |   21 | M    |    172 | 2015-11-11 |
|  7 | Lily   |       6 |   22 | F    |    165 | 2016-02-26 |
|  8 | Susan  |       4 |   23 | F    |    170 | 2015-10-01 |
|  9 | Thomas |       3 |   22 | M    |    178 | 2016-06-07 |
| 10 | Tom    |       4 |   23 | M    |    165 | 2016-08-05 |
+----+--------+---------+------+------+--------+------------+
10 rows in set (0.26 sec)
```

如果在 SELECT 命令后指定字段名，则结果只显示这些字段，相当于投影操作

```mysql
SELECT <字段名1>,<字段名2>,…,<字段名n> FROM < 表名 >;
```

查询多个字段的示例如下

```mysql
mysql> SELECT id,name,height
    -> FROM tb_students_info;
+----+--------+--------+
| id | name   | height |
+----+--------+--------+
|  1 | Dany   |    160 |
|  2 | Green  |    158 |
|  3 | Henry  |    185 |
|  4 | Jane   |    162 |
|  5 | Jim    |    175 |
|  6 | John   |    172 |
|  7 | Lily   |    165 |
|  8 | Susan  |    170 |
|  9 | Thomas |    178 |
| 10 | Tom    |    165 |
+----+--------+--------+
10 rows in set (0.00 sec)
```

### 1.2 条件查询

使用 WHERE 关键字来指定查询条件，相当于选择操作。查询条件可以是：

- 带比较运算符和逻辑运算符的查询条件
- 带 IS NULL 或 IS NOT NULL 关键字的查询条件
- 带 BETWEEN AND 关键字的查询条件
- 带 IN 或 NOT IN 关键字的查询条件
- 带 LIKE 关键字的查询条件

查询条件可以是一条，也可以是多条，如下例

```mysql
# 单一条件
mysql> SELECT name,height FROM tb_students_info
    -> WHERE height=170;
+-------+--------+
| name  | height |
+-------+--------+
| Susan |    170 |
+-------+--------+
1 row in set (0.17 sec)
# 多条件
mysql> SELECT name,age,height FROM tb_students_info 
    -> WHERE age>21 AND height>=175;
+--------+------+--------+
| name   | age  | height |
+--------+------+--------+
| Henry  |   23 |    185 |
| Jim    |   24 |    175 |
| Thomas |   22 |    178 |
+--------+------+--------+
3 rows in set (0.00 sec)
```

比较运算符和逻辑运算符的时候大家都很熟悉，下面介绍其它几种运算

#### 空值查询

IS NULL 和 IS NOT NULL 关键字用来进行判空查询，判断某个元组的分量是不是等于 NULL。

这里要注意的是 MySQL 中空值不等于 0，也不等于空字符串。一个使用示例如下

```msyql
mysql> SELECT `name`,`login_date` FROM tb_students_info 
    -> WHERE login_date IS NULL;
+--------+------------+
| NAME   | login_date |
+--------+------------+
| Dany   | NULL       |
| Green  | NULL       |
| Henry  | NULL       |
| Jane   | NULL       |
| Thomas | NULL       |
| Tom    | NULL       |
+--------+------------+
6 rows in set (0.01 sec)
```

#### 范围查询

BETWEEN AND 关键字用来进行范围查询，判断某个属性是否在指定的范围内，语法格式如下

```mysql
[NOT] BETWEEN 取值1 AND 取值2
```

使用示例如下

```mysql
mysql> SELECT name,age FROM tb_students_info 
    -> WHERE age BETWEEN 20 AND 23;
+--------+------+
| name   | age  |
+--------+------+
| Green  |   23 |
| Henry  |   23 |
| Jane   |   22 |
| John   |   21 |
| Lily   |   22 |
| Susan  |   23 |
| Thomas |   22 |
| Tom    |   23 |
+--------+------+
8 rows in set (0.00 sec)
```

#### IN/NOT IN

MySQL 中的 IN 用来判断表达式的值是否位于给出的列表中；如果是，返回值为 1，否则返回值为 0。语法格式如下

```mysql
expr IN ( value1, value2, value3 ... valueN )
expr NOT IN ( value1, value2, value3 ... valueN )
```

示例如下

```mysql
 SELECT 2 IN (1,3,5,'this'), 'this' NOT IN (1,3,5,'this');
+---------------------+------------------------------+
| 2 IN (1,3,5,'this') | 'this' NOT IN (1,3,5,'this') |
+---------------------+------------------------------+
|                   0 |                            0 |
+---------------------+------------------------------+
1 row in set, 2 warnings (0.00 sec)
```

当 IN 运算符的两侧有一个为空值 NULL 时，如果找不到匹配项，则返回值为 NULL；如果找到了匹配项，则返回值为 1。

NOT IN 的作用和 IN 恰好相反。

#### 模糊查询

使用 LIKE 关键字可以搜索匹配字段中的指定内容，语法格式如下

```mysql
[NOT] LIKE  '字符串'
```

- NOT ：可选参数，字段中的内容与指定的字符串不匹配时满足条件。
- 字符串：指定用来匹配的字符串。“字符串”可以是一个很完整的字符串，也可以包含通配符。

LIKE 关键字支持百分号 `%` 和下划线 `_` 通配符。

```mysql
# %通配符,代表任何长度的字符串，字符串的长度可以为 0
mysql> SELECT name FROM tb_students_info
    -> WHERE name LIKE 'T%';
+--------+
| name   |
+--------+
| Thomas |
| Tom    |
+--------+
2 rows in set (0.12 sec)
# _通配符,只能代表单个字符，字符的长度不能为 0
mysql> SELECT name FROM tb_students_info
    -> WHERE name LIKE '____y'; # 4个下划线
+-------+
| name  |
+-------+
| Henry |
+-------+
1 row in set (0.00 sec)
```

默认情况下，LIKE 关键字匹配字符的时候是不区分大小写的。如果需要区分大小写，可以加入 BINARY 关键字。

```mysql
mysql> SELECT name FROM tb_students_info WHERE name LIKE BINARY 't%';
Empty set (0.01 sec)
```

如果查询内容中包含通配符，可以使用“\”转义符

```mysql
mysql> SELECT NAME FROM test.`tb_students_info` WHERE NAME LIKE '%\%';
+-------+
| NAME  |
+-------+
| Dany% |
+-------+
1 row in set (0.00 sec)
```

### 1.3 分组查询

通过 WHERE 子句筛选后，可能会使用 GROUP BY 关键字根据一个或多个字段对查询结果进行分组。语法格式如下

```mysql
GROUP BY  <字段名>
```

其中，<字段名> 表示需要分组的字段名称，多个字段时用逗号隔开。

#### 单个字段分组

单独使用时，查询结果只显示每个分组的第一条记录，但要记得，每个分组一般不会只有这一条记录

```mysql
mysql> SELECT `name`,`sex` FROM tb_students_info 
    -> GROUP BY sex;
+-------+------+
| name  | sex  |
+-------+------+
| Henry | 女   |
| Dany  | 男   |
+-------+------+
2 rows in set (0.01 sec)
```

使用 GROUP_CONCAT() 函数可以在一条记录中把该分组所有的结果显示出来

```mysql
mysql> SELECT `sex`, GROUP_CONCAT(name) FROM tb_students_info 
    -> GROUP BY sex;
+------+----------------------------+
| sex  | GROUP_CONCAT(name)         |
+------+----------------------------+
| 女   | Henry,Jim,John,Thomas,Tom  |
| 男   | Dany,Green,Jane,Lily,Susan |
+------+----------------------------+
2 rows in set (0.00 sec)
```

#### 多个字段分组

多个字段分组查询时，会先按照第一个字段进行分组。如果第一个字段中有相同的值，MySQL 才会按照第二个字段进行分组。如果第一个字段中的数据都是唯一的，那么 MySQL 将不再对第二个字段进行分组。

```mysql
mysql> SELECT age,sex,GROUP_CONCAT(name) FROM tb_students_info 
    -> GROUP BY age,sex;
+------+------+--------------------+
| age  | sex  | GROUP_CONCAT(name) |
+------+------+--------------------+
|   21 | 女   | John               |
|   22 | 女   | Thomas             |
|   22 | 男   | Jane,Lily          |
|   23 | 女   | Henry,Tom          |
|   23 | 男   | Green,Susan        |
|   24 | 女   | Jim                |
|   25 | 男   | Dany               |
+------+------+--------------------+
7 rows in set (0.00 sec)
```

#### 聚合函数

聚合函数包括 COUNT()，SUM()，AVG()，MAX() 和 MIN()。其中，COUNT() 用来统计记录的条数；SUM() 用来计算字段值的总和；AVG() 用来计算字段值的平均值；MAX() 用来查询字段的最大值；MIN() 用来查询字段的最小值。这些都是分组查询时常用的函数。

```mysql
mysql> SELECT sex,COUNT(sex) FROM tb_students_info 
    -> GROUP BY sex;
+------+------------+
| sex  | COUNT(sex) |
+------+------------+
| 女   |          5 |
| 男   |          5 |
+------+------------+
2 rows in set (0.00 sec)
```

#### WITH ROLLUP

WITH POLLUP 关键字用来在所有记录的最后加上一条记录，这条记录是上面所有记录的总和，即统计记录数量。

```mysql
mysql> SELECT sex,GROUP_CONCAT(name) FROM tb_students_info 
    ->GROUP BY sex WITH ROLLUP;
+------+------------------------------------------------------+
| sex  | GROUP_CONCAT(name)                                   |
+------+------------------------------------------------------+
| 女   | Henry,Jim,John,Thomas,Tom                            |
| 男   | Dany,Green,Jane,Lily,Susan                           |
| NULL | Henry,Jim,John,Thomas,Tom,Dany,Green,Jane,Lily,Susan |
+------+------------------------------------------------------+
3 rows in set (0.00 sec)
```

### 1.4 过滤分组

在 MySQL 中，可以使用 HAVING 子句对分组后的数据进行过滤。语法格式如下

```mysql
HAVING <查询条件>
```

HAVING 关键字和 WHERE 关键字都可以用来过滤数据，且 HAVING 支持 WHERE 关键字中所有的操作符和语法。但是 WHERE 和 HAVING 关键字也存在以下几点差异：

- 一般情况下，WHERE 用于过滤数据行，而 HAVING 用于过滤分组。
- WHERE 查询条件中不可以使用聚合函数，而 HAVING 查询条件中可以使用聚合函数。
- WHERE 在数据分组前进行过滤，而 HAVING 在数据分组后进行过滤 。
- WHERE 查询条件中不可以使用字段别名，而 HAVING 查询条件中可以使用字段别名。

```mysql
# 使用 WHERE
mysql> SELECT name,sex FROM tb_students_info 
    -> WHERE height>150;
+--------+------+
| name   | sex  |
+--------+------+
| Dany   | 男   |
| Green  | 男   |
| Henry  | 女   |
| Jane   | 男   |
| Jim    | 女   |
| John   | 女   |
| Lily   | 男   |
| Susan  | 男   |
| Thomas | 女   |
| Tom    | 女   |
+--------+------+
10 rows in set (0.00 sec)

# 使用 HAVING
mysql> SELECT GROUP_CONCAT(name),sex,height FROM tb_students_info 
    -> GROUP BY height 
    -> HAVING AVG(height)>170;
+--------------------+------+--------+
| GROUP_CONCAT(name) | sex  | height |
+--------------------+------+--------+
| John               | 女   |    172 |
| Jim                | 女   |    175 |
| Thomas             | 女   |    178 |
| Henry              | 女   |    185 |
+--------------------+------+--------+
4 rows in set (0.00 sec)
```

### 1.5 排序

ORDER BY 关键字用来将查询结果中的数据按照一定的顺序进行排序。其语法格式如下：

```mysql
ORDER BY <字段名> [ASC|DESC]
```

语法说明如下。

- 字段名：表示需要排序的字段名称，多个字段时用逗号隔开。
- ASC|DESC：`ASC`表示字段按升序排序；`DESC`表示字段按降序排序。其中`ASC`为默认值。

使用 ORDER BY 关键字应该注意以下几个方面：

- ORDER BY 关键字后可以跟子查询。
- 当排序的字段中存在空值时，ORDER BY 会将该空值作为最小值来对待。
- 在对多个字段进行排序时，排序的第一个字段必须有相同的值，才会对第二个字段进行排序。如果第一个字段数据中所有的值都是唯一的，MySQL 将不再对第二个字段进行排序。

```mysql
# 单字段排序
mysql> SELECT * FROM tb_students_info ORDER BY height;
+----+--------+---------+------+------+--------+------------+
| id | name   | dept_id | age  | sex  | height | login_date |
+----+--------+---------+------+------+--------+------------+
|  2 | Green  |       3 |   23 | F    |    158 | 2016-10-22 |
|  1 | Dany   |       1 |   25 | F    |    160 | 2015-09-10 |
|  4 | Jane   |       1 |   22 | F    |    162 | 2016-12-20 |
|  7 | Lily   |       6 |   22 | F    |    165 | 2016-02-26 |
| 10 | Tom    |       4 |   23 | M    |    165 | 2016-08-05 |
|  8 | Susan  |       4 |   23 | F    |    170 | 2015-10-01 |
|  6 | John   |       2 |   21 | M    |    172 | 2015-11-11 |
|  5 | Jim    |       1 |   24 | M    |    175 | 2016-01-15 |
|  9 | Thomas |       3 |   22 | M    |    178 | 2016-06-07 |
|  3 | Henry  |       2 |   23 | M    |    185 | 2015-05-31 |
+----+--------+---------+------+------+--------+------------+
10 rows in set (0.08 sec)
# 多字段排序
mysql> SELECT name,height FROM tb_students_info ORDER BY height,name;
+--------+--------+
| name   | height |
+--------+--------+
| Green  |    158 |
| Dany   |    160 |
| Jane   |    162 |
| Lily   |    165 |
| Tom    |    165 |
| Susan  |    170 |
| John   |    172 |
| Jim    |    175 |
| Thomas |    178 |
| Henry  |    185 |
+--------+--------+
10 rows in set (0.09 sec)
```

默认情况下，查询数据按字母升序进行排序（A～Z），但数据的排序并不仅限于此，还可以使用 ORDER BY 中的 DESC 对查询结果进行降序排序（Z～A）。

### 1.6 去重

SELECT 选择属性列后，新表可能出现重复的元组，可以使用 DISTINCT 关键字去重

```mysql
SELECT DISTINCT <字段名> FROM <表名>;
```

其中，“字段名”为需要消除重复记录的字段名称，多个字段时用逗号隔开。

使用 DISTINCT 关键字时需要注意以下几点：

- DISTINCT 关键字只能在 SELECT 语句中使用。
- 在对一个或多个字段去重时，DISTINCT 关键字必须在所有字段的最前面。
- 如果 DISTINCT 关键字后有多个字段，则会对多个字段进行组合去重，也就是说，只有多个字段组合起来完全是一样的情况下才会被去重。

示例如下

```mysql
# 原表
mysql> SELECT * FROM test.student;
+----+----------+------+-------+
| id | name     | age  | stuno |
+----+----------+------+-------+
|  1 | zhangsan |   18 |    23 |
|  2 | lisi     |   19 |    24 |
|  3 | wangwu   |   18 |    25 |
|  4 | zhaoliu  |   18 |    26 |
|  5 | zhangsan |   18 |    27 |
|  6 | wangwu   |   20 |    28 |
+----+----------+------+-------+
6 rows in set (0.00 sec)
# 对 name 和 age 字段搜索结果去重
mysql> SELECT DISTINCT name,age FROM student;
+----------+------+
| name     | age  |
+----------+------+
| zhangsan |   18 |
| lisi     |   19 |
| wangwu   |   18 |
| zhaoliu  |   18 |
| wangwu   |   20 |
+----------+------+
5 rows in set (0.00 sec)
```

因为 DISTINCT 只能返回它的目标字段，而无法返回其它字段，所以在实际情况中，我们经常使用 DISTINCT 关键字来返回不重复字段的条数。

```mysql
mysql> SELECT COUNT(DISTINCT name,age) FROM student;
+--------------------------+
| COUNT(DISTINCT name,age) |
+--------------------------+
|                        5 |
+--------------------------+
1 row in set (0.01 sec)
```

值得一提的是，聚合函数中也有 DISTINCT 的选项，如下

```mysql
AVG([distinct] expr)
COUNT({*|[distinct] } expr)
MAX([distinct] expr)
MIN([distinct] expr)
SUM([distinct] expr)
```

这意味着 HAVING 子句也可以出现该关键字，比如

```mysql
SELECT class
FROM courses
GROUP BY class
HAVING COUNT(DISTINCT student) >= 5
```

### 1.7 设置别名

当表名或字段名很长或者执行一些特殊查询的时候，为了查询方便，可以使用 AS 关键字来为表和字段指定别名。

注意：表的别名不能与该数据库的其它表同名。字段的别名不能与该表的其它字段同名。在条件表达式中不能使用字段的别名，否则会出现 `ERROR 1054 (42S22): Unknown column` 这样的错误提示信息。

语法格式如下，其中 AS 关键字可省略，省略后原名和别名用空格隔开

```mysql
<表名> [AS] <别名> # 为表指定别名
<字段名> [AS] <别名> # 为字段指定别名
```

示例如下

```mysql
# 为表指定别名
mysql> SELECT stu.name,stu.height FROM tb_students_info AS stu;
+--------+--------+
| name   | height |
+--------+--------+
| Dany   |    160 |
| Green  |    158 |
| Henry  |    185 |
| Jane   |    162 |
| Jim    |    175 |
| John   |    172 |
| Lily   |    165 |
| Susan  |    170 |
| Thomas |    178 |
| Tom    |    165 |
+--------+--------+
10 rows in set (0.04 sec)

# 为字段指定别名
mysql> SELECT name AS student_name, age AS student_age FROM tb_students_info;
+--------------+-------------+
| student_name | student_age |
+--------------+-------------+
| Dany         |          25 |
| Green        |          23 |
| Henry        |          23 |
| Jane         |          22 |
| Jim          |          24 |
| John         |          21 |
| Lily         |          22 |
| Susan        |          23 |
| Thomas       |          22 |
| Tom          |          23 |
+--------------+-------------+
10 rows in set (0.00 sec)
```

### 1.8 限制查询结果的条数

使用 LIMIT 关键字限制查询结果返回的条数。LIMIT 关键字有 3 种使用方式，即指定初始位置、不指定初始位置以及与 OFFSET 组合使用。

基本语法格式如下，第一条记录的位置是 0，两个参数必须都是正整数。

```mysql
LIMIT 初始位置，记录数
```

示例

```mysql
mysql> SELECT * FROM tb_students_info LIMIT 3,5;
+----+-------+---------+------+------+--------+------------+
| id | name  | dept_id | age  | sex  | height | login_date |
+----+-------+---------+------+------+--------+------------+
|  4 | Jane  |       1 |   22 | F    |    162 | 2016-12-20 |
|  5 | Jim   |       1 |   24 | M    |    175 | 2016-01-15 |
|  6 | John  |       2 |   21 | M    |    172 | 2015-11-11 |
|  7 | Lily  |       6 |   22 | F    |    165 | 2016-02-26 |
|  8 | Susan |       4 |   23 | F    |    170 | 2015-10-01 |
+----+-------+---------+------+------+--------+------------+
5 rows in set (0.00 sec)
```

省略初始位置时默认从第一条记录开始，如果表中剩余记录数不够，则返回所有剩余记录

```mysql
mysql> SELECT * FROM tb_students_info LIMIT 4;
+----+-------+---------+------+------+--------+------------+
| id | name  | dept_id | age  | sex  | height | login_date |
+----+-------+---------+------+------+--------+------------+
|  1 | Dany  |       1 |   25 | F    |    160 | 2015-09-10 |
|  2 | Green |       3 |   23 | F    |    158 | 2016-10-22 |
|  3 | Henry |       2 |   23 | M    |    185 | 2015-05-31 |
|  4 | Jane  |       1 |   22 | F    |    162 | 2016-12-20 |
+----+-------+---------+------+------+--------+------------+
4 rows in set (0.00 sec)
```

LIMIT 可以和 OFFSET 组合使用，不过仅仅是换了一种写法，没有其它语义

```mysql
LIMIT 记录数 OFFSET 初始位置
```

### 1.9 多表查询

前面所讲的查询语句都是针对一个表的，但是在关系型数据库中，表与表之间是有联系的，所以在实际应用中，经常使用多表查询。多表查询就是同时查询两个或两个以上的表，主要有交叉连接、内连接和外连接。

#### 交叉连接

交叉连接（CROSS JOIN）一般用来返回连接表的笛卡尔积。语法格式如下

```mysql
SELECT <字段名> FROM <表1> CROSS JOIN <表2> [WHERE子句]
SELECT <字段名> FROM <表1>, <表2> [WHERE子句] 
```

以上两种语法的返回结果是相同的，但是第一种语法是官方建议的标准写法。

当存在 WHERE 语句时，执行完交叉连接会执行筛选，当没有 WHERE 语句，返回的就是完整的笛卡尔积，但是，不管哪种情况，都会先生成完整的笛卡尔积结果，这个结果中包含的元组数目非常大，所以一般不建议使用交叉连接。

为了减少结果结果中数据行的数目，应尽量使用内连接，内连接使用 ON 关键字设置连接条件。我们从 WHERE 语句 和 ON 语句的执行顺序可以理解这一点，WHERE 在 连接之后执行，而 ON 语句在连接之前进行。

#### 内连接

内连接（INNER JOIN）就是利用条件表达式来消除交叉连接的某些数据行。

内连接使用 INNER JOIN 关键字连接两张表，并使用 ON 子句来设置连接条件。如果没有连接条件，INNER JOIN 和 CROSS JOIN 在语法上是等同的，两者可以互换，因此 ON 子句的作用比较重要。内连接的语法格式如下

```mysql
SELECT <字段名> FROM <表1> INNER JOIN <表2> [ON子句]
```

示例

```mysql
mysql> SELECT s.name,c.course_name FROM tb_students_info s INNER JOIN tb_course c 
    -> ON s.course_id = c.id;
+--------+-------------+
| name   | course_name |
+--------+-------------+
| Dany   | Java        |
| Green  | MySQL       |
| Henry  | Java        |
| Jane   | Python      |
| Jim    | MySQL       |
| John   | Go          |
| Lily   | Go          |
| Susan  | C++         |
| Thomas | C++         |
| Tom    | C++         |
+--------+-------------+
10 rows in set (0.00 sec)
```

注意：当对多个表进行查询时，要在 SELECT 语句后面指定字段是来源于哪一张表。因此，在多表查询时，SELECT 语句后面的写法是表名.列名。另外，如果表名非常长的话，也可以给表设置别名，这样就可以直接在 SELECT 语句后面写上表的别名.列名。

#### 外连接

内连接的查询结果都是符合连接条件的记录，而外连接会先将连接的表分为基表和参考表，再以基表为依据返回满足和不满足条件的记录。外连接可以分为左外连接和右外连接。

**左外连接**使用 LEFT OUTER JOIN 关键字，语法格式如下

```mysql
SELECT <字段名> FROM <表1> LEFT OUTER JOIN <表2> <ON子句>
```

上述语法中，“表1”为基表，“表2”为参考表。左外连接查询时，可以查询出“表1”中的所有记录和“表2”中匹配连接条件的记录。如果“表1”的某行在“表2”中没有匹配行，那么在返回结果中，“表2”的字段值均为空值（NULL）。示例如下

```mysql
# 两张原始表
mysql> SELECT * FROM tb_course;
+----+-------------+
| id | course_name |
+----+-------------+
|  1 | Java        |
|  2 | MySQL       |
|  3 | Python      |
|  4 | Go          |
|  5 | C++         |
|  6 | HTML        |
+----+-------------+
6 rows in set (0.00 sec)

mysql> SELECT * FROM tb_students_info;
+----+--------+------+------+--------+-----------+
| id | name   | age  | sex  | height | course_id |
+----+--------+------+------+--------+-----------+
|  1 | Dany   |   25 | 男   |    160 |         1 |
|  2 | Green  |   23 | 男   |    158 |         2 |
|  3 | Henry  |   23 | 女   |    185 |         1 |
|  4 | Jane   |   22 | 男   |    162 |         3 |
|  5 | Jim    |   24 | 女   |    175 |         2 |
|  6 | John   |   21 | 女   |    172 |         4 |
|  7 | Lily   |   22 | 男   |    165 |         4 |
|  8 | Susan  |   23 | 男   |    170 |         5 |
|  9 | Thomas |   22 | 女   |    178 |         5 |
| 10 | Tom    |   23 | 女   |    165 |         5 |
| 11 | LiMing |   22 | 男   |    180 |         7 |
+----+--------+------+------+--------+-----------+
11 rows in set (0.00 sec)
# 左外连接查询结果
mysql> SELECT s.name,c.course_name FROM tb_students_info s LEFT OUTER JOIN tb_course c 
    -> ON s.`course_id`=c.`id`;
+--------+-------------+
| name   | course_name |
+--------+-------------+
| Dany   | Java        |
| Henry  | Java        |
| NULL   | Java        |
| Green  | MySQL       |
| Jim    | MySQL       |
| Jane   | Python      |
| John   | Go          |
| Lily   | Go          |
| Susan  | C++         |
| Thomas | C++         |
| Tom    | C++         |
| LiMing | NULL        |
+--------+-------------+
12 rows in set (0.00 sec)
```

**右外连接** 是左外连接的反向连接，使用 RIGHT OUTER JOIN 关键字，大致内容相同，不再赘述。

### 1.10 子查询

子查询指将一个查询语句嵌套在另一个查询语句中，通过这种方式可以实现多表查询。子查询可以在 SELECT、INSERT、UPDATE 和 DELETE 语句中使用，而且可以进行多层嵌套。

子查询可以在使用表达式的任何地方使用，不过在实际开发时出现在 WHERE 子句和 FROM 子句中比较多。语法格式如下

```mysql
WHERE <表达式> <操作符> (子查询)
```

其中，操作符可以是比较运算符和 IN、NOT IN、EXISTS、NOT EXISTS 等关键字。

1. **IN | NOT IN**：当表达式与子查询返回的结果集中的某个值相等时，返回 TRUE，否则返回 FALSE；若使用关键字 NOT，则返回值正好相反。

   ```mysql
   mysql> SELECT name FROM tb_students_info 
       -> WHERE course_id IN (SELECT id FROM tb_course WHERE course_name = 'Java');
   +-------+
   | name  |
   +-------+
   | Dany  |
   | Henry |
   +-------+
   2 rows in set (0.01 sec)
   ```

2. **EXISTS | NOT EXISTS**：用于判断子查询的结果集是否为空，若子查询的结果集不为空，返回 TRUE，否则返回 FALSE；若使用关键字 NOT，则返回的值正好相反。

   ```mysql
   mysql> SELECT * FROM tb_students_info
       -> WHERE EXISTS(SELECT course_name FROM tb_course WHERE id=1);
   +----+--------+------+------+--------+-----------+
   | id | name   | age  | sex  | height | course_id |
   +----+--------+------+------+--------+-----------+
   |  1 | Dany   |   25 | 男   |    160 |         1 |
   |  2 | Green  |   23 | 男   |    158 |         2 |
   |  3 | Henry  |   23 | 女   |    185 |         1 |
   |  4 | Jane   |   22 | 男   |    162 |         3 |
   |  5 | Jim    |   24 | 女   |    175 |         2 |
   |  6 | John   |   21 | 女   |    172 |         4 |
   |  7 | Lily   |   22 | 男   |    165 |         4 |
   |  8 | Susan  |   23 | 男   |    170 |         5 |
   |  9 | Thomas |   22 | 女   |    178 |         5 |
   | 10 | Tom    |   23 | 女   |    165 |         5 |
   | 11 | LiMing |   22 | 男   |    180 |         7 |
   +----+--------+------+------+--------+-----------+
   11 rows in set (0.01 sec)
   ```

习惯上，外层的 SELECT 查询称为父查询，圆括号中嵌入的查询称为子查询（子查询必须放在圆括号内）。MySQL 在处理上例的 SELECT 语句时，执行流程为：先执行子查询，再执行父查询。

子查询的功能也可以通过表连接完成，但是子查询会使 SQL 语句更容易阅读和编写。一般来说，表连接（内连接和外连接等）都可以用子查询替换，但反过来却不一定，有的子查询不能用表连接来替换。子查询比较灵活、方便、形式多样，适合作为查询的筛选条件，而表连接更适合于查看连接表的数据。

### 1.11 正则表达式查询

MySQL 中，使用 **REGEXP** 关键字指定正则表达式的字符匹配模式，其基本语法格式如下：

```mysql
属性名 REGEXP '匹配方式'
```

其中，“属性名”表示需要查询的字段名称；“匹配方式”表示以哪种方式来匹配查询，符合正则表达式的写法。一个示例如下

```mysql
mysql> SELECT * FROM tb_students_info 
    -> WHERE name REGEXP '^J';
+----+------+------+------+--------+-----------+
| id | name | age  | sex  | height | course_id |
+----+------+------+------+--------+-----------+
|  4 | Jane |   22 | 男   |    162 |         3 |
|  5 | Jim  |   24 | 女   |    175 |         2 |
|  6 | John |   21 | 女   |    172 |         4 |
+----+------+------+------+--------+-----------+
3 rows in set (0.01 sec)
```

## 2. 插入数据

INSERT 语句有两种语法形式，分别是 INSERT…VALUES 语句和 INSERT…SET 语句。

1. INSERT…VALUES

   ```mysql
   INSERT INTO <表名> [ <列名1> [ , … <列名n>] ]
   VALUES (值1) [… , (值n) ];
   ```

   语法说明如下。

   - `<表名>`：指定被操作的表名。
   - `<列名>`：指定需要插入数据的列名。若向表中的所有列插入数据，则全部的列名均可以省略，直接采用 INSERT<表名>VALUES(…) 即可。
   - `VALUES` 或 `VALUE` 子句：该子句包含要插入的数据清单。数据清单中数据的顺序要和列的顺序相对应。

2. INSERT…SET

   ```mysql
   INSERT INTO <表名>
   SET <列名1> = <值1>,
       <列名2> = <值2>,
       …
   ```

   此语句用于直接给表中的某些列指定对应的列值，即要插入的数据的列名在 SET 子句中指定，col_name 为指定的列名，等号后面为指定的数据，而对于未指定的列，列值会指定为该列的默认值。

   由 INSERT 语句的两种形式可以看出：

   - 使用 INSERT…VALUES 语句可以向表中插入一行数据，也可以插入多行数据；
   - 使用 INSERT…SET 语句可以指定插入行中每列的值，也可以指定部分列的值；
   - INSERT…SELECT 语句向表中插入其他表的数据。
   - 采用 INSERT…SET 语句可以向表中插入部分列的值，这种方式更为灵活；
   - INSERT…VALUES 语句可以一次插入多条数据。

在 MySQL 中，用单条 INSERT 语句处理多个插入要比使用多条 INSERT 语句更快。

当使用单条 INSERT 语句插入多行数据的时候，只需要将每行数据用圆括号括起来即可。

```mysql
mysql> INSERT INTO tb_courses
    -> (course_name,course_grade,course_info)
    -> VALUES('System',3,'Operation System');
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            3 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```

可以使用 INSERT INTO…SELECT…FROM 语句从一个或多个表中取出数据，并将这些数据作为行数据插入另一个表中。

```mysql
mysql> INSERT INTO tb_courses_new
    -> (course_id,course_name,course_grade,course_info)
    -> SELECT course_id,course_name,course_grade,course_info
    -> FROM tb_courses;
Query OK, 4 rows affected (0.17 sec)
Records: 4  Duplicates: 0  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            3 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```

## 3. 修改数据

使用 UPDATE 语句修改单个表，语法格式为：

```mysql
UPDATE <表名> SET 字段 1=值 1 [,字段 2=值 2… ] [WHERE 子句 ]
[ORDER BY 子句] [LIMIT 子句]
```

语法说明如下：

- `<表名>`：用于指定要更新的表名称。
- `SET` 子句：用于指定表中要修改的列名及其列值。其中，每个指定的列值可以是表达式，也可以是该列对应的默认值。如果指定的是默认值，可用关键字 DEFAULT 表示列值。
- `WHERE` 子句：可选项。用于限定表中要修改的行。若不指定，则修改表中所有的行。
- `ORDER BY` 子句：可选项。用于限定表中的行被修改的次序。
- `LIMIT` 子句：可选项。用于限定被修改的行数。

> 注意：修改一行数据的多个列值时，SET 子句的每个值用逗号分开即可。

```mysql
# 更新所有行的 course_grade 字段值为 4
mysql> UPDATE tb_courses_new
    -> SET course_grade=4;
Query OK, 3 rows affected (0.11 sec)
Rows matched: 4  Changed: 3  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            4 | Computer Network |
|         2 | Database    |            4 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            4 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)

# 更新 course_id 值为 2 的记录，将 course_grade 字段值改为 3.5，将 course_name 字段值改为“DB”
mysql> UPDATE tb_courses_new
    -> SET course_name='DB',course_grade=3.5
    -> WHERE course_id=2;
Query OK, 1 row affected (0.13 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            4 | Computer Network |
|         2 | DB          |          3.5 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            4 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```

## 4. 删除数据

使用 DELETE 语句从单个表中删除数据，语法格式为：

```mysql
DELETE FROM <表名> [WHERE 子句] [ORDER BY 子句] [LIMIT 子句]
```

语法说明如下：

- `<表名>`：指定要删除数据的表名。
- `ORDER BY` 子句：可选项。表示删除时，表中各行将按照子句中指定的顺序进行删除。
- `WHERE` 子句：可选项。表示为删除操作限定删除条件，若省略该子句，则代表删除该表中的所有行。
- `LIMIT` 子句：可选项。用于告知服务器在控制命令被返回到客户端前被删除行的最大值。


注意：在不使用 WHERE 条件的时候，将删除所有数据。

```mysql
# 删除表中全部数据
mysql> DELETE FROM tb_courses_new;
Query OK, 3 rows affected (0.12 sec)
mysql> SELECT * FROM tb_courses_new;
Empty set (0.00 sec)

# 删除 course_id 为 4 的记录
mysql> DELETE FROM tb_courses
    -> WHERE course_id=4;
Query OK, 1 row affected (0.00 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
+-----------+-------------+--------------+------------------+
3 rows in set (0.00 sec)
```

**TRUNCATE** 关键字可以用于完全清空一个表

```msyql
TRUNCATE [TABLE] 表名
```

其中，TABLE 关键字可省略。

```msyql
mysql> TRUNCATE TABLE tb_student_course;
Query OK, 0 rows affected (0.04 sec)

mysql> SELECT * FROM tb_student_course;
Empty set (0.00 sec)
```

**TRUNCATE 和 DELETE 的区别**

从逻辑上说，TRUNCATE 语句与 DELETE 语句作用相同，但是在某些情况下，两者在使用上有所区别。

- DELETE 是 DML 类型的语句；TRUNCATE 是 DDL 类型的语句。它们都用来清空表中的数据。
- DELETE 是逐行一条一条删除记录的；TRUNCATE 则是直接删除原来的表，再重新创建一个一模一样的新表，而不是逐行删除表中的数据，执行数据比 DELETE 快。因此需要删除表中全部的数据行时，尽量使用 TRUNCATE 语句， 可以缩短执行时间。
- DELETE 删除数据后，配合事件回滚可以找回数据；TRUNCATE 不支持事务的回滚，数据删除后无法找回。
- DELETE 删除数据后，系统不会重新设置自增字段的计数器；TRUNCATE 清空表记录后，系统会重新设置自增字段的计数器。
- DELETE 的使用范围更广，因为它可以通过 WHERE 子句指定条件来删除部分数据；而 TRUNCATE 不支持 WHERE 子句，只能删除整体。
- DELETE 会返回删除数据的行数，但是 TRUNCATE 只会返回 0，没有任何意义。

**总结**

当不需要该表时，用 DROP；当仍要保留该表，但要删除所有记录时，用 TRUNCATE；当要删除部分记录时，用 DELETE
