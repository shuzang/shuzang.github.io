---
title: 基于Go语言开发在线论坛2-通过模型类与MySQL数据库交互
author: xueyuanjun
date: 2020-05-27T18:25:20+08:00 
lastmod: 2020-06-14
tags: [Go实战]
categories: [Golang学习之路]
slug: Development of online forum based on golang 2-Interact with MySQL 
---

在本篇教程中，我们将在 MySQL 中创建一个 `chitchat` 数据库作为论坛项目的数据库。我选择了在本地安装 MySQL Server，但也可以基于 Docker 容器运行。转自学院君的教程，略有改动。

<!--more-->

## 1. 项目初始化

首先创建项目目录，命名为 `chitchat`，然后初始化目录结构如下

![初始化的目录结构](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200527_%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)

各个子目录/文件的作用介绍如下：

- `main.go`：应用入口文件
- `config.json`：全局配置文件
- `handlers`：用于存放处理器代码（可类比为 MVC 模式中的控制器目录）
- `logs`：用于存放日志文件
- `models`：用于存放与数据库交互的模型类
- `public`：用于存放前端资源文件，比如图片、CSS、JavaScript 等
- `routes`：用于存放路由文件和路由器实现代码
- `views`：用于存放视图模板文件

在 Github 网页端创建同名仓库，然后在本地执行如下命令初始化仓库（我们使用 Github 存储代码）

```bash
echo "# chitchat" >> README.md
git init
git add README.md
git commit -m "Initialize project directory"
git remote add origin https://github.com/shuzang/chitchat.git
git push -u origin master
```

然后在 `chitchat` 目录下初始化 Go 项目， 后续通过 Go Module 来管理依赖

```bash
$ go mod init github.com/shuzang/chitchat
```

## 2. 创建数据表

在 MySQL Server 中创建 `chitchat` 数据库，然后创建数据表，数据表对应的 SQL 语句如下

```mysql
create table users (
  id         serial primary key,
  uuid       varchar(64) not null unique,
  name       varchar(255),
  email      varchar(255) not null unique,
  password   varchar(255) not null,
  created_at timestamp not null
);
    
create table sessions (
  id         serial primary key,
  uuid       varchar(64) not null unique,
  email      varchar(255),
  user_id    integer references users(id),
  created_at timestamp not null
);
    
create table threads (
  id         serial primary key,
  uuid       varchar(64) not null unique,
  topic      text,
  user_id    integer references users(id),
  created_at timestamp not null
);
    
create table posts (
  id         serial primary key,
  uuid       varchar(64) not null unique,
  body       text,
  user_id    integer references users(id),
  thread_id  integer references threads(id),
  created_at timestamp not null
);
```

使用 Navicat for MySQL 进行连接测试

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200527_%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E6%B5%8B%E8%AF%95.png)

大量的语句逐条执行很容易出错，可以通过脚本形式批量执行[^sql脚本]。

