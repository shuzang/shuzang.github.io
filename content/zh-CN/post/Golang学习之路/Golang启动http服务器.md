---
title: Golang启动HTTP服务器
date: 2020-05-29T09:32:00+08:00
lastmod: 2020-05-29
tags: [Go实战]
categories: [Golang学习之路]
slug: Golang Start HTTP server 
---

本文介绍 Golang 如何实现 HTTP 服务端及客户端。

<!--more-->

## 1. HTTP协议客户端实现

Go语言标准库内置了net/http包，涵盖了HTTP客户端和服务端具体的实现方式。内置的net/http包提供了最简洁的HTTP客户端实现方式，无须借助第三方网络通信库，就可以直接使用HTTP中用得最多的GET和POST方式请求数据。

实现HTTP客户端就是客户端通过网络访问向服务端发送请求，服务端发送响应信息，并将相应信息输出到客户端的过程。实现客户端有多种方式，具体如下所示。

### 1.1 使用http.NewRequest()方法

首先创建一个client（客户端）对象，其次创建一个request（请求）对象，最后使用client发送request。

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	testHttpNewRequest()
}

func testHttpNewRequest() {
	//1.创建一个客户端
	client := http.Client{}
	//2.创建一个请求，请求方式可以是GET或POST
	request, err := http.NewRequest("GET", "http://www.baidu.com", nil)
	checkErr(err)
	//3.客户端发送请求
	cookName := &http.Cookie{Name: "username", Value: "Steven"}

	//添加cookie
	request.AddCookie(cookName)
	response, err := client.Do(request)
	checkErr(err)
	//设置请求头
	request.Header.Set("Accept-Lanauage", "zh-cn")
	defer response.Body.Close()
	//查看请求头的数据
	fmt.Printf("Header:%+v\n", request.Header)
	fmt.Printf("响应状态码: %v\n", response.StatusCode)

	//4.操作数据
	if response.StatusCode == 200 {
		fmt.Println("网络请求成功")
		checkErr(err)
	} else {
		fmt.Println("网络请求失败", response.Status)
	}
}

//检查错误
func checkErr(err error) {
	defer func() {
		if ins, ok := recover().(error); ok {
			fmt.Println("程序出现异常: ", ins.Error())
		}
	}()
	if err != nil {
		panic(err)
	}
}
```

运行结果如下

```bash
$ go run main.go
Header:map[Accept-Lanauage:[zh-cn] Cookie:[username=Steven]]
响应状态码: 200
网络请求成功 
```

### 1.2 调用client.Get() 方法

这种方法总共两个步骤，先创建一个client（客户端）对象，然后使用client调用Get()方法。

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	testClientGet()
}

func testClientGet() {
	//1.创建一个客户端
	client := http.Client{}
	//2.通过client请求
	response, err := client.Get("http://www.baidu.com")
	checkErr(err)

	fmt.Printf("响应状态码: %v\n", response.StatusCode)

	if response.StatusCode == 200 {
		fmt.Println("网络请求成功")
		defer response.Body.Close()
	}
}

//检查错误
func checkErr(err error) {
	defer func() {
		if ins, ok := recover().(error); ok {
			fmt.Println("程序出现异常: ", ins.Error())
		}
	}()
	if err != nil {
		panic(err)
	}
}

```

运行结果如下

```bash
$ go run main.go
响应状态码: 200
网络请求成功
```

### 1.3 使用client.Post()或client.PostForm()方法

这种方法也是两个步骤，先创建一个client（客户端）对象，然后使用client调用Post()或PostForm()方法。其实client的Post()或PostForm()方法，就是对http.NewRequest()的封装。

```go
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
```

### 1.4 使用http.Get() 方法

这种方式只有一个步骤，http的Get()方法就是对DefaultClient.Get()的封装。

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	testHttpGet()
}

func testHttpGet() {
	//获取服务器数据
	response, err := http.Get("http://www.baidu.com")
	checkErr(err)
	fmt.Printf("响应状态码: %v\n", response.StatusCode)

	if response.StatusCode == 200 {
		fmt.Println("网络请求成功")
		defer response.Body.Close()
		checkErr(err)
	} else {
		fmt.Println("请求失败", response.Status)
	}
}

