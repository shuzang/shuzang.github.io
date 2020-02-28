# Golang语法基础1-背景与开发环境准备


## 1. 起源与发展

Go/Golang起源于2007年，并于2009年正式对外发布，是一个完全开源的项目，背后的支持者是谷歌公司，核心设计者是三位著名IT工程师：Ken Thompson，Rob Pike，Robert Griesemer。分别是如下从左到右三位

![Go语言核心设计者](https://s2.ax1x.com/2020/02/19/3AxHPJ.jpg)

其中Ken Thompson是 Unix操作系统的设计者，并因此获得图灵奖，也是C语言前身B语言的设计者，UTF-8编码设计者之一，计算机史的重要人物，2006年加入谷歌，和另外两人一起设计了Go语言。 Rob Pike是Ken的老搭档。

随后又有lan Lance Taylor和Russ Cox两人加入团队，前者是gccgo编译器的作者和cgo工具链的维护者，后者加入团队后着手Go语言标准包的开发。下图分别是他们两个

![Go语言核心开发者](https://s2.ax1x.com/2020/02/19/3AxT54.jpg)

Go语言以囊地鼠(Gopher)为图标和吉祥物，这是一种原产于加拿大的啮齿类动物，Go语言开发者也一般自称为Gopher。下图中左边是囊地鼠，右边是Go logo.

![Gopher](https://s2.ax1x.com/2020/02/19/3AxXKx.jpg)

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

使用Go的国内公司有：七牛、字节跳动、bilibili、京东、[百度]( https://github.com/baidu/bfe )，其实大部分公司包括小米、腾讯、阿里等都在使用Go

## 2. 跟踪最新动态

最直接的方式是跟踪Go语言的源码库，关注提交历史和[issue]( https://github.com/golang/go/issues )

1. 原始代码库： https://go.googlesource.com/go
2. github镜像： https://github.com/golang/go

其它活跃论坛或动态

- [golang-dev](https://groups.google.com/forum/#!forum/golang-dev)：Google邮件列表的Go开发组讨论区
- [golang-nuts](https://groups.google.com/forum/#!forum/golang-nuts)：Google邮件列表的Go讨论社区
- [golang-announce](https://groups.google.com/forum/#!forum/golang-announce)：发布Go版本或Go开发的最新状态
- [go.dev](https://go.dev/)：刚刚上线(2019.11.14)的Go开发人员中心
- [golangclub](https://golangclub.com/)：go.dev在国内对应的本地化站点
- [gotime](https://changelog.com/gotime)：Go的一个播客，每周一更，内容有干货
- [@golang](https://twitter.com/golang)：Go 语言在 Twitter 的官方帐号 

此外还有每年举办的几个大会

- Gopher Con，举办地在美国，时间不定，今年在7月，2020年会在6月份举行
  - 官网：https://www.gophercon.com/ 
  - github会议总结： https://github.com/gophercon 
- GopherChina，举办地在中国，每年4月份
  - 官网： https://gopherchina.org/ 
  - github会议总结： https://github.com/gopherchina/conference 
- dotGo，举办地在欧洲，每年3月份，官网为 https://www.dotgo.eu/ 
- 详细的会议列表可查看 https://github.com/golang/go/wiki/Conferences 

Go下载和相关的文档、标准库等本来都应该在官网 https://golang.org/ 查看，但国内由于某些原因无法访问，可以访问国内的镜像网站 http://docscn.studygolang.com/ 。该镜像网站由Go语言中文网维护，同时，[Go语言中文网]( https://studygolang.com/ )也是国内的活跃社区，每周还会发行一份[Go语言爱好者周刊](https://studygolang.com/go/weekly )

Go相关资料聚集最多的还是[go wiki](https://github.com/golang/go/wiki/Articles)

## 3. 下载安装

go语言环境搭建是一切操作的基础，windowns下快速安装可以使用 chocolatey ，执行如下命令即可

```powershell
> choco install golang
```

安装完甚至不需要自己配置环境变量，重启终端执行`go version`命令即可证明安装成功。后面的内容就开始介绍正常的安装方法，以win10和Ubuntu为例介绍windows和linux下的安装与环境配置。

### 3.1 下载

打开Golang中国官网 [golang.google.cn](https://golang.google.cn/) 

![golang中国区官网](https://s2.ax1x.com/2020/02/19/3AziRA.png)

点击首页的`Download Go`按钮，将跳转到[golang.google.cn/dl](https://golang.google.cn/dl/)页面，在该页面Golang为windows，macOS和Linux三种环境都提供了安装包，还提供了源码的打包文件。这里不介绍macOS的安装，因为没有使用经验。

目前的最新版本是本月刚发布的Go1.13

![golang下载页面](https://s2.ax1x.com/2020/02/19/3AzEsP.png)

windows默认下载文件为`go1.13.windows-amd64.msi`，双击启动即可安装，默认安装位置为`C:\Go`，并将自动设置环境变量。如果下载了以`.zip`为后缀的版本，则需要自己解压到合适的路径，并自己设置环境变量。

Linux默认下载文件为`go1.13.linux-amd64.tar.gz`，将其解压缩到`/usr/local`，同时需要手动将`/usr/local/go/bin`添加到PATH环境变量中。

windows和Linux都同时提供了32位和64位的安装版本，除此外，Linux还提供针对其它架构（如arm）的版本。

![golang安装版本](https://s2.ax1x.com/2020/02/19/3AzmdS.png)

### 3.2 安装和配置环境变量

#### windows

一般选择`msi`后缀的安装版本，双击即可安装，环境变量将会自动设置完成，打开powershell，使用`go env`命令查看环境变量

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
set GONOPROXY=
set GONOSUMDB=
set GOOS=windows
set GOPATH=C:\Users\lylw1\go
set GOPRIVATE=
set GOPROXY=https://proxy.golang.org,direct
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
set GOGCCFLAGS=-m64 -mthreads -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=C:\Users\lylw1\AppData\Local\Temp\go-build821698526=/tmp/go-build -gno-record-gcc-switches
```

我们需要关心的唯一一个环境变量是GOPATH，可以看到，默认的GOPATH是`c:\Users\lylw1\go`，其中`lylw1`是自己的用户名。

GOPATH所代表的路径是go的工作区，可以选择任意自己喜欢的文件夹作为go的工作区，如果不想使用默认的路径，需要自己进行修改，最简单的修改方式为使用`setx`命令，更多的环境变量修改方式可以查看[Setting GOPATH](https://github.com/golang/go/wiki/SettingGOPATH#windows)，需要注意的一点是GOPATH不能设置为Go的安装目录`C:\go`

```powershell
> setx GOPATH d:\go
```

go的版本号使用`go version`命令查看

```go
> go version
go version go1.13.4 windows/amd64
```

####  Linux

主要以Ubuntu为例，操作系统版本位Ubuntu18.04 LTS。首先在[golang.org/dl](https://golang.org/dl/)页面右击要下载的版本获取下载链接，使用wget下载文件。直接下载同样可以。

```bash
$ wget https://dl.google.com/go/go1.13.linux-amd64.tar.gz
```

将文件解压缩到指定目录/usr/local，`-C`参数用于指定目标文件夹，解压缩后删除压缩文件。

```bash
$ sudo tar -xzf go1.13.linux-amd64.tar.gz -C /usr/local
$ rm go1.13.linux-amd64.tar.gz
```

![解压后的golang文件](https://s2.ax1x.com/2020/02/19/3EFd7q.png)

设置环境变量

```bash
$ sudo nano /etc/profile
# 在打开的文件末尾添加下列语句
export PATH=$PATH:/usr/local/go/bin
```

![Linux环境变量设置](https://s2.ax1x.com/2020/02/19/3EF2u9.png)

更新的环境变量可以通过下面的命令使其直接生效

```bash
$ source /etc/profile
# 查看生效后的环境变量设置
$ go version
go version go1.13 linux/amd64
```

![linux go环境变量查看](https://s2.ax1x.com/2020/02/19/3EkN8O.png)

上述命令只能暂时使go可用，正式生效需重启系统。

注：将环境变量添加到/etc/profile可使go全局启用，也可添加到$HOME/.profile，对当前登录用户起作用。

*注：可以适用apt-get命令安装，但安装的go不会是最新版本，同样，也可以使用snap安装，可以安装最新版本。*

### 3.3 环境变量说明

常规的使用除了GOPATH外无需关心其它环境变量，但在进行交叉编译时，需要进行一定的设置，详细的环境变量说明可以查看[$GOPATH environment variable set correctly](https://golang.org/doc/install/source#gopath)，以下解释几个主要的环境变量。

- `GOOS`与`GOARCH`：目标操作系统和处理器架构，
- `GOHOSTOS`和`GOHOSTARCH`是宿主机操作系统和处理器架构。

- `GOBIN`：编译器和链接器的安装位置，默认是`$GOROOT/bin`，一般将其设置为空值，Go会使用默认值。

- `GOROOT`：电脑上安装的Go的根目录，Linux下一般为`$HOME/go`，windows下为`C:\go`

下面对`GOPATH`作进一步的说明。go install、go tool和go get命令默认路径都是`GOPATH`，作为编译后二进制文件的存放目的地、import包时的搜索路径以及下载后包的存放路径，如

- `$GOPATH/src/A/B`编译后二进制包放在`$GOPATH/bin/B`目录下。
- 导入包`X/Y/Z`会到`$GOPATH/pkg/$GOOS_$GOARCH/X/Y/Z.a`下寻找二进制包，到`$GOPATH/src/X/Y/Z`下寻找源码包。
- `go get github.com/go-kit/kit`命令，会将远程包克隆到`$GOPATH/src/github.com/go-kit/kit`

`GOPATH`可以是一个路径列表，也就是说在多个目录下编写go代码不会出问题，但是这种做法不被官方建议，即使拥有多个工作目录，包的导入与搜索也总是在第一个路径下。

### 3.4 代码目录结构

go中对代码目录结构的说明是

1. 所有的Go代码在一个工作区
2. 一个工作区包含多个版本控制仓库
3. 每个仓库包含一个或多个包
4. 每个包是一个文件夹，通常不含子文件夹，里面有一个或多个Go源码文件

以上提到的`工作区`、`版本控制仓库`、`包`都是一个个文件夹，工作区的根目录下初始拥有两个文件夹：

- `src`包含Go源码
- `bin`包含Go的可执行命令文件，如`gofmt.exe`

go工具链会将二进制包编译和安装到`bin`目录下，`src`的子目录则是一个个版本控制仓库，比如git。下面是一个工作区的目录示例：

```bash
bin/
    hello                          # command executable
    outyet                         # command executable
src/
    github.com/golang/example/
    	.git/                      # Git repository metadata
		hello/
	    	hello.go               # command source
		outyet/
	    	main.go                # command source
	    	main_test.go           # test source
		stringutil/
	    	reverse.go             # package source
	    	reverse_test.go        # test source
    golang.org/x/image/
        .git/                      # Git repository metadata
		bmp/
	    	reader.go              # package source
	    	writer.go              # package source
    ... (many more repositories and packages omitted) ...
```

上面的示例中一共包含两个仓库：`example`和`image`，前者是github的仓库，后者是golang本身的仓库。`example`仓库中包含两个命令(`hello`和`outyet`)和一个库(`stringutil`)，`image`仓库中包含`bmp`包以及一些[其它的东西](https://godoc.org/golang.org/x/image)

一个典型的Go工作区会包含多个仓库，每个仓库包含多个命令和包。

### 3.5 安装目录清单

安装完成的Go根目录（$GOROOT）文件夹结构除了README.md，AUTHORS，LICENSE等常规文件外，基本结构应如下所示：

- `/bin`：包含可执行文件，如：编译器，Go 工具
- `/doc`：包含示例程序，代码工具，本地文档等
- `/lib`：包含文档模版
- `/misc`：包含与支持 Go 编辑器有关的配置文件以及 cgo 的示例
- `/pkg/os_arch`：包含标准库的包的对象文件（`.a`）
- `/src`：包含源代码构建脚本和标准库的包的完整源代码（Go 是一门开源语言）
- `/src/cmd`：包含 Go 和 C 的编译器和命令行脚本

这是之前的介绍，网上搜到的多是这个说明，但当前还多了`api`和`test`两个文件夹。

## 4. 第一个程序

编辑并运行我们的第一个程序，按照编程界的惯例，输出`hello, world!`。

首先在工作区创建相应的包目录，注意下面的`$GOPATH`只是指代Go的工作区，windows下这样输入命令是无法执行的，需要替换为实际的路径

```powershell
$ mkdir $GOPATH/src/github.com/shuzang/hello
```

然后在该目录下创建`hello.go`文件，使其包含如下代码

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```

使用`go install`命令编译安装该包

```powershell
> go install github.com/shuzang/hello
```

上面的命令会进入GOPATH目录寻找`github.com/shuzang/hello`包然后编译安装，也可以进入包目录下直接执行`go install`

```powershell
> cd $GOPATH/src/github.com/shuzang/hello
> go install
```

该命令编译并生成可执行文件，放在工作区的`bin`目录下，windows下文件名为`hello.exe`，编译好的二进制文件已经放在了`bin`目录下，以完整路径运行该程序

```powershell
PS C:\Users\lylw1\go> ./bin/hello
Hello, world!
```

也可以将`$GOPATH/bin`添加到电脑的`PATH`环境变量中，这样可以直接执行

```powershell
PS C:\Users\lylw1\go> hello
Hello, world!
```

可能是因为我使用chocolatey安装的原因，这一环境变量也已经自动设好了，所以我可以直接执行命令。

完成编辑后，就可以使用版本控制工具初始化仓库并上传代码了，示例如下

```powershell
$ cd $GOPATH/src/github.com/shuzang/hello
$ git init
Initialized empty Git repository in C:/Users/lylw1/go/src/github.com/shuzang/hello/.git/
$ git add hello.go
$ git commit -m "initial commit"
[master (root-commit) 0b4507d] initial commit
 1 file changed, 7 insertion(+)
 create mode 100644 hello.go
```

## 5. 第一个库

现在来写一个用于`hello`程序的库

首先依然是选择合适的路径创建包文件夹

```powershell
> mkdir $GOPATH/src/github.com/shuzang/stringutil
```

在该文件夹下创建`reverse.go`文件，使其包含以下代码

```go
// Package stringutil contains utility functions for working with strings.
package stringutil

// Reverse returns its argument string reversed rune-wise left to right.
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

然后使用`go build`编译

```powershell
> go build github.com/shuzang/stringutil
```

如果位于`stringutil`包的根目录，也可以不用路径

```powershell
> cd $GOPATH/src/github.com/shuzang/stringutil
> go build
```

`go build`的作用仅仅是编译，不包括安装，因此不会生成任何输出文件，编译后的结果仅位于本地缓存。

确认编译完成后，修改`hello.go`文件，引入`stringutil`包

```go
package main

import (
	"fmt"

	"github.com/shuzang/stringutil"
)

func main() {
	fmt.Println(stringutil.Reverse("!oG ,olleH"))
}
```

然后使用`go install`编译安装，最后执行查看结果

```powershell
> go install github.com/shuzang/hello
> hello
Hello, Go!
```

以上步骤结束后，工作区的目录应该如下所示

```powershell
bin/
    hello                 # command executable
src/
    github.com/user/
        hello/
            hello.go      # command source
        stringutil/
            reverse.go    # package source
```

## 6. 编辑器、IDE和其它工具

### 6.1 概述

对开发工具的需求一般包括

1. 语法高亮，注释，代码补全提示，自动保存，显示代码所在的行数，括号匹配
2. 方便的项目文件管理，可以同时编辑多个源文件，读取保持最近的文件或项目
3. 能够跳转到某个函数或类型的定义部分。
4. 完美的查找和替换功能，替换之前最好还能预览结果。
5. 当有编译错误时，双击错误提示可以跳转到发生错误的位置，可以进行断点调试
6. 跨平台，免费，开源
7. 能够通过插件架构来轻易扩展和替换某个功能
8. 构建出的程序需要能够通过命令行或 IDE 内部的控制台运行
9. 内置 Go 的相关工具
10. 能够导出不同格式的代码文件，如：PDF，HTML 或格式化后的代码
11. 集成像 git 这样的版本控制工具
12. ...

以这些要求寻找，列举工具如下：

1. VScode
2. Goland
3. LiteIDE
4. vim-go
5. Atom

我当前用的是VScode，网上对Goland评价很高，其它的就随意了，也有不少直接用普通编辑器配合Go自带的工具使用的，实际上vscode和vim就属于这种

完整的IDE和插件列表可以查看

-  https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins 

### 6.2 VScode

VScode中的Go扩展提供了大量的特性，如自动补全、悬停信息显示、括号匹配等，下面进行具体介绍，在VScode软件内的插件商店直接搜索`Go`安装即可，详细的特性说明查看[官网](https://code.visualstudio.com/docs/languages/go)。

#### Go工具链

微软在开发 VS Code 过程中, 定义一种协议, [语言服务器协议](https://link.zhihu.com/?target=https%3A//microsoft.github.io/language-server-protocol/) ，用来为每种语言提供诸如自动完成，代码提示等功能。gopls就是Go语言的服务器, 安装命令为

```bash
$ go get -v golang.org/x/tools/gopls
```

不过VScode会在右下角弹出提示，只要直接点击`Install`即可，不需要自己输入命令。同时，因为没有设置go.toolsGopath，默认使用了gopath作为安装路径

```bash
go.toolsGopath setting is not set. Using GOPATH C:\Users\lylw1\go
Installing 1 tool at C:\Users\lylw1\go\bin in module mode.
  gopls

Installing golang.org/x/tools/gopls SUCCEEDED

Reload VS Code window to use the Go language server
All tools successfully installed. You're ready to Go :).

```

Go代码的调试还需要delve工具

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
golint
```

所有这些工具只需要在右下角弹出提示后点击`Install`即可

不过上述提到的很多插件不翻墙都无法下载，可以自己手动到github下载然后解压到`GOPATH/src`，建立起对应的目录结构，然后运行`go install`将其安装

#### 用户和工作区设置

使用VScode需要关心的一个重要部分是用户和工作区设置，几乎所有的事情都和它们有关。

![](https://code.visualstudio.com/assets/docs/getstarted/settings/hero.png)

这是两种不同的设置范围

- 用户设置：是一个全局的设置，适用于打开的任何VScode窗口
- 工作区设置：如题，项目工作区的设置，只适用于对应的工作区

工作区的设置会覆盖掉用户设置，它针对具体的项目，配置文件位于项目根目录`.vscode`文件夹，可与其它开发者共享。`.vscode`文件夹还用于存放调试配置和任务配置。

点击左下角的齿轮，选择`设置`，默认的设置界面如上，是一个可视化的界面，不过也可以使用`settings.json`配置文件

- 用户设置文件在windows中位于`%APPDATA%\Code\User\settings.json`
- 工作区设置文件位于根目录的`.vscode`文件夹中

最后，VScode大量的操作都可以通过命令完成，使用快捷键`Ctrl+Shift+P`可以打开命令输入框。

#### 特性说明

在用户或工作区设置中，将 `go.autocompleteUnimportedPackages` 设为`true`  ，可以在代码中点击包名跳转查看包的具体内容。

鼠标悬停在变量、函数和结构体的名称上方可以查看它们的签名等信息，这一功能由前面安装的`godef`工具实现，同样可完成这一功能的工具还有`godoc`和`gogetdoc`，通过在用户或工作区设置中调整`go.docsTool`来切换工具。

代码导航功能无需设置默认实现。

对源码的保存操作会自动触发格式化、编译和代码质量检查。格式化工具为`gofmt`，可选的替代工具有`goreturns`和`goimports`，在用户或工作区设置中调整`go.formatTool`来设置。编译的过程使用`go build`命令。代码质量检查的工具为`golint`，也可以使用`gometalinter`，用来检查代码的规范性，检查得到的`errors`和`warning`会在编辑器里以红色/绿色波浪线标出来，下面的输出窗口也会显示详细信息。

所有可选的替代工具都需要自己安装，使用快捷键打开命令面板，输入 `Go: Install/Update Tools `，选择要安装的工具，点击确定即可。

#### 重命名符号

普通情况重命名一般使用替换功能，实际上有更为简单的方式：选中要重命名的符号，按快捷键`F2`即可

#### 调试

调试使用的是前面安装的`delve`工具。在VScode中，按`F5`启动调试，一般情况下使用默认的调试配置即可，不过还是应当对调试配置选项有一定的了解。

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

#### 运行

快捷键`Ctrl+F5`用来运行程序，和调试使用的是同一个配置文件。
