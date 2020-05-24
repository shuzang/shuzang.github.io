# Mysql学习6-操作表中数据


本文详细介绍表中数据地增删查改操作。

<!--more-->

## 1. 查询数据表

使用 SELECT 语句查询数据，语法格式如下

```mysql
SELECT
{* | <字段列名>}
[
FROM <表 1>, <表 2>…
[WHERE <表达式>
[GROUP BY <group by definition>
[HAVING <expression> [{<operator> <expression>}…]]
[ORDER BY <order by definition>]
[LIMIT[<offset>,] <row count>]
]
```

其中，各条子句的含义如下：

- `{*|<字段列名>}`包含星号通配符的字段列表，表示所要查询字段的名称。
- `<表 1>，<表 2>…`，表 1 和表 2 表示查询数据的来源，可以是单个或多个。
- `WHERE <表达式>`是可选项，如果选择该项，将限定查询数据必须满足该查询条件。
- `GROUP BY< 字段 >`，该子句告诉 MySQL 如何显示查询出来的数据，并按照指定的字段分组。
- `[ORDER BY< 字段 >]`，该子句告诉 MySQL 按什么样的顺序显示查询出来的数据，可以进行的排序有升序（ASC）和降序（DESC），默认情况下是升序。
- `[LIMIT[，]]`，该子句告诉 MySQL 每次显示查询出来的数据条数。

### 1.1 查询所有字段

使用 `*` 通配符，语法格式如下

```mysql
SELECT * FROM 表名;
```

### 1.2 查询指定字段

语法格式为

```mysql
SELECT <字段名1>,<字段名2>,…,<字段名n> FROM < 表名 >;
```

可以是一列也可以是多列，多列示例如下

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

### 1.3 去重

使用 DISTINCT 关键字去重

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

### 1.4 设置别名

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

### 1.5 限制查询结果的条数

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

### 1.6 对查询结果排序

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
- ORDER BY 指定多个字段进行排序时，MySQL 会按照字段的顺序从左到右依次进行排序。

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

在对多个字段进行排序时，排序的第一个字段必须有相同的值，才会对第二个字段进行排序。如果第一个字段数据中所有的值都是唯一的，MySQL 将不再对第二个字段进行排序。

默认情况下，查询数据按字母升序进行排序（A～Z），但数据的排序并不仅限于此，还可以使用 ORDER BY 中的 DESC 对查询结果进行降序排序（Z～A）。

### 1.7 条件查询

可以使用 WHERE 关键字来指定查询条件，查询条件可以是：

- 带比较运算符和逻辑运算符的查询条件
- 带 BETWEEN AND 关键字的查询条件
- 带 IS NULL 关键字的查询条件
- 带 IN 关键字的查询条件
- 带 LIKE 关键字的查询条件

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

### 1.8 模糊查询

使用 LIKE 关键字搜索匹配字段中的指定内容，语法格式如下

```mysql
[NOT] LIKE  '字符串'
```

其中：

- NOT ：可选参数，字段中的内容与指定的字符串不匹配时满足条件。
- 字符串：指定用来匹配的字符串。“字符串”可以是一个很完整的字符串，也可以包含通配符。

LIKE 关键字支持百分号“%”和下划线“_”通配符。

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

### 1.9 范围查询

使用比较运算符中提到的 BETWEEN AND 关键字，语法格式如下

```mysql
[NOT] BETWEEN 取值1 AND 取值2
```

示例

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

### 1.10 空值查询

使用 IS NULL 关键字，空值不同于 0，也不同于空字符串。语法格式为

```mysql
IS [NOT] NULL
```

示例

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

### 1.11 分组查询

GROUP BY 关键字可以根据一个或多个字段对查询结果进行分组。语法格式如下

```mysql
GROUP BY  <字段名>
```

其中，“字段名”表示需要分组的字段名称，多个字段时用逗号隔开。

#### 单独使用

单独使用时，查询结果只显示每个分组的第一条记录

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

#### 与 GROUP_CONCAT() 一起使用

和 GROUP_CONCAT() 函数一起使用，GROUP_CONCAT() 函数会把每个分组的字段值都显示出来。

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

#### 与聚合函数一起使用

聚合函数包括 COUNT()，SUM()，AVG()，MAX() 和 MIN()。其中，COUNT() 用来统计记录的条数；SUM() 用来计算字段值的总和；AVG() 用来计算字段值的平均值；MAX() 用来查询字段的最大值；MIN() 用来查询字段的最小值。

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

#### 与 WITH ROLLUP 一起使用

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

### 1.12 过滤分组

在 MySQL 中，可以使用 HAVING 关键字对分组后的数据进行过滤。语法格式如下

```mysql
HAVING <查询条件>
```

HAVING 关键字和 WHERE 关键字都可以用来过滤数据，且 HAVING 支持 WHERE 关键字中所有的操作符和语法。

但是 WHERE 和 HAVING 关键字也存在以下几点差异：

- 一般情况下，WHERE 用于过滤数据行，而 HAVING 用于过滤分组。
- WHERE 查询条件中不可以使用聚合函数，而 HAVING 查询条件中可以使用聚合函数。
- WHERE 在数据分组前进行过滤，而 HAVING 在数据分组后进行过滤 。
- WHERE 针对数据库文件进行过滤，而 HAVING 针对查询结果进行过滤。也就是说，WHERE 根据数据表中的字段直接进行过滤，而 HAVING 是根据前面已经查询出的字段进行过滤。
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

### 1.13 多表查询

前面所讲的查询语句都是针对一个表的，但是在关系型数据库中，表与表之间是有联系的，所以在实际应用中，经常使用多表查询。多表查询就是同时查询两个或两个以上的表，主要有交叉连接、内连接和外连接。


