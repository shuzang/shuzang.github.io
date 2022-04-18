---
title: Golang语法基础1-背景与初探
date: 2019-09-13
lastmod: 2020-06-13
tags: [Go语法]
categories: [Golang学习之路]
slug: Golang Basic Grammar 1-Background and Preliminary Exploration 
---

## 1. 起源与发展

Go/Golang 起源于2007年，并于2009年正式对外发布，是一个完全开源的项目，背后的支持者是谷歌公司，核心设计者是三位著名IT工程师：Ken Thompson，Rob Pike，Robert Griesemer。分别是如下从左到右三位

![Go语言核心设计者](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20190913_3AxHPJ.jpg)

其中 Ken Thompson 是 Unix 操作系统的设计者，并因此获得图灵奖，也是C语言前身B语言的设计者，UTF-8 编码设计者之一，计算机史的重要人物，2006年加入谷歌，和另外两人一起设计了Go语言。 Rob Pike 是 Ken 的老搭档。

随后又有 lan Lance Taylor 和 Russ Cox 两人加入团队，前者是 gccgo 编译器的作者和 cgo工具链的维护者，后者加入团队后着手Go语言标准库的开发。下图分别是他们两个

![Go语言核心开发者](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20190913_3AxT54.jpg)

Go语言以囊地鼠(Gopher)为图标和吉祥物，这是一种原产于加拿大的啮齿类动物，Go语言开发者也一般自称为 Gopher。下图中左边是囊地鼠，右边是 Go logo.

![Gopher](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20190913_3AxXKx.jpg)

Go语言相比于其它语言的最大优势在于它的执行性能与开发效率，这得益于Go在并发编程、内存回收等许多方面的良好设计，并因此大规模用于服务器编程、网络编程、数据库和云平台领域。

比较出名的Go语言项目有(不限于这些)

- Go语言本身： https://github.com/golang/go 
- Docker： https://www.docker.com/ 
- kubernetes： https://github.com/kubernetes/kubernetes 
- Ethereum： https://github.com/ethereum/go-ethereum 
- fabric： https://github.com/hyperledger/fabric 
- Hugo： https://github.com/gohugoio/hugo 
- TiDB： https://github.com/pingcap/tidb 
- InfluxDB： https://github.com/influxdata/influxdb 
- ETCD： https://github.com/etcd-io/etcd 

使用Go的国外公司有：Google, Docker, NetFlix, CloudFlare, Dropbox, MongoDB, Uber等

