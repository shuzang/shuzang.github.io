---
title: Golang数据库编程
date: 2020-05-26
lastmod: 2020-05-28
tags: [Go实战]
categories: [Golang学习之路]
slug: Golang interact with MySQL 
---

Go 官方提供了database 包，database 包下有 sql/driver。该包用来定义操作数据库的接口，这保证了无论使用哪种数据库，操作方式都是相同的。但 Go 官方并没有提供连接数据库的 driver，如果要操作数据库，还需要第三方的 driver 包。这里介绍 [go-mysql-driver](https://github.com/go-sql-driver/mysql) 的使用。

## 1. 安装

在执行了 `go mod` 的项目目录下执行如下安装命令

```bash
$ go get -u github.com/go-sql-driver/mysql
```

Win10 下，go-sql-driver 包将被安装到 `%GOPATH%\pkg\mod\github.com\go-sql-driver\mysql@v1.5.0` 目录下，其它项目使用时不必重复下载，执行上述命令即可直接引入。

## 2. 导入

示例代码如下

```go
import (
	"database/sql"
    "fmt"
    _ "github.com/go-sql-driver/mysql"
)
```

Golang 提供了database/sql 包，用于对 SQL 数据库的访问。它提供了一系列接口方法，用于访问关系数据库但并不会提供数据库特有的方法，那些特有的方法交给数据库驱动去实现。

对于数据库操作来说，开发者不应该直接使用导入的驱动包所提供的方法，而应该使用 sql.DB 对象所提供的统一的方法。因此在导入 MySQL 驱动时，使用了匿名导入包的方式，即将 go-sql-driver 包重命名为特殊符号 `_`。采用这种方式只会执行其中的 init 函数和初始化其全局变量，无法调用函数。

## 3. 连接数据库

连接数据库使用 sql 包中的 Open() 函数，原型如下

```go
func Open(driverName, dataSourceName string) (*DB, error)
```

- driverName：使用的驱动名。这个名字其实就是数据库驱动注册到 database/sql 时所使用的名字
- dataSourceName：数据库连接信息。它包含了数据库的用户名、密码、数据库主机以及需要连接的数据库名等信息。

使用示例如下

```go
db, err := sql.Open("mysql", "用户名:密码@tcp(IP:端口)/数据库?charset=utf8")
```

sql.Open() 返回的 sql.DB 对象是 Goroutine 并发安全的。sql.DB 通过数据库驱动为开发者提供管理底层数据库连接的打开和关闭操作。sql.DB 帮助开发者管理数据库连接池。正在使用的连接被标记为繁忙，用完后回到连接池等待下次使用。所以，如果开发者没有把连接释放回连接池，会导致过多连接使系统资源耗尽。

sql.DB 的设计目标就是作为长连接（一次连接多次数据交互）使用，不宜频繁开关。比较好的做法是，为每个不同的 datastore 建一个DB 对象，保持这些对象打开。如果需要短连接（一次连接一次数据交互），就把 DB 作为参数传入函数，而不要在函数中开关。

## 4. 增删改数据

直接调用DB对象的 Exec() 方法如下所示

```go
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
```

通过db.Exec()插入数据，通过返回的 err 可知插入失败的原因，通过返回的结果可以进一步查询本次插入数据影响的行数（RowsAffected）和最后插入的ID（如果数据库支持查询最后插入ID）。事实上，Result 是对已执行的 SQL 命令的总结，类型定义如下

```go
type Result interface {
    // LastInsertId返回一个数据库生成的回应命令的整数。
    // 当插入新行时，一般来自一个"自增"列。
    // 不是所有的数据库都支持该功能，该状态的语法也各有不同。
    LastInsertId() (int64, error)

    // RowsAffected返回被update、insert或delete命令影响的行数。
    // 不是所有的数据库都支持该功能。
    RowsAffected() (int64, error)
}
```

Exec() 方法的使用方式如下所示

```go
result,err := db.Exec("INSERT INTO user_info (username, departname, created) VALUES (?,?,?)", "Steven", "区块链教学部"， "2017-10-1")
```

预编译语句（PreparedStatement）提供了诸多好处。PreparedStatement 可以实现自定义参数的查询，通常来说比手动拼接字符串SQL语句高效；PreparedStatement 还可以防止SQL注入攻击。因此，开发中应尽量使用它。

通常使用 PreparedStatement 和 Exec() 完成 INSERT、UPDATE、DELETE 操作。使用DB对象的Prepare() 方法获得预编译对象 stmt，然后调用 Exec() 方法，语法如下所示。

```go
func (db *DB) Prepare(query string) (*Stmt, error)
```

具体用法如下

```go
stmt, err := db.Prepare("INSERT INTO user_info SET username=?, departnamt=?, created=?")
result, err := stmt.Exec("Jackson", "研发部", "2017-10-1")
```

获取影响数据库的行数，可以根据该数值判断是否操作（插入、删除或修改）成功。语法如下所示。

```go
count, err := result.RowsAffected()
```

预编译对象 stmt 属于 Stmt 类型，是一个准备好的状态，可以安全地被多个 Goroutine 同时使用，类型定义与方法集如下，定义在 DB 对象上的 Exec 和 Stmt 对象上的 Exec 传入参数有区别，后者不需要 SQL 语句，但作用应该是相同的。

```go
type Stmt struct {
    // 内含隐藏或非导出字段
}

func (s *Stmt) Exec(args ...interface{}) (Result, error)
func (s *Stmt) Query(args ...interface{}) (*Rows, error)
func (s *Stmt) QueryRow(args ...interface{}) *Row
func (s *Stmt) Close() error
```

## 5. 查询数据

数据查询的一般步骤如下

1. 调用 db.Query() 方法执行 SQL 语句，此方法返回一个 Rows 作为查询结果，语法如下所示

   ```go
   func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
   ```

   注意，Rows 作为查询结果，其游标指向结果集的第零行

2. 将 rows.Next() 方法的返回值作为 for 循环的条件，迭代查询数据，语法如下所示。

   ```go
   func (rs *Rows) Next() bool
   ```

   Next 的返回值不是简单的一个对下一个结果是否存在的判断，而是准备下一行结果用于 Scan 方法进行扫描，如果准备好，返回 true，如果没有下一行或准备时出现错误，返回 false

3. 在循环中，通过 rows.Scan()方法读取每一行数据，语法如下所示。

   ```go
   func (rs *Rows) Scan(dest ...interface{}) error
   ```

   rows.Scan() 方法的参数顺序很重要，必须和查询结果的列相对应（数量和顺序都需要一致）。假设 `SELECT * From user_info where age ＞=20 AND age ＜ 30` 查询的列顺序是 `id, name, age`，和插入操作顺序相同，rows.Scan() 的参数传入也需要按照此顺序 `rows.Scan(＆id, ＆name, ＆age)`，不然会造成数据读取的错位。

4. 调用db.Close()关闭查询，Close 关闭 DB对象，释放任何打开的资源，但实际上因为 DB 句柄通常被多个 Go 协程共享，不会被关闭。

查询多行数据的一个完整示例如下所示

```go
stmt, err := db.Prepare("SELECT * FROM user_info WHERE uid<?")
rows, err := stmt.Query(10)  // 注意这里的 Query 也是 stmt 的方法
user := new(UserTable)
for rows.Next() {
    err := rows.Scan(&user.Uid, &user.Username, &user.Department, &user.Created)
    if err != nil {
        panic(err)
        continue
    }
    fmt.Println(*user)
}
```

因为 Golang 是强类型语言，所以查询数据时先定义数据类型。数据库中的数据有3种可能状态：存在值、存在零值、未赋值，因此可以将待查询的数据类型定义为 sql.NullString、sql.NullInt64 类型等。可以通过 Valid 值来判断查询到的值是赋值状态还是未赋值状态。

每次 db.Query() 操作后，都建议调用 rows.Close()。因为 db.Query() 会从数据库连接池中获取一个连接，这个底层连接在结果集（rows）未关闭前会被标记为处于繁忙状态。当遍历读到最后一条记录时，会发生一个内部 EOF 错误，自动调用 rows.Close()。但如果出现异常，提前退出循环，rows 不会关闭，连接不会回到连接池中，连接也不会关闭，则此连接会一直被占用。因此通常使用 defer rows.Close() 来确保数据库连接可以正确放回到连接池中。rows.Close() 操作是幂等操作，而一个幂等操作的特点是：其任意多次执行所产生的影响与一次执行的影响相同。所以即便对已关闭的 rows 再执行 close() 也没关系。

谈到这里，我们可以注意到 Close 方法一共有三种，分别定义在 DB、Stmt、Rows 三个方法上，用于关闭数据库连接、预准备状态和查询结果，不过本质上都是释放某个连接池中的连接。

单条数据通过 QueryRow() 方法查询，语法如下所示。

```go
func (db *DB) QueryRow(query string, args ...interface{}) *Row
```

使用示例如下

```go
var username, departname, created string
err := db.QueryRow("SELECT username, departname, created FROM user_info WHERE uid=?", 3).Scan(&username, &departname, &created)
```

## 6. 示例代码

定义一个表结构如下

```msyql
mysql> CREATE TABLE user_info (
    -> uid INT(10) NOT NULL AUTO_INCREMENT,
    -> username VARCHAR(64) DEFAULT NULL,
    -> departname VARCHAR(64) DEFAULT NULL,
    -> created DATE DEFAULT NULL,
    -> PRIMARY KEY(uid)
    -> );
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> DESC user_info;
+------------+-------------+------+-----+---------+----------------+
| Field      | Type        | Null | Key | Default | Extra          |
+------------+-------------+------+-----+---------+----------------+
| uid        | int         | NO   | PRI | NULL    | auto_increment |
| username   | varchar(64) | YES  |     | NULL    |                |
| departname | varchar(64) | YES  |     | NULL    |                |
| created    | date        | YES  |     | NULL    |                |
+------------+-------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```

编写一个完整的测试代码，使用上述提到的所有数据库操作

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

// 定义数据库连接信息
type DbConn struct {
	Dsn string // 数据库驱动字符串
	Db  *sql.DB
}

// 数据库中的用户表的映射对象
type UserTable struct {
	Uid        int
	Username   string
	Department string
	Created    string
}

func main() {
	var err error
	dbConn := DbConn{
		Dsn: "root:@tcp(127.0.0.1:3306)/testdb?charset=utf8",
	}
	dbConn.Db, err = sql.Open("mysql", dbConn.Dsn)
	if err != nil {
		fmt.Println(err)
	}
	defer dbConn.Db.Close()

	//1. 测试直接使用 DB 的 Exec() 方法增删改
	execData(&dbConn)
	//2. 测试使用预编译语句和 Exec() 方法增删改
	preExecData(&dbConn)
	//3. 查询单行数据(以最后一条数据为例)
	result := dbConn.QueryRowData("select * from user_info where uid = (select max(uid) from user_info)")
	fmt.Println(result)
	//4. 查询多行数据
	result1 := dbConn.QueryData("select * from user_info where uid<10")
	fmt.Println(len(result1))
	//5. 查询多行数据（使用预编译语句）
	result2 := dbConn.PreQueryData("select * from user_info where uid<? order by uid desc", 10)
	fmt.Println(len(result2))
	//遍历查询的结果集
	for k, v := range result2 {
		fmt.Println("uid: ", k, v)
	}
}

func execData(dbConn *DbConn) {
	count, id, err := dbConn.ExecData("insert user_info (username,departname,created) values ('Josh', 'business group','2020-06-14')")
	if err != nil {
		fmt.Println(err.Error())
	} else {
		fmt.Println("受影响的行数：", count)
		fmt.Println("新添加数据的id：", id)
	}
}

func (dbConn *DbConn) ExecData(sqlString string) (count, id int64, err error) {
	result, err := dbConn.Db.Exec(sqlString)
	if err != nil {
		fmt.Println(err)
		return
	}
	if id, err = result.LastInsertId(); err != nil {
		fmt.Println(err)
		return
	}
	if count, err = result.RowsAffected(); err != nil {
		fmt.Println(err)
		return
	}
	return count, id, nil
}

func preExecData(dbConn *DbConn) {
	count, id, err := dbConn.PreExecData("insert user_info (username,departname,created) values ('Jackson', 'Education Department','2020-06-14')")
	if err != nil {
		fmt.Println(err.Error())
	} else {
		fmt.Println("受影响的行数：", count)
		fmt.Println("新添加数据的id：", id)
	}
}

func (dbConn *DbConn) PreExecData(sqlString string, args ...interface{}) (count, id int64, err error) {
	stmt, err := dbConn.Db.Prepare(sqlString)
	defer stmt.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	result, err := stmt.Exec(args...)
	if err != nil {
		fmt.Println(err)
		return
	}
	if id, err = result.LastInsertId(); err != nil {
		fmt.Println(err)
		return
	}
	if count, err = result.RowsAffected(); err != nil {
		fmt.Println(err)
		return
	}
	return count, id, nil
}

func (dbConn *DbConn) QueryRowData(sqlString string) (data UserTable) {
	user := new(UserTable)
	err := dbConn.Db.QueryRow(sqlString).Scan(&user.Uid, &user.Username, &user.Department, &user.Created)
	if err != nil {
		fmt.Println(err)
		return
	}
	return *user
}

func (dbConn *DbConn) QueryData(sqlString string) (resultSet map[int]UserTable) {
	rows, err := dbConn.Db.Query(sqlString)
	defer rows.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	resultSet = make(map[int]UserTable)
	user := new(UserTable)
	for rows.Next() {
		err := rows.Scan(&user.Uid, &user.Username, &user.Department, &user.Created)
		if err != nil {
			fmt.Println(err)
			continue
		}
		resultSet[user.Uid] = *user
	}
	return resultSet
}

func (dbConn *DbConn) PreQueryData(sqlString string, args ...interface{}) (resultSet map[int]UserTable) {
	stmt, err := dbConn.Db.Prepare(sqlString)
	defer stmt.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	rows, err := stmt.Query(args...)
	defer rows.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	resultSet = make(map[int]UserTable)
	user := new(UserTable)
	for rows.Next() {
		err := rows.Scan(&user.Uid, &user.Username, &user.Department, &user.Created)
		if err != nil {
			fmt.Println(err)
			continue
		}
		resultSet[user.Uid] = *user
	}
	return resultSet
}
```

运行结果为

```bash
$ go run main.go
受影响的行数： 1
新添加数据的id： 2
受影响的行数： 1
新添加数据的id： 3
{3 Jackson Education Department 2020-06-14}        
3
3
uid:  3 {3 Jackson Education Department 2020-06-14}
uid:  2 {2 Josh business group 2020-06-14}
uid:  1 {1 Steven 区块链教学部 2017-10-01}
```



https://weread.qq.com/web/reader/df83279071b1ee24df86404k07c32570311607cdfd23f04