//检查错误
func checkErr(err error) {
	defer func() {
		if ins, ok := recover().(error); ok {
			fmt.Println("程序出现异常: ", ins.Error())
		}
	}()
	if err != nil {
		panic(err)
	}
}

```

运行结果为

```bash
$ go run main.go
响应状态码: 200
网络请求成功
```

### 1.5 使用http.Post()或http.PostForm()方法

http的Post()函数或PostForm()，就是对DefaultClient.Post()或DefaultClient.PostForm()的封装。这种方法也只需要一个步骤

## 2. HTTP协议服务端实现

使用Go语言标准库内置的net/http包，就可以实现一个基本的HTTP服务端。一个基本的HTTP服务器主要应完成如下功能

1. 处理动态请求：处理浏览网站，登录帐户或发布图片等用户传入的请求。
2. 提供静态文件：将JavaScript，CSS和图像等静态文件提供给浏览器，服务于用户。
3. 接受连接请求：HTTP服务器必须监听指定端口从而接收来自网络的连接请求。

### 2.1 处理动态请求

我们可以使用`http.HandleFunc`函数注册一个新的 Handler 来处理动态请求。它的第一个参数是请求路径的匹配模式，第二个参数是一个函数类型，表示针对这个请求要执行的功能。下例中针对请求返回一个欢迎访问的提示语。

```go
http.HandleFunc("/", func (w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Welcome to my website!")
})
```

`http.ResponseWriter`类型包含了服务器端给客户端的响应数据。服务器端往里面写入了什么内容，浏览器的网页源码就是什么内容。`*http.Request`包含了客户端发送给服务器端的请求信息（路径、浏览器类型等）。

### 2.2 提供静态文件

使用`http.FileServer()` 方法提供 Javascript，CSS或图片等静态文件。它的参数是文件系统接口，可以使用`http.Dir()`来指定文件所在的路径。如果该路径中有index.html文件，则会优先显示html文件，否则会显示文件目录。

```go
fs := http.FileServer(http.Dir("static/"))
```

`http.FileServer()`的返回值正好是 Handler 类型，也就是可以提供文件访问服务的HTTP处理器。现在，我们只需要将一个URL指向它，期间我们可以使用`http.StripPrefix()` 去除某些URL前缀，返回值同样是一个 Handler类型

```go
http.Handle("/static/", http.StripPrefix("/static/", fs))
```

### 2.3 接收连接请求

`http.ListenAndServer()`函数用来启动HTTP服务器，并且在指定的 IP 地址和端口上监听客户端请求

```go
http.ListenAndServe(":80", nil)
```

函数实现如下，其中第一个参数为监听地址，第二个参数表示一个HTTP处理器 Handler。可以看到，底层调用的是 `net/http` 包的 `ListenAndServe` 方法，首先会初始化一个 `Server` 对象，然后调用该 `Server` 实例的 `ListenAndServe` 方法，进而调用 `net.Listen("tcp", addr)`，也就是基于 TCP 协议创建 Listen Socket，并在传入的IP 地址和端口号上监听请求。

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}

func (srv *Server) ListenAndServe() error {
	if srv.shuttingDown() {
		return ErrServerClosed
	}
	addr := srv.Addr
	if addr == "" {
		addr = ":http"
	}
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(ln)
}
```

最终我们看到调用了  `Server` 实例的 `Serve(net.Listener)` 方法，这个方法里面起了一个 `for` 循环，在循环体中首先通过 `net.Listener`（即上一步监听端口中创建的 Listen Socket）实例的 `Accept` 方法接收客户端请求，接收到请求后根据请求信息创建一个 `conn` 连接实例，最后单独开了一个 goroutine，把这个请求的数据当做参数扔给这个 `conn` 去服务：