使用Go的国内公司有：七牛、字节跳动、bilibili、京东、[百度]( https://github.com/baidu/bfe )，其它公司如小米、腾讯、阿里等也都在使用Go，但可能不是主力

## 2. 跟踪最新动态

最直接的方式是跟踪Go语言的源码库，关注提交历史和[issue]( https://github.com/golang/go/issues )

1. 原始代码库： https://go.googlesource.com/go
2. github镜像： https://github.com/golang/go

其它活跃论坛或动态

- [golang-dev](https://groups.google.com/forum/#!forum/golang-dev)：Google邮件列表的Go开发组讨论区
- [golang-nuts](https://groups.google.com/forum/#!forum/golang-nuts)：Google邮件列表的Go讨论社区
- [golang-announce](https://groups.google.com/forum/#!forum/golang-announce)：发布Go版本或Go开发的最新状态
- [go.dev](https://go.dev/)：刚刚上线(2019.11.14)的Go开发人员中心
- [gotime](https://changelog.com/gotime)：Go的一个播客，每周一更，内容有干货
- [@golang](https://twitter.com/golang)：Go 语言在 Twitter 的官方帐号 

此外还有每年举办的几个大会

- [Gopher Con](https://www.gophercon.com/ )，举办地在美国，时间不定，今年在7月，2020年会在6月份举行。[会议总结](https://github.com/gophercon)
- [GopherChina](https://gopherchina.org/ )，举办地在中国，每年4月份。[会议总结](https://github.com/gopherchina/conference)
- [dotGo](https://www.dotgo.eu/ )，举办地在欧洲，每年3月份
- 详细的会议列表可查看 https://github.com/golang/go/wiki/Conferences 

Go 下载地址和相关的文档、标准库等访问地址为

- 官网 https://golang.org/ 
- 国内的镜像网站 https://golang.google.cn/

[Go语言中文网]( https://studygolang.com/ ) 是国内最活跃的Go社区，每周会发行一份[Go语言爱好者周刊](https://studygolang.com/go/weekly )

Go相关资料聚集最多的还是[go wiki](https://github.com/golang/go/wiki/Articles)

## 3. 下载安装

Windowns下快速安装可以使用 chocolatey ，执行如下命令即可

```powershell
> choco install golang
```

自动配置环境变量，安装完重启终端即可使用。下面开始介绍常规的安装方法，以win10和Ubuntu为例。

### 3.1 下载

Golang中国官网下载页面为 [golang.google.cn/dl](https://golang.google.cn/dl/)，为windows，macOS和Linux三种环境都提供了安装包。

![Golang下载页面](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20190913_Golang%E4%B8%8B%E8%BD%BD%E9%A1%B5%E9%9D%A2.png)

windows默认下载文件为`go1.14.windows-amd64.msi`，双击启动即可安装，默认安装位置为`C:\Go`，环境变量将自动设置。但如果下载了以`.zip`为后缀的版本，则需要自己解压到合适的路径，并自己设置环境变量。

Linux默认下载文件为`go1.14.linux-amd64.tar.gz`，将其解压缩到`/usr/local`，然后手动将`/usr/local/go/bin`添加到 PATH 环境变量中。

### 3.2 安装

#### windows

`msi`后缀的安装版本双击安装，环境变量将会自动设置，打开powershell，使用`go env`命令查看环境变量

```powershell
> go env
set GO111MODULE=
set GOARCH=amd64
set GOBIN=
set GOCACHE=C:\Users\lylw1\AppData\Local\go-build
set GOENV=C:\Users\lylw1\AppData\Roaming\go\env
set GOEXE=.exe
set GOFLAGS=
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOINSECURE=
set GONOPROXY=
set GONOSUMDB=
set GOOS=windows
set GOPATH=C:\Users\lylw1\go
set GOPRIVATE=
set GOPROXY=https://goproxy.cn,direct
set GOROOT=c:\go
set GOSUMDB=sum.golang.org
set GOTMPDIR=
set GOTOOLDIR=c:\go\pkg\tool\windows_amd64
set GCCGO=gccgo
set AR=ar
set CC=gcc
set CXX=g++
set CGO_ENABLED=1
set GOMOD=
set CGO_CFLAGS=-g -O2
set CGO_CPPFLAGS=
set CGO_CXXFLAGS=-g -O2
set CGO_FFLAGS=-g -O2
set CGO_LDFLAGS=-g -O2
set PKG_CONFIG=pkg-config
set GOGCCFLAGS=-m64 -mthreads -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=C:\Users\lylw1\AppData\Local\Temp\go-build192094374=/tmp/go-build -gno-record-gcc-switches
```

使用 `go version` 查看版本同时验证安装

```go
> go version
go version go1.13.4 windows/amd64
```

####  Linux

以Ubuntu 20.04 LTS 为例说明，首先在 [golang.org/dl](https://golang.org/dl/) 页面右击要下载的版本获取下载链接，使用wget下载文件。也可以直接下载。

```bash
$ wget https://dl.google.com/go/go1.14.linux-amd64.tar.gz
```

将文件解压缩到指定目录/usr/local，`-C`参数用于指定目标文件夹，解压缩后删除压缩文件。

```bash
$ sudo tar -xzf go1.14.linux-amd64.tar.gz -C /usr/local
$ rm go1.14.linux-amd64.tar.gz
```

查看安装目录

```bash
$ ls /usr/local
bin  etc  games  go  include  lib  man  sbin  share  src
$ ls /usr/local/go
AUTHORS          CONTRIBUTORS  PATENTS    SECURITY.md  api  doc          lib   pkg         src
CONTRIBUTING.md  LICENSE       README.md  VERSION      bin  favicon.ico  misc  robots.txt  test
```

设置环境变量

```bash
$ sudo nano /etc/profile
# 在打开的文件末尾添加下列语句
export PATH=$PATH:/usr/local/go/bin
```

更新的环境变量可以通过下面的命令使其直接生效

```bash
$ source /etc/profile
# 查看生效后的环境变量设置
$ go version
go version go1.14 linux/amd64
```

Linux 全部环境变量如下

```bash
$ go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/home/shuzang/.cache/go-build"
GOENV="/home/shuzang/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOINSECURE=""
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/home/shuzang/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build153919612=/tmp/go-build -gno-record-gcc-switches"
```

注：将环境变量添加到/etc/profile可使go全局启用，也可添加到$HOME/.profile，对当前登录用户起作用。

*注：可以使用apt-get命令安装，但安装的go不会是最新版本，同样，也可以使用snap安装，可以安装最新版本。*

### 3.3 环境变量说明

常规的使用除了GOPATH外无需关心其它环境变量，但在进行交叉编译时，需要进行一定的设置，详细的环境变量说明可以查看 [$GOPATH environment variable set correctly](https://golang.org/doc/install/source#gopath)，以下解释几个主要的环境变量。

- `GOOS`与`GOARCH`：目标操作系统和处理器架构，
- `GOHOSTOS`和`GOHOSTARCH`是宿主机操作系统和处理器架构。
- `GOBIN`：编译器和链接器的安装位置，默认是`$GOROOT/bin`，一般将其设置为空值，Go会使用默认值。
- `GOROOT`：电脑上安装的Go的根目录，Linux下一般为`$HOME/go`，windows下为`C:\go`
- `GOPATH`：go install、go get等命令默认路径都是`GOPATH`，是编译后二进制文件的存放目的地、下载后包的存放路径以及 import 包时的搜索路径

我们需要关心的唯一一个环境变量是GOPATH，可以看到，默认的GOPATH是`c:\Users\lylw1\go`，其中`lylw1`是自己的用户名。

GOPATH 所代表的路径是 go 的工作区，可以选择任意自己喜欢的文件夹作为go的工作区，如果不想使用默认的路径，需要自己进行修改，使用 `go env` 命令

```powershell
> go env -w GOPATH=/somewhere/else
```

更多的环境变量修改方式可以查看[Setting GOPATH](https://github.com/golang/go/wiki/SettingGOPATH#windows)，需要注意的一点是GOPATH不能设置为Go的安装目录`C:\go`

### 3.4 安装目录清单

安装完成的Go根目录（$GOROOT）文件夹结构除了README.md，AUTHORS，LICENSE等常规文件外，基本结构应如下所示：

- `/bin`：包含可执行文件，如：编译器，Go 工具
- `/doc`：包含示例程序，代码工具，本地文档等
- `/lib`：包含文档模版
- `/misc`：包含与支持 Go 编辑器有关的配置文件以及 cgo 的示例
- `/pkg/os_arch`：包含标准库的包的对象文件（`.a`）
- `/src`：包含源代码构建脚本和标准库的包的完整源代码（Go 是一门开源语言）
- `/src/cmd`：包含 Go 和 C 的编译器和命令行脚本

这是之前的介绍，网上搜到的多是这个说明，但当前还多了`api`和`test`两个文件夹。

### 3.5 代码目录结构

Go 1.13之前，所有代码在一个工作区，Go 1.13及之后，使用 Go module 进行管理，项目代码不再必须放到 GOPATH 下。本文假设使用 Go 1.13 及以后的版本。对于之前的方式不再介绍。

Go 程序以包（package）的形式进行组织，一个包就是一个目录，该目录下可能有一个或多个源文件，同一个包的所有代码会被一起编译。另外，同一个包中定义的变量、常量、类型、函数，即使在不同的源文件中，也可以相互访问。

Go 项目以模块的方式进行组织，模块（module）是一组相关包的集合。一个项目通常是一个仓库（repository），而一个 Go 仓库（repository）只能包含一个模块，但可导入使用多个其它模块，仓库根目录下的 `go.mod` 文件用来声明这些导入模块的路径。和 `go.mod` 文件一起位于仓库根目录的是一个一个的子目录，每个子目录可能是一个包，也可能是一个子模块，是否为子模块要看子目录下是否存在 `go.mod` 文件。

值得注意的是，尽管项目代码以仓库的方式进行组织，但在编写完成构建之前，代码不需要上传到云端，模块可以本地存在，仓库形式仅仅是因为我们总有一天会将代码上传到云的。

模块的路径不仅作为导入路径，同样也说明了模块的下载路径，比如 `golang.org/x/tools`， Go 命令将知道从 `https://golang.org/x/tools` 去下载它。包的导入路径是模块路径 + 包名，比如，模块 `github.com/google/go-cmp` 包含一个包（子目录）`cmp`，这个包的导入路径就是 `github.com/google.go-cmp/cmp`。Go 的标准库导入不需要模块路径前缀，可以仅使用包名。

最后，所有的模块都会下载到 GOPATH 的 `pkg/mod` 目录下，并被所有项目共享，因此，模块代码是只读的。

本部分说明可以阅读第二篇文章后再来看。

## 4. 第一个程序

编辑并运行我们的第一个程序，按照编程界的惯例，输出`hello, world!`。

首先创建项目文件夹并初始化模块，模块路径自行选择

```bash
$ mkdir hello
$ cd hello
$ go mod init example.com/user/hello
go: creating new go.mod: module example.com/user/hello
$ cat go.mod
module example.com/user/hello

go 1.14
$
```

然后在该目录下创建`hello.go`文件，使其包含如下代码

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```

使用`go install`命令编译安装

```bash
> go install github.com/shuzang/hello
```

该命令会编译程序并产生一个二进制包，然后将该二进制包安装到 `$HOME/go/bin/hello` ，如果是 Windows，会安装到 ` %USERPROFILE%\go\bin\hello.exe`。

安装目录受 GOPATH 和 GOBIN 影响，如果 GOBIN 被设置，则二进制包会被安装到设置的目录，如果 GOPATH 被设置，二进制包将按照到 GOPATH 列表的第一个目录的 bin 子目录，如果都没有被设置，就会安装到默认 GOPATH 的 bin 子目录(`$HOME/go` or `%USERPROFILE%\go`)。

`go install` 命令执行需要在仓库根目录下，且下面三个命令是等效的

```bash
$ go install example.com/user/hello
$ go install .
$ go install
```

将`$GOPATH/bin`添加到电脑的`PATH`环境变量中，这样可以直接执行

```powershell
$ hello
Hello, world!
```

可能是因为我使用chocolatey安装的原因，这一环境变量也已经自动设好了，所以我可以直接执行命令。

完成编辑后，就可以使用版本控制工具初始化仓库并上传代码了，示例如下（这一步不是必须的）

```powershell
$ git init
Initialized empty Git repository in C:/Users/lylw1/go/src/github.com/shuzang/hello/.git/
$ git add go.mod hello.go
$ git commit -m "initial commit"
[master (root-commit) 0b4507d] initial commit
 1 file changed, 7 insertion(+)
 create mode 100644 go.mod hello.go
```

一种最方便的方式是将模块路径与仓库路径匹配，比如将以上模块声明为 `github.com/username/hello`，多数版本控制服务商都支持这种格式。

## 5. 编辑器/IDE

Golang 开发最流行的两个工具是 Goland 和 VScode，我自己是 VScode 的使用者。除了这两个工具外，官方还提供了一份[IDE和插件列表](https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins )。

VScode 中的 Go 扩展提供了大量的特性，如自动补全、悬停信息显示、括号匹配等，原本属于第三方开发者维护，现在交给了 Go 团队。详细的特性说明查看[官网](https://code.visualstudio.com/docs/languages/go)，下面进行一些简单介绍。

### 5.1 Go工具链

微软在开发 VS Code 过程中, 定义了一种协议： [语言服务器协议](https://link.zhihu.com/?target=https%3A//microsoft.github.io/language-server-protocol/) ，用来为每种语言提供诸如自动完成，代码提示等功能。gopls 就是Go语言的服务器, 安装命令为

```bash
$ go get -v golang.org/x/tools/gopls
```

事实上，编辑 Go 代码时如果没有安装，VScode 会在右下角弹出提示，只要直接点击 `Install` 即可，不需要自己输入命令。同时，因为没有设置go.toolsGopath，默认使用了 GOPATH 作为安装路径

```bash
go.toolsGopath setting is not set. Using GOPATH C:\Users\lylw1\go
Installing 1 tool at C:\Users\lylw1\go\bin in module mode.
  gopls

Installing golang.org/x/tools/gopls SUCCEEDED

Reload VS Code window to use the Go language server
All tools successfully installed. You're ready to Go :).

```

Go代码的调试需要 delve 工具

```go
$ go get -v github.com/go-delve/delve/cmd/dlv
```

除此之外，还有一批需要安装的分析工具，如下

```bash
# Below tools are needed for the basic features of the Go extension.
gocode
gopkgs
go-outline
gocode-gomod
godef
goreturns
```

所有这些工具只需要在右下角弹出提示后点击 `Install` 即可

~~不过上述提到的很多插件不翻墙都无法下载，可以自己手动到github下载然后解压到`GOPATH/src`，建立起对应的目录结构，然后运行`go install`将其安装~~

微软设置了默认的代理服务器，现在(2020.03.18)所有插件均可在不翻墙的情况下顺利安装，而且速度很快

### 5.2 用户和工作区设置

使用 VScode 需要关心的一个重要部分是用户和工作区设置，几乎所有的事情都和它们有关。

![](https://code.visualstudio.com/assets/docs/getstarted/settings/hero.png)

这是两种不同的设置范围

- 用户设置：是一个全局的设置，适用于打开的任何VScode窗口
- 工作区设置：如题，项目工作区的设置，只适用于对应的工作区

工作区的设置会覆盖掉用户设置，它针对具体的项目，配置文件位于项目根目录`.vscode`文件夹，可与其它开发者共享。`.vscode`文件夹还用于存放调试配置和任务配置。

点击左下角的齿轮，选择`设置`，默认的设置界面如上，是一个可视化的界面，不过也可以使用`settings.json`配置文件

- 用户设置文件在windows中位于`%APPDATA%\Code\User\settings.json`
- 工作区设置文件位于根目录的`.vscode`文件夹中

最后，VScode大量的操作都可以通过命令完成，使用快捷键`Ctrl+Shift+P`可以打开命令输入框。

### 5.3 特性说明

在用户或工作区设置中，将 `go.autocompleteUnimportedPackages` 设为`true`  ，可以在代码中点击包名跳转查看包的具体内容。

鼠标悬停在变量、函数和结构体的名称上方可以查看它们的签名等信息，这一功能由前面安装的`godef`工具实现，同样可完成这一功能的工具还有`godoc`和`gogetdoc`，通过在用户或工作区设置中调整`go.docsTool`来切换工具。

代码导航功能无需设置默认实现。

对源码的保存操作会自动触发格式化、编译和代码质量检查。格式化工具为`gofmt`，可选的替代工具有`goreturns`和`goimports`，在用户或工作区设置中调整`go.formatTool`来设置。编译的过程使用`go build`命令。代码质量检查的工具为`golint`，也可以使用`gometalinter`，用来检查代码的规范性，检查得到的`errors`和`warning`会在编辑器里以红色/绿色波浪线标出来，下面的输出窗口也会显示详细信息。

所有可选的替代工具都需要自己安装，使用快捷键打开命令面板，输入 `Go: Install/Update Tools `，选择要安装的工具，点击确定即可。

### 5.4 使用

**调试**使用的是前面安装的`delve`工具。在 VScode 中，按`F5`启动调试，一般情况下使用默认的调试配置即可，不过还是应当对调试配置选项有一定的了解。

默认是没有调试配置文件的，当我们需要进行配置时，输入命令 `Debug: Open launch.json `，第一次会在工作区根目录的`.vscode`文件夹中创建`launch.json`文件，文件中默认的配置信息如下

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${fileDirname}",
            "env": {},
            "args": []
        }
    ]
}
```

可设置的配置属性可查看[Debugging Go code using VS Code](https://github.com/Microsoft/vscode-go/wiki/Debugging-Go-code-using-VS-Code)，更多关于VScode中Go调试的相关信息都可查看该文档。

这里需要注意的是`args`，调试时程序需要传入的参数数组是该配置参数的值

**运行**程序使用快捷键`Ctrl+F5`，和调试使用的是同一个配置文件。