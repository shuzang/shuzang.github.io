# Golang日志系统


日志是一个系统必不可少的部分，本篇介绍Golang中的日志系统。

<!--more-->

## 1. 内置log包

官方提供的 log 包实现了简单的日志服务。该包定义了一个 Logger 类型，提供了一些格式化输出方法，为了更容易地使用，对该类型提供了一个标准 Logger 实现，该 Logger 会打印每条日志信息的日期、时间、默认输出到标准错误，Fatal系列函数会在写入日志信息后调用 os.Exit(1)。Panic系列函数会在写入日志信息后panic。

### 1.1 内置实现

一个内置实现使用的简单例子如下

```go
// This sample program demonstrates how to use the base log package.
package main

import (
    "log"
)

func init() {
    log.SetPrefix("TRACE: ")
    log.SetFlags(log.Ldate | log.Lmicroseconds | log.Llongfile)
}

func main() {
    // Println writes to the standard logger.
    log.Println("message")

    // Fatalln is Println() followed by a call to os.Exit(1).
    log.Fatalln("fatal message")

    // Panicln is Println() followed by a call to panic().
    log.Panicln("panic message")
}
-------------------------------
TRACE: 2019/04/09 14:24:32.868375 D:/go/TestFile/src/main/TestLog.go:15: message
TRACE: 2019/04/09 14:24:32.962329 D:/go/TestFile/src/main/TestLog.go:18: fatal message

Process finished with exit code 1
```

原型函数的说明如下

```go
func Flags() int            		// Flags返回标准logger的输出选项
func SetFlags(flag int)   		// SetFlags设置标准logger的输出选项
func Prefix() string      		// Prefix返回标准logger的输出前缀
func SetPrefix(prefix string) 		// SetPrefix设置标准logger的输出前缀
func SetOutput(w io.Writer)		// SetOutput设置标准logger的输出目的地，默认是标准错误输出

// Printf调用Output将生成的格式化字符串输出到标准logger，参数用和fmt.Printf相同的方法处理。
func Printf(format string, v ...interface{})
// Print调用Output将生成的格式化字符串输出到标准logger，参数用和fmt.Print相同的方法处理。
func Print(v ...interface{})
// Println调用Output将生成的格式化字符串输出到标准logger，参数用和fmt.Println相同的方法处理。
func Println(v ...interface{})

func Fatalf(format string, v ...interface{})	// Fatalf等价于{Printf(v...); os.Exit(1)}
func Fatal(v ...interface{})			// Fatal等价于{Print(v...); os.Exit(1)}
func Fatalln(v ...interface{})			// Fatalln等价于{Println(v...); os.Exit(1)}

func Panicf(format string, v ...interface{})	// Panicf等价于{Printf(v...); panic(...)}
func Panic(v ...interface{})			// Panic等价于{Print(v...); panic(...)}
func Panicln(v ...interface{})			// Panicln等价于{Println(v...); panic(...)}
```

SetPrefix 设置输出前缀，SetfFlags 设置输出选项，为了理解输出前缀与选项，我们先来看一个标准输出

```go
TRACE: 2019/04/09 14:24:32.868375 D:/go/TestFile/src/main/TestLog.go:15: message
```

其中，`TRACE` 就是输出前缀，可以通过 SetPrefix 设置，通过 Prefix 输出，用来在普通的程序输出中分布出日志。后面冒号前的信息就是输出选项，通过 SetFlags 设置，通过 Flags 输出。输出选项的结构定义如下

```go
const (
    // 字位共同控制输出日志信息的细节。不能控制输出的顺序和格式。
    // 在所有项目后会有一个冒号：2009/01/23 01:23:23.123123 /a/b/c/d.go:23: message
    Ldate         = 1 << iota     // 日期：2009/01/23
    Ltime                         // 时间：01:23:23
    Lmicroseconds                 // 微秒分辨率：01:23:23.123123（用于增强Ltime位）
    Llongfile                     // 文件全路径名+行号： /a/b/c/d.go:23
    Lshortfile                    // 文件无路径名+行号：d.go:23（会覆盖掉Llongfile）
    LstdFlags     = Ldate | Ltime // 标准logger的初始值
)
```

log 包有一个很方便的地方就是，这些日志记录器是多 goroutine 安全的。这意味着在多个goroutine 可以同时调用来自同一个日志记录器的这些函数，而不会有彼此间的写冲突。标准日志记录器具有这一性质，用户定制的日志记录器也应该满足这一性质。

### 1.2 基于Logger自定义