```go
for {
		rw, err := l.Accept()
		if err != nil {
			...
		}
		connCtx := ctx
		if cc := srv.ConnContext; cc != nil {
			connCtx = cc(connCtx, rw)
			if connCtx == nil {
				panic("ConnContext returned nil")
			}
		}
		tempDelay = 0
		c := srv.newConn(rw)
		c.setState(c.rwc, StateNew) // before Serve can return
		go c.serve(connCtx)
	}
```

用户的每一次请求都是在一个新的 goroutine 去服务，相互不影响。客户端请求的具体处理逻辑都是在 `c.serve` 中完成的。 `conn` 实例的 `serve` 方法首先会通过 `c.readRequest()` 解析请求，然后在 `serverHandler{c.server}.ServeHTTP(w, w.req)` 的 `ServeHTTP` 方法中获取相应的 `handler`：`handler := c.server.Handler`，也就是我们刚才在调用函数 `ListenAndServe` 时候的第二个参数。

```go
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}
	handler.ServeHTTP(rw, req)
}
```

我们发现当 handler 为 nil，也就是 ListenAndServe() 的第二个参数为 nil 时，使用了默认的 http.DefaultServeMux，这是 ServeMux的默认实例

```go
var DefaultServeMux = &defaultServeMux
var defaultServeMux ServeMux
```

ServeMux的数据结构如下

```go
type ServeMux struct {
    mu    sync.RWMutex. // 由于请求涉及到并发处理，因此这里需要一个锁机制
    m     map[string]muxEntry // 路由规则字典，存放 URL 路径与处理器的映射关系
    es    []muxEntry // MuxEntry 切片（按照最长到最短排序）
    hosts bool       // 路由规则中是否包含 host 信息
}
```

这里，我们需要重点关注的是 `muxEntry` 结构：

```go
type muxEntry struct {
    h   Handler       // 处理器具体实现
    pattern string    // 模式匹配字符串
}
```

最后我们来看一下 `Handler` 的定义，这是一个接口：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request) // 路由处理实现方法
}
```

当请求路径与 `pattern` 匹配时，就会调用 `Handler` 的 `ServeHTTP` 方法来处理请求。

```go
http.HandleFunc("/", sayHelloWorld)
```

当我们使用一个自定义的处理函数时，如上面的sayHelloWorld，并没有实现 `Handler` 接口，之所以可以成功添加到路由映射规则，是因为在底层通过 `HandlerFunc()` 函数将其强制转化为了 `HandlerFunc` 类型，而 `HandlerFunc` 类型实现了 `ServeHTTP` 方法，这样，`sayHelloWorld` 方法也就变相实现了 `Handler` 接口

```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    if handler == nil {
		  panic("http: nil handler")
    }
    mux.Handle(pattern, HandlerFunc(handler))
}

...

type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

对于 `sayHelloWorld` 方法来说，它已然变成了 `HandlerFunc` 类型的函数类型，当我们在其实例上调用 `ServeHTTP` 方法时，调用的是 `sayHelloWorld` 方法本身。

前面我们提到，`DefaultServeMux` 是 `ServeMux` 的默认实例，当我们在 `HandleFunc` 中调用 `mux.Handle` 方法时，实际上是将其路由映射规则保存到 `DefaultServeMux` 路由处理器的数据结构中：

```go
func (mux *ServeMux) Handle(pattern string, handler Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	if pattern == "" {
		panic("http: invalid pattern")
	}
	if handler == nil {
		panic("http: nil handler")
	}
	if _, exist := mux.m[pattern]; exist {
		panic("http: multiple registrations for " + pattern)
	}

	if mux.m == nil {
		mux.m = make(map[string]muxEntry)
	}
	e := muxEntry{h: handler, pattern: pattern}
	mux.m[pattern] = e
	if pattern[len(pattern)-1] == '/' {
		mux.es = appendSorted(mux.es, e)
	}

	if pattern[0] != '/' {
		mux.hosts = true
	}
}
```

还是以 `sayHelloWorld` 为例，这里的 `pattern` 字符串对应的是请求路径 `/`，`handler` 对应的是 `sayHelloWorld` 函数。

