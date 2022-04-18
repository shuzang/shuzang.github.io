---
title: 基于Go语言开发在线论坛3-访问论坛首页
author: xueyuanjun
date: 2020-05-29T09:08:00+08:00 
lastmod: 2020-06-14
tags: [Go实战]
categories: [Golang学习之路]
slug: Development of online forum based on golang 3-Visit Forum Homepage 
---

前两篇分别介绍了整体设计及数据表的创建、模型类的编写，本篇了解如何在服务端处理用户请求，并启动论坛首页。文章转自学院君的教程，略有改动。

<!--more-->

![首页视图](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200529_Jietu20200330-021118.jpg)

用户请求的处理流程如下：

1. 客户端发送请求；
2. 服务端路由器（multiplexer）将请求分发给指定处理器（handler）；
3. 处理器处理请求，完成对应的业务逻辑；
4. 处理器调用模板引擎生成 HTML 并将响应返回给客户端。

接下来我们按照这个流程来编写服务端代码。

## 1. 路由器定义

这里我们基于 [gorilla/mux](https://github.com/gorilla/mux) 来实现路由器，所以需要安装对应依赖：

```bash
$ go get -u github.com/gorilla/mux
```

将路由器定义在 `routes` 目录下的 `router.go` 中

```go
package routes

import "github.com/gorilla/mux"

// 返回一个 mux.Router 类型指针，从而可以当作处理器使用
func NewRouter() *mux.Router {

    // 创建 mux.Router 路由器示例
    router := mux.NewRouter().StrictSlash(true)

    // 遍历 web.go 中定义的所有 webRoutes
    for _, route := range webRoutes {
        // 将每个 web 路由应用到路由器
        router.Methods(route.Method).
            Path(route.Pattern).
            Name(route.Name).
            Handler(route.HandlerFunc)
    }

    return router
}
```

将所有路由定义在同一目录的 `routes.go` 中：

```go
package routes

import "net/http"

// 定义一个 WebRoute 结构体用于存放单个路由
type WebRoute struct {
    Name        string
    Method      string
    Pattern     string
    HandlerFunc http.HandlerFunc
}

// 声明 WebRoutes 切片存放所有 Web 路由
type WebRoutes []WebRoute

// 定义所有 Web 路由
var webRoutes = WebRoutes{

}
```

## 2. 启动HTTP服务器

最后在项目根目录下的 `main.go` 中引入上述路由器来启动 HTTP 服务器：

```go
package main

import (
    . "github.com/xueyuanjun/chitchat/routes"
    "log"
    "net/http"
)

func main()  {
    startWebServer("8080")
}

// 通过指定端口启动 Web 服务器
func startWebServer(port string)  {
    r := NewRouter()
    http.Handle("/", r)  // 通过 router.go 中定义的路由器来分发请求

    log.Println("Starting HTTP service at " + port)
    err := http.ListenAndServe(":" + port, nil) // 启动协程监听请求

    if err != nil {
        log.Println("An error occured starting HTTP listener at port " + port)
        log.Println("Error: " + err.Error())
    }
}
```

这里我们指定 HTTP 服务器监听 `8080` 端口，使用的路由器正是上述 `router.go` 中 `NewRouter` 方法返回的 `mux.Router` 指针类型实例，这里可以看到引用的时候并没有带上包名前缀，之所以可以这么做是因为通过特殊符号 `.` 引入了 `routes`包，通过这种方式引入的包可以直接调用包中对外可见的变量、方法和结构体，而不需要加上包名前缀。

## 3. 处理静态资源

在线论坛涉及到前端静态资源文件的处理，我们可以在 `startWebServer` 方法中新增如下这两行代码：

```go
r := NewRouter() // 通过 router.go 中定义的路由器来分发请求

// 处理静态资源文件
assets := http.FileServer(http.Dir("public"))
r.PathPrefix("/static/").Handler(http.StripPrefix("/static/", assets))

http.Handle("/", r) // 应用路由器到 HTTP 服务器

...
```

其中 `http.FileServer` 用于初始化文件服务器和目录为当前目录下的 `public` 目录。

然后在第二段代码中指定静态资源路由及处理逻辑：将 `/static/` 前缀的 URL 请求去除 `static` 前缀，然后在文件服务器查找指定文件路径是否存在（`public` 目录下的相对地址）。

比如 URL 请求路径为 `http://localhost:8080/static/css/bootstrap.min.css`，对应的查找路径是：

```go
<application root>/public/css/bootstrap.min.css
```

对于静态资源文件直接返回文件内容，不会进行额外处理。

## 4. 编写处理器实现

### 4.1 首页处理器方法

做好上述准备工作后，接下来，我们来创建论坛首页的路由处理器，在 `handlers` 目录下新增一个 `index.go` 来定义首页的处理器方法：

```go
package handlers

import (
    "github.com/xueyuanjun/chitchat/models"
    "html/template"
    "net/http"
)

// 论坛首页路由处理器方法
func Index(w http.ResponseWriter, r *http.Request) {
    files := []string{"views/layout.html", "views/navbar.html", "views/index.html",}
    templates := template.Must(template.ParseFiles(files...))
    threads, err := models.Threads();
    if err == nil {
        templates.ExecuteTemplate(w, "layout", threads)
    }
}
```

### 4.2 创建视图模板

这里我们使用 Go 自带的 `html/template` 作为模板引擎，需要传入位于 `views` 目录下的视图模板文件，这里传入了多个模板文件，包括主布局文件 `layout.html`：

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
	</head>
	<body>
    {{ template "navbar" . }}

	<div class="container">

        {{ template "content" . }}

	</div> <!-- /container -->

	<script src="/static/js/jquery-2.1.1.min.js"></script>
	<script src="/static/js/bootstrap.min.js"></script>
	</body>
	</html>

{{ end }}
```

顶部导航模板 `navbar.html`：

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
					<li><a href="/login">Login</a></li>
				</ul>
			</div>
		</div>
	</div>
{{ end }}
```

以及首页视图模板 `index.html`：

```html
{{ define "content" }}
	<p class="lead">
		<a href="/thread/new">Start a thread</a> or join one below!
	</p>

    {{ range . }}
		<div class="panel panel-default">
			<div class="panel-heading">
				<span class="lead"> <i class="fa fa-comment-o"></i> {{ .Topic }}</span>
			</div>
			<div class="panel-body">
				Started by {{ .User.Name }} - {{ .CreatedAtDate }} - {{ .NumReplies }} posts.
				<div class="pull-right">
					<a href="/thread/read?id={{.Uuid }}">Read more</a>
				</div>
			</div>
		</div>
    {{ end }}

{{ end }}
```

引入多个视图模板是为了提高模板代码的复用性，因为对于同一个应用的不同页面来说，可能基本布局、页面顶部导航和页面底部组件都是一样的，关于视图模板的细节，我们在后面视图模板部分会详细介绍，这里简单了解下即可。

### 4.3 渲染视图模板

我们可以从数据库查询群组数据并将该数据传递到模板文件，最后将模板视图渲染出来，对应代码如下：

```go
threads, err := models.Threads();
if err == nil {
    templates.ExecuteTemplate(w, "layout", threads)
}
```

编译多个视图模板时，默认以第一个模板名作为最终视图模板名，所以这里第二个参数传入的是 `layout`，第三个参数传入要渲染的数据 `threads`，对应的渲染逻辑位于 `views/index.html` 中：

```go
{{ range . }}
	<div class="panel panel-default">
		<div class="panel-heading">
			<span class="lead"> <i class="fa fa-comment-o"></i> {{ .Topic }}</span>
		</div>
		<div class="panel-body">
			Started by {{ .User.Name }} - {{ .CreatedAtDate }} - {{ .NumReplies }} posts.
			<div class="pull-right">
				<a href="/thread/read?id={{.Uuid }}">Read more</a>
			</div>
		</div>
	</div>
{{ end }}
```

其中 `{{ range . }}` 表示将处理器方法传入的变量，这里是 `threads` 进行循环。

### 4.4 注册首页路由

最好，我们在 `routes/routes.go` 中注册首页路由及对应的处理器方法 `Index`：

```go
import "github.com/xueyuanjun/chitchat/handlers"

// 定义所有 Web 路由
var webRoutes = WebRoutes{
    {
        "home",
        "GET",
        "/",
        handlers.Index,
    },
}
```

## 5. 访问论坛首页

访问论坛首页之前，我们将相应的前端资源文件拷贝到 `public` 目录下，此时项目整体目录结构如下：

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200529_%E7%9B%AE%E5%BD%95%E6%A0%BC%E5%BC%8F.png)

然后我们在项目根目录下运行如下代码启动 HTTP 服务器

```bash
$ go run main.go
```

然后我们在浏览器访问论坛首页 `http://localhost:8080`：

![首页](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200529_%E9%A6%96%E9%A1%B5.png)