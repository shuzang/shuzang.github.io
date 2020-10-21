---
title: Go并发学习2-channel
date: 2020-07-16T19:20:00+08:00 
lastmod: 2020-07-16
tags: [Go并发]
categories: [Golang学习之路]
draft: true
slug: Goroutine learning 1-channel: 
---

本篇介绍 Go 中的通道

<!--more-->

## 1. channel 基础

直观的翻译就是通道，是 Goroutine 间通信的工具，类似于缓冲区。在 Go 中一般有两个方面的作用

1. 传递方面：消息传递、任务发送和事件广播
2. 控制方面：资源争抢、并发控制和流程开关

### 1.1 声明与初始化

声明通道的基本形式如下，未初始化的通道值为nil

```go
var identifier chan datatype
```

通道只能传输一种类型的数据，比如`chan int`或`chan string`，可以是任意类型，包括空接口`interface{}`和通道自己

和map相同，通道也是引用类型，因此使用`make`进行初始化，可以指定第二个参数用来指定缓冲区的大小，即通道可容纳的数据个数，这一个值默认是0，意思是无缓冲，无缓冲的通道将通信、值的交换、同步三者结合，保证两个协程的计算处于已知状态。

```go
var ci chan string
ci = make(chan string)			// unbuffered channel of integers

cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

### 1.2 通信操作符<-

这一操作符直观的表示数据的传输：信息按照箭头方向流动。

流向通道(发送)用`ch <- int1`表示，意为利用通道ch发送变量int1

从通道流出(接收)用`int2 = <- ch`表示，意为变量int2从通道ch接收数据，如果int2没有声明过，可以使用`int2 :=  <- ch`

`<- ch`则用于表示丢弃当前值，获取通道的下一个值，可以用来验证，如

```go
if <- ch != 1000 {
    ...
}
```

为了可读性通道的命名通常以 `ch` 开头或者包含 `chan`。通道的发送和接收都是原子操作，总是互不干扰的完成。下面的示例展示了通信操作符的使用。 

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)

	go sendData(ch)
	go getData(ch)

	time.Sleep(1e9)
}

func sendData(ch chan string) {
	ch <- "Washington"
	ch <- "Tripoli"
	ch <- "London"
	ch <- "Beijing"
	ch <- "Tokyo"
}

func getData(ch chan string) {
	var input string
	// time.Sleep(2e9)
	for {
		input = <-ch
		fmt.Printf("%s ", input)
	}
}
//Output:
Washington Tripoli London Beijing tokyo
```

如果2个协程需要通信，必须给它们同一个通道作为参数。上例中 `main()` 函数中启动了两个协程：`sendData()` 通过通道 ch 发送了 5 个字符串，`getData()` 按顺序接收它们并打印出来。 一些同步的细节如下：

- main() 等待了 1 秒让两个协程完成，如果不这样(注释掉`time.Sleep(1e9)`)，sendData() 就没有机会输出。
- getData() 使用了无限循环：它随着 sendData() 的发送完成和 ch 变空也结束了。
- 如果我们移除一个或所有 `go` 关键字，程序无法运行，Go 运行时会抛出 panic。这是因为运行时(runtime)会检查所有的协程是否在等待什么东西(从通道读取或写入某个通道)，这意味着陷入死锁，程序无法继续执行。

通道的发送和接收顺序是无法预知的，如果使用打印状态来输出，由于两者间的时间延迟，打印的顺序和真实发生的顺序是不同的。

## 2. 一些理解

以下面的简单例子为例

```go
package main

import "fmt"

func main() {
	ch := make(chan int)
	fmt.Println(ch)

	go func() {
		ch <- 1
	}()

	fmt.Println(<-ch)
	close(ch)
	fmt.Println(ch)
}
// Output:
0xc000016180
1
0xc000016180
```

最后输出通道接收到的值为 1 是正常的，通道使用完毕后进行关闭。但是，我们将初始化后的通道值和通道关闭后的值进行输出，发现得到了两个相同的地址，这是因为 make 得到的是 channel 类型变量的地址。那么如果在关闭后继续使用通道传入和接收会怎么样呢，如下，会抛出 panic

```go
package main

import "fmt"

func main() {
	ch := make(chan int)
	fmt.Println(ch)

	go func() {
		ch <- 1
	}()

	fmt.Println(<-ch)
	close(ch)
	fmt.Println(ch)
	ch <- 2
	fmt.Println(<-ch)
}
// Output:
0xc0001020c0
1
0xc0001020c0
panic: send on closed channel

goroutine 1 [running]:
main.main()
	f:/Gotest/main.go:16 +0x22f
Process exiting with code: 0
```

抛出的异常告诉我们，已关闭的 channel 不允许再传入数据，另外，我们还应该注意到这里得到的 channel 的地址不一样了，这是很正常的，因为每次运行为 channel 分配的内存都不太可能是同一块

最后，如果我们关闭后不传入数据，仅查看呢，其值为0，事实上，这里会是 chan 所修饰的类型的零值，比如，如果定义的是 ch := make(chan bool)，关闭 channel 后输出内容会是 false

```go
package main

import "fmt"

func main() {
	ch := make(chan int)

	go func() {
		ch <- 1
	}()

	fmt.Println(<-ch)
	close(ch)

	fmt.Println(<-ch)
}
// Output:
1
0
```

## 3. 资源争抢

channel 有时候也用于资源争抢，举例如下，100个人抢10个鸡蛋，每次的结果是随机的

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	eggs := make(chan int, 10)

	for i := 0; i < 10; i++ {
		eggs <- i
	}

	var wg sync.WaitGroup

	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func(nums int) {
			select {
			case egg := <-eggs:
				fmt.Printf("People %d get egg %d\n", nums, egg)
			default:
			}
			wg.Done()
		}(i)
	}
	wg.Wait()
}
// Output:
People 0 get egg 0
People 6 get egg 5
People 5 get egg 7
People 8 get egg 8
People 7 get egg 9
People 10 get egg 6
People 1 get egg 1
People 3 get egg 2
People 2 get egg 3
People 4 get egg 4
```

资源争抢的运用领域非常广泛，比如电商的秒杀活动，出行的司机抢单，股票抢购，系统的计算资源争抢等

## 4. channel阻塞

channel 阻塞就是资源的等待，比如使用 <-ch 时还没有数值被送入通道，就需要等待。

阻塞产生的条件有两种

1. 输入时缓冲区满，此时需要等待 channel 中的值被取出；
2. 取出时缓冲区空，此时需要等待有值放入 channel；

阻塞可能造成下游接收数据的一方收不到数据，从而导致数据丢失，如果阻塞位置位于主流程，那么可能导致整个程序休眠，这时候影响就比较大。