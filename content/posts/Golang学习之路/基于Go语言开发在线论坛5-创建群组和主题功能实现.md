---
title: 基于Go语言开发在线论坛5-创建群组和主题功能实现
author: xueyuanjun
date: 2020-06-05T11:41:00+08:00 
lastmod: 2020-06-05
tags: [Go实战]
categories: [Golang学习之路]
slug: Development of online forum based on golang 5-Create thread and post 
---

上篇文章基于 Cookie + Session 实现了简单的用户认证功能，用户认证之后，就可以创建群组和主题了，本篇介绍如何创建群组和主题，并将其渲染到前端页面。

<!--more-->

## 1. 群组的创建和浏览

### 1.1 处理器方法

在 `handlers` 目录下新增 `thread.go` 编写群组创建与获取方法：

```go
package handlers

import (
    "fmt"
    "github.com/xueyuanjun/chitchat/models"
    "net/http"
)

// GET /threads/new
// 创建群组页面
func NewThread(writer http.ResponseWriter, request *http.Request) {
    _, err := session(writer, request)
    if err != nil {
        http.Redirect(writer, request, "/login", 302)
    } else {
        generateHTML(writer, nil, "layout", "auth.navbar", "new.thread")
    }
}

// POST /thread/create
// 执行群组创建逻辑
func CreateThread(writer http.ResponseWriter, request *http.Request) {
    sess, err := session(writer, request)
    if err != nil {
        http.Redirect(writer, request, "/login", 302)
    } else {
        err = request.ParseForm()
        if err != nil {
            fmt.Println("Cannot parse form")
        }
        user, err := sess.User()
        if err != nil {
            fmt.Println("Cannot get user from session")
        }
        topic := request.PostFormValue("topic")
        if _, err := user.CreateThread(topic); err != nil {
            fmt.Println("Cannot create thread")
        }
        http.Redirect(writer, request, "/", 302)
    }
}

// GET /thread/read
// 通过 ID 渲染指定群组页面
func ReadThread(writer http.ResponseWriter, request *http.Request) {
    vals := request.URL.Query()
    uuid := vals.Get("id")
    thread, err := models.ThreadByUUID(uuid)
    if err != nil {
        fmt.Println("Cannot read thread")
    } else {
        _, err := session(writer, request)
        if err != nil {
            generateHTML(writer, &thread, "layout", "navbar", "thread")
        } else {
            generateHTML(writer, &thread, "layout", "auth.navbar", "auth.thread")
        }
    }
}
```

其中定义了三个方法，分别用于渲染群组创建表单页面、处理提交表单执行群组创建逻辑、以及根据指定 ID 渲染对应群组页面。前两个方法需要认证后才能访问，否则将用户重定向到登录页，群组详情页不需要认证即可访问，不过会根据是否认证返回不同的视图模板。

在这里，仍然通过辅助函数 `session` 判断用户是否认证，其他的业务逻辑也都非常简单，无非是获取表单输入、查询数据库、写入数据库、返回响应视图等操作，后面我们会在介绍处理 HTTP 请求时详细解释其中的细节，这里，我们先了解下全貌即可。

### 1.2 视图模板

然后我们需要创建几个新的视图模板，在 `views` 目录下 `new.thread.html` 来编写创建群组表单：

```html
{{ define "content" }}

	<form role="form" action="/thread/create" method="post">
		<div class="lead">Start a new thread with the following topic</div>
		<div class="form-group">
			<textarea class="form-control" name="topic" id="topic" placeholder="Thread topic here" rows="4"></textarea>
			<br/>
			<br/>
			<button class="btn btn-lg btn-primary pull-right" type="submit">Start this thread</button>
		</div>
	</form>

{{ end }}
```

然后创建 `thread.html` 编写未认证情况下渲染的群组详情页视图（其中还包含了对群组主题的遍历和渲染）：

```HTML
{{ define "content" }}

<div class="panel panel-default">
  <div class="panel-heading">
    <span class="lead"> <i class="fa fa-comment-o"></i> {{ .Topic }}</span>
    <div class="pull-right">
      Started by {{ .User.Name }} - {{ .CreatedAtDate }}
    </div>

  </div>
  
  {{ range .Posts }}
  <div class="panel-body">
    <span class="lead"> <i class="fa fa-comment"></i> {{ .Body }}</span>
    <div class="pull-right">
      {{ .User.Name }} - {{ .CreatedAtDate }}
    </div>    
  </div>
  {{ end }}    

</div>

{{ end }}
```

以及 `auth.thread.html` 编写认证后的群组详情页视图（在未认证视图模板的基础上新增了提交主题的表单区块）：

