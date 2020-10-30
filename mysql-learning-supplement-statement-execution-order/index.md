# Mysql学习补充-语句执行顺序


这篇文章用来仔细思考 MySQL 查询语句的执行顺序。

<!--more-->

手写 MySQL  语句的顺序通常如下

```mysql
select <select_list>
from <table_name>
<join_type> join <join_table> on <join_condition>
where <where_condition>
group by <group_by_list>
having <having_condition>
order by <order_by_condition>
limit <limt_number>
```

MySQL 语句的执行顺序如下

```mysql
from <left table>
on <on_condition>
<join_type> join <join_table>
where <where_condition>
group by <group_by_list>
<sum()avg()等聚合函数>
having <having_condition>
select <select_list>
distinct
order by <order_by_condition>
limit <limit_number>
```

下面做一下解释

**第一步**：如果有连接运算，加载 from 子句指定的表中的前两个计算笛卡尔积，生成虚拟表 vt1；

**第二步**：对虚拟表 vt1 执行 on 表达式，筛选符合条件的元组，生成虚拟表 vt2，如果是外连接，基表中的数据会全部保留；

**第三步**：如果 from 子句中的表数量大于 2，则重复前两部，直至所有的表都连接完成，得到虚拟表 vt3；

**第四步**：执行 where 表达式，筛选符合条件的数据生成 vt4；

**第五步**：执行 group by 子句。group by 子句执行过后，会对子句组合成唯一值并且对每个唯一值只包含一行，生成 vt5,。一旦执行group by，后面的所有步骤只能得到 vt5 中的列（group by的子句包含的列）和聚合函数。

**第六步**：执行聚合函数，生成 vt6；

**第七步**：执行 having 表达式，筛选 vt6 中的数据，生成vt7。having是唯一一个在分组后的条件筛选;

**第八步**：执行 select 语句，从 vt7 中筛选列，生成 vt8；

**第九步**：执行 distinct，对 vt8 去重，生成 vt9。如果执行过 group by 就没必要再去执行 distinct，因为分组后，每组只会有一条数据，并且每条数据都不相同。

**第十步**：对 vt9 进行排序，此处返回的不是一个虚拟表，而是一个游标，记录了数据的排序顺序，此处可以使用别名；

**第十一步**：执行 limit 语句，将结果返回给客户端。

----



**参考**

[1] 樱桃mayue，51CTO博客，[MySql学习笔记（二）：SQL执行顺序](https://blog.51cto.com/13593129/2357192?source=dra)，2019.03.02


