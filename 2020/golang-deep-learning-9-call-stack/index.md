# Golang深入学习9-调用栈


本篇介绍如何分析 Go 的调用栈。

<!--more-->

测试使用的版本是Go 1.14.3，下面介绍如何在Go调试的时候查看堆栈跟踪信息及识别传递的参数。

示例程序如下

```go
package main

import (
	"runtime/debug"
)

func main() {
	slice := make([]string, 2, 4)
	Example(slice, "hello", 10)
}
func Example(slice []string, str string, i int) {
	debug.PrintStack()
}
```

在程序中我们使用了 runtime/debug 下的 PrintStack() 函数将调用栈的返回信息打印到标准错误输出，如下所示

```go
goroutine 1 [running]:
runtime/debug.Stack(0x0, 0x0, 0x0)
	c:/go/src/runtime/debug/stack.go:24 +0xa8
runtime/debug.PrintStack()
	c:/go/src/runtime/debug/stack.go:16 +0x29
main.Example(0xc000077f38, 0x2, 0x4, 0x4a8534, 0x5, 0xa)
	f:/Gotest/main.go:13 +0x2e
main.main()
	f:/Gotest/main.go:9 +0xb1
Process exiting with code: 0
```

注意，在其它编程语言如C中，运行一个程序会启动一个线程来执行，在Go中，启动的是一个 Goroutine。上面第一行就说明了启动了一个 Goroutine，Goroutine ID 为1，其后各行是不同层次的调用，最深的调用最先打印，最浅的调用最后打印。各行说明如下

第8、9行：main package 的 main 函数，代码文件路径为 `f:/Gotest/main.go`，调用出现在 main.go 文件的第9行

第6、7行：main 函数调用 Example 函数

第4、5行：Example 函数调用 debug.PrintStack 函数

第2，3行：debug.PrintStack 函数调用 debug.Stack 函数

Example 函数传参信息如下

```go
// 调用
slice := make([]string, 2, 4)
Example(slice, "hello", 10)
// 栈追踪
main.Example(0xc000077f38, 0x2, 0x4, 0x4a8534, 0x5, 0xa)
```

堆栈跟踪信息中，前三个参数分别代表切片的指针、长度、容量，第4和第5个参数代表字符串的指针和大小，最后一个参数指向整型数值

```go
// 切片
Pointer: 0xc000077f38
Length: 0x2
Capacity: 0x4
// 字符串
Pointer: 0x4a8534
Length: 0x5
// 整数
base 16: 0xa
```

如果是调用方法，跟踪信息会显示接收者

```go
// 程序
package main

import (
   "fmt"
   "runtime/debug"
)

type trace struct{}

func main() {
   slice := make([]string, 2, 4)
   var t trace
   t.Example(slice, "hello", 10)
}
func (t *trace) Example(slice []string, str string, i int) {
   fmt.Printf("Receiver Address: %p\n", t)
   debug.PrintStack()
}
// 堆栈信息
Receiver Address: 0x5781c8
goroutine 1 [running]:
runtime/debug.Stack(0x15, 0xc000071ef0, 0x1)
	C:/Go/src/runtime/debug/stack.go:24 +0xae
runtime/debug.PrintStack()
	C:/Go/src/runtime/debug/stack.go:16 +0x29
main.(*trace).Example(0x5781c8, 0xc000071f48, 0x2, 0x4, 0x4c04bb, 0x5, 0xa)
	D:/gopath/src/example/example/main.go:17 +0x7c
main.main()
	D:/gopath/src/example/example/main.go:13 +0x9a
```

传递的参数全部为值类型时，可能会防止一个32位的字中

```go
// 程序
import (
   "runtime/debug"
)

func main() {
   Example(true, false, true, 25)
}
func Example(b1, b2, b3 bool, i uint8) {

   debug.PrintStack()
}
// 堆栈信息
goroutine 1 [running]:
runtime/debug.Stack(0x4, 0xc00007a010, 0xc000077f88)
	C:/Go/src/runtime/debug/stack.go:24 +0xae
runtime/debug.PrintStack()
	C:/Go/src/runtime/debug/stack.go:16 +0x29
main.Example(0xc019010001)
	D:/gopath/src/example/example/main.go:12 +0x27
main.main()
	D:/gopath/src/example/example/main.go:8 +0x30
```

可以看到 Example 的参数只有一个，实际上底层四个参数放在一个字中

```go
// Parameter values
true, false, true, 25

// Word value
Bits    Binary      Hex   Value
00-07   0000 0001   01    true
08-15   0000 0000   00    false
16-23   0000 0001   01    true
24-31   0001 1001   19    25
```