官方的预置实现是基于Logger类型的，我们也可以基于Logger类型自己进行实现

Logger类型表示一个活动状态的记录日志的对象，它会生成一行行的输出写入一个io.Writer接口。每一条日志操作会调用一次io.Writer接口的Write方法。Logger类型的对象可以被多个线程安全的同时使用，它会保证对io.Writer接口的顺序访问。

```go
type Logger struct {
    // contains filtered or unexported fields
}
```

Logger类型的方法就是官方实现的那些函数

```go
func (l *Logger) Fatal(v ...interface{})
func (l *Logger) Fatalf(format string, v ...interface{})
func (l *Logger) Fatalln(v ...interface{})
func (l *Logger) Flags() int
func (l *Logger) Output(calldepth int, s string) error
func (l *Logger) Panic(v ...interface{})
func (l *Logger) Panicf(format string, v ...interface{})
func (l *Logger) Panicln(v ...interface{})
func (l *Logger) Prefix() string
func (l *Logger) Print(v ...interface{})
func (l *Logger) Printf(format string, v ...interface{})
func (l *Logger) Println(v ...interface{})
func (l *Logger) SetFlags(flag int)
func (l *Logger) SetPrefix(prefix string)
```

有区别的是，还有一个 New 函数用来创建一个 Logger，其中的参数 out 用于设置日志信息写入的目的地，prefix 设置前缀，flag 设置选项。

```go
func New(out io.Writer, prefix string, flag int) *Logger
```

一个简单的使用示例如下

```go
var buf bytes.Buffer
logger := log.New(&buf, "logger: ", log.Lshortfile)
logger.Print("Hello, log file!")
fmt.Print(&buf)
// Output
logger: example_test.go:16: Hello, log file!
```

一般情况下，我们需要区分不同的日志级别：Info、Warning 和 Error，这里有一个参考实现

```go
// 这个示例程序展示如何创建定制的日志记录器
package main

import (
    "io"
    "io/ioutil"
    "log"
    "os"
)

var (
    Trace   *log.Logger // 记录所有日志
    Info    *log.Logger // 重要的信息
    Warning *log.Logger // 需要注意的信息
    Error   *log.Logger // 非常严重的问题
)

func init() {
    file, err := os.OpenFile("errors.txt",
        os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalln("Failed to open error log file:", err)
    }

    Trace = log.New(ioutil.Discard,
        "TRACE: ",
        log.Ldate|log.Ltime|log.Lshortfile)

    Info = log.New(os.Stdout,
        "INFO: ",
        log.Ldate|log.Ltime|log.Lshortfile)

    Warning = log.New(os.Stdout,
        "WARNING: ",
        log.Ldate|log.Ltime|log.Lshortfile)

    Error = log.New(io.MultiWriter(file, os.Stderr),
        "ERROR: ",
        log.Ldate|log.Ltime|log.Lshortfile)
}

func main() {
    Trace.Println("I have something standard to say")
    Info.Println("Special Information")
    Warning.Println("There is something you need to know about")
    Error.Println("Something has failed")
}
------------------------------------
INFO: 2019/04/09 14:37:11 TestCustomLog.go:44: Special Information
ERROR: 2019/04/09 14:37:11 TestCustomLog.go:46: Something has failed
WARNING: 2019/04/09 14:37:11 TestCustomLog.go:45: 
There is something you need to know about
```

这里写入的目的地为 ioutil.Discard，这是一个 io.Writer 接口，调用它的 Write 方法将不做任何事情并且始终成功返回。

OpenFile几个选项说明如下

- O_CREATE   // 如果不存在将创建一个新文件
- O_WRONLY  // 只写模式打开文件
- O_APPEND  // 写操作时将数据附加到文件尾部

## 2. 第三方log库

比较流行且近期还在更新的日志库有 [logrus](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsirupsen%2Flogrus)、[zap](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fuber-go%2Fzap)、[oklog](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Foklog%2Foklog)、[zerolog](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frs%2Fzerolog)

可以考虑选择 JSON 格式的日志，参考 [最佳日志实践（v2.0）](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F27363484) 一文

关于性能可以参考 [Go零消耗debug log技巧](https://links.jianshu.com/go?to=https%3A%2F%2Fmzh.io%2Fgolang-build-tags-for-debug)，使用官方模块在生产环境中存在性能瓶颈

## 3.参考

[1] 合肥懒皮，简书，[Golang log日志](https://www.jianshu.com/p/73ae6dc4d16a)，2019.04.09

[2] [Go语言标准库文档](https://studygolang.com/pkgdoc)




