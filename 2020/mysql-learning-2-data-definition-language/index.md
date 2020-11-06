# Mysql学习2-数据库操作


本文介绍对数据库的操作，包括创建、删除、修改等。

<!--more-->

## 1. 查看

查看数据库的语法格式为：

```mysql
SHOW DATABASES [LIKE '数据库名'];
```

语法说明如下：

- LIKE 从句是可选项，用于匹配指定的数据库名称。LIKE 从句可以部分匹配（使用 %），也可以完全匹配。
- 数据库名由单引号`' '`包围。

```mysql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sakila             |
| sys                |
| world              |
+--------------------+
6 row in set (0.22 sec)
```

如上所示，安装 MySQL 时系统自动创建 6 个数据库，各自功能如下

- information_schema：主要存储了系统中的一些数据库对象信息，比如用户表信息、列信息、权限信息、字符集信息和分区信息等。
- mysql：MySQL 的核心数据库，类似于 SQL Server 中的 master 表，主要负责存储数据库用户、用户访问权限等 MySQL 自己需要使用的控制和管理信息。常用的比如在 mysql 数据库的 user 表中修改 root 用户密码。
- performance_schema：主要用于收集数据库服务器性能参数。
- sakila：MySQL 提供的样例数据库，该数据库共有 16 张表，这些数据表都是比较常见的，在设计数据库时，可以参照这些样例数据表来快速完成所需的数据表。
- sys：主要提供一些视图，数据都来自于 performation_schema，主要是让开发者和使用者更方便地查看性能问题。
- world：MySQL 自动创建的数据库，只包括 3 张数据表，分别保存城市，国家和国家使用的语言等内容。

## 2. 创建

创建数据库的语法格式为：

```mysql
CREATE DATABASE [IF NOT EXISTS] <数据库名>
[[DEFAULT] CHARACTER SET <字符集名>] 
[[DEFAULT] COLLATE <校对规则名>];
```

`[]` 中的内容可选，语法说明如下

- 数据库名不能以数字开头，尽量有实际意义，不区分大小写；
- `[IF NOT EXISTS]` 用来避免数据库已存在而重复创建的错误
- `[DEFAULT] CHARACTER SET` 指定字符集，避免数据乱码，不指定则使用默认字符集
- `[DEFAULT] COLLATE` 指定字符集默认校对规则

注：字符集是用来定义 MySQL 存储字符串的方式，校对规则定义了比较字符串的方式。

示例

```mysql
mysql> CREATE DATABASE test_db; # 创建数据库
mysql> CREATE DATABASE IF NOT EXISTS test_db_char # 创建数据库的同时指定字符集和校对规则
    -> DEFAULT CHARACTER SET utf8
    -> DEFAULT COLLATE utf8_chinese_ci;
```

## 3. 修改

只能对数据库使用的字符集和校对规则进行修改，语法格式为：

```mysql
ALTER DATABASE [数据库名] { 
[ DEFAULT ] CHARACTER SET <字符集名> |
[ DEFAULT ] COLLATE <校对规则名>}
```

语法说明如下：

- ALTER DATABASE 用于更改数据库的全局特性。
- 使用 ALTER DATABASE 需要获得数据库 ALTER 权限。
- 数据库名称可以忽略，此时语句对应于默认数据库。
- CHARACTER SET 子句用于更改默认的数据库字符集。

## 4. 删除

删除数据库的语法格式为：

```mysql
DROP DATABASE [ IF EXISTS ] <数据库名>
```

执行该命令会将数据库中存储的所有数据表和数据一同删除，且不能恢复。因此最好在删除数据库之前先将数据库进行备份。

IF EXISTS 用于防止当数据库不存在时发生错误。

系统自动创建的 information_schema 和 mysql 两个数据库存放一些和数据库相关的信息，删除后MySQL 将不能正常工作。

## 5. 选择

创建数据库之后，该数据库不会自动成为当前数据库，需要用 USE 来指定当前数据库。其语法格式为：

```mysql
USE <数据库名>
```

该语句可以通知 MySQL 把 `<数据库名>` 所指示的数据库作为当前数据库。该数据库保持为默认数据库，直到语段的结尾，或者直到遇见一个不同的 USE 语句。

## 6. 其它

单行注释使用 `#`，多行注释使用 `/*……*/`

不同操作系统大小写是否区分的规则如下表

|          | Windows          | Linux |
| -------- | ---------------- | ----- |
| 数据库名 | 否（忽略大小写） | 是    |
| 表名     | 否               | 是    |
| 表别名   | 否               | 是    |
| 列名     | 否               | 否    |
| 列别名   | 否               | 否    |
| 变量名   | 否               | 是    |
