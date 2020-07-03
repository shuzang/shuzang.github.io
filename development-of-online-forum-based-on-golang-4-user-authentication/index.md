# 基于Go语言开发在线论坛4-通过Cookie和Session实现用户认证


上篇演示了首页路由、处理器、视图模板的实现，接着就要实现群组、主题的增删查改，不过，需要在认证后才能执行这些操作，所以本篇介绍如何实现用户认证相关功能。

<!--more-->

## 1. 全局辅助函数

我们先在 `handlers` 目录下创建一个 `helper.go` 文件，用于定义一些全局辅助函数（主要用在处理器中）：

```go
package handlers

import (
    "errors"
    "fmt"
    "github.com/xueyuanjun/chitchat/models"
    "html/template"
    "net/http"
)

// 通过 Cookie 判断用户是否已登录
func session(writer http.ResponseWriter, request *http.Request) (sess models.Session, err error) {
    cookie, err := request.Cookie("_cookie")
    if err == nil {
        sess = models.Session{Uuid: cookie.Value}
        if ok, _ := sess.Check(); !ok {
            err = errors.New("Invalid session")
        }
    }
    return
}

// 解析 HTML 模板（应对需要传入多个模板文件的情况，避免重复编写模板代码）
func parseTemplateFiles(filenames ...string) (t *template.Template) {
    var files []string
    t = template.New("layout")
    for _, file := range filenames {
        files = append(files, fmt.Sprintf("views/%s.html", file))
    }
    t = template.Must(t.ParseFiles(files...))
    return
}

// 生成响应 HTML
func generateHTML(writer http.ResponseWriter, data interface{}, filenames ...string) {
    var files []string
    for _, file := range filenames {
        files = append(files, fmt.Sprintf("views/%s.html", file))
    }

    templates := template.Must(template.ParseFiles(files...))
    templates.ExecuteTemplate(writer, "layout", data)
}

// 返回版本号
func Version() string {
    return "0.1"
}
```

目前提供了版本信息，判断用户是否登录，HTML 模板的解析与生成等逻辑，我们将 HTML 模板解析与生成逻辑提取出来，主要是为了避免重复编写类似的模板代码，比如现在，我们可以将 `handlers/index.go` 中的 `Index` 方法改写如下：

```go
func Index(w http.ResponseWriter, r *http.Request) {
    threads, err := models.Threads();
    if err == nil {
        generateHTML(w, threads, "layout", "navbar", "index")
    }
}
```

是不是看起来简单多了，更重要的是提高了代码的复用性。

在 `session` 函数中，通过从请求中获取指定 Cookie 字段里面存放的 Session ID，然后从 Session 存储器（这里存储驱动是数据库）查询对应 Session 是否存在来判断用户是否已认证，如果已认证则返回的 `sess` 不为空。

## 2. 用户认证相关处理器

### 2.1 编写处理器代码

接下来，在 `handlers` 目录下创建一个 `auth.go` 来存放用户认证相关处理器：

```go
package handlers

import (
    "fmt"
    "github.com/xueyuanjun/chitchat/models"
    "net/http"
)

// GET /login
// 登录页面
func Login(writer http.ResponseWriter, request *http.Request) {
    t := parseTemplateFiles("auth.layout", "navbar", "login")
    t.Execute(writer, nil)
}

// GET /signup
// 注册页面
func Signup(writer http.ResponseWriter, request *http.Request) {
    generateHTML(writer, nil, "auth.layout", "navbar", "signup")
}

// POST /signup
// 注册新用户
func SignupAccount(writer http.ResponseWriter, request *http.Request) {
    err := request.ParseForm()
    if err != nil {
        fmt.Println("Cannot parse form")
    }
    user := models.User{
        Name:     request.PostFormValue("name"),
        Email:    request.PostFormValue("email"),
        Password: request.PostFormValue("password"),
    }
    if err := user.Create(); err != nil {
        fmt.Println("Cannot create user")
    }
    http.Redirect(writer, request, "/login", 302)
}

// POST /authenticate
// 通过邮箱和密码字段对用户进行认证
func Authenticate(writer http.ResponseWriter, request *http.Request) {
    err := request.ParseForm()
    user, err := models.UserByEmail(request.PostFormValue("email"))
    if err != nil {
        fmt.Println("Cannot find user")
    }
    if user.Password == models.Encrypt(request.PostFormValue("password")) {
        session, err := user.CreateSession()
        if err != nil {
            fmt.Println("Cannot create session")
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

// GET /logout
// 用户退出
func Logout(writer http.ResponseWriter, request *http.Request) {
    cookie, err := request.Cookie("_cookie")
    if err != http.ErrNoCookie {
        fmt.Println("Failed to get cookie")
        session := models.Session{Uuid: cookie.Value}
        session.DeleteByUUID()
    }
    http.Redirect(writer, request, "/", 302)
}
```

