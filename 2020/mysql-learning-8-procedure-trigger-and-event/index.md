# Mysql学习8-存储过程、触发器和事件


存储过程是在数据库中定义一些 SQL 语句的集合，可以直接调用这些存储过程来执行已经定义好的 SQL 语句。避免了开发人员重复编写相同 SQL 语句的问题。

触发器和存储过程相似，都是嵌入到 MySQL 中的一段程序。触发器是由事件来触发某个操作。当数据库执行这些事件时，就会激活触发器来执行相应的操作。

本篇介绍这两个概念

<!--more-->

## 1. 存储过程

数据库的实际操作中，经常会有需要多条 SQL 语句处理多个表才能完成的操作。例如，为了确认学生能否毕业，需要同时查询学生档案表、成绩表和综合表，此时就需要使用多条 SQL 语句来针对这几个数据表完成处理要求。**存储过程**就是这样一组为了完成特定功能的 SQL 语句的集合。

使用存储过程的目的是将常用或复杂的工作预先用 SQL 语句写好并用一个指定名称存储起来，这个过程经编译和优化后存储在数据库服务器中，因此称为存储过程。当以后需要数据库提供与已定义好的存储过程的功能相同的服务时，只需调用“CALL存储过程名字”即可自动完成

常用操作数据库的 SQL 语句在执行的时候需要先编译，然后执行。存储过程则采用另一种方式来执行 SQL 语句。

一个存储过程是一个可编程的函数，它在数据库中创建并保存，一般由 SQL 语句和一些特殊的控制结构组成。当希望在不同的应用程序或平台上执行相同的特定功能时，存储过程尤为合适。

MySQL 5.0 版本以前并不支持存储过程，这使 MySQL 在应用上大打折扣。MySQL 从 5.0 版本开始支持存储过程，既提高了数据库的处理速度，同时也提高了数据库编程的灵活性

存储过程是数据库中的一个重要功能，存储过程可以用来转换数据、数据迁移、制作报表，它类似于编程语言，一次执行成功，就可以随时被调用，完成指定的功能操作。

使用存储过程不仅可以提高数据库的访问效率，同时也可以提高数据库使用的安全性。

对于调用者来说，存储过程封装了 SQL 语句，调用者无需考虑逻辑功能的具体实现过程。只是简单调用即可，它可以被 Java 和 C# 等编程语言调用。

### 1.1 创建存储过程

创建存储过程使用 CREATE PROCEDURE 语句，语法格式如下

```mysql
CREATE PROCEDURE <过程名> ( [过程参数[,…] ] ) <过程体>
```

其中，**过程参数**是存储过程的参数列表。MySQL 存储过程支持三种类型的参数，即输入参数、输出参数和输入/输出参数，分别用 IN、OUT 和 INOUT 三个关键字标识。其中，输入参数可以传递给一个存储过程，输出参数用于存储过程需要返回一个操作结果的情形，而输入/输出参数既可以充当输入参数也可以充当输出参数。格式如下

```mysql
[ IN | OUT | INOUT ] <参数名> <类型>
```

**过程体**是存储过程的主体部分，包含在过程调用的时候必须执行的 SQL 语句。这个部分以关键字 **BEGIN** 开始，以关键字 **END** 结束。若存储过程体中只有一条 SQL 语句，则可以省略 BEGIN-END 标志。

在 MySQL 中，服务器处理 SQL 语句默认是以分号作为语句结束标志的。然而，在创建存储过程时，存储过程体可能包含有多条 SQL 语句，这些 SQL 语句如果仍以分号作为语句结束符，那么 MySQL 服务器在处理时会以遇到的第一条 SQL 语句结尾处的分号作为整个程序的结束符，而不再去处理存储过程体中后面的 SQL 语句，这样显然不行。为解决以上问题，通常使用 **DELIMITER** 命令将结束命令修改为其他字符。语法格式如下：

```mysql
DELIMITER $$
```

语法说明如下：

- $$ 是用户定义的结束符，通常这个符号可以是一些特殊的符号，如两个“?”或两个“￥”等。
- 当使用 DELIMITER 命令时，应该避免使用反斜杠“\”字符，因为它是 MySQL 的转义字符。

在 MySQL 命令行客户端输入如下 SQL 语句。

