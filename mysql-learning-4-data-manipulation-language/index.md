# Mysql学习4-数据表操作


介绍数据表的基本操作，主要包括创建数据表、查看数据表结构、修改数据表和删除数据表等。

<!--more-->

## 1. 创建

创建数据表的语法格式为

```mysql
CREATE TABLE <表名> ([表定义选项])[表选项][分区选项];
```

其中，`[表定义选项]`的格式为：

```mysql
<列名1> <类型1> [,…] <列名n> <类型n>
```

语句的主要语法及使用说明如下：

- CREATE TABLE：用于创建给定名称的表，必须拥有表CREATE的权限。
- <表名>：指定要创建表的名称，在 CREATE TABLE 之后给出，必须符合标识符命名规则。表名称被指定为 db_name.tbl_name，以便在特定的数据库中创建表。无论是否有当前数据库，都可以通过这种方式创建。在当前数据库中创建表时，可以省略 db-name。如果使用加引号的识别名，则应对数据库和表名称分别加引号。例如，'mydb'.'mytbl' 是合法的，但 'mydb.mytbl' 不合法。
- <表定义选项>：表创建定义，由列名（col_name）、列的定义（column_definition）以及可能的空值说明、完整性约束或表索引组成。
- 默认的情况是，表被创建到当前的数据库中。若表已存在、没有当前数据库或者数据库不存在，则会出现错误。

示例如下

```mysql
mysql> USE test_db;
Database changed
mysql> CREATE TABLE tb_emp1
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT
    -> );
Query OK, 0 rows affected (0.37 sec)
```

## 2. 查看

使用 `SHOW TABLES` 查看数据表

```mysql
mysql> SHOW TABLES;
+--------------------+
| Tables_in_test_db  |
+--------------------+
| tb_emp1            |
+--------------------+
1 rows in set (0.00 sec)
```

使用 `DESCRIBE/DESC <表名> ` 查看表的字段信息，包括字段名、字段数据类型、是否为主键、是否有默认值等

```mysql
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```

其中，各个字段的含义如下：

- Null：表示该列是否可以存储 NULL 值。
- Key：表示该列是否已编制索引。PRI 表示该列是表主键的一部分，UNI 表示该列是 UNIQUE 索引的一部分，MUL 表示在列中某个给定值允许出现多次。
- Default：表示该列是否有默认值，如果有，值是多少。
- Extra：表示可以获取的与给定列有关的附加信息，如 AUTO_INCREMENT 等。

使用 `SHOW CREATE TABLE <表名>` 显示创建表时的CREATE TABLE语句、存储引擎和字符编码，末尾使用 `\G` 可以使显示结果更规整

```mysql
mysql> SHOW CREATE TABLE tb_emp1\G
*************************** 1. row ***************************
       Table: tb_emp1
Create Table: CREATE TABLE `tb_emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(25) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.03 sec)
```

## 3. 修改

修改表指的是修改数据库中已经存在的数据表的结构，例如增加或删减列、更改原有列类型、重新命名列或表等，前提是数据库中已经存在该表。语法格式如下

```mysql
ALTER TABLE <表名> [修改选项]
```

修改选项的语法格式如下

```mysql
{ ADD COLUMN <列名> <类型>
| CHANGE COLUMN <旧列名> <新列名> <新列类型>
| ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT }
| MODIFY COLUMN <列名> <类型>
| DROP COLUMN <列名>
| RENAME TO <新表名>
| CHARACTER SET <字符集名>
| COLLATE <校对规则名> }
```

一些示例如下

```mysql
ALTER TABLE <旧表名> RENAME [TO] <新表名>; # 修改表名
ALTER TABLE 表名 [DEFAULT] CHARACTER SET <字符集名> [DEFAULT] COLLATE <校对规则名>; # 修改表字符集
```

一个具体的修改字段名的例子如下

```mysql
mysql> ALTER TABLE tb_emp1
    -> CHANGE col1 col3 CHAR(30);
Query OK, 0 rows affected (0.76 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col3   | char(30)    | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
5 rows in set (0.01 sec)
```

有意思的是，添加字段时可以选择位置

```mysql
mysql> ALTER TABLE student ADD age INT(4); # 默认在末尾添加
mysql> ALTER TABLE student ADD stuId INT(4) FIRST; # 在开头添加
mysql> ALTER TABLE student ADD stuno INT(11) AFTER name; # 在中间位置添加
```

## 4. 删除

删除数据表的语法格式如下

```mysql
DROP TABLE [IF EXISTS] 表名1 [ ,表名2, 表名3 ...]
```

在删除表的同时，表的结构和表中所有的数据都会被删除，因此在删除数据表之前最好先备份，以免造成无法挽回的损失。不过应注意

1. 用户必须拥有执行 DROP TABLE 命令的权限，否则数据表不会被删除。
2. 表被删除时，用户在该表上的权限不会自动删除。

数据表之间还经常存在外键关联的情况，这时如果直接删除父表，会破坏数据表的完整性，也会删除失败。此时有以下两种方法：

- 先删除与它关联的子表，再删除父表；但是这样会同时删除两个表中的数据。
- 将关联表的外键约束取消，再删除父表；适用于需要保留子表的数据，只删除父表的情况。

假设tb_emp5 表为子表，具有名称为 fk_emp4_emp5 的外键约束；tb_emp4 为父表，其主键 id 被子表 tb_ emp5 所关联。删除被数据表 tb_emp5 关联的数据表 tb_emp4，SQL 语句如下：

```mysql
# 解除外键约束
mysql> ALTER TABLE tb_emp5 DROP FOREIGN KEY fk_emp4_emp5;
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0
# 删除父表
mysql> DROP TABLE tb_emp4;
```


