# Mysql学习1-入门


本文介绍数据库和 MySQL的基础知识。得庆幸当初数据库课学的还可以，大部分内容看一眼都能想起来，不需要去理解，就是单纯的忘了一些语句写法。

<!--more-->

## 1. 关系型数据库

关系型数据库是建立在关系模型基础上的数据库，借助于集合代数等数学概念和方法来处理数据库中的数据。简单说，关系型数据库是由多张能互相连接的表组成的数据库。

#### 优点

1. 都是使用表结构，格式一致，易于维护。
2. 使用通用的 SQL 语言操作，使用方便，可用于复杂查询。
3. 数据存储在磁盘中，安全。

#### 缺点

1. 读写性能比较差，不能满足海量数据的高效率读写。
2. 不节省空间。因为建立在关系模型上，就要遵循某些规则，比如数据中某字段值即使为空仍要分配空间。
3. 固定的表结构，灵活度较低。

常见的关系型数据库有 Oracle、DB2、PostgreSQL、Microsoft SQL Server、Microsoft Access 和 MySQL 等。

## 2. NoSQL

非关系型数据库被称为 [NoSQL](http://m.biancheng.net/nosql/)（Not Only SQL )，意为不仅仅是 SQL。通常指数据以对象的形式存储在数据库中，而对象之间的关系通过每个对象自身的属性来决定。

#### 优点

1. 非关系型数据库存储数据的格式可以是 key-value 形式、文档形式、图片形式等。使用灵活，应用场景广泛，而关系型数据库则只支持基础类型。
2. 速度快，效率高。 NoSQL 可以使用硬盘或者随机存储器作为载体，而关系型数据库只能使用硬盘。
3. 海量数据的维护和处理非常轻松。
4. 非关系型数据库具有扩展简单、高并发、高稳定性、成本低廉的优势。
5. 可以实现数据的分布式处理。

#### 缺点

1. 非关系型数据库暂时不提供 SQL 支持，学习和使用成本较高。
2. 非关系数据库没有事务处理，没有保证数据的完整性和安全性。适合处理海量数据，但是不一定安全。
3. 功能没有关系型数据库完善。

常见的非关系型数据库有 Neo4j、[MongoDB](http://m.biancheng.net/mongodb/)、[Redis](http://m.biancheng.net/redis/)、Memcached、MemcacheDB 和 [HBase](http://m.biancheng.net/hbase/) 等。

## 3. MySQL

MySQL 是最流行的数据库之一，是一个免费开源的关系型数据库管理系统，由瑞典 MySQL AB 公司开发，目前属于 Oracle 公司。MySQL 适合中小型软件，被个人用户以及中小企业青睐。

针对不同的用户，MySQL 分为两个版本：

- MySQL Community Server（社区版）：该版本是自由下载且完全免费的，但是官方不提供技术支持。
- MySQL Enterprise Server（企业版）：该版本是收费的，而且不能下载，但是该版本拥有完善的技术支持

MySQL 的命名机制由 3 个数字和 1 个后缀组成，例如 mysql-5.7.20：

- 第 1 个数字“5”是主版本号，用于描述文件的格式，所有版本 5 的发行版都有相同的文件夹格式。
- 第 2 个数字“7”是发行级别，主版本号和发行级别组合在一起便构成了发行序列号。
- 第 3 个数字“20”是在此发行系列的版本号，随每次新发行的版本递增。通常选择已经发行的最新版本。

MySQL 的主要特点就是免费，并且在任何平台上都能使用，占用空间相对较小。

MySQL 是 C/S 架构，有 Client 和 Server 两部分，可以都安装在一台电脑上，也可以独立安装。

学习 MySQL 必须掌握的知识点如下：

- MySQL 的下载安装。熟悉 MySQL 的配置文件，目录结构。
- MySQL 服务器的启动，登录与退出。
- MySQL 常用命令及语法规范。
- MySQL 数据类型与数据表的操作。例如，数据表的增删改查、单表查询、多表查询等。
- MySQL 运算符和函数，例如，日期函数，时间函数，信息函数，聚合函数，加密函数，自定义函数等。
- MySQL 存储过程，存储过程的调用。
- MySQL 各个存储引擎的特点，如何选择合适的存储引擎等。
- MySQL 事务的概念和使用等。
- MySQL 权限管理和用户管理等。

## 4. 安装与配置

官方下载页面：https://dev.mysql.com/downloads/

Windows 中 配置环境变量为 `C：\Program Files\MySQL\MySQL Server 5.7\bin`(默认安装路径)，配置后才能在普通命令行中使用。

启动和停机命令为 `net start DB` 和 `net stop DB`，其中 `DB` 为自己的 MySQL 服务器名，在安装时指定。

使用 `mysql -h 127.0.0.1 -u root -p` 登录

- -h 后面的参数是服务器的主机地址，从服务器登录省略该参数
- -u 后面跟登录数据库的用户名称
- -p 后面是用户登录密码，中间没有空格，也可以置空随后跟随提示输入

```mysql
PS C:\Users\lylw1> mysql -u root -p
Enter password: ****
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

这些说明性语句介绍如下：

- Commands end with; or\g：说明 mysql 命令行下的命令是以分号（;）或“\g”来结束的
- Your MySQL connection id is 13：id 表示 MySQL 数据库的连接次数。
- Server version: 8.0.19 MySQL Community Server - GPL：Server version 后面说明数据库的版本，Community 表示该版本是社区版。
- Type 'help;' or '\h' for help：表示输入”help;“或者”\h“可以看到帮助信息。
- Type '\c' to clear the current input statement：表示遇到”\c“就清除前面的命令。

Linux 以 Ubuntu 为例，官网下载 deb 包，使用 `dpkg` 命令安装

```bash
$ sudo dpkg -i mysql-apt-config*
```

`mysql-apt-config*` 为包名，使用如下命令验证安装

```bash
$ sudo systemctl status mysql.service
```

如果服务没有启动，使用如下命令

```bash
$ sudo systemctl start mysql.service
```

## 5. 图形化管理工具

常用的图形化管理工具有 [MySQL Workbench](http://dev.mysql.com/downloads/workbench/) 和 [Navicat](http://www.avicat.com/)，前者是官方提供的工具，但后者比较好用。

## 6. SQL

对数据库进行查询和修改操作的语言叫做 SQL（Structured Query Language，结构化查询语言）。SQL 语言是目前广泛使用的关系数据库标准语言，是各种数据库交互方式的基础。SQL包括四部分

1. 数据定义语言（Data Definition Language，DDL）：用来创建或删除数据库以及表等对象，主要包含以下几种命令：
   - DROP：删除数据库和表等对象
   - CREATE：创建数据库和表等对象
   - ALTER：修改数据库和表等对象的结构

2. 数据操作语言（Data Manipulation Language，DML）：用来变更表中的记录，主要包含以下几种命令：- 
   - SELECT：查询表中的数据
   - INSERT：向表中插入新数据
   - UPDATE：更新表中的数据
   - DELETE：删除表中的数据

3. 数据查询语言（Data Query Language，DQL）：用来查询表中的记录，主要包含 SELECT 命令，来查询表中的数据。

4. 数据控制语言（Data Control Language，DCL）：用来确认或者取消对数据库中的数据进行的变更。除此之外，还可以对数据库中的用户设定权限。主要包含以下几种命令：
   - GRANT：赋予用户操作权限
   - REVOKE：取消用户的操作权限
   - COMMIT：确认对数据库中的数据进行的变更
   - ROLLBACK：取消对数据库中的数据进行的变更

SQL 的几个易错书写规则如下

1. 以分号 `;` 结尾
2. SQL 语句不区分大小写
3. 单词需要半角空格或换行来分隔
