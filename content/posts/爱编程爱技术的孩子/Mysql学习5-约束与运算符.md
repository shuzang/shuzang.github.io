---
title: Mysql学习5-约束与运算符
date: 2020-05-24T12:10:00+08:00
tags: [计算机基础]
categories: [爱编程爱技术的孩子]
slug: mysql learning 5 Constraints and operators 
---

约束是一种限制，它通过限制表中的数据，来确保数据的完整性和唯一性。本文介绍 MySQL 的约束和运算符。

<!--more-->

## 1. 约束

MySQL 主要支持 6 种约束

1. **主键约束**：唯一标识该表中的每条信息。例如，学生信息表中的学号是唯一的。
2. **外键约束**：经常和主键约束一起使用，用来确保数据的一致性。
3. **唯一约束**：保证列的唯一性，但与主键约束不同的是，可以为表中多个字段设置唯一约束，并且设置唯一约束的列是允许有空值的，虽然只能有一个空值。
4. **检查约束**：检查数据表中字段值是否有效
5. **非空约束**：约束表中的字段不能为空
6. **默认值约束**：用来约束当数据表中某个字段不输入值时，自动为其添加一个已经设置好的值。通常用在已经设置了非空约束的列，这样能够防止数据表在录入数据时出现错误。

以上 6 种约束中，一个数据表中只能有一个主键约束，其它约束可以有多个。使用 SHOW CREATE TABLE 语句可查看表中的约束。

### 1.1 主键

主键分为单字段主键和多字段联合主键，联合主键指一张表的主键由多个字段组成，比如学生选课数据表，学生编号和课程编号都不能单独作为主键，因为要限定一个学生只能选择同一课程一次，要将两者一起作为主键。不论哪种情况，每个表只能定义一个主键

#### 创建表时设置

使用 `PRIMARY KEY` 关键字在定义字段时指定主键，适用于单字段主键设置，语法格式如下

```mysql
<字段名> <数据类型> PRIMARY KEY [默认值]
```

示例

```mysql
mysql> CREATE TABLE tb_emp3
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT
    -> );
Query OK, 0 rows affected (0.37 sec)
```

或者在定义完所有字段后指定，多字段主键设置只能用这种方法。语法格式如下

```mysql
[CONSTRAINT <约束名>] PRIMARY KEY [字段名]
```

示例

```mysql
# 单字段主键设置
mysql> CREATE TABLE tb_emp4
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.37 sec)

# 多字段主键设置
mysql> CREATE TABLE tb_emp5
    -> (
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> PRIMARY KEY(id,deptId)
    -> );
Query OK, 0 rows affected (0.37 sec)
```

#### 修改表时设置

主键约束可在修改表时添加，应保证原表没有主键，要设置成主键的字段没有空值且没有重复，语法格式如下

```mysql
ALTER TABLE <数据表名> ADD PRIMARY KEY(<字段名>);
```

示例

```mysql
mysql> ALTER TABLE tb_emp2
    -> ADD PRIMARY KEY(id);
Query OK, 0 rows affected (0.94 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

#### 删除主键

语法格式如下

```mysql
ALTER TABLE <数据表名> DROP PRIMARY KEY;
```

#### 主键自增长

给字段添加 `AUTO_INCREMENT` 属性可以实现主键自增长，语法格式如下

```mysql
字段名 数据类型 AUTO_INCREMENT
```

- 默认情况下，AUTO_INCREMENT 的初始值是 1，每新增一条记录，字段值自动加 1。
- 一个表中只能有一个字段使用 AUTO_INCREMENT 约束，且该字段必须有唯一索引，以避免序号重复（即为主键或主键的一部分）。
- AUTO_INCREMENT 约束的字段必须具备 NOT NULL 属性。
- AUTO_INCREMENT 约束的字段只能是整数类型（TINYINT、SMALLINT、INT、BIGINT 等）。
- AUTO_INCREMENT 约束字段的最大值受该字段的数据类型约束，如果达到上限，AUTO_INCREMENT 就会失效。

示例

```mysql
mysql> CREATE TABLE tb_student(
    -> id INT(4) PRIMARY KEY AUTO_INCREMENT,
    -> name VARCHAR(25) NOT NULL
    -> );
