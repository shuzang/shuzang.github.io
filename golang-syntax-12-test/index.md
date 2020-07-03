# Golang语法基础12-测试


实际开发中对代码进行测试是不可缺少的工作，在go中可以通过`testing`包来进行代码的测试。`testing`包和`go test`命令相互配合，能够完成代码的自动化测试。

在实际开发中，对代码进行测试是不可缺少的工作，在go中可以通过`testing`包对代码进行单元测试和性能测试。

#### 基本说明

`testing`包是与`go test`命令配合使用的，编写测试代码需要使用`testing`包，而执行测试需要使用`go test`命令。执行测试时，会自动读取源码目录下名为`*_test.go`的文件，生成并运行测试用的可执行文件，并最终在终端输出测试信息。输出的信息类似于

```go
ok   archive/tar   0.011s
FAIL archive/zip   0.022s
ok   compress/gzip 0.033s
...
```

`testing`包的说明位于：[Package testing](https://golang.org/pkg/testing/)

`go test`的说明可通过执行` go help test`查看

#### 单元测试

假设当前我们在源代码目录拥有名为`hello.go`的源码文件，其内容如下：

```go
package hello

import "fmt"

//Hello print "hello!
func Hello() {
	fmt.Println("hello!")
}
```

首先编写测试用例，即在`hello.go`文件同目录下创建`hello_test.go`文件，编辑其内容如下：

```go
package hello

import "testing"

func TestHello(t *testing.T) {
	Hello()
}
```

注意，单元测试的函数头一般符合如下形式，其中`Xxx`是测试函数名。无论原函数名首字母是否大写，测试函数中函数名的首字母必须大写，原话为`where Xxx does not start with a lowercase letter.`

```go
func TestXxx(t *testing.T)
```

在这两个文件目录下执行`go test`命令

```powershell
PS F:\Go> ls
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2019/9/19     19:31            154 hello.go
-a----        2019/9/19     19:45            222 hello_test.go

PS F:\Go> go test
hello!
PASS
ok      _/F_/Go 0.248s
```

根据输出结果可知测试通过

#### 性能测试

在`hello.go`中新增`Add`函数

```go
package hello

import "fmt"

//Hello print "hello!
func Hello() {
	fmt.Println("hello!")
}

//Add add para
func Add(i int) int {
	return i
}
```

在`hello_test.go`文件中添加`Add`的性能测试函数

```go
package hello

import (
	"fmt"
	"testing"
)

func TestHello(t *testing.T) {
	Hello()
}

func BenchmarkAdd(b *testing.B) {
	var sum int
	for i := 0; i < b.N; i++ {
		sum += Add(1)
	}
	fmt.Println(sum)
}
```

性能测试的函数头也符合与单元测试相似的形式

```go
func BenchmarkXxx(b *testing.B)
```

执行带`bench`参数的`go test`命令

```powershell
PS F:\Go> go test -bench .
hello!
1
goos: windows
goarch: amd64
BenchmarkHello-4        100
10000
1000000
100000000
2000000000
2000000000               0.32 ns/op
PASS
ok      _/F_/Go 1.019s
```

以上命令只执行了性能测试函数，BenchmarkHello执行了2000000000次，每次的平均执行时间为0.32ns，总的执行时间为1.019s。测试通过。

#### go test命令的参数

以上是基本的单元测试和性能测试命令，`go test`命令还拥有大量的参数，可以对测试进行定制。

```bash
usage: go test [build/test flags] [packages] [build/test flags & test binary flags]
```

如上，`go test`命令接受用于本身的两个参数和用于生成的测试结果文件的两个参数。

用于test命令本身的参数列表如下

```bash
-bench regexp
    只运行与正则表达式匹配的性能测试，默认不执行任何性能测试。
    对于测试，正则表达式由斜杠（/）字符拆分为正则表达式序列
    要运行所有的性能测试, 使用 '-bench .' 或 '-bench=.'.

-benchtime t
    Run enough iterations of each benchmark to take t, specified
    as a time.Duration (for example, -benchtime 1h30s).
    The default is 1 second (1s).
    The special syntax Nx means to run the benchmark N times
    (for example, -benchtime 100x).

-count n
    运行每个测试和性能测试n次（默认1次）
    如果设置了 -cpu 参数, 为每个 GOMAXPROCS 值运行 n 次

-cover
    启用测试覆盖率分析。这里要注意，测试覆盖率分析是通过在编译前注释源码来实现的，因此，出现错误时报告的行     号可能不对应。

-covermode set,count,atomic
    为正在测试的包设置测试覆盖率分析的模式。 
    默认为 "set" 启用了 -race 则为 "atomic".
    各模式说明：
	set: bool: 这个声明运行了吗
	count: int: 这个声明运行了多少次
	atomic: int: count, but correct in multithreaded tests;
		significantly more expensive.
    需要设置 -cover

-coverpkg pattern1,pattern2,pattern3
    在每个测试中对与模式匹配的包应用测试覆盖率分析。默认情况下，每个测试只分析正在测试的包。
    运行 'go help packages' 可获得包模式的说明
    需要设置 -cover

-cpu 1,2,4
    指定应为其执行测试或性能测试的GOMAXPROCS值的列表。默认值是GOMAXPROCS的当前值。

-failfast
    第一次测试失败后不开启新的测试

-list regexp
    列出与正则表达式匹配的测试、性能测试或示例测试。
    但不会运行任何测试、性能测试或示例测试。
    只列出顶级测试。不会显示子测试或子性能测试。

-parallel n
    允许调用t.parallel的测试函数并行执行。
  	n是要同时运行的最大测试数；默认设置为GOMAXPROCS的值。
  	该参数只适用于单个测试二进制文件。
  	并行测试不同的package参加 'go help build' 命令的说明

-run regexp
    仅运行与正则表达式匹配的那些测试和示例测试。
    对于测试，正则表达式由斜杠（/）字符拆分为正则表达式序列

-short
    测试长时间运行时声明需要缩短时间，默认关闭。

-timeout d
    如果测试文件的运行时间长于 d, panic。
    如果d为 0, 未超时。
    默认为 10 分钟 (10m).

-v
    详细输出：记录所有运行的测试。即使测试成功，也打印日志。

-vet list
    在“go test”期间配置“go vet”的调用，从而使用以逗号分隔的vet检查列表。
    如果list为空，则“go test”运行“go vet”，其中列出了一系列被认为值得解决的检查。
    如果list是“off”，则“go test”根本不会运行“go vet”。
```

用于生成的测试结果文件的参数列表如下

```bash
-benchmem
    打印性能测试的内存分配统计信息

-blockprofile block.out
    Write a goroutine blocking profile to the specified file
    when all tests are complete.
    Writes test binary as -c would.

-blockprofilerate n
    Control the detail provided in goroutine blocking profiles by
    calling runtime.SetBlockProfileRate with n.
    See 'go doc runtime.SetBlockProfileRate'.
    The profiler aims to sample, on average, one blocking event every
    n nanoseconds the program spends blocked. By default,
    if -test.blockprofile is set without this flag, all blocking events
    are recorded, equivalent to -test.blockprofilerate=1.

-coverprofile cover.out
    所有测试完成后，将测试覆盖率分析结果写入指定文件，需要设置 -cover

-cpuprofile cpu.out
    退出前将CPU分析结果写入指定文件
    Writes test binary as -c would.

-memprofile mem.out
    所有测试完成后，将内存分配分析结果写入指定文件
    Writes test binary as -c would.

-memprofilerate n
    通过设置 runtime.MemProfileRate 启用更精确（也更昂贵）的内存分配分析. 
    参见 'go doc runtime.MemProfileRate'.
    要分析所有的内存分配, 使用 -test.memprofilerate=1.

-mutexprofile mutex.out
    当所有测试完成时，将互斥竞争分析结果写入指定文件。
    Writes test binary as -c would.

-mutexprofilefraction n
    Sample 1 in n stack traces of goroutines holding a contended mutex.

-outputdir directory
    将分析结果文件放到指定目录。默认目录为 "go test" 运行的目录

-trace trace.out
    退出前将执行追踪结果写入指定文件
```

每个参数也可以通过`test.`前缀来使用，当调用测试生成的二进制结果文件时（`go test -c`命令会编译生成结果文件但不会执行），前缀是必须的。

如下列两个命令等价，参见[Testing flags](https://golang.org/cmd/go/#hdr-Testing_flags)

```bash
$ go test -v -myflag testdata -cpuprofile=prof.out -x
$ pkg.test -test.v -myflag testdata -test.cpuprofile=prof.out
```