上述代码中定义了用户注册、登录、退出相关业务逻辑，非常简单，和 Laravel 认证脚手架生成的默认认证相关控制器非常相似。

### 2.2 用户注册

用户注册逻辑比较简单，无非是填写注册表单（`Signup` 处理器方法），提交注册按钮将用户信息保存到数据库（`SignupAccount` 处理器方法）。

### 2.3 用户登录

接下来，服务端会将用户重定向到登录页面（`Login` 处理器方法），用户填写登录表单后，就可以通过 `Authenticate` 处理器方法执行认证操作。

用户认证是基于 Cookie + Session 实现的，Session 的数据结构如下所示：

```go
type Session struct {
    Id        int
    Uuid      string
    Email     string
    UserId    int
    CreatedAt time.Time
}
```

通过 `Uuid` 字段可以唯一标识这个 Session，因此可以看作是对外可见的全局 Session ID，在客户端 Cookie 存储的 Session ID 也是这个 `Uuid`。当用户认证成功之后，就会创建 Session，有了 Session 之后，就可以创建 Cookie 并写到响应中：

```go
cookie := http.Cookie{
    Name:     "_cookie",
    Value:    session.Uuid,
    HttpOnly: true,
}
http.SetCookie(writer, &cookie)
```

这样，下次用户访问在线论坛页面就会在请求头中带上包含 Session ID 的 Cookie，服务端通过解析这个 Uuid 并查询 Session 存储器（这里存储驱动是数据库）判断该用户 Session 是否存在，如果存在则用户认证通过，也就是前面辅助函数 `session` 所做的事情。

### 2.3 用户退出

上述 Cookie 未设置过期时间，所以生命周期和 Session 一致，当浏览器关闭时，Cookie 就自动删除，下次打开浏览器需要重新认证。

最后用户退出处理器方法 `Logout` 方法则是方便用户主动退出，当用户点击退出按钮，可以执行该处理器方法销毁当前用户 Session 和认证 Cookie，并将用户重定向到首页。

## 3. 用户认证相关视图模板

定义好认证处理器后，我们来编写与认证相关的视图模板，主要是登录页面和注册页面，在 `views` 目录下新增 `login.html` 编写登录页面：

```html
{{ define "content" }}

<form class="form-signin center" role="form" action="/authenticate" method="post">
  <h2 class="form-signin-heading">
    <i class="fa fa-comments-o">
      ChitChat
    </i>
  </h2>
  <input type="email" name="email" class="form-control" placeholder="Email address" required autofocus>
  <input type="password" name="password" class="form-control" placeholder="Password" required>
  <br/>
  <button class="btn btn-lg btn-primary btn-block" type="submit">Sign in</button>
  <br/>
  <a class="lead pull-right" href="/signup">Sign up</a>
</form>

{{ end }}
```

然后创建 `signup.html` 编写注册页面：