```html
{{ define "content" }}

	<div class="panel panel-default">
		<div class="panel-heading">
			<span class="lead"> <i class="fa fa-comment-o"></i> {{ .Topic }}</span>
			<div class="pull-right">
				Started by {{ .User.Name }} - {{ .CreatedAtDate }}
			</div>

		</div>

        {{ range .Posts }}
			<div class="panel-body">
				<span class="lead"> <i class="fa fa-comment"></i> {{ .Body }}</span>
				<div class="pull-right">
                    {{ .User.Name }} - {{ .CreatedAtDate }}
				</div>
			</div>
        {{ end }}

	</div>

	<div class="panel panel-info">
		<div class="panel-body">
			<form role="form" action="/thread/post" method="post">
				<div class="form-group">
					<textarea class="form-control" name="body" id="body" placeholder="Write your reply here" rows="3"></textarea>
					<input type="hidden" name="uuid" value="{{ .Uuid }}">
					<br/>
					<button class="btn btn-primary pull-right" type="submit">Reply</button>
				</div>
			</form>
		</div>
	</div>

{{ end }}
```

### 1.3 注册路由

最后在 `routes/routes.go` 中注册群组相关路由：

```go
var webRoutes = WebRoutes{
    ... // 其他路由
    {
            "newThread",
            "GET",
            "/thread/new",
            handlers.NewThread,
    },
    {
        "createThread",
        "POST",
        "/thread/create",
        handlers.CreateThread,
    },
    {
        "readThread",
        "GET",
        "/thread/read",
        handlers.ReadThread,
    },
}
```

### 1.4 测试群组创建和浏览

这样，我们就完成了在线论坛项目群组创建和浏览的所有相关路由、处理器、视图编码，重新启动 HTTP 服务器，就可以在首页点击「Start a thread」链接创建新的群组了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15858871392315.jpg)

如果没有登录，会先跳转到登录页面，登录之后再次点击该链接就可以进入群组创建页面：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15858873399471.jpg)

我们在输入框中输入群组主题「Golang」并点击右下角提交按钮，就可以成功创建一个新的群组并在首页看到了：

！![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15858874341284.jpg)

然后，我们可以点击该群组的「Read more」链接进入群组详情页：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15858874780161.jpg)

目前还没有任何主题，接下来，我们来编写创建主题的后端处理器方法和路由实现。

## 2. 创建新主题

### 2.1 处理器方法

我们在 `handlers` 目录下新增 `post.go` 来存放主题相关处理器方法：

```go
package handlers

import (
    "fmt"
    "github.com/xueyuanjun/chitchat/models"
    "net/http"
)

// POST /thread/post
// 在指定群组下创建新主题
func PostThread(writer http.ResponseWriter, request *http.Request) {
    sess, err := session(writer, request)
    if err != nil {
        http.Redirect(writer, request, "/login", 302)
    } else {
        err = request.ParseForm()
        if err != nil {
            fmt.Println("Cannot parse form")
        }
        user, err := sess.User()
        if err != nil {
            fmt.Println("Cannot get user from session")
        }
        body := request.PostFormValue("body")
        uuid := request.PostFormValue("uuid")
        thread, err := models.ThreadByUUID(uuid)
        if err != nil {
            fmt.Println("Cannot read thread")
        }
        if _, err := user.CreatePost(thread, body); err != nil {
            fmt.Println("Cannot create post")
        }
        url := fmt.Sprint("/thread/read?id=", uuid)
        http.Redirect(writer, request, url, 302)
    }
}
```

我们只定义了一个创建主题的处理器方法，在该处理器方法中，仍然会验证用户是否已认证，只有认证用户才能创建主题，我们最后会调用 `user.CreatePost` 方法根据群组 ID、用户 ID 和主题内容创建新的主题记录，保存成功后，会返回创建该主题的群组详情页，并将与该群组关联的所有主题渲染出来。关于数据库和视图模板引擎的语法细节，后面我们会在对应的独立教程中详细介绍。

### 2.2 注册路由

由于主题没有独立的视图模板，所以我们只需要在路由文件中注册创建主题对应的路由就可以了：

```go
{
    "postThread",
    "POST",
    "/thread/post",
    handlers.PostThread,
},
```

### 2.3 测试主题创建

再次重启 HTTP 服务器，就可以在之前的群组详情页创建新主题了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15858883014649.jpg)

点击「REPLY」按钮提交，页面会刷新并渲染主题内容：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15858883342441.jpg)

回到论坛首页，你可以看到每个群组下的主题数目：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15858883971308.jpg)