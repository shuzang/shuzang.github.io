---
title: Golang深入学习5-使用dlv调试程序
date: 2020-07-26T09:43:00+08:00
lastmod: 2020-07-26
tags: [Go语法]
categories: [Golang学习之路]
slug: Golang deep learning 5- debug with dlv 
---

在C/C++中，使用 GDB 调试程序，但在Go中，看过网上的一些文章后，发现 dlv 是个更好的选择，本文进行介绍。

<!--more-->

实际上，[delve](https://github.com/go-delve/delve) 才是全称，dlv 只是启动命令，如果使用 VScode，默认使用的调试器就是基于 delve 的。

## 1. 基本命令

使用如下命令安装

```go
go get github.com/go-delve/delve/cmd/dlv
```

安装后执行 `dlv` 命令查看基本信息

```bash
$ dlv
Delve is a source level debugger for Go programs.

Delve enables you to interact with your program by controlling the execution of the process,
evaluating variables, and providing information of thread / goroutine state, CPU register state and more.

The goal of this tool is to provide a simple yet powerful interface for debugging Go programs.

Pass flags to the program you are debugging using `--`, for example:

`dlv exec ./hello -- server --config conf/config.toml`

Usage:
  dlv [command]

Available Commands:
  attach      Attach to running process and begin debugging.
  connect     Connect to a headless debug server.
  core        Examine a core dump.
  debug       Compile and begin debugging main package in current directory, or the package specified.
  exec        Execute a precompiled binary, and begin a debug session.
  help        Help about any command
  run         Deprecated command. Use 'debug' instead.
  test        Compile test binary and begin debugging program.
  trace       Compile and begin tracing program.
  version     Prints version.

Flags:
      --accept-multiclient   Allows a headless server to accept multiple client connections.
      --api-version int      Selects API version when headless. (default 1)
      --backend string       Backend selection (see 'dlv help backend'). (default "default")
      --build-flags string   Build flags, to be passed to the compiler.
      --check-go-version     Checks that the version of Go in use is compatible with Delve. (default true)
      --headless             Run debug server only, in headless mode.
      --init string          Init file, executed by the terminal client.
  -l, --listen string        Debugging server listen address. (default "127.0.0.1:0")
      --log                  Enable debugging server logging.
      --log-dest string      Writes logs to the specified file or file descriptor (see 'dlv help log').
      --log-output string    Comma separated list of components that should produce debug output (see 'dlv help log')
      --only-same-user       Only connections from the same user that started this instance of Delve are allowed to connect. (default true)
      --wd string            Working directory for running the program. (default ".")

Additional help topics:
  dlv backend Help about the --backend flag.
  dlv log     Help about logging flags.

Use "dlv [command] --help" for more information about a command.
```

进入调试模式有以下几种办法

1. dlv attach pid：对正在运行的进程直接进行调试（pid 为进程id）；
2. dlv debug：编译源文件并开始调试，这里应和 main 函数位于同一目录，或者指定完整的 main 函数路径
3. dlv exec filename：从二进制文件启动调试

我们以下面的程序为例进行说明，使用 dlv debug 进入调试

```go
package main

import "fmt"

func main() {
	a := 10
	fmt.Println(a)
}
```

进入调试，使用 help 可以查看所有可用命令

```bash
$ dlv debug main.go
Type 'help' for list of commands.
(dlv) help
The following commands are available:
    args ------------------------ Print function arguments.
    break (alias: b) ------------ Sets a breakpoint.
    breakpoints (alias: bp) ----- Print out info for active breakpoints.
    call ------------------------ Resumes process, injecting a function call (EXPERIMENTAL!!!)
    clear ----------------------- Deletes breakpoint.
    clearall -------------------- Deletes multiple breakpoints.
    condition (alias: cond) ----- Set breakpoint condition.
    config ---------------------- Changes configuration parameters.
    continue (alias: c) --------- Run until breakpoint or program termination.
    deferred -------------------- Executes command in the context of a deferred call.
    disassemble (alias: disass) - Disassembler.
    down ------------------------ Move the current frame down.
    edit (alias: ed) ------------ Open where you are in $DELVE_EDITOR or $EDITOR
    exit (alias: quit | q) ------ Exit the debugger.
    frame ----------------------- Set the current frame, or execute command on a different frame.
    funcs ----------------------- Print list of functions.
    goroutine (alias: gr) ------- Shows or changes current goroutine
    goroutines (alias: grs) ----- List program goroutines.
    help (alias: h) ------------- Prints the help message.
    libraries ------------------- List loaded dynamic libraries
    list (alias: ls | l) -------- Show source code.
    locals ---------------------- Print local variables.
    next (alias: n) ------------- Step over to next source line.
    on -------------------------- Executes a command when a breakpoint is hit.
    print (alias: p) ------------ Evaluate an expression.
    regs ------------------------ Print contents of CPU registers.
    restart (alias: r) ---------- Restart process.
    set ------------------------- Changes the value of a variable.
    source ---------------------- Executes a file containing a list of delve commands
    sources --------------------- Print list of source files.
    stack (alias: bt) ----------- Print stack trace.
    step (alias: s) ------------- Single step through program.
    step-instruction (alias: si)  Single step a single cpu instruction.
    stepout (alias: so) --------- Step out of the current function.
    thread (alias: tr) ---------- Switch to the specified thread.
    threads --------------------- Print out info for every traced thread.
    trace (alias: t) ------------ Set tracepoint.
    types ----------------------- Print list of types
    up -------------------------- Move the current frame up.
    vars ------------------------ Print package variables.
    whatis ---------------------- Prints type of an expression.
Type help followed by a command for full documentation.
```

常用的命令总结如下：

| 命令     | 含义                                           |
| -------- | ---------------------------------------------- |
| b        | 设置断点                                       |
| bp       | 打印正活动的断点信息                           |
| clear    | 删除断点                                       |
| clearall | 删除所有断点                                   |
| c        | 运行直到断点处或程序终止                       |
| n        | 下一步，不会进入函数                           |
| s        | 下一步，会进入函数                             |
| so       | 跳出当前函数                                   |
| args     | 查看函数参数                                   |
| locals   | 查看所有局部变量                               |
| list     | 打印当前源代码                                 |
| on       | 运行到某断点然后执行相应的命令，比如 on 2 list |
| bt       | 打印当前调用栈                                 |
| exit     | 退出                                           |

## 2. 命令使用

下面通过实践说明这些简单命令的使用

### 2.1 断点设置

```bash
(dlv) b main.go:6
Breakpoint 1 set at 0x4bd2f8 for main.main() F:/Gotest/main.go:6
```

### 2.2 打印断点信息

```bash
(dlv) bp
Breakpoint runtime-fatal-throw at 0x4377e0 for runtime.fatalthrow() c:/go/src/runtime/panic.go:1162 (0)
Breakpoint unrecovered-panic at 0x437860 for runtime.fatalpanic() c:/go/src/runtime/panic.go:1189 (0)  
        print runtime.curg._panic.arg
Breakpoint 1 at 0x4bd2f8 for main.main() F:/Gotest/main.go:6 (0)
```

### 2.3 运行直到断点处

```bash
(dlv) c
> main.main() F:/Gotest/main.go:6 (hits goroutine(1):1 total:1) (PC: 0x4bd2f8)
     1: package main
     2: 
     3: import "fmt"
     4: 
     5: func main() {
=>   6:         a := 10       
     7:         fmt.Println(a)
     8: }
```

### 2.4 下一步

```bash
(dlv) n
> main.main() F:/Gotest/main.go:7 (PC: 0x4bd301)
     2: 
     3: import "fmt"
     4: 
     5: func main() {
     6:         a := 10
=>   7:         fmt.Println(a)
     8: }
```

### 2.5 查看局部变量

```bash
(dlv) locals
a = 10
```

### 2.6 查看当前调用栈

```bash
(dlv) bt
0  0x00000000004bd301 in main.main
   at F:/Gotest/main.go:7
1  0x0000000000439cfa in runtime.main
   at c:/go/src/runtime/proc.go:203
2  0x00000000004643d1 in runtime.goexit
   at c:/go/src/runtime/asm_amd64.s:1373
```

### 2.7 打印源代码

```bash
(dlv) list
> main.main() F:/Gotest/main.go:7 (PC: 0x4bd301)
     2: 
     3: import "fmt"
     4: 
     5: func main() {
     6:         a := 10
=>   7:         fmt.Println(a)
     8: }
```

### 2.8 运行到断点处执行某个命令

1 是断点ID，p a 代表指定到断点处打印变量 a 的值

```bash
(dlv) on 1 p a
(dlv) c
10
Process 6540 has exited with status 0
```

## 3. Goroutine 调试

Go 的优势在协程，dlv 相比 GDB 的优点也在于对协程调试的支持，我们以下面的程序为例，给出示例

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	a := 10
	go printA(a)

	time.Sleep(1e9)
}