```mysql
mysql > DELIMITER ??
```

成功执行这条 SQL 语句后，任何命令、语句或程序的结束标志就换为两个问号“??”了。

若希望换回默认的分号“;”作为结束标志，则在 MySQL 命令行客户端输入下列语句即可：

```mysql
mysql > DELIMITER ;
```

注意：DELIMITER 和分号“;”之间一定要有一个空格。在创建存储过程时，必须具有 CREATE ROUTINE 权限。

假如创建名称为 GetScoreByStu 的存储过程，输入参数是学生姓名。存储过程的作用是通过输入的学生姓名从学生成绩信息表中查询指定学生的成绩信息，输入的 SQL 语句和执行过程如下所示。

```mysql
mysql> DELIMITER //
mysql> CREATE PROCEDURE GetScoreByStu
    -> (IN name VARCHAR(30))
    -> BEGIN
    -> SELECT student_score FROM tb_students_score
    -> WHERE student_name=name;
    -> END //
Query OK, 0 rows affected (0.01 sec)
```

### 1.2 查看存储过程

创建好存储过程后，可以使用 SHOW CREATE 语句查看存储过程的定义，语法格式如下

```mysql
SHOW PROCEDURE STATUS LIKE 存储过程名;
```

举例如下

```mysql
mysql> SHOW PROCEDURE STATUS LIKE 'showstuscore' \G
*************************** 1. row ***************************
                  Db: test
                Name: showstuscore
                Type: PROCEDURE
             Definer: root@localhost
            Modified: 2020-02-20 13:34:50
             Created: 2020-02-20 13:34:50
       Security_type: DEFINER
             Comment:
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.01 sec)
```

也可以查看存储过程的定义，语法格式如下

```mysql
SHOW CREATE PROCEDURE 存储过程名
```

举例

```mysql
mysql> SHOW CREATE PROCEDURE showstuscore \G
*************************** 1. row ***************************
           Procedure: showstuscore
            sql_mode: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `showstuscore`()
BEGIN
SELECT id,name,score FROM studentinfo;
END
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.01 sec)
```

### 1.3 修改存储过程

使用 ALTER PROCEDURE 语句修改存储过程，语法格式如下

```mysql
ALTER PROCEDURE 存储过程名 [ 特征 ... ]
```

`特征`指定了存储过程的特性，可能的取值有：

- CONTAINS SQL 表示子程序包含 SQL 语句，但不包含读或写数据的语句。
- NO SQL 表示子程序中不包含 SQL 语句。
- READS SQL DATA 表示子程序中包含读数据的语句。
- MODIFIES SQL DATA 表示子程序中包含写数据的语句。
- SQL SECURITY { DEFINER |INVOKER } 指明谁有权限来执行。
- DEFINER 表示只有定义者自己才能够执行。
- INVOKER 表示调用者可以执行。
- COMMENT 'string' 表示注释信息。

下面修改存储过程 showstuscore 的定义，将读写权限改为 MODIFIES SQL DATA，并指明调用者可以执行，代码如下：

```mysql
mysql> ALTER PROCEDURE showstuscore MODIFIES SQL DATA SQL SECURITY INVOKER;
Query OK, 0 rows affected (0.01 sec)
```

### 1.4 删除存储过程

存储过程被创建后，就会一直保存在数据库服务器上，直至被删除。当 MySQL 数据库中存在废弃的存储过程时，我们需要将它从数据库中删除。

使用 DROP PROCEDURE 语句删除存储过程，语法格式如下

```mysql
DROP PROCEDURE [ IF EXISTS ] <过程名>
```

注意：存储过程名称后面没有参数列表，也没有括号，在删除之前，必须确认该存储过程没有任何依赖关系，否则会导致其他与之关联的存储过程无法运行。

下面删除存储过程 ShowStuScore，SQL 语句和运行结果如下：

```mysql
mysql> DROP PROCEDURE ShowStuScore;
Query OK, 0 rows affected (0.08 sec)
```

## 2. 触发器

MySQL 的触发器和存储过程一样，都是嵌入到 MySQL 中的一段程序，是 MySQL 中管理数据的有力工具。不同的是执行存储过程要使用 CALL 语句来调用，而触发器的执行不需要使用 CALL 语句来调用，也不需要手工启动，而是通过对数据表的相关操作来触发、激活从而实现执行。比如当对 student 表进行操作（INSERT，DELETE 或 UPDATE）时就会激活它执行。