Query OK, 0 rows affected (0.07 sec)
```

如果第一条记录设置了该字段的初始值，那么新增加的记录就从这个初始值开始自增。

```mysql
mysql> CREATE TABLE tb_student2 (
    -> id INT NOT NULL AUTO_INCREMENT,
    -> name VARCHAR(20) NOT NULL,
    -> PRIMARY KEY(ID)
    -> )AUTO_INCREMENT=100;
Query OK, 0 rows affected (0.03 sec) # 插入的数据从100开始增加，而不是1
```

### 1.2 外键

对于两个具有关联关系的表而言，相关联字段中主键所在的表就是主表（父表），外键所在的表就是从表（子表）。外键用来建立主表与从表的关联关系，为两个表的数据建立连接，约束两个表中数据的一致性和完整性。主表删除某条记录时，从表中与之对应的记录也必须有相应的改变。一些规则如下

- 主表必须已经存在于数据库中，或者是当前正在创建的表。如果是后一种情况，则主表与从表是同一个表，这样的表称为自参照表，这种结构称为自参照完整性。
- 必须为主表定义主键。
- 一个表可以有一个或多个外键。
- 主键不能包含空值，但允许在外键中出现空值。也就是说，只要外键的每个非空值出现在指定的主键中，这个外键的内容就是正确的。
- 从表的外键关联的必须是主表的主键，且主键和外键的数据类型必须一致。

可在创建表时使用 `FOREIGN KEY` 指定外键，语法格式如下

```mysql
[CONSTRAINT <外键名>] FOREIGN KEY 字段名 [，字段名2，…]
REFERENCES <主表名> 主键列1 [，主键列2，…]
```

示例

```mysql
# 创建主表
mysql> CREATE TABLE tb_dept1
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) NOT NULL,
    -> location VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.37 sec)
# 创建从表并设定外键
mysql> CREATE TABLE tb_emp6
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> CONSTRAINT fk_emp_dept1
    -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
    -> );
Query OK, 0 rows affected (0.37 sec)
```

也可在修改表时添加外键约束，前提是从表中外键列中的数据必须与主表中主键列中的数据一致或者是没有数据。语法格式如下

```mysql
ALTER TABLE <数据表名> ADD CONSTRAINT <外键名>
FOREIGN KEY(<列名>) REFERENCES <主表名> (<列名>);
```

示例

```mysql
mysql> ALTER TABLE tb_emp2
    -> ADD CONSTRAINT fk_tb_dept1
    -> FOREIGN KEY(deptId)
    -> REFERENCES tb_dept1(id);
Query OK, 0 rows affected (1.38 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除外键约束语法如下

```mysql
ALTER TABLE <表名> DROP FOREIGN KEY <外键约束名>;
```

### 1.3 唯一约束

使字段值不能重复出现，一张表主键约束只有一个，唯一约束可有多个，虽然允许有空值，但只能有一个。

可在创建表时使用 `UNIQUE` 关键字指定，如下示例

```mysql
mysql> CREATE TABLE tb_dept2
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) UNIQUE,
    -> location VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.37 sec)
```

也可在修改表时使用如下语法

```mysql
ALTER TABLE <数据表名> ADD CONSTRAINT <唯一约束名> UNIQUE(<列名>);
```

示例

```mysql
mysql> ALTER TABLE tb_dept1
    -> ADD CONSTRAINT unique_name UNIQUE(name);
Query OK, 0 rows affected (0.63 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除唯一约束的语法如下

```mysql
ALTER TABLE <表名> DROP INDEX <唯一约束名>;
```

### 1.4 检查约束

检查字段有效性，使用 SQL 表达式指定需要检查的条件，放在某个列的定义后称为基于列的 CHECK 约束，放在所有列的定义以及主键约束和外键定义之后，称为基于表的 CHECK 约束

创建表时使用 `CHECK` 关键字指定

```mysql
mysql> CREATE TABLE tb_emp7
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> CHECK(salary>0 AND salary<100),
    -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
    -> );