func printA(a int) {
	fmt.Println(a)
}
```

设置断点并启动运行

```bash
> b main.go:12
Breakpoint 1 set at 0x4bd5be for main.main() F:/Gotest/main.go:12
(dlv) c
> main.main() F:/Gotest/main.go:12 (hits goroutine(1):1 total:1) (PC: 0x4bd5be)
     7: 
     8: func main() {
     9:         a := 10
    10:         go printA(a)
    11: 
=>  12:         time.Sleep(1e9)
    13: }
    14: 
    15: func printA(a int) {
    16:         fmt.Println(a)
    17: }
```

查看当前启动的协程，其中 Goroutine 1 是主协程，Goroutine 6 是自行启动的子协程

```bash
(dlv) goroutines
* Goroutine 1 - User: F:/Gotest/main.go:12 main.main (0x4bd5be) (thread 11412)
  Goroutine 2 - User: c:/go/src/runtime/proc.go:305 runtime.gopark (0x43a092)
  Goroutine 3 - User: c:/go/src/runtime/proc.go:305 runtime.gopark (0x43a092)
  Goroutine 4 - User: c:/go/src/runtime/proc.go:305 runtime.gopark (0x43a092)
  Goroutine 5 - User: c:/go/src/runtime/proc.go:305 runtime.gopark (0x43a092)
  Goroutine 6 - User: F:/Gotest/main.go:15 main.printA (0x4bd5e0)
[6 goroutines]
```

打印主协程当前的变量

```bash
(dlv) locals
a = 10
```

如果要查看子协程的情况，需要先切换到子协程

```bash
(dlv) goroutine 6
Switched from 1 to 6 (thread 11412)
(dlv) bt
0  0x00000000004bd5e0 in main.printA
   at F:/Gotest/main.go:15
1  0x0000000000464621 in runtime.goexit
   at c:/go/src/runtime/asm_amd64.s:1373
(dlv) args
a = 10
```

## 4. 后记

写到这里好像没发现直接使用 dlv 有什么优势，VScode 在 delve 基础上完成的调试功能更方便，比自己使用命令逐步执行好多了。