触发器与数据表关系密切，主要用于保护表中的数据。特别是当有多个表具有一定的相互联系的时候，触发器能够让不同的表保持数据的一致性。

在 MySQL 中，只有执行 INSERT、UPDATE 和 DELETE 操作时才能激活触发器，其它 SQL 语句则不会激活触发器。

那么为什么要使用触发器呢？比如，在实际开发项目时，我们经常会遇到以下情况：

- 在学生表中添加一条关于学生的记录时，学生的总数就必须同时改变。
- 增加一条学生记录时，需要检查年龄是否符合范围要求。
- 删除一条学生信息时，需要删除其成绩表上的对应记录。
- 删除一条数据时，需要在数据库存档表中保留一个备份副本。


虽然上述情况实现的业务逻辑不同，但是它们都需要在数据表发生更改时，自动进行一些处理。这时就可以使用触发器处理。例如，对于第一种情况，可以创建一个触发器对象，每当添加一条学生记录时，就执行一次计算学生总数的操作，这样就可以保证每次添加一条学生记录后，学生总数和学生记录数是一致的。

### 2.1 优缺点

触发器的优点如下：

- 触发器的执行是自动的，当对触发器相关表的数据做出相应的修改后立即执行。
- 触发器可以实施比 FOREIGN KEY 约束、CHECK 约束更为复杂的检查和操作。
- 触发器可以实现表数据的级联更改，在一定程度上保证了数据的完整性。


触发器的缺点如下：

- 使用触发器实现的业务逻辑在出现问题时很难进行定位，特别是涉及到多个触发器的情况下，会使后期维护变得困难。
- 大量使用触发器容易导致代码结构被打乱，增加了程序的复杂性，
- 如果需要变动的数据量较大时，触发器的执行效率会非常低。

### 2.2 MySQL支持的触发器

在实际使用中，MySQL 所支持的触发器有三种：INSERT 触发器、UPDATE 触发器和 DELETE 触发器。

#### INSERT 触发器

在 INSERT 语句执行之前或之后响应的触发器。

使用 INSERT 触发器需要注意以下几点：

- 在 INSERT 触发器代码内，可引用一个名为 NEW（不区分大小写）的虚拟表来访问被插入的行。
- 在 BEFORE INSERT 触发器中，NEW 中的值也可以被更新，即允许更改被插入的值（只要具有对应的操作权限）。
- 对于 AUTO_INCREMENT 列，NEW 在 INSERT 执行之前包含的值是 0，在 INSERT 执行之后将包含新的自动生成值。

#### UPDATE 触发器

在 UPDATE 语句执行之前或之后响应的触发器。

使用 UPDATE 触发器需要注意以下几点：

- 在 UPDATE 触发器代码内，可引用一个名为 NEW（不区分大小写）的虚拟表来访问更新的值。
- 在 UPDATE 触发器代码内，可引用一个名为 OLD（不区分大小写）的虚拟表来访问 UPDATE 语句执行前的值。
- 在 BEFORE UPDATE 触发器中，NEW 中的值可能也被更新，即允许更改将要用于 UPDATE 语句中的值（只要具有对应的操作权限）。
- OLD 中的值全部是只读的，不能被更新。

注意：当触发器设计对触发表自身的更新操作时，只能使用 BEFORE 类型的触发器，AFTER 类型的触发器将不被允许。

#### DELETE 触发器

在 DELETE 语句执行之前或之后响应的触发器。

使用 DELETE 触发器需要注意以下几点：

- 在 DELETE 触发器代码内，可以引用一个名为 OLD（不区分大小写）的虚拟表来访问被删除的行。
- OLD 中的值全部是只读的，不能被更新。

总体来说，触发器使用的过程中，MySQL 会按照以下方式来处理错误。

对于事务性表，如果触发程序失败，以及由此导致的整个语句失败，那么该语句所执行的所有更改将回滚；对于非事务性表，则不能执行此类回滚，即使语句失败，失败之前所做的任何更改依然有效。

若 BEFORE 触发程序失败，则 MySQL 将不执行相应行上的操作。