Query OK, 0 rows affected (0.37 sec)
```

修改表时使用如下语法

```mysql
ALTER TABLE tb_emp7 ADD CONSTRAINT <检查约束名> CHECK(<检查约束>)
```

示例

```mysql
mysql> ALTER TABLE tb_emp7
    -> ADD CONSTRAINT check_id
    -> CHECK(id>0);
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除检查约束的语法如下

```mysql
ALTER TABLE <数据表名> DROP CONSTRAINT <检查约束名>;
```

### 1.5 默认值

创建表时使用 `DEFAULT` 关键字设定默认值

```mysql
mysql> CREATE TABLE tb_dept3
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22),
    -> location VARCHAR(50) DEFAULT 'Beijing'
    -> );
Query OK, 0 rows affected (0.37 sec)
```

修改表时使用如下语法

```mysql
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <数据类型> DEFAULT <默认值>;
```

示例

```mysql
mysql> ALTER TABLE tb_dept3
    -> CHANGE COLUMN location
    -> location VARCHAR(50) DEFAULT 'Shanghai';
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除默认值的语法如下

```mysql
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> 
<字段名> <数据类型> DEFAULT NULL;
```

示例

```mysql
mysql> ALTER TABLE tb_dept3
    -> CHANGE COLUMN location
    -> location VARCHAR(50) DEFAULT NULL;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

### 1.6 非空约束

创建表时使用 `NOT NULL` 关键字设置非空约束

```mysql
mysql> CREATE TABLE tb_dept4
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) NOT NULL,
    -> location VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.37 sec)
```

修改表时使用如下语法

```mysql
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名>
<字段名> <数据类型> NOT NULL;
```

删除非空约束语法如下

```mysql
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> 
<字段名> <数据类型> NULL;
```

## 2. 运算符

在 MySQL 中，可以通过运算符来获取表结构以外的另一种数据。例如，学生表中存在一个 birth 字段，这个字段表示学生的出生年份。如果想得到这个学生的实际年龄，可以使用 MySQL 中的算术运算符用当前的年份减学生出生的年份，求出的结果就是这个学生的实际年龄了。

### 2.1 常用运算符

MySQL 支持的 4 种运算符分别是

1. 算术运算符：加减乘除、取余
2. 比较运算符：等于、安全的等于（<=>）、不等于、大于等于、大于、小于等于、小于、 IS NULL、IS NOT NULL、BETWEEN AND
3. 逻辑运算符：与、或、非、异或
4. 位运算符：按位与、按位或、按位取反、按位异或、按位左移、按位右移

**算术运算示例**如下

```mysql
# 创建和插入数据
CREATE TABLE temp(num INT);
INSERT INTO temp VALUE (64);
# 利用运算符查看
mysql> SELECT num, num+10, num-2.4, num*2, num/2, num%3, MOD (num,3), num/0, num%0 FROM temp;
+------+--------+---------+-------+---------+-------+-------------+-------+-------+
| num  | num+10 | num-2.4 | num*2 | num/2   | num%3 | MOD (num,3) | num/0 | num%0 |
+------+--------+---------+-------+---------+-------+-------------+-------+-------+
|   64 |     74 |    61.6 |   128 | 32.0000 |     1 |           1 |  NULL |  NULL |
+------+--------+---------+-------+---------+-------+-------------+-------+-------+
1 row in set (0.01 sec)
```

可以注意到以下几点

1. 无法整除时保留小数点后4位
2. 取余有两种写法
3. 除数为 0 没有意义返回 NULL

**逻辑运算符示例**如下

```mysql
SELECT NOT 1, NOT NULL, !0, 1 AND 0, 1 OR 0, 1 XOR 0;
+-------+----------+----+---------+--------+---------+
| NOT 1 | NOT NULL | !0 | 1 AND 0 | 1 OR 0 | 1 XOR 0 |
+-------+----------+----+---------+--------+---------+
|     0 |     NULL |  1 |       0 |      1 |       1 |
+-------+----------+----+---------+--------+---------+
1 row in set, 1 warning (0.00 sec)
```