保存好路由映射规则之后，客户端请求的处理就默认调用`ServeMux` 实现的 `ServeHTTP` 方法：

```go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    if r.RequestURI == "*" {
        w.Header().Set("Connection", "close")
        w.WriteHeader(StatusBadRequest)
        return
    }
    
    h, _ := mux.Handler(r)
    h.ServeHTTP(w, r)
}
```

如上所示，路由处理器接收到请求之后，如果 URL 路径是 `*`，则关闭连接，否则调用 `mux.Handler(r)` 返回对应请求路径匹配的处理器，然后执行 `h.ServeHTTP(w, r)`，也就是调用对应路由 `handler` 的 `ServerHTTP` 方法，以 `/` 路由为例，调用的就是 `sayHelloWorld` 函数本身。

**总结**

现在我们来捋一下，当我们调用 http.ListenAndServe，首先建立了一个 Server 实例，然后把两个参数都赋给了该实例，之后我们在该实例的基础上调用底层 net 包监听端口、创建socket并开启连接，最后把这个连接交给了 Server实例的 handler处理，这个handler 正是我们在 ListenAndServe 中传入的第二个参数。

当第二个参数为 nil 时调用了 ServeMux 的默认实例 DefaultServeMux ，该实例实现了一个 ServeMux 结构体，而这个结构体中最重要的一个字段就是muxEntry 结构体，包含 pattern 和 handler 两部分。所以我们实现 Handle 和 HandleFunc 都是在将路由映射规则保存到 `DefaultServeMux` 路由处理器的 muxEntry 结构体的这两个字段。

客户端请求的处理就默认调用`ServeMux` 实现的 `ServeHTTP` 方法，把对应的请求交给对应的处理器。

### 2.4 获取客户端提交的数据

前面已经提到，客户端提交的数据全部位于 *http.Request 中，下面的例子虽然做了声明，但没有使用，本节介绍一下如何从 *http.request 中提取想要的数据

```go
http.HandleFunc("/", func (w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Welcome to my website!")
})
```

Request的部分结构如下

```go
type Request struct {
    ...
    // Method指定HTTP方法（GET、POST、PUT等）。对客户端，""代表GET。
    Method string
    // Form是解析好的表单数据，包括URL字段的query参数和POST或PUT的表单数据。
    Form url.Values
    // PostForm是解析好的POST或PUT的表单数据。
    PostForm url.Values
    ...
}
```

使用ParseForm解析URL中的查询字符串，并将解析结果更新到r.Form字段。对于POST或PUT请求，ParseForm还会将body当作表单解析，并将结果既更新到r.PostForm也更新到r.Form。解析结果中，POST或PUT请求主体要优先于URL查询字符串（同名变量，主体的值在查询字符串的值前面）。

```go
func (r *Request) ParseForm() error
```

然后使用 FormValue 返回以 key 为健查询 r.Form 得到的第一个值

```go
func (r *Request) FormValue(key string) string
```

PostFormValue则返回key为键查询r.PostForm字段得到的第一个值，用于POST和PUT

```go
func (r *Request) PostFormValue(key string) string
```

当提交的请求数据中有文件时，使用FormFile，可以返回以key为键查询r.MultipartForm字段得到结果中的第一个文件和它的信息。

```go
func (r *Request) FormFile(key string) (multipart.File, *multipart.FileHeader, error)
```

一个简单的实现如下

```go
func loginActionHandler(w http.ResponseWriter, r *http.Request) {
	r.ParseForm()
	if r.Method == "GET" && r.ParseForm() == nil {
		username := r.FormValue("username")
		pwd := r.FormValue("password")
		if len(username) < 4 || len(username) > 10 {
			w.Write([]byte("用户名不符合规范"))
		}
		if len(pwd) < 6 || len(pwd) > 16 {
			w.Write([]byte("密码不符合规范"))
		}
		http.Redirect(w, r, "/list", http.StatusFound)
		return
	} else {
		w.Write([]byte("请求方式不对"))
		return
	}
	w.Write([]byte("登录失败"))
}
```



