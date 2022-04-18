# 基于Go语言开发在线论坛6-日志与错误处理


到现在为止，我们已经完成了在线论坛项目基本功能的开发，对 Go 语言 Web 编程中如何实现 MVC 架构模式以及 CRUD（数据库增删改查）基本操作有了初步的认识。不过现在所有的日志和错误处理都是杂糅在业务代码中，本篇介绍如何对它们统一进行处理，使得业务代码和日志及错误处理逻辑分离。

<!--more-->

## 1. 日志处理

### 1.1 初始化日志处理器

首先来看日志处理，在 `handlers/helper.go` 中，新增如下日志处理器初始化代码：

```go
import (
    "log"
    "os"
)

var logger *log.Logger

func init()  {
    file, err := os.OpenFile("logs/chitchat.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalln("Failed to open log file", err)
    }
    logger = log.New(file, "INFO ", log.Ldate|log.Ltime|log.Lshortfile)
}
```

这里我们借助 Go 官方提供的 `log` 包进行日志处理，首先声明一个 `*log.Logger` 类型的 `logger` 变量作为日志处理器，以便可以全局使用。默认的日志文件位于 `logs/chitchat.log`，我们通过 `os.OpenFile` 打开这个日志文件句柄，如果文件不存在，则自动创建。然后我们通过 `log.New` 初始化日志处理器并赋值给 `logger`，该方法需要传入日志文件、默认日志级别、以及日志格式，关于该方法的细节，我们后面在日志章节会详细介绍。

### 1.2 定义日志函数

然后我们就可以通过 `logger` 这个日志处理器来记录日志了，在 `helper.go` 中新增如下几个日志函数：

```go
func info(args ...interface{}) {
    logger.SetPrefix("INFO ")
    logger.Println(args...)
}
    
// 为什么不命名为 error？避免和 error 类型重名
func danger(args ...interface{}) {
    logger.SetPrefix("ERROR ")
    logger.Println(args...)
}
    
func warning(args ...interface{}) {
    logger.SetPrefix("WARNING ")
    logger.Println(args...)
}
```

非常简单，我们定义了三个日志函数来记录三个日志级别，分别是 INFO（普通）、ERROR（错误）、WARNING（警告），然后通过调用 `logger.Println` 传入参数记录日志信息到日志文件即可，这里的参数类型是 `...interface{}`，表示可以传入参数支持任意类型、任意个数。

### 1.3 重构业务代码

接下来，我们到业务处理器中，将原来的日志打印代码都重构为调用对应的日志函数，以 `handlers/auth.go` 为例，修改日志处理代码如下：

```go
// src/github.com/xueyuanjun/chitchat/handlers/auth.go
// 注册新用户
func SignupAccount(writer http.ResponseWriter, request *http.Request) {
    err := request.ParseForm()
    if err != nil {
        danger(err, "Cannot parse form")
    }
    user := models.User{
        Name:     request.PostFormValue("name"),
        Email:    request.PostFormValue("email"),
        Password: request.PostFormValue("password"),
    }
    if err := user.Create(); err != nil {
        danger(err, "Cannot create user")
    }
    http.Redirect(writer, request, "/login", 302)
}

// 用户认证
func Authenticate(writer http.ResponseWriter, request *http.Request) {
    err := request.ParseForm()
    user, err := models.UserByEmail(request.PostFormValue("email"))
    if err != nil {
        danger(err, "Cannot find user")
    }
    if user.Password == models.Encrypt(request.PostFormValue("password")) {
        session, err := user.CreateSession()
        if err != nil {
            danger(err, "Cannot create session")
        }
        cookie := http.Cookie{
            Name:     "_cookie",
            Value:    session.Uuid,
            HttpOnly: true,
        }
        http.SetCookie(writer, &cookie)
        http.Redirect(writer, request, "/", 302)
    } else {
        http.Redirect(writer, request, "/login", 302)
    }
}

// 用户退出
func Logout(writer http.ResponseWriter, request *http.Request) {
    cookie, err := request.Cookie("_cookie")
    if err != http.ErrNoCookie {
        warning(err, "Failed to get cookie")
        session := models.Session{Uuid: cookie.Value}
        session.DeleteByUUID()
    }
    http.Redirect(writer, request, "/", 302)
}
```

