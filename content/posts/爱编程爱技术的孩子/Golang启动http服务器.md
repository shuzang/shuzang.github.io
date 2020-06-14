---
title: Golang启动HTTP服务器
date: 2020-05-29T09:32:00+08:00
lastmod: 2020-05-29
tags: [Golang, Go Web]
categories: [爱编程爱技术的孩子]
slug: Golang Start HTTP server
typora-root-url: ..\..\..\static
---

本文介绍 HTTP 协议及 Golang 如何实现 HTTP 服务端及客户端。

<!--more-->

## 1. HTTP概述

### 1.1 HTTP的概念

超文本传输协议（HyperText Transfer Protocol，HTTP）定义了浏览器（客户端进程）如何向网络上的服务器请求网络文档，以及服务器如何将文档传送给浏览器。从层次角度看，HTTP 是一个应用层协议，使用网络层的 TCP 进行可靠传输。

一个大致的工作过程如下所述。

每个网络节点都有一个服务器进程，它在后台不间断地监听着 TCP 的 80 端口，以便发现是否有来自浏览器的连接建立请求。一旦监听到连接请求并遵照握手协议建立 TCP 连接后，浏览器就会向该服务器发出浏览某个页面的请求，服务器就返回对应的页面作为响应。最后，TCP 连接被释放掉。HTTP 就是浏览器和服务器之间请求和响应的交互需要遵循的格式与规则。

HTTP 规定浏览器与服务器的交互是一个 ASCII 码串，这段字符串的格式就是 HTTP 报文格式。因为 HTTP 是建立在 TCP 上的协议，因此这段字符串也是 TCP 报文的数据部分。

无论是用户主动在浏览器地址栏输入了某个 URL，还是在页面上点击了某个元素，在背后都会转化为一个链接，然后浏览器就会在网络上找到链接对应的页面。假设我输入的 URL 或点击的元素指向了「清华大学院系设置」页面，具体的链接为 http://www.tsinghua.edu.cn/chn/yxsz/index.htm，之后发生的事情如下所述：

1. 浏览器向 DNS 请求解析 www.tsinghua.edu.cn 的 IP 地址；
2. 域名系统返回清华大学服务器的地址 166.111.4.100；
3. 浏览器与服务器建立 TCP 连接；
4. 浏览器发出取文件命令：GET /chn/yxsz/index.htm；
5. 服务器给出响应，把文件 index.htm 发给浏览器；
6. 释放 TCP 连接；
7. 浏览器渲染并显示 index.htm 文件，显示的页面就是「清华大学院系设置」页面

数据的可靠性由底层的 TCP 保证，HTTP 本身是无连接的，因此从上面的过程可以看出，通信双方并不需要建立和释放 HTTP 连接。HTTP 也是无状态的，无状态的含义是，如果此时浏览器再次访问「清华大学院系设置」页面，服务器会执行一遍重复的过程，再返回一次 index.htm 页面，因为服务器不记得这个浏览器曾经访问过。这种无状态特性既有好处也有坏处，后面会介绍。

### 1.2 HTTP 报文结构

HTTP 有两类报文：

1. 请求报文—从客户向服务器发送请求报文；
2. 响应报文—服务器向客户返回响应；

上一小节已经提到过，HTTP 是面向文本的，本质上一串 ASCII 码字符串，报文的格式就是对字符串的各部分含义进行规定，各部分长度也是不固定的。

![HTTP请求与响应报文](/images/Golang启动http服务器/epub_655484_323.jpg)

HTTP 请求与响应报文都是由三个部分组成：开始行、首部行和实体主体。

- **实体主体**就是数据部分，请求报文一般都不用，响应报文中也可能没有这个字段。
- **首部行**不一定是一行，可能有多行，也可能没有，每行都是「键：值」形式，其中键叫做首部字段名，每一行结束都要跟一个回车和换行。整个首部行部分结束，还要加一个空行和实体主体分开。首部行的作用是说明浏览器、服务器或报文主体的一些基本信息，首部字段名大都是规定好的。
- **开始行**是请求和响应报文唯一不同的地方。请求报文的开始行可以叫做请求行，响应报文的开始行叫做状态行，开始行的三个部分都以空格分开，末尾要添加 回车+换行 与首部行部分区分。

#### 请求报文

请求报文的第一行请求行包括三部分：方法、URL和版本。

**方法**是浏览器希望执行的操作，其实就是一些命令，比如提到的 GET，请求报文的类型一般根据方法的类型进行区分，下表列举了请求报文常用的一些方法：

| 方法（操作） | 意义                              |
| ------------ | --------------------------------- |
| OPTION       | 请求一些选项的信息                |
| GET          | 请求读取由 URL 所标志的信息       |
| HEAD         | 请求读取由 URL 所标志的信息的首部 |
| POST         | 给服务器添加信息                  |
| PUT          | 在指明的 URL 下存储一个文档       |
| DELETE       | 删除指明的 URL 所标志的资源       |
| TRACE        | 用来进行环回测试的请求报文        |
| CONNECT      | 用于代理服务器                    |

**URL** 则是请求资源的 URL，**版本**是指 HTTP 协议的版本，现在一般是 HTTP/1.1。前面 1.1 小节提到的例子，请求「清华大学院系设置」页面，其 HTTP 请求报文的开始行就应当是

```http
GET http://www.tsinghua.edu.cn/chn/yxsz/index.htm HTTP/1.1
```

这里给出一个完整的请求报文的例子，请求行使用相对 URL 是因为首部行给出了主机域名。