注意到以下几点

1. 逻辑非有 NOT 和 ! 两种写法，逻辑与有 AND 和 && 两种写法，逻辑或有 OR 和 || 两种写法，逻辑异或只有 XOR 一种写法
2. 操作数为 NULL，返回值与 NULL 相关，操作数为 0 ，返回值与 0 相关，这是两种不同情况

**比较运算符示例**如下

```mysql
mysql> SELECT 1=0, NULL<=>NULL, 1 != 2, 1 <= 2, 1 >= 2, 1 IS NULL, 1 IS NOT NULL, 1 BETWEEN 1 AND 3;
+-----+-------------+--------+--------+--------+-----------+---------------+-------------------+
| 1=0 | NULL<=>NULL | 1 != 2 | 1 <= 2 | 1 >= 2 | 1 IS NULL | 1 IS NOT NULL | 1 BETWEEN 1 AND 3 |
+-----+-------------+--------+--------+--------+-----------+---------------+-------------------+
|   0 |           1 |      1 |      1 |      0 |         0 |             1 |                 1 |
+-----+-------------+--------+--------+--------+-----------+---------------+-------------------+
1 row in set (0.00 sec)
```

注意到

1. <=> 指安全的等于，用来判断 NULL 值，使结果返回 1 或 0 而不是 NULL；
2. BETWEEN AND 判断的数可以位于区间边界；

**位运算符示例**如下

```mysql
mysql> SELECT 10|15, 10&15, 10^15, 1<<2, 1>>1, ~1;
+-------+-------+-------+------+------+----------------------+
| 10|15 | 10&15 | 10^15 | 1<<2 | 1>>1 | ~1                   |
+-------+-------+-------+------+------+----------------------+
|    15 |    10 |     5 |    4 |    0 | 18446744073709551614 |
+-------+-------+-------+------+------+----------------------+
1 row in set (0.00 sec)
```

位运算一般用于操作整数，对整数进行位运算才有实际的意义。整数在内存中是以补码形式存储的，正数的补码形式和原码形式相同，而负数的补码形式和它的原码形式是不一样的，这意味着对负数进行位运算时，操作的是它的补码，而不是它的原码。

### 2.2 运算符优先级表

| 优先级由低到高排列 | 运算符                                                       |
| ------------------ | ------------------------------------------------------------ |
| 1                  | =(赋值运算）、:=                                             |
| 2                  | II、OR                                                       |
| 3                  | XOR                                                          |
| 4                  | &&、AND                                                      |
| 5                  | NOT                                                          |
| 6                  | BETWEEN、CASE、WHEN、THEN、ELSE                              |
| 7                  | =(比较运算）、<=>、>=、>、<=、<、<>、!=、 IS、LIKE、REGEXP、IN |
| 8                  | \|                                                           |
| 9                  | &                                                            |
| 10                 | <<、>>                                                       |
| 11                 | -(减号）、+                                                  |
| 12                 | *、/、%                                                      |
| 13                 | ^                                                            |
| 14                 | -(负号）、〜（位反转）                                       |
| 15                 | !                                                            |

### 2.3 IN 和 NOT IN

MySQL 中的 IN 运算符用来判断表达式的值是否位于给出的列表中；如果是，返回值为 1，否则返回值为 0。NOT IN 的作用和 IN 恰好相反。语法格式如下

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

## 3. MySQL 函数

MySQL 提供了一些内部函数帮助用户更方便地处理表中数据，包括数学函数、字符串函数、日期和时间函数、条件判断函数、系统信息函数和加密函数等。SELECT、INSERT、UPDATE 和 DELETE 语句及其子句（例如 WHERE、ORDER BY、HAVING 等）中都可以使用 MySQL 函数。实际应用时可以查阅：

- [MySQL 常用函数汇总](http://m.biancheng.net/mysql/function/)
- [MySQL 官方参考文档](https://dev.mysql.com/doc/refman/5.7/en/)