[^sql脚本]:[mysql下如何执行sql脚本](https://www.cnblogs.com/kenkofox/archive/2011/01/14/1935422.html)

## 3. 与数据库交互

### 3.1 数据库驱动

数据表创建完成后，接下来，需要在 Go 应用代码中与数据库交互，Go 语言开发组并没有为此提供官方的数据库驱动实现，只是提供了数据库交互接口，我们可以通过实现这些接口的第三方扩展包完成与 MySQL 数据库的交互，本项目选择的扩展包是 [go-mysql-driver](https://github.com/go-sql-driver/mysql) 。

我们可以在 Go 应用中编写模型类基于这个扩展包提供的方法与 MySQL 交互完成增删改查操作，开始之前，可以运行如下命令安装这个依赖：

```bash
$ go get -u github.com/go-sql-driver/mysql
```

### 3.2 数据库连接

然后在 `chitchat/models` 目录下创建 `db.go`，并编写数据库连接初始化方法以及生成 UUID、哈希加密方法：

```go
package models

import (
	"crypto/rand"
	"crypto/sha256"
	"database/sql"
	"fmt"
	"log"

	_ "github.com/go-sql-driver/mysql"
)

var Db *sql.DB

func init() {
	var err error
	Db, err = sql.Open("mysql", "root:root@/chitchat?charset=utf8&parseTime=true")
	if err != nil {
		log.Fatal(err)
	}
}

// create a random UUID with from RFC 4122
// adapted from http://github.com/nu7hatch/gouuid
func createUUID() (uuid string) {
	u := new([16]byte)
	_, err := rand.Read(u[:])
	if err != nil {
		log.Fatalln("Cannot generate UUID", err)
	}

	// 0x40 is reserved variant from RFC 4122
	u[8] = (u[8] | 0x40) & 0x7F
	// Set the four most significant bits (bits 12 through 15) of the
	// time_hi_and_version field to the 4-bit version number.
	u[6] = (u[6] & 0xF) | (0x4 << 4)
	uuid = fmt.Sprintf("%x-%x-%x-%x-%x", u[0:4], u[4:6], u[6:8], u[8:10], u[10:])
	return
}

// hash plaintext with sha-256
func Encrypt(plaintext string) (cryptext string) {
	cryptext = fmt.Sprintf("%x", sha256.Sum256([]byte(plaintext)))
	return
}
```

其中，`Db` 变量代表数据库连接池，通过 `init` 方法在 Web 应用启动时自动初始化数据库连接，这样，我们就可以在应用中通过 `Db` 变量对数据库进行增删改查操作了，这也是该变量首字母大写的原因，方便在 `models` 包之外被引用，具体的操作实现我们放到独立的模型文件中处理。

注：这里通过 `sql.Open` 初始化数据库连接，我们写死了数据库连接配置，在实际生产环境，这块配置值应该从配置文件或系统环境变量获取。

### 3.3 用户模型相关类

有了代表数据库连接池的 `Db` 变量之后，就可以为每个数据表编写对应的模型类实现增删改查操作了，首先在 `models` 目录下创建 `user.go` 用于定义用户模型类 `User` 与 `users` 表进行交互，以及与 `sessions` 表进行关联：

```go
package models

import "time"

type User struct {
    Id        int
    Uuid      string
    Name      string
    Email     string
    Password  string
    CreatedAt time.Time
}

// Create a new session for an existing user
func (user *User) CreateSession() (session Session, err error) {
    statement := "insert into sessions (uuid, email, user_id, created_at) values (?, ?, ?, ?)"
    stmtin, err := Db.Prepare(statement)
    if err != nil {
        return
    }
    defer stmtin.Close()

    uuid := createUUID()
    stmtin.Exec(uuid, user.Email, user.Id, time.Now())

    stmtout, err := Db.Prepare("select id, uuid, email, user_id, created_at from sessions where uuid = ?")
    if err != nil {
        return
    }
    defer stmtout.Close()
    // use QueryRow to return a row and scan the returned id into the Session struct
    err = stmtout.QueryRow(uuid).Scan(&session.Id, &session.Uuid, &session.Email, &session.UserId, &session.CreatedAt)
    return
}

// Get the session for an existing user
func (user *User) Session() (session Session, err error) {
    session = Session{}
    err = Db.QueryRow("SELECT id, uuid, email, user_id, created_at FROM sessions WHERE user_id = ?", user.Id).
        Scan(&session.Id, &session.Uuid, &session.Email, &session.UserId, &session.CreatedAt)
    return
}

// Create a new user, save user info into the database
func (user *User) Create() (err error) {
    // Postgres does not automatically return the last insert id, because it would be wrong to assume
    // you're always using a sequence.You need to use the RETURNING keyword in your insert to get this
    // information from postgres.
    statement := "insert into users (uuid, name, email, password, created_at) values (?, ?, ?, ?, ?)"
    stmtin, err := Db.Prepare(statement)
    if err != nil {
        return
    }
    defer stmtin.Close()

    uuid := createUUID()
    stmtin.Exec(uuid, user.Name, user.Email, Encrypt(user.Password), time.Now())

    stmtout, err := Db.Prepare("select id, uuid, created_at from users where uuid = ?")
    if err != nil {
        return
    }
    defer stmtout.Close()
    // use QueryRow to return a row and scan the returned id into the User struct
    err = stmtout.QueryRow(uuid).Scan(&user.Id, &user.Uuid, &user.CreatedAt)
    return
}

// Delete user from database
func (user *User) Delete() (err error) {
    statement := "delete from users where id = ?"
    stmt, err := Db.Prepare(statement)
    if err != nil {
        return
    }
    defer stmt.Close()

    _, err = stmt.Exec(user.Id)
    return
}

// Update user information in the database
func (user *User) Update() (err error) {
    statement := "update users set name = ?, email = ? where id = ?"
    stmt, err := Db.Prepare(statement)
    if err != nil {
        return
    }
    defer stmt.Close()

    _, err = stmt.Exec(user.Name, user.Email, user.Id)
    return
}

// Delete all users from database
func UserDeleteAll() (err error) {
    statement := "delete from users"
    _, err = Db.Exec(statement)
    return
}

// Get all users in the database and returns it
func Users() (users []User, err error) {
    rows, err := Db.Query("SELECT id, uuid, name, email, password, created_at FROM users")
    if err != nil {
        return
    }
    for rows.Next() {
        user := User{}
        if err = rows.Scan(&user.Id, &user.Uuid, &user.Name, &user.Email, &user.Password, &user.CreatedAt); err != nil {
            return
        }
        users = append(users, user)
    }
    rows.Close()
    return
}

// Get a single user given the email
func UserByEmail(email string) (user User, err error) {
    user = User{}
    err = Db.QueryRow("SELECT id, uuid, name, email, password, created_at FROM users WHERE email = ?", email).
        Scan(&user.Id, &user.Uuid, &user.Name, &user.Email, &user.Password, &user.CreatedAt)
    return
}

// Get a single user given the UUID
func UserByUUID(uuid string) (user User, err error) {
    user = User{}
    err = Db.QueryRow("SELECT id, uuid, name, email, password, created_at FROM users WHERE uuid = ?", uuid).
        Scan(&user.Id, &user.Uuid, &user.Name, &user.Email, &user.Password, &user.CreatedAt)
    return
}
```

创建 `session.go` 用于定义会话模型类 `Session`：

```go
package models

import "time"

type Session struct {
    Id        int
    Uuid      string
    Email     string
    UserId    int
    CreatedAt time.Time
}

// Check if session is valid in the database
func (session *Session) Check() (valid bool, err error) {
    err = Db.QueryRow("SELECT id, uuid, email, user_id, created_at FROM sessions WHERE uuid = ?", session.Uuid).
        Scan(&session.Id, &session.Uuid, &session.Email, &session.UserId, &session.CreatedAt)
    if err != nil {
        valid = false
        return
    }
    if session.Id != 0 {
        valid = true
    }
    return
}

// Delete session from database
func (session *Session) DeleteByUUID() (err error) {
    statement := "delete from sessions where uuid = ?"
    stmt, err := Db.Prepare(statement)
    if err != nil {
        return
    }
    defer stmt.Close()

    _, err = stmt.Exec(session.Uuid)
    return
}

// Get the user from the session
func (session *Session) User() (user User, err error) {
    user = User{}
    err = Db.QueryRow("SELECT id, uuid, name, email, created_at FROM users WHERE id = ?", session.UserId).
        Scan(&user.Id, &user.Uuid, &user.Name, &user.Email, &user.CreatedAt)
    return
}

// Delete all sessions from database
func SessionDeleteAll() (err error) {
    statement := "delete from sessions"
    _, err = Db.Exec(statement)
    return
}
```

这里面定义了基于 `Db` 数据库连接实例实现用户模型和会话模型相关的增删改查操作，具体的语法可以参考 `go-mysql-driver` 的 [官方文档](https://github.com/go-sql-driver/mysql)。

### 3.4 主题模型相关类

编写好用户相关模型类后，接下来在同级目录下创建 `thread.go`，定义群组模型类 `Thread` 与 `threads` 表进行交互：

```go
package models

import "time"

type Thread struct {
    Id        int
    Uuid      string
    Topic     string
    UserId    int
    CreatedAt time.Time
}

// format the CreatedAt date to display nicely on the screen
func (thread *Thread) CreatedAtDate() string {
    return thread.CreatedAt.Format("Jan 2, 2006 at 3:04pm")
}

// get the number of posts in a thread
func (thread *Thread) NumReplies() (count int) {
    rows, err := Db.Query("SELECT count(*) FROM posts where thread_id = ?", thread.Id)
    if err != nil {
        return
    }
    for rows.Next() {
        if err = rows.Scan(&count); err != nil {
            return
        }
    }
    rows.Close()
    return
}

// get posts to a thread
func (thread *Thread) Posts() (posts []Post, err error) {
    rows, err := Db.Query("SELECT id, uuid, body, user_id, thread_id, created_at FROM posts where thread_id = ?", thread.Id)
    if err != nil {
        return
    }
    for rows.Next() {
        post := Post{}
        if err = rows.Scan(&post.Id, &post.Uuid, &post.Body, &post.UserId, &post.ThreadId, &post.CreatedAt); err != nil {
            return
        }
        posts = append(posts, post)
    }
    rows.Close()
    return
}

// Get all threads in the database and returns it
func Threads() (threads []Thread, err error) {
    rows, err := Db.Query("SELECT id, uuid, topic, user_id, created_at FROM threads ORDER BY created_at DESC")
    if err != nil {
        return
    }
    for rows.Next() {
        conv := Thread{}
        if err = rows.Scan(&conv.Id, &conv.Uuid, &conv.Topic, &conv.UserId, &conv.CreatedAt); err != nil {
            return
        }
        threads = append(threads, conv)
    }
    rows.Close()
    return
}

// Get a thread by the UUID
func ThreadByUUID(uuid string) (conv Thread, err error) {
    conv = Thread{}
    err = Db.QueryRow("SELECT id, uuid, topic, user_id, created_at FROM threads WHERE uuid = ?", uuid).
        Scan(&conv.Id, &conv.Uuid, &conv.Topic, &conv.UserId, &conv.CreatedAt)
    return
}

// Get the user who started this thread
func (thread *Thread) User() (user User) {
    user = User{}
    Db.QueryRow("SELECT id, uuid, name, email, created_at FROM users WHERE id = ?", thread.UserId).
        Scan(&user.Id, &user.Uuid, &user.Name, &user.Email, &user.CreatedAt)
    return
}
```

以及 `post.go` 编写主题模型类与 `posts` 表进行交互：

```go
package models

import "time"

type Post struct {
    Id        int
    Uuid      string
    Body      string
    UserId    int
    ThreadId  int
    CreatedAt time.Time
}

func (post *Post) CreatedAtDate() string {
    return post.CreatedAt.Format("Jan 2, 2006 at 3:04pm")
}

// Get the user who wrote the post
func (post *Post) User() (user User) {
    user = User{}
    Db.QueryRow("SELECT id, uuid, name, email, created_at FROM users WHERE id = ?", post.UserId).
        Scan(&user.Id, &user.Uuid, &user.Name, &user.Email, &user.CreatedAt)
    return
}
```

此外，我们到 `user.go` 中为 `User` 模型新增如下两个方法与 `Thread`、`Post` 模型进行关联，用于创建新的群组和主题：

```go
// Create a new thread
func (user *User) CreateThread(topic string) (conv Thread, err error) {
    statement := "insert into threads (uuid, topic, user_id, created_at) values (?, ?, ?, ?)"
    stmtin, err := Db.Prepare(statement)
    if err != nil {
        return
    }
    defer stmtin.Close()

    uuid := createUUID()
    stmtin.Exec(uuid, topic, user.Id, time.Now())

    stmtout, err := Db.Prepare("select id, uuid, topic, user_id, created_at from threads where uuid = ?")
    if err != nil {
        return
    }
    defer stmtout.Close()

    // use QueryRow to return a row and scan the returned id into the Session struct
    err = stmtout.QueryRow(uuid).Scan(&conv.Id, &conv.Uuid, &conv.Topic, &conv.UserId, &conv.CreatedAt)
    return
}

// Create a new post to a thread
func (user *User) CreatePost(conv Thread, body string) (post Post, err error) {
    statement := "insert into posts (uuid, body, user_id, thread_id, created_at) values (?, ?, ?, ?, ?)"
    stmtin, err := Db.Prepare(statement)
    if err != nil {
        return
    }
    defer stmtin.Close()

    uuid := createUUID()
    stmtin.Exec(uuid, body, user.Id, conv.Id, time.Now())

    stmtout, err := Db.Prepare("select id, uuid, body, user_id, thread_id, created_at from posts where uuid = ?")
    if err != nil {
        return
    }
    defer stmtout.Close()

    // use QueryRow to return a row and scan the returned id into the Session struct
    err = stmtout.QueryRow(uuid).Scan(&post.Id, &post.Uuid, &post.Body, &post.UserId, &post.ThreadId, &post.CreatedAt)
    return
}
```

## 4. 小结

在上述编写的模型类中，模型类与数据表的映射由 `go-mysql-driver` 底层实现，每次从数据库查询到结果之后，可以通过 `Scan` 方法将数据表字段值映射到对应的结构体模型类，而将模型类保存到数据库时，又可以基于字段映射关系将结构体属性值转化为对应的数据表字段值。

底层数据库交互逻辑定义好了之后，接下来，我们就可以编写上层实现代码了，下一篇介绍在线论坛项目上层路由和处理器方法的实现。