```http
GET /chn/yxsz/index.htm HTTP/1.1
Host: www.tsinghua.edu.cn  // 主机域名
Connection: close  // 告诉服务器发送完请求的文档后就可以释放连接
User-Agent: Mozilla/5.0  // 表明用户代理是使用 Netscape 浏览器
Accept-Language: cn  // 表示用户希望优先得到中文版本的文档
// 这里有一个空行，后面的实体主体部分为空
```

#### 响应报文

响应报文的状态行同样分三部分：版本，状态码和短语。

版本依然是 HTTP 协议版本，状态码用来说明不同的响应情况，短语是对状态码的简单说明。状态行的示例如下

```http
HTTP/1.1 202 Accepted
```

HTTP 状态码(tatus Code)都是三位数字，分为 5 大类共 33 种（见RFC 2616），大类的含义如下表

| 分类 | 分类描述                                       |
| :--- | :--------------------------------------------- |
| 1**  | 通知信息，请求收到了或正在处理                 |
| 2**  | 成功，操作被成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

若请求的网页从 http://www.ee.xyz.edu/index.html 转移到了一个新的地址，则响应报文的状态行和一个首部行就是下面的形式：

```http
HTTP/1.1 301 Moved Permanently
Location: http://www.xyz.edu/ee/index.html
```

### 1.3 HTTPS

安全超文本传输协议（Secure Hypertext Transfer Protocol，HTTPS）比HTTP更加安全。

HTTPS 是基于 SSL/TLS 的 HTTP，HTTP 是应用层协议，TLS 是传输层协议，在应用层和传输层之间，增加了一个安全套接层 SSL。

服务器用 RSA 生成公钥和私钥，把公钥放在证书里发送给客户端，私钥自己保存。客户端首先向一个权威的服务器求证证书的合法性，如果证书合法，客户端产生一段随机数，这段随机数就作为通信的密钥，称为对称密钥。这段随机数以公钥加密，然后发送到服务器，服务器用密钥解密获取对称密钥，最后，双方以对称密钥进行加密解密通信。

HTTPS的作用首先是内容加密，建立一个信息安全通道，来保证数据传输的安全；其次是身份认证，确认网站的真实性；最后是保证数据完整性，防止内容被第三方替换或者篡改。

HTTPS和HTTP有一定的区别。HTTPS协议需要到CA申请证书。HTTP是超文本传输协议，信息是明文传输；HTTPS则是具有安全性的SSL加密传输协议。HTTP使用的是80端口，而HTTPS使用的是443端口。HTTP的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的、可进行加密传输和身份认证的网络协议，比HTTP协议安全。

## 2. HTTP协议客户端实现

Go语言标准库内置了net/http包，涵盖了HTTP客户端和服务端具体的实现方式。内置的net/http包提供了最简洁的HTTP客户端实现方式，无须借助第三方网络通信库，就可以直接使用HTTP中用得最多的GET和POST方式请求数据。

实现HTTP客户端就是客户端通过网络访问向服务端发送请求，服务端发送响应信息，并将相应信息输出到客户端的过程。实现客户端有多种方式，具体如下所示。

### 2.1 使用http.NewRequest()方法

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

### 2.2 调用client.Get() 方法

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

### 2.3 使用client.Post()或client.PostForm()方法

这种方法也是两个步骤，先创建一个client（客户端）对象，然后使用client调用Post()或PostForm()方法。其实client的Post()或PostForm()方法，就是对http.NewRequest()的封装。

```go
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
```

### 2.4 使用http.Get() 方法

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

### 2.5 使用http.Post()或http.PostForm()方法

http的Post()函数或PostForm()，就是对DefaultClient.Post()或DefaultClient.PostForm()的封装。这种方法也只需要一个步骤

## 3. HTTP协议服务端实现

使用Go语言标准库内置的net/http包，就可以实现一个基本的HTTP服务端。一个基本的HTTP服务器主要应完成如下功能

1. 处理动态请求：处理浏览网站，登录帐户或发布图片等用户传入的请求。
2. 提供静态文件：将JavaScript，CSS和图像等静态文件提供给浏览器，服务于用户。
3. 接受连接请求：HTTP服务器必须监听指定端口从而接收来自网络的连接请求。

### 3.1 处理动态请求

我们可以使用`http.HandleFunc`函数注册一个新的 Handler 来处理动态请求。它的第一个参数是请求路径的匹配模式，第二个参数是一个函数类型，表示针对这个请求要执行的功能。下例中针对请求返回一个欢迎访问的提示语。

```go
http.HandleFunc("/", func (w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Welcome to my website!")
})
```

`http.ResponseWriter`类型包含了服务器端给客户端的响应数据。服务器端往里面写入了什么内容，浏览器的网页源码就是什么内容。`*http.Request`包含了客户端发送给服务器端的请求信息（路径、浏览器类型等）。

### 3.2 提供静态文件

使用`http.FileServer()` 方法提供 Javascript，CSS或图片等静态文件。它的参数是文件系统接口，可以使用`http.Dir()`来指定文件所在的路径。如果该路径中有index.html文件，则会优先显示html文件，否则会显示文件目录。

```go
fs := http.FileServer(http.Dir("static/"))
```

`http.FileServer()`的返回值正好是 Handler 类型，也就是可以提供文件访问服务的HTTP处理器。现在，我们只需要将一个URL指向它，期间我们可以使用`http.StripPrefix()` 去除某些URL前缀，返回值同样是一个 Handler类型

```go
http.Handle("/static/", http.StripPrefix("/static/", fs))
```

### 3.3 接收连接请求

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

### 3.4 获取客户端提交的数据

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