其他处理器方法参照这个示例进行调整即可

## 2. 错误处理

Go 语言并没有像 PHP、Java 那样提供异常这种类型，只有 `error` 和 `panic`，对于 Go Web 应用中的错误处理，不影响程序继续往后执行的，可以通过日志方式记录下来，如果某些错误导致程序无法往后执行，比如浏览群组详情页，对应群组不存在，这个时候，我们就应该直接返回 404 响应或者将用户重定向到 404 页面，而不能继续往后执行，对于这种错误，只能通过单独的处理逻辑进行处理，这种错误类似于 Laravel 中的中断异常处理。

### 2.1 重定向到错误页面

在这个项目中，我们通过重定向到错误页面的方式处理这种类型的错误，在 `handlers/helper.go` 中新增 `error_message` 函数：

```go
// 异常处理统一重定向到错误页面
func error_message(writer http.ResponseWriter, request *http.Request, msg string) {
    url := []string{"/err?msg=", msg}
    http.Redirect(writer, request, strings.Join(url, ""), 302)
}
```

调用该方法会将用户重定向到错误处理页面（由 `err` 路由对应处理器方法渲染），响应状态码为 302，并且带上错误消息 `msg`，以便客户端感知错误原因。

### 2.2 编写错误页面相关代码

为此，我们还要编写用于处理应用出错的路由、处理器和视图实现。

**处理器方法**

首先在 `handlers/index.go` 中编写全局的、渲染错误页面的处理器方法：

```go
func Err(writer http.ResponseWriter, request *http.Request)  {
    vals := request.URL.Query()
    _, err := session(writer, request)
    if err != nil {
        generateHTML(writer, vals.Get("msg"), "layout", "navbar", "error")
    } else {
        generateHTML(writer, vals.Get("msg"), "layout", "auth.navbar", "error")
    }
}
```

我们可以通过 `vals.Get` 方法从查询字符串获取 `msg` 参数，并将其渲染到错误视图 `error.html` 中。

**错误视图**

然后在 `views` 目录下新增 `error.html` 用来定义错误视图：

```go
{{ define "content" }}
    
<p class="lead">{{ . }}</p>
    
{{ end }}
```

非常简单，只是通过 `{{ . }}` 获取 `msg` 变量的值并渲染出来。

**注册路由**

最后在 `routes/routes.go` 中注册错误路由：

```go
{
    "error",
    "GET",
    "/err",
    handlers.Err,
},
```

### 2.3 重构业务代码

在必要的地方调用错误处理函数 `error_message` 将用户重定向到错误页面，比如在 `handlers/thread.go` 中，在浏览群组详情页时，如果指定 ID 对应群组不存在，则将用户重定向到错误页面：

```go
// 通过 ID 渲染指定群组页面
func ReadThread(writer http.ResponseWriter, request *http.Request) {
    vals := request.URL.Query()
    uuid := vals.Get("id")
    thread, err := models.ThreadByUUID(uuid)
    if err != nil {
        error_message(writer, request, "Cannot read thread")
    } else {
        ...
    }
}
```

又比如 `handlers/post.go` 中，在创建新主题时，如果获取不到主题归属的群组，则将用户重定向到错误页面：

```go
// 在指定群组下创建新主题
func PostThread(writer http.ResponseWriter, request *http.Request) {
    sess, err := session(writer, request)
    if err != nil {
        http.Redirect(writer, request, "/login", 302)
    } else {
        ... 
        thread, err := models.ThreadByUUID(uuid)
        if err != nil {
            error_message(writer, request, "Cannot read thread")
        }
        ...
    }
}
```

## 3. 整体测试

至此，我们已经完成了日志和错误统一处理的代码重构，接下来，可以进行简单的测试，重启 HTTP 服务器，访问应用首页，此时会引入 `helper.go`，执行 `init` 方法，创建日志文件，我们试图使用错误的用户名密码登录：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15862426490215.jpg)

测试就可以在 `logs/chitchat.log` 中看到错误日志了：

```go
ERROR 2020/04/07 14:55:39 helper.go:71: sql: no rows in result set Cannot find user
```

接下来，我们访问一个不存在的群组 `http://localhost:8080/thread/read?id=100`，页面就会重定向到错误页面：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15862427655179.jpg)