```html
{{ define "content" }}

<form class="form-signin" role="form" action="/signup_account" method="post">
  <h2 class="form-signin-heading">
    <i class="fa fa-comments-o">
      ChitChat
    </i>
  </h2>
  <div class="lead">Sign up for an account below</div>
  <input id="name" type="text" name="name" class="form-control" placeholder="Name" required autofocus>
  <input type="email" name="email" class="form-control" placeholder="Email address" required>
  <input type="password" name="password" class="form-control" placeholder="Password" required>
  <button class="btn btn-lg btn-primary btn-block" type="submit">Sign up</button>
</form>

{{ end }}
```

此外，我们还为登录和注册页面定义了单独的布局模板 `auth.layout.html`：

```html
{{ define "layout" }}

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=9">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>ChitChat</title>
    <link href="/static/css/bootstrap.min.css" rel="stylesheet">
    <link href="/static/css/font-awesome.min.css" rel="stylesheet">
    <link href="/static/css/login.css" rel="stylesheet">
  </head>
  <body>
    <div class="container">
      
      {{ template "content" . }}
      
    </div> <!-- /container -->
    
    <script src="/static/js/jquery-2.1.1.min.js"></script>
    <script src="/static/js/bootstrap.min.js"></script>
  </body>
</html>

{{ end }}
```

以上视图模板已经在认证处理器方法中引用。

## 4. 注册用户认证路由

最后，我们需要在 `routes/routes.go` 中注册用户认证相关路由：

```go
// 定义所有 Web 路由
var webRoutes = WebRoutes{
    ... // 其他路由
    {
        "signup",
        "GET",
        "/signup",
        handlers.Signup,
    },
    {
        "signupAccount",
        "POST",
        "/signup_account",
        handlers.SignupAccount,
    },
    {
        "login",
        "GET",
        "/login",
        handlers.Login,
    },
    {
        "auth",
        "POST",
        "/authenticate",
        handlers.Authenticate,
    },
    {
        "logout",
        "GET",
        "/logout",
        handlers.Logout,
    },
}
```

## 5. 测试用户认证功能

这样一来，我们就可以重启应用并访问用户注册页面 `http://localhost:8080/signup` 进行注册了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15856711017140.jpg)

注册成功后，页面会跳转到登录页面 `http://localhost:8080/login`：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15856711769523.jpg)

输入刚才填写的注册邮箱和密码，点击「SIGN IN」按钮登录成功后，页面跳转到首页。

我们还没有对首页做额外的认证判断和处理，所以此时显示的页面效果和之前一样，为了区别用户认证与未认证状态，我们可以基于认证状态渲染不同的导航模板，对于认证用户，渲染 `auth.navbar` 模板，对于未认证用户，还是保持和之前一样，为此，我们需要在 views 目录下新增 `auth.navbar.html` 视图：

```html
{{ define "navbar" }}
<div class="navbar navbar-default navbar-static-top" role="navigation">
  <div class="container">
    <div class="navbar-header">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target=".navbar-collapse">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <a class="navbar-brand" href="/">
        <i class="fa fa-comments-o"></i>
        ChitChat
      </a>
    </div>
    <div class="navbar-collapse collapse">
      <ul class="nav navbar-nav">
        <li><a href="/">Home</a></li>
      </ul>
      <ul class="nav navbar-nav navbar-right">
        <li><a href="/logout">Logout</a></li>
      </ul>
    </div>
  </div>
</div>
{{ end }}
```

同时还要修改 `handlers.Index` 处理器方法实现：

```go
func Index(writer http.ResponseWriter, request *http.Request) {
    threads, err := models.Threads();
    if err == nil {
        _, err := session(writer, request)
        if err != nil {
            generateHTML(writer, threads, "layout", "navbar", "index")
        } else {
            generateHTML(writer, threads, "layout", "auth.navbar", "index")
        }
    }
}
```

再次重启应用，刷新首页，导航条的展示效果就不一样了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15856718177791.jpg)

此时显示的是「Logout」链接，点击即可退出应用：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15856719117027.jpg)
