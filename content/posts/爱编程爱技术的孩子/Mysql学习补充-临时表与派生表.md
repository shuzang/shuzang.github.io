---
title: Mysql学习补充-临时表与派生表
date: 2020-10-13T19:15:00+08:00
lastmod: 2020-10-13
tags: [计算机基础]
categories: [爱编程爱技术的孩子]
slug: mysql learning supplement-temporary table and derived table 
---

区分 MySQL 中临时表与派生表的概念，明确它们的用途。

<!--more-->

## 1. 临时表

本节参考 [易百教程-MySQL临时表](https://www.yiibai.com/mysql/temporary-table.html)

### 1.1 简介

就像它的名字，临时表是一个临时的结果集，一般在多表连接时使用，存储一个临时的结果以便另一个查询来处理。

临时表的一些注意如下

- 在 `CREATE` 和 `TABLE` 中间添加 `TEMPORARY` 关键字来创建临时表，即 `CREATE TEMPOPARY TABLE`；
- 连接结束时，临时表会被自动删除，当然，也可以使用 DROP TABLE 显式删除。注意，这里的连接结束指的是数据库连接，当开发时使用连接池或者持久连接时，无法保证临时表在程序终止时自动删除，因为程序结束时连接不一定结束，而是会放到连接池；
- 临时表只能被创建它的客户端看到和访问，因此不同的客户端可以创建具有相同名称的临时表，不会导致冲突；
- 临时表可以与数据库中的普通表（永久表）具有相同的名称，但会屏蔽掉永久表，只有临时表被删除后永久表才能再次访问。但不建议这样做，因为如果服务器断线重连，将无法区分临时表和永久表，此时发起 DELETE TABLE 可能会删除掉永久表。

### 1.2 创建

如上所述，使用 `CREATE TEMPORARY TABLE` 创建临时表。下例中创建了一个临时表，按照收入存储前 10 名客户

```mysql
CREATE TEMPORARY TABLE top10customers
SELECT p.customerNumber, 
       c.customerName, 
       FORMAT(SUM(p.amount),2) total
FROM payments p
INNER JOIN customers c ON c.customerNumber = p.customerNumber
GROUP BY p.customerNumber
ORDER BY total DESC
LIMIT 10;
```

从创建的临时表中查询数据如下

```mysql
SELECT * FROM top10customers;
```

### 1.3 删除

使用 `DROP TEMPORARY TABLE` 删除临时表，如下

```mysql
DROP TEMPORARY TABLE top10customers;
```

关键词 `TEMPORARY` 可以省略，但为了避免删除永久表的错误，最好还是加上该关键词。

## 2. 派生表

本节参考 [易百教程-MySQL派生](https://www.yiibai.com/mysql/derived-table.html)

当在 SELECT 语句的 FROM 子句中使用独立子查询时，将其称为派生表。

```mysql
SELECT column_list
FROM ( # 圆括号中的子查询结果即为派生表
	SELECT column_list
    FROM table_1
) derived_table_name # 派生表必须具有别名
WHERE derived_table_name.c1 > 0;
```

派生表必须具有别名，以便在稍后的查询中引用，否则，MySQL 将给出如下错误

```mysql
Every derived table must have its own alias.
```

所以，我们可以看到，派生表不需要像临时表那样需要创建。