若在 BEFORE 或 AFTER 触发程序的执行过程中出现错误，则将导致调用触发程序的整个语句失败。

仅当 BEFORE 触发程序和行操作均已被成功执行，MySQL 才会执行 AFTER 触发程序。

## 3. 事件

在数据库管理中，经常要周期性的执行某一命令或 SQL 语句，于是 MySQL 5.1 版本以后就提供了事件，它可以很方便的实现 MySQL 数据库的计划任务，定期运行指定命令，使用起来非常简单方便。

事件（Event）也可称为事件调度器（Event Scheduler），是用来执行定时任务的一组 SQL 集合，可以通俗理解成 MySQL 中的定时器。一个事件可调用一次，也可周期性的启动。

事件可以作为定时任务调度器，取代部分原来只能用操作系统的计划任务才能执行的工作。另外，更值得一提的是，MySQL 的事件可以实现每秒钟执行一个任务，非常适合对实时性要求较高的环境，而操作系统的计划任务只能精确到每分钟一次。

事件和触发器类似，都是在某些事情发生时启动。当数据库启动一条语句的时候，触发器就启动了，而事件是根据调度事件来启动的。由于他们彼此相似，所以事件也称为临时性触发器。

### 3.1 查看事件是否开启

在 MySQL 中，调度器 event_scheduler 负责调用事件。我们可以通过以下几种命令查看事件是否开启，一般情况下默认值为 OFF。SQL 命令和运行结果如下：

```
mysql> SHOW VARIABLES LIKE 'event_scheduler';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| event_scheduler | OFF   |
+-----------------+-------+
1 row in set, 1 warning (0.02 sec)

mysql> SELECT @@event_scheduler;
+-------------------+
| @@event_scheduler |
+-------------------+
| OFF               |
+-------------------+
1 row in set (0.00 sec)

mysql> SHOW PROCESSLIST;
+----+------+-----------------+------+---------+------+----------+------------------+
| Id | User | Host            | db   | Command | Time | State    | Info             |
+----+------+-----------------+------+---------+------+----------+------------------+
|  2 | root | localhost:56279 | NULL | Query   |    0 | starting | SHOW PROCESSLIST |
+----+------+-----------------+------+---------+------+----------+------------------+
1 row in set (0.01 sec)
```

从结果可以看出，事件没有开启。因为参数 event_scheduler 的值为 OFF，并且在 PROCESSLIST 中查看不到 event_scheduler 的信息。如果参数 event_scheduler 的值为 ON，或者在 PROCESSLIST 中显示了 event_scheduler 的信息，则说明事件已经开启。

### 3.2 开启事件

开启事件主要通过以下两种方式实现。 

#### 通过设置全局参数修改

可以使用 SET GLOBAL 命令设定全局变量 event_scheduler 的值，开启或关闭事件。将 event_scheduler 参数的值设置为 ON，表示开启事件；设置为 OFF，则关闭事件。

例如，要开启事件可以在命令行窗口中输入以下命令。

```
mysql> SET GLOBAL event_scheduler = ON ;
Query OK, 0 rows affected (0.06 sec)

mysql> SHOW VARIABLES LIKE 'event_scheduler';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| event_scheduler | ON    |
+-----------------+-------+
1 row in set, 1 warning (0.01 sec)
```

结果显示，event_scheduler 的值为 ON，表示事件已经开启。


通过 SET GLOBAL 命令开启或关闭事件，MySQL 重启服务后事件又会回到原来的状态，如果想要始终开启或关闭事件，可以修改 MySQL 配置文件。

#### 更改配置文件

在 MySQL 配置文件中找到 [mysqld] 选项，然后在下面添加以下代码开启事件。

event_scheduler = ON

在配置文件中添加代码并保存文件后，重启 MySQL 服务才能生效。

通过该方法开启或关闭事件，重启 MySQL 服务后，不会回到原来的状态。例如，此时重启 MySQL 服务器，然后查看事件是否开启。

```
mysql> SHOW VARIABLES LIKE 'event_scheduler';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| event_scheduler | ON    |
+-----------------+-------+
1 row in set, 1 warning (0.01 sec)
```

结果显示，参数 event_scheduler 的值为 ON，表示已经开启。
