---
title: Golang Web开发1 HTTP服务器
date: 2020-01-27
tags: [Golang]
categories: [爱编程爱技术的孩子]
---

学自[无闻](https://github.com/unknwon?tab=repositories)大佬的 Go Web 基础课程[^1]，本系列为学习笔记，很多内容可能会直接复制过来。

[^1]:https://github.com/unknwon/building-web-applications-in-go

之后有一个关于建立博客的实战[^blog]

[^blog]:https://study.163.com/course/courseLearn.htm?courseId=328001#/learn/video?lessonId=442046&courseId=328001

第一篇首先学习如何在 Go 语言中启动一个 HTTP 服务器用于接受和响应来自客户端的 HTTP 请求。虽然 Web 应用协议不止于 HTTP（HyperText Transfer Protocol），还包括常见的 Socket、WebSocket 和 SPDY 等等，但 HTTP 是当下最简单和最常见的交互形式。与其它语言所不同的是，Go 语言的标准库自带了一系列结构和方法来帮助开发者简化 HTTP 服务开发的相关流程。因此，我们不需要依赖任何第三方组件就能构建并启动一个高并发的 HTTP 服务器。

## Hello world!

第一步，就让我们使用 Go 语言来搭建一个 HTTP 版的 ”Hello world“ 程序吧！

我们先创建一个名为 `http_server.go` 的文件，然后输入以下代码（*示例文件 [http_server.go](https://github.com/unknwon/building-web-applications-in-go/blob/master/listings/01/http_server.go)*）：

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello world!"))
	})

	log.Println("Starting HTTP server...")
	log.Fatal(http.ListenAndServe("localhost:4000", nil))
}
```

当我们通过终端运行这段代码的时候，就能够看到如下输出：

```go
➜ go run http_server.go
2018/11/25 07:34:49 Starting HTTP server...
```

此时，我们的 HTTP 服务器已经监听在本机的 4000 端口，并准备好接受来自客户端的请求了。有两种方式可以验证这个小程序的正确性：终端调用 cURL 或者打开浏览器访问。

调用 cURL 的命令非常简单，回车运行之后即可看到服务端响应的字符串 ”Hello world!“：

```bash
➜ curl http://localhost:4000
Hello world!
```

同样的，如果直接打开浏览器访问地址 [http://localhost:4000](http://localhost:4000/) 的话，我们也可以看到一样的字符串：

[![img](https://github.com/unknwon/building-web-applications-in-go/raw/master/images/01-http-browser-hello-word.png)](https://github.com/unknwon/building-web-applications-in-go/blob/master/images/01-http-browser-hello-word.png)

是不是感觉几个简单的步骤就实现了一个完全可用的 HTTP 服务器？这就是 Go 语言的魅力之一！

接下来，让我们分析一下这段代码具体都做了什么事情。

我们需要明白这里有三个关键点：

1. [`http.HandleFunc`](https://gowalker.org/net/http#HandleFunc) 函数的作用是将某一个函数与一个路由规则(路由就是URL到函数的映射[^2])进行绑定，当用户访问指定路由时（某个路由规则匹配成功），所绑定的函数就会被执行。它接受两个参数，第一个参数就是指定的路由规则，本例中我们使用 `/` 来表示根路径；第二个参数就是与该路由进行绑定的函数。

   [^2]:https://zhuanlan.zhihu.com/p/24814675

2. [`http.HandleFunc`](https://gowalker.org/net/http#HandleFunc) 的第二个参数必须符合函数签名 `func(http.ResponseWriter, *http.Request)`，这个函数同样接受两个参数，第一个参数是请求所对应的响应对象 [`http.ResponseWriter`](https://gowalker.org/net/http#ResponseWriter)，包括响应码（Response Code）、响应头（Response Header）和响应体（Response Body）等等，我们就是通过调用这个对象的 `Write` 方法向响应体写入 ”Hello world!“ 字符串的；第二个参数则是请求所对应的请求对象 [`*http.Request`](https://gowalker.org/net/http#Request)，该对象包含当前这个 HTTP 请求所有的信息，包括请求头（Request Header）、请求体（Request Body）和其它相关的内容。

3. [`http.ListenAndServe`](https://gowalker.org/net/http#ListenAndServe) 函数的作用就是启动 HTTP 服务器，并监听发送到指定地址和端口号的 HTTP 请求，本例中我们要求 HTTP 服务器监听并接受发送到地址 localhost 且端口号为 4000 的 HTTP 请求。这个函数也接受两个参数，我们目前只使用到了第一个参数，即监听地址和端口号；第二个参数会在后文讲解，因此暂时可以使用 nil 作为它的值。另外，如果监听地址为 127.0.0.1 或者 localhost，则可以使用更加简洁的写法，即 `http.ListenAndServe(":4000", nil) `。

除此之外，你可能已经注意到为了节省代码行数，我们在这段代码中使用了匿名函数来编写 HTTP 请求的处理逻辑。这在编写简单的逻辑时非常方便，但当逻辑处理较为复杂时，应该定义一个独立的函数以提升代码的可读性。我们可以将这段代码等价地转化为如下形式（*示例文件 [http_server_2.go](https://github.com/unknwon/building-web-applications-in-go/blob/master/listings/01/http_server_2.go)*）：

```
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", hello)

	log.Println("Starting HTTP server...")
	log.Fatal(http.ListenAndServe(":4000", nil))
}

func hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello world!"))
}
```

重新运行这段代码在输出结果上并没有任何不同，只是代码结构稍稍改变了。

实际上，`http.HandleFunc` 也是标准库提供给用户的一种简便写法，它的第二个参数的函数签名必须为 `func(http.ResponseWriter, *http.Request)` 是因为在 `http.HandleFunc` 函数内部会将我们传入的绑定函数转化为类型 [`http.HandlerFunc`](https://gowalker.org/net/http#HandlerFunc)，即一个 Go 语言中标准的 HTTP 请求处理器对象，这个对象类型实现了 [`http.Handler`](https://gowalker.org/net/http#Handler) 接口：

```
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

通过 `http.Handler` 的接口定义我们发现，函数签名 `func(http.ResponseWriter, *http.Request)` 的由来是因为要实现接口的 `ServeHTTP` 方法。

现在我们知道了 `http.HandleFunc` 的根本作用是将一个函数转化为一个实现了 `http.Handler` 接口的类型（`http.HandlerFunc`），那么我们可不可以自己创建一个类型并实现 `http.Handler` 接口呢？答案当然是肯定的。

一模一样的功能，下面的代码使用了更加复杂的用法（*示例文件 [http_server_3.go](https://github.com/unknwon/building-web-applications-in-go/blob/master/listings/01/http_server_3.go)*）：

```
package main

import (
	"log"
	"net/http"
)

func main() {
	http.Handle("/", &helloHandler{})

	log.Println("Starting HTTP server...")
	log.Fatal(http.ListenAndServe(":4000", nil))
}

type helloHandler struct{}

func (*helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello world!"))
}
```

这段代码不再使用 `http.HandleFunc` 函数，取而代之的是直接调用 [`http.Handle`](https://gowalker.org/net/http#Handle) 并传入我们自定义的 `http.Handler` 实现。

初学者有一点需要特别注意，即 Go 语言是一门大小写敏感的语言（否则无法通过首字母大小写区分一个对象是公开的还是私有的）。因此，想要实现`http.Handler` 接口，方法名称必须连大小写也保持一致，即这里的方法名称必须是 `ServeHTTP` 而不可以是 `ServeHttp`。