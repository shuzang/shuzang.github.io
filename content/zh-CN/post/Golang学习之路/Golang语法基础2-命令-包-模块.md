---
title: Golang语法基础2-命令、包与模块
date: 2019-09-19
lastmod: 2020-09-28
tags: [Go语法]
categories: [Golang学习之路]
slug: Golang basic grammer 2-command package module
---

本篇介绍Go中的基础命令，包和模块。

## 1. 命令

Go 在安装后自带一个命令行工具，名为 `go`，用来下载、编译、安装、测试 Go 的包和源文件，关于 Go CLI 的发展历史和设计理念，可以查看 [About the go command](https://golang.google.cn/doc/articles/go_command.html)，这里只介绍如何使用这些命令。

Go 命令的用法如下

```bash
go <command> [arguments]
```

可用命令如下，使用 `go help <command>` 可获取对应命令的帮助信息

```bash
bug         start a bug report
build       compile packages and dependencies
clean       remove object files and cached files
doc         show documentation for package or symbol
env         print Go environment information
fix         update packages to use new APIs
fmt         gofmt (reformat) package sources
generate    generate Go files by processing source
get         add dependencies to current module and install them
install     compile and install packages and dependencies
list        list packages or modules
mod         module maintenance
run         compile and run Go program
test        test packages
tool        run specified go tool
version     print Go version
vet         report likely mistakes in packages
```

其它的一些帮助主题如下，使用 `go help <topic>` 可以查看相关主题的说明

```bash
buildmode   build modes
c           calling between Go and C
cache       build and test caching
environment environment variables
filetype    file types
go.mod      the go.mod file
gopath      GOPATH environment variable
gopath-get  legacy GOPATH go get
goproxy     module proxy protocol
importpath  import path syntax
modules     modules, module versions, and more
module-get  module-aware go get
module-auth module authentication using go.sum
module-private module configuration for non-public modules
packages    package lists and patterns
testflag    testing flags
testfunc    testing functions
```

我们之前已经接触过几种命令，包括查看环境变量的 `go env`，查看go版本的 `go version`，用于编译和安装的`go install`和`go build`，已经对它们有了一定的了解，下面详细解释所有的命令，明确它们的作用，弄清楚它们的区别。

### 1.1 command

当前命令一共17个，使用` go help <command>`可以查看这些命令的详细说明

#### go build

```bash
go build [-o output] [-i] [build flags] [packages]
```

`go build` 编译导入的包和依赖，但会忽略掉以 `_test.go` 结尾的文件，因为这些文件是用来测试的。

编译`main`包时，输出的可执行文件会放到**当前**目录下，可执行文件的后缀取决于操作系统，只有在windows下后缀才会是`.exe`，而可执行文件的名字同编译的go文件名相同，如ed.go会编译成ed.exe。

编译多个包或单个非`main`包时，编译器不会输出可执行文件，仅仅作为这些包是否可编译的一个检查。

以 hello world 程序为例，编译后的 hello.exe 文件位于项目根目录下

```bash
$ go build github.com/shuzang/hello
$ ls
go.mod  go.sum  gotest.exe  main.go
```

加入 `-o` 参数可以指定输出文件，`-i`则在编译后自动执行安装过程（默认只会编译不会安装）

#### go install

`go install` 命令在编译的基础上增添了安装这一步，`安装` 的基本含义是将生成的可执行文件放到指定的目录，默认为 GOBIN 环境变量指定的目录，即 `$GOPATH/bin`。仍以`hello`为例，如下命令执行后，`hello.exe`文件将位于`$GOPATH/bin`目录下

```powershell
> go install github.com/shuzang/hello
```

#### go get

```bash
go get [-d] [-t] [-u] [-v] [-insecure] [build flags] [packages]
```

`go get` 相比 `go install` 又多了一步：解析与添加依赖到当前包。完成这一步后自动编译和安装它们。

`go get` 下载的默认路径是 `GOPATH/pkg/mod` ，默认下载最新版本，但版本的选择规则比较复杂，可以查看命令说明。下面是一个使用实例

```powershell
> go get -v github.com/google/codesearch/index
github.com/google/codesearch (download)
github.com/google/codesearch/sparse
github.com/google/codesearch/index
```

`-v`参数输出下载安装的详细过程，并输出debug信息，其它的一些可选参数说明如下

- `-d`只下载不安装
- `-fix`在对下载的包解析依赖项或编译前先运行修复工具
- `-t`下载为指定的包生成测试需要的包
- `-u`用于更新已有的包和依赖

#### go run

以上三个命令虽然都包含编译过程，但也到此为止，在生成可执行文件后将不再做任何操作，需要自己来执行。`go run`命令则在编译后直接执行运行操作，以`hello`为例

```powershell
> go run github.com/shuzang/hello
Hello, Go!
```

#### go version

`go version`在没有参数时，会打印自身的版本信息

```powershell
> go version
go version go1.13.4 windows/amd64
```

但是`go version`后也可以跟一个目录，这时会递归的查找该目录下的可执行文件，并打印它们的版本信息，以`bin`目录为例

```powershell
PS C:\Users\lylw1\go> go version bin
bin\cobra.exe: go1.13.4
bin\dlv.exe: go1.13.4
bin\go-outline.exe: go1.13.4
bin\go-symbols.exe: go1.13.4
bin\gocode-gomod.exe: go1.13.4
bin\gocode.exe: go1.13.4
bin\godef.exe: go1.13.4
bin\golint.exe: go1.13.4
bin\gopkgs.exe: go1.13.4
bin\gorename.exe: go1.13.4
bin\goreturns.exe: go1.13.4
bin\guru.exe: go1.13.4
bin\hello.exe: go1.13.4
```

添加`-m`参数会打印包导入的模块信息，添加`-v`参数会将无法识别的文件信息也打印出来，以`stringutil`为例

```powershell
> go version -v src/github.com/shuzang/stringutil
src\github.com\shuzang\stringutil\reverse.go: not executable file
```

#### go env

```bash
go env [-json] [-u] [-w] [var ...]
```

`go env`的基本作用是打印go的环境变量信息，添加`-json`参数可以以 JSON 的格式打印，但没有默认的脚本形式可读性高。添加 `-w` 参数可以设置某个环境变量的值，比如

```powershell
> go env -w GOPATH=d:\go
```

与之相反，重置某个环境变量为默认值可以使用`-u`参数

```powershell
> go env -u GOPATH
```

#### 其它

其它的命令使用频率没有上面几个高，因此只简单介绍其作用，用的时候再去查用法即可，`go test`和`go mod`两个命令以后单独介绍。

- `go bug`，作用是打开默认浏览器并启动新的 Bug 报告，该报告包含有用的系统信息。
- `go clean`，移除源码包中编译生成的文件
- `go doc`，显示包或符号的文档
- `go fix`，用来更新老版本的代码到新版本
- `go fmt`，go的代码有严格的格式要求，该命令用来做格式化，但一般IDE都会帮忙做这件事
- `go generate`，通过处理源码生成go文件
- `go list`，列出当前安装的包或模块
- `go mod`，模块维持
- `go test`，自动读取源码目录下名为`*_test.go`的文件进行测试
- `go tool`，运行指定的go工具，后面跟的参数是其它命令
- `go vet`，用于检查Go语言源码中的静态错误

### 1.2 topic

topic有15个，本质是对Go中的一些概念作解释，所以它实际上是一些文档说明，使用`go help <topic>`查看，这些topic包括

- buildmode：构建模式的描述
- c：Go和c的相互调用
- cache：构建和测试缓存
- environment：环境变量
- filetype：文件类型
- go.mod：go.mod文件
- gopath：GOPATH环境变量
- gopath-get：
- goproxy：模块代理协议
- importpath：导入路径语法
- modules：模块，模块版本等
- modules-get：
- packages：包列表的描述
- testflag：测试符号描述
- testfunc：测试函数描述

## 2. 包

包是Go语言代码组织和代码编译的一个基本结构，一个包可能由一个或多个`.go`文件组成，而一个或多个包可以构成完整的项目(仓库)。

### 2.1 包名

每个Go源文件(`.go`文件)都必须在非注释的首行声明属于哪个包

```go
package name
```

`name`就是包名，包名的命名遵循命名规范，而且不得使用大写字母。每个Go应用程序都必须包含一个`main`包

一个Go程序通过`import`关键字将一组包衔接在一起，如

```go
import "fmt"
```

包名需要用双引号包围，多个包可以使用多个`import`语句，也可以使用一个小括号全部放在一起，如

```go
import (
   "fmt"
   "os"
    "crypto/rot13"
    "github.com/shuzang/hello"
)
```

包名也是导入路径的最后一个字段，比如`crypto/rot13`，其包名为`rot13`；`github.com/shuzang/hello`，其包名为`hello`

没有必要刻意使用不同的包名，只要导入路径保持唯一即可

### 2.2 标准库

像 `fmt`、`os` 等这样具有常用功能的内置包在 Go 语言中有 150 个以上，它们被称为标准库，大部分内置于Go本身，位于`C:\Go\pkg\windows_amd64`，即`$GOROOT/pkg/$GOOS__$GOARCH`目录下，不需要额外的下载、安装和编译，详细列表和说明可以在 [Go Packages](https://golang.google.cn/pkg/) 查看，这里简单介绍一些常用包的基本功能

- `unsafe`: 包含了一些打破 Go 语言“类型安全”的命令，一般的程序中不会被使用，可用在 C/C++ 程序的调用中
- 系统操作类
  - `os`: 提供给我们一个平台无关性的操作系统功能接口，采用类Unix设计，隐藏了不同操作系统间差异，让不同的文件系统和操作系统对象表现一致
  - `os/exec`: 提供我们运行外部操作系统命令和程序的方式
  - `syscall`: 底层的外部包，提供了操作系统底层调用的基本接口
-  `archive/tar` 和 `/zip-compress`：压缩(解压缩)文件功能。
- 输入输出类
  - `fmt`: 提供了格式化输入输出功能
  - `io`: 提供了基本输入输出功能，大多数是围绕系统功能的封装
  - `bufio`: 缓冲输入输出功能的封装
  - `path/filepath`: 用来操作在当前系统中的目标文件名路径
  - `flag`: 对命令行参数的操作　
- 字符串操作类
  - `strings`: 提供对字符串的操作
  - `strconv`: 提供将字符串转换为基础类型的功能
  - `unicode`: 为 unicode 型的字符串提供特殊的功能
  - `regexp`: 正则表达式功能
  - `bytes`: 提供对字符型分片的操作
  - `index/suffixarray`: 子字符串快速查询
- 数学
  - `math`: 基本的数学函数
  - `math/cmath`: 对复数的操作
  - `math/rand`: 伪随机数生成
  - `sort`: 为数组排序和自定义集合
  - `math/big`: 大数的实现和计算
- 数据结构
  - `list`: 双链表
  - `ring`: 环形链表
- 时间
  - `time`: 日期和时间的基本操作
  - `log`: 记录程序运行时产生的日志,我们将在后面的章节使用它
- 编/解码
  - `encoding/json`: 读取并解码和写入并编码 JSON 数据
  - `encoding/xml`:简单的 XML1.0 解析器
  - `text/template`:生成像 HTML 一样的数据与文本混合的数据驱动模板
- 网络
  - `net`: 网络数据的基本操作。
  - `http`: 提供了一个可扩展的 HTTP 服务器和基础客户端，解析 HTTP 请求和回复。
  - `html`: HTML5 解析器
- `runtime`: Go 程序运行时的交互操作，例如垃圾回收和协程创建。
- `reflect`: 实现通过程序运行时反射，让程序操作任意类型的变量。

Go的生态远不止这些标准库，社区里存在大量的第三方包或项目，在开发自己的项目时，最好先查找下是否有已存在的第三方包或可用的库。 [Go Walker](https://gowalker.org/) 支持根据包名在海量数据中查询。 

### 2.3 导入包

导入包的基本格式如下

```go
import "包的路径或 URL 地址" 
```

”包的路径“指可以是以下这种形式

```go
import "./pack1/pack1"
```

而URL地址指的是下载的外部安装包

```go
import "github.com/org1/pack1”
```

导入时可以对包进行重命名，比如

```go
import packx "github.com/org1/pack1”
```

也可以重命名为两种特殊符号：`.`和`_`，前者可以在使用时省略包名，直接使用对外部可见的函数和变量，后者则只执行其中的init函数和初始化其全局变量，无法调用函数

```go
import . "./pack1"
import _ "./pack1/pack1"
```

### 2.4 使用脚本编译或安装

 在 Linux/OS X 下可以用 Makefile 脚本 实现自动检测机器架构并调用正确的编译器和链接器

```go
include $(GOROOT)/src/Make.inc
TARG=pack1
GOFILES=\
 	pack1.go\
 	pack1b.go\
include $(GOROOT)/src/Make.pkg
```

 通过 `chmod 777 ./Makefile`确保它的可执行性，然后在终端使用make工具

## 3. 模块

自 Go 1.11 起，开始支持使用 Go Module 进行包管理，这里参考 Go Blog 中的 [Using Go Modules](https://blog.golang.org/using-go-modules) 一文对其进行介绍。

模块(module)通过项目根目录下`go.mod`文件起作用，项目中使用的所有包的集合都在该文件中定义。`go.mod`文件定义了模块路径，用于项目中包的导入路径。以 delve 的`go.mod`文件为例

```go
module github.com/go-delve/delve

go 1.10

require (
	github.com/cosiner/argv v0.0.0-20170225145430-13bacc38a0a5
	github.com/cpuguy83/go-md2man v1.0.8 // indirect
	github.com/inconshreveable/mousetrap v1.0.0 // indirect
	github.com/kr/pretty v0.1.0 // indirect
	github.com/mattn/go-colorable v0.0.0-20170327083344-ded68f7a9561
	github.com/mattn/go-isatty v0.0.3
	github.com/onsi/ginkgo v1.8.0 // indirect
	github.com/onsi/gomega v1.5.0 // indirect
	github.com/peterh/liner v0.0.0-20170317030525-88609521dc4b
	github.com/pkg/profile v0.0.0-20170413231811-06b906832ed0
	github.com/russross/blackfriday v0.0.0-20180428102519-11635eb403ff // indirect
	github.com/sirupsen/logrus v0.0.0-20180523074243-ea8897e79973
	github.com/spf13/cobra v0.0.0-20170417170307-b6cb39589372
	github.com/spf13/pflag v0.0.0-20170417173400-9e4c21054fa1 // indirect
	github.com/stretchr/testify v1.3.0 // indirect
	go.starlark.net v0.0.0-20190702223751-32f345186213
	golang.org/x/arch v0.0.0-20171004143515-077ac972c2e4
	golang.org/x/crypto v0.0.0-20180614174826-fd5f17ee7299 // indirect
	golang.org/x/sys v0.0.0-20190626221950-04f50cda93cb
	golang.org/x/tools v0.0.0-20181120060634-fc4f04983f62
	gopkg.in/airbrake/gobrake.v2 v2.0.9 // indirect
	gopkg.in/check.v1 v1.0.0-20180628173108-788fd7840127 // indirect
	gopkg.in/gemnasium/logrus-airbrake-hook.v2 v2.1.2 // indirect
	gopkg.in/yaml.v2 v2.2.1
)
```

注意，使用模块进行包管理独立于原来的使用 gopath 进行包管理，可以在任意位置建立项目。如果`go.mod`文件放在`$GOPATH/src`，即原来的统一工作区，是不起作用的，会被旧的GOPATH模式屏蔽。自 Go 1.13 开始，module 模式已经成为了 Go 开发的默认模式。

### 3.1 创建新模块

在`$GOPATH/src`外的任意位置创建项目目录，`cd`进入该目录，创建`hello.go`文件

```go
package hello

func Hello() string {
    return "Hello, world."
}
```

为它写一个测试文件`hello_test.go`

```go
package hello

import "testing"

func TestHello(t *testing.T) {
    want := "Hello, world."
    if got := Hello(); got != want {
        t.Errorf("Hello() = %q, want %q", got, want)
    }
}
```

现在，项目中有了一个包，因为项目目录位于`F:/hello`，执行`go test`的结果如下

```bash
$ go test
PASS
ok      _/F_/hello 0.240s
```

因为我们在 `$GOPATH` 外进行的测试，而且现在还没有`go.mod`文件，所以Go命令不知道导入路径，只能根据目录名生成一个导入路径。现在我们来创建`go.mod`文件并再次执行`go test`命令

```bash
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
$ go test
PASS
ok      example.com/hello       0.214s
```

`go mod init`命令用于创建`go.mod`文件，初始化Go模块，其内容如下

```bash
$ pwd
/f/hello
$ ls
go.mod  hello.go  hello_test.go
$ cat go.mod
module example.com/hello

go 1.13
```

`go.mod`文件位于项目根目录，其中的模块路径也只显示到项目根目录，子目录中包的导入路径由模块路径+子目录路径组成。我们在当前项目目录下创建`world`子目录，其导入路径将会是`example.com/hello/world`

### 3.2 添加依赖

使用Go模块的首要目的是提升别的开发者使用我们编写的代码的体验。编辑`hello.go`文件，导入`rsc.io/quote`，然后用它来实施`Hello`

```go
package hello

import "rsc.io/quote"

func Hello() string {
    return quote.Hello()
}
```

现在运行`go test`

```bash
$ go test
go: finding rsc.io/quote v1.5.2
go: downloading rsc.io/quote v1.5.2
go: extracting rsc.io/quote v1.5.2
go: finding rsc.io/sampler v1.3.0
go: finding golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
go: downloading rsc.io/sampler v1.3.0
go: extracting rsc.io/sampler v1.3.0
go: downloading golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
go: extracting golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
PASS
ok      example.com/hello    0.023s
```

Go使用`go.mod`文件中列举的依赖模块版本来解析导入的包，但是当在`go.mod`文件中找不到这个包是，就会自动的查找包含这个包的模块的最新版本，然后添加到`go.mod`文件。上例中自动下载了模块`rsc.io/quote v1.5.2`，还下载了它依赖的两个其它模块：`rsc.io/sampler`和`golang.org/x/text`。不过只有直接依赖会被记录到`go.mod`文件

```bash
$ cat go.mod
module example.com/hello

go 1.13

require rsc.io/quote v1.5.2
```

下载的包位于`$GOPATH/pkg/mod`，再次运行`go test`命令，就不会重新下载了，因为已经存在。

```bash
$ go test
PASS
ok      example.com/hello    0.020s
```

需要注意的是，尽管使用模块使得添加包管理简单轻松，但并非没有代价，当前代码的安全性、正确性和许可等都依赖于导入的模块，关于这个问题的详细描述可以查看 [Our Software Dependency Problem](https://research.swtch.com/deps). 

如上所述，添加一个直接依赖通常会引入一些间接的依赖，使用`go list -m all`可以列举当前模块和它们所有的依赖。

```bash
$ go list -m all
example.com/hello
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
rsc.io/quote v1.5.2
rsc.io/sampler v1.3.0
```

在`go list`的输出中，当前模块是主模块，总是位于第一行，其它的模块按模块路径顺序显示。

`go.mod`文件外，Go还维持一个名为`go.sum`的文件，包含特定模块版本内容的加密哈希。Go使用该文件确保将来下载该模块时内容与第一次下载的内容相同，从而保证所依赖的模块没有被恶意或非恶意的更改。

```bash
$ ls
go.mod  go.sum  hello.go  hello_test.go
$ cat go.sum
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c h1:qgOY6WgZO...
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:Nq...
rsc.io/quote v1.5.2 h1:w5fcysjrx7yqtD/aO+QwRjYZOKnaM9Uh2b40tElTs3...
rsc.io/quote v1.5.2/go.mod h1:LzX7hefJvL54yjefDEDHNONDjII0t9xZLPX...
rsc.io/sampler v1.3.0 h1:7uVkIFmeBqHfdjD+gZwtXXI+RODJ2Wc4O7MPEh/Q...
rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9...
```

`go.mod`和`go.sum`文件都应该包含到版本控制系统中，也就是说随着源代码文件一起提交到远程仓库。

### 3.3 更新依赖

Go模块使用的版本号分三部分：主版本号(major)，次版本号(minor)和修订版本号(patch)。例如，`v0.1.2`，主版本号是`0`，次版本号是`1`，修订版本号是`2`。首先以一个次版本号的更新举例说明

前面`go list -m all`的输出中我们可以看到`golang.org/x/text`使用的不是标准的版本号，我们可以使用`go get`命令将其更新

```bash
$ go get golang.org/x/text
go: finding golang.org/x/text v0.3.0
go: downloading golang.org/x/text v0.3.0
go: extracting golang.org/x/text v0.3.0
$ go test
PASS
ok      example.com/hello    0.013s
```

现在再次列举所有的模块以及查看`go.mod`文件

```bash
$ go list -m all
example.com/hello
golang.org/x/text v0.3.0
rsc.io/quote v1.5.2
rsc.io/sampler v1.3.0
$ cat go.mod
module example.com/hello

go 1.12

require (
    golang.org/x/text v0.3.0 // indirect
    rsc.io/quote v1.5.2
)
```

可以看到两者中`golang.org/x/text`都已经更新到了最新的版本号`v0.3.0`，注释中的`indirect`说明这个依赖不是直接被引用的，只是被其它的模块间接引用。

现在我们更新`rsc.io/sampler`的次版本号

```bash
$ go get rsc.io/sampler
go: finding rsc.io/sampler v1.99.99
go: downloading rsc.io/sampler v1.99.99
go: extracting rsc.io/sampler v1.99.99
$ go test
--- FAIL: TestHello (0.00s)
    hello_test.go:8: Hello() = "99 bottles of beer on the wall, 99 bottles of beer, ...", want "Hello, world."
FAIL
exit status 1
FAIL    example.com/hello    0.014s
```

测试失败说明版本不匹配，列举所有可用版本

```bash
$ go list -m -versions rsc.io/sampler
rsc.io/sampler v1.0.0 v1.2.0 v1.2.1 v1.3.0 v1.3.1 v1.99.99
```

之前使用的是v1.3.0，v1.99.99也被证明无法使用，因此换用v1.3.1

```bash
$ go get rsc.io/sampler@v1.3.1
go: finding rsc.io/sampler v1.3.1
go: downloading rsc.io/sampler v1.3.1
go: extracting rsc.io/sampler v1.3.1
$ go test
PASS
ok      example.com/hello    0.022s
```

通过`@`来指定具体的版本号，`go get`的参数实际上应该带有版本号，不明确指定会使用默认的`@latest`，从而使用最新版本。

### 3.4 在主版本号上添加依赖

主版本号的更新是不同的，会看作一个独立的依赖。在`hello.go`中添加一个新的函数`Proverb()`，它使用了`rsc.io/quote`的`v3`版本，导入格式为`rsc.io/quote/v3`

```go
package hello

import (
    "rsc.io/quote"
    quoteV3 "rsc.io/quote/v3"
)

func Hello() string {
    return quote.Hello()
}

func Proverb() string {
    return quoteV3.Concurrency()
}
```

然后为其添加测试函数

```go
func TestProverb(t *testing.T) {
    want := "Concurrency is not parallelism."
    if got := Proverb(); got != want {
        t.Errorf("Proverb() = %q, want %q", got, want)
    }
}
```

测试结果如下

```bash
$ go test
go: finding rsc.io/quote/v3 v3.1.0
go: downloading rsc.io/quote/v3 v3.1.0
go: extracting rsc.io/quote/v3 v3.1.0
PASS
ok      example.com/hello    0.024s
```

现在我们的模块同时依赖于`rsc.io/quote`和`rsc.io/quote/v3`

```bash
$ go list -m rsc.io/q...
rsc.io/quote v1.5.2
rsc.io/quote/v3 v3.1.0
```

Go模块的每个主版本号(v1, v2, and so on)都使用一个不同的模块路径，从`v2`开始，路径必须以主版本号结尾，比如`rsc.io/quote`的`v3`版本模块路径为`rsc.io/quote/v3`，这种惯例称作 [semantic import versioning](https://research.swtch.com/vgo-import) ，给不兼容的包提供了不同的名字。相反，同一个主版本号，如`v1.6.0`应该向前兼容，和`rsc.io/quote`使用同一个名字，在同一个项目中，每个主版本号Go只允许出现一种，比如`v1.5.2`和`v1.6.0`不能同时存在，但不同的主版本号可以同时存在，这是为了使程序可以逐步过渡到新的版本。

### 3.5 更新依赖到新的主版本号

比如从`rsc.io/quote`迁移到`rsc.io/quote/v3`，由于大版本更新，某些不兼容的API可能被删除、重命名或做其它的更改，阅读文档，我们可以发现`HelloV3`相对于`Hello`做了如下改动

```bash
$ go doc rsc.io/quote/v3
package quote // import "rsc.io/quote"

Package quote collects pithy sayings.

func Concurrency() string
func GlassV3() string
func GoV3() string
func HelloV3() string
func OptV3() string
```

检查并更新我们的源码

```go
package hello

import quoteV3 "rsc.io/quote/v3"

func Hello() string {
    return quoteV3.HelloV3()
}

func Proverb() string {
    return quoteV3.Concurrency()
}
```

由于没有冲突，这里导入的包不再需要重命名，因此可以继续简化

```go
package hello

import "rsc.io/quote/v3"

func Hello() string {
    return quote.HelloV3()
}

func Proverb() string {
    return quote.Concurrency()
}
```

运行测试

```bash
$ go test
PASS
ok      example.com/hello       0.014s
```

### 3.6 移除不使用的依赖

我们已经不再使用`rsc.io/quote`，但它依然存在于`go list -m all`的输出和`go.mod`文件中

```bash
$ go list -m all
example.com/hello
golang.org/x/text v0.3.0
rsc.io/quote v1.5.2
rsc.io/quote/v3 v3.1.0
rsc.io/sampler v1.3.1
$ cat go.mod
module example.com/hello

go 1.12

require (
    golang.org/x/text v0.3.0 // indirect
    rsc.io/quote v1.5.2
    rsc.io/quote/v3 v3.0.0
    rsc.io/sampler v1.3.1 // indirect
)
```

这是因为编译单个包时无法自动检测是否可以安全的移除某个包，只能通过手动执行`go mod tidy`命令清理不再使用的依赖

```bash
$ go mod tidy
$ go list -m all
example.com/hello
golang.org/x/text v0.3.0
rsc.io/quote/v3 v3.1.0
rsc.io/sampler v1.3.1
$ cat go.mod
module example.com/hello

go 1.12

require (
    golang.org/x/text v0.3.0 // indirect
    rsc.io/quote/v3 v3.1.0
    rsc.io/sampler v1.3.1 // indirect
)

$ go test
PASS
ok      example.com/hello    0.020s
$
```

最后是关于发布自己的模块版本的注意，参考两篇文章

- [Publishing Go Modules](https://blog.golang.org/publishing-go-modules)
- [Go Modules: v2 and Beyond](https://blog.golang.org/v2-go-modules)





