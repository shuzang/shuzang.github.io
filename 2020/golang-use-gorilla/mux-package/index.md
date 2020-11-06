# Golang使用gorilla、mux包


本文介绍 [gorilla/mux](https://github.com/gorilla/mux) 包的使用。

<!--more-->

我们已知 Go 标准库 net/http 提供的默认路由是 DefaultServeMux，虽然简单易上手，但存在很多不足，比如

- 不支持参数设定，例如 `/user/:uid` 这种泛类型匹配；
- 对 REST 风格接口支持不友好，无法限制访问路由的方法；
- 对于拥有很多路由规则的应用，编写大量路由规则非常繁琐。

为此，我们可以使用第三方库 `gorilla/mux` 提供的更加强大的路由处理器（`mux` 代表 `HTTP request multiplexer`，即 HTTP 请求多路复用器），和 `http.ServeMux` 实现原理一样，`gorilla/mux` 提供的路由器实现类 `mux.Router` 也会匹配用户请求与系统注册的路由规则，然后将用户请求转发过去。

```go
type Router struct {
	// Configurable Handler to be used when no route matches.
	NotFoundHandler http.Handler
	// Configurable Handler to be used when the request method does not match the route.
	MethodNotAllowedHandler http.Handler
	// Routes to be matched, in order.
	routes []*Route
	// Routes by name for URL building.
	namedRoutes map[string]*Route
	// If true, do not clear the request context after handling the request.
	// Deprecated: No effect, since the context is stored on the request itself.
	KeepContext bool
	// Slice of middlewares to be called after a match is found
	middlewares []middleware
	// configuration shared with `Route`
	routeConf
}
```

`mux.Router` 主要具备以下特性：

- 实现了 `http.Handler` 接口，所以和 `http.ServeMux` 完全兼容；
- 可以基于 URL 主机、路径、前缀、scheme、请求头、请求参数、请求方法进行路由匹配；
- URL 主机、路径、查询字符串支持可选的正则匹配；
- 支持构建或反转已注册的 URL 主机，以便维护对资源的引用；
- 支持路由嵌套，以便不同路由可以共享通用条件，比如主机、路径前缀等。

## 1. 使用入门

运行如下命令进行安装

```bash
$ go get -u github.com/gorilla/mux
```

一个简单的示例如下

```go
func main() {
    r := mux.NewRouter()
    r.HandleFunc("/", HomeHandler)
    r.HandleFunc("/products", ProductsHandler)
    r.HandleFunc("/articles", ArticlesHandler)
    http.ListenAndServe(":8080", r)
}
```

`main` 函数中的第一行显式初始化了 `mux.Router` 作为路由器，然后在这个路由器中注册路由规则，最后将这个路由器传入 `http.ListenAndServe` 方法，整个调用过程和之前并无二致，因为`mux.Router` 也实现了 `Handler` 接口。

路径中可以包含变量。变量的定义形式为 `{name}` 或 `{name:pattern}`，只能是小写字母，不支持其它字符，同时，name 可以是正则表达式，如下面的例子所示

```go
r := mux.NewRouter()
r.HandleFunc("/products/{key}", ProductHandler)
r.HandleFunc("/articles/{category}/", ArticlesCategoryHandler)
r.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)
```

相应地，在闭包处理函数中，我们使用 `mux.Vars()` 解析路由参数：

```go
func ArticlesCategoryHandler(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, "Category: %v\n", vars["category"])
}
```

## 2. 路由代码拆分

比较简单的情况下，所有的路由、处理器都放在应用入口文件中，一般是 main.go，但如果项目比较大，甚至仅仅是博客这种级别的项目，就要处理文章、用户、图片等众多资源，所以我们需要针对这种情况进行一定的优化。

优化的办法就是将路由器与控制器分离，为了使代码结构更加清晰明了，我们把服务器、路由器、路由定义、处理器方法全都拆分开。

### 2.1 路由器

我们在项目根目录下新建 `routes` 目录用来存放路由定义及实现。

首先在 `routes` 目录下创建 `routes.go` 存放路由定义，文件内容如下

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

在这里，我们定义了一个 `WebRoute` 结构体来表示单个路由，其中包含了路由名称、请求方法、匹配字符串模式、以及对应的处理器方法，路由器可以根据这些配置请求请求分发。

然后定义了一个 `WebRoutes` 切片来存放所有 `WebRoute` 类型的路由，最后初始化这个切片为空，表示还没有定义任何路由。

接下来，在 `routes` 目录下创建 `router.go` 用来编写路由器实现

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

StrictSlash 定义斜杠尾随行为，意思是，传入 true 时，如果路由路径是 `/path` 这种形式，将重定向到 `/path/`  ，反之亦然。传入 false 时，不会重定向，这两种情况不会看作一种。

Methods，Path，Name，Handler分别用来限定请求方法、匹配字符串模式、路由名和处理器方法。通过这种方式，我们将 routes.go 中定义的所有 Web 路由都应用到了使用 mux.NewRouter 创建的路由器，以便可以处理用户请求的路由匹配和分发。

### 2.2 服务器

在入口文件 main.go 中使用如下方法启动服务器

```go
package main

import (
     . "github.com/shuzang/projectname/routes"
    "log"
    "net/http"
)

func main()  {
    startWebServer("8080")
}

func startWebServer(port string)  {
    r := NewRouter()
    http.Handle("/", r)

    log.Println("Starting HTTP service at " + port)
    err := http.ListenAndServe(":"+port, nil) // Goroutine will block here

    if err != nil {
        log.Println("An error occured starting HTTP listener at port " + port)
        log.Println("Error: " + err.Error())
    }
}
```

我们将 Web 服务器启动逻辑封装到 `startWebServer` 方法中实现，该方法需要传入端口参数。在具体实现时，我们调用了 `routes/router.go` 中定义的 `NewRouter` 方法，将其返回值作为处理器传入 `http.Handle` 方法，最后调用 `http.ListenAndServe` 启动 Web 服务器并监听传入的端口号。

最后在 `main` 方法中调用 `startWebServer` 方法即可。

### 2.3 处理器

上层代码写完后，现在定义处理器方法。在项目根目录下新建 `handlers` 目录存放处理器方法，这里举 2 个示例，分别定义在 `common.go` 和 `user.go` 两个文件中，用来处理通用请求和用户资源。

首先在 `common.go` 中编写首页请求处理器方法

```go
package handlers

import (
    "io"
    "net/http"
)

func Home(w http.ResponseWriter, r *http.Request)  {
    io.WriteString(w, "Welcome to my site")
}
```

然后在 `user.go` 中定义获取指定用户对应处理器方法

```go
package handlers

import (
    "github.com/gorilla/mux"
    "io"
    "net/http"
)

func GetUser(w http.ResponseWriter, r *http.Request)  {
    // Get user from DB by id...
    params := mux.Vars(r)
    id := params["id"]
    io.WriteString(w, "Return user info with id = " + id)
}
```

这时要记得，`routes/routes.go` 中的路由切片还是空的，用实现的处理器填充它

```go
var webRoutes = WebRoutes{
    WebRoute{
        "Home",
        "GET",
        "/",
        handlers.Home,
    },
    WebRoute{
        "User",
        "GET",
        "/user/{id}",
        handlers.GetUser,
    },
}
```

## 3. 路由匹配规则

第一部分的路由匹配规则只是简单介绍，实际上，gorilla/mux 实现的匹配规则非常强大。

### 3.1 常用匹配规则

**限定请求方法**

```go
r.HandleFunc("/books/{title}", CreateBook).Methods("POST")
r.HandleFunc("/books/{title}", ReadBook).Methods("GET")
r.HandleFunc("/books/{title}", UpdateBook).Methods("PUT")
r.HandleFunc("/books/{title}", DeleteBook).Methods("DELETE")
```

**限定主机名或子域名**

```go
r.HandleFunc("/books/{title}", BookHandler).Host("www.mybookstore.com")
```

**限定 Scheme**

```go
r.HandleFunc("/secure", SecureHandler).Schemes("https")
r.HandleFunc("/insecure", InsecureHandler).Schemes("http")
```

**限定前缀和子路由**

```go
bookrouter := r.PathPrefix("/books").Subrouter()
bookrouter.HandleFunc("/", AllBooks)
bookrouter.HandleFunc("/{title}", GetBook)
```

**限定请求参数**

```go
r.HandleFunc("/request/header", func(w http.ResponseWriter, r *http.Request) {
    header := "X-Requested-With"
    fmt.Fprintf(w, "包含指定请求头[%s=%s]", header, r.Header[header])
}).Headers("X-Requested-With", "XMLHttpRequest")
```

### 3.2 自定义匹配规则

`gorilla/mux` 路由支持通过 `MatcherFunc` 方法自定义路由匹配规则，在该方法中，可以获取到请求实例 `request`，这样我们就可以拿到所有的用户请求信息，并对其进行判断，符合我们预期的请求才能匹配并访问该方法应用到的路由。

比如下面这个示例，我们限定只有来自 `https://baidu.com` 域名的请求才可以匹配到 `/custom/matcher` 路由

```go
r.HandleFunc("/custom/matcher", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "请求来自指定域名: %s", r.Referer())
}).MatcherFunc(func(request *http.Request, match *mux.RouteMatch) bool {
    return request.Referer() == "https://baidu.com"
})
```

### 3.3 路由命名

通过 `Name` 方法在路由规则中指定

```go
postRouter := r.PathPrefix("/posts").Subrouter()
postRouter.HandleFunc("/", listPosts).Methods("GET").Name("posts.index")
postRouter.HandleFunc("/create", createPost).Methods("POST").Name("posts.create")
```

## 4. 路由中间件



## 5. 处理静态资源响应

使用默认`http`包处理静态资源的方法如下

```go
fs := http.FileServer(http.Dir("assets/"))
http.Handle("/static/", http.StripPrefix("/static/", fs))
```

使用 gorilla/mux 时，处理方法很相似

```go
r := NewRouter() // 通过 router.go 中定义的路由器来分发请求

assets := http.FileServer(http.Dir("public"))
r.PathPrefix("/static/").Handler(http.StripPrefix("/static/", assets))

http.Handle("/", r) // 应用路由器到 HTTP 服务器

...
```

虽然 `gorilla/mux` 路由器提供了对静态资源的支持，但是通常我们还是会基于 Nginx 来处理静态资源，然后将动态请求转发给 Go HTTP 服务器，因为 Nginx 作为一款强大的反向代理服务器，并发处理静态资源的能力非常强悍，没必要自己去处理这块逻辑。




