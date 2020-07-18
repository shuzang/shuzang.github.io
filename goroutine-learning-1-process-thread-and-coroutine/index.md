# Go并发学习1-进程、线程与协程


本篇介绍进程、线程、协程和Goroutine

<!--more-->

## 1. 进程和线程

进程(processes)是一个程序动态执行的过程，也是操作系统调度的一个基本单位执行的基本单位，运行在一个独立的内存地址空间中。从windows资源管理器可以看到系统正在运行的进程，如下，每个应用程序是一个进程。

![进程与线程](/images/Go并发学习1-进程线程与协程/3EkjsJ.png)

一个进程由多个线程(threads)组成，线程的存在是为了能够同时执行多个任务，最大化利用时间，防止产生等待，线程间是共享内存地址空间的。线程也是操作系统调度的单位，而且是调度的最小单位。

## 2. 协程

协程英文为 coroutine，比线程更轻量，其区别可以按如下理解

1. 协程的内存消耗更小：一个线程可以包含多个协程，线程大约8MB的内存申请量，协程大约2KB的内存申请量；
2. 上下文切换更快：协程少一道手续，线程申请内存，需要走过内核，协程申请内存，不需要走过内核

Go 中的并发单位也叫做协程，但命名为 Goroutine，具有如下特点

1. 去掉了冗余的协程生命周期管理：协程创建、协程完成和协程重用；
2. 降低了额外的延迟和开销：主要是由于协程间频繁交互导致的
3. 降低加锁、解锁的频率：降低一部分额外的开销

### 2.1 多协程设计

企业级应用中，用到协程的场景一般有两种，第一种是单独监控任务，比如服务器端口，以便前端的访问，第二种就是多协程并发处理程序。

多协程的定义：官方定义是一段时间内协程的并行，而从任务的角度，是某个任务使用多个协程同时进行处理

多协程的必要条件：协程任务间有关联性，一般是将一个任务分成几个小部分，分别交给不同的协程同时处理，然后将结果汇总，这个步骤可以总结为

1. 任务切分/分配
2. 启动多个协程
3. 合并多个协程的结果

要注意的是多协程不是万能的，有它的使用场景，一般情况下，用于运算量比较多的流程，协程间的依赖性也应该比较弱。另外，任务的分配与结果的合并也是需要耗费时间的，如果这部分时间占比过大，那么使用协程是得不偿失的。最后，协程的使用可能会增加内存的消耗。

多协程设计的主要目标就是提高运行速度，其评价标准主要就是耗时，这里要注意，时间的减少必然带来内存消耗的增加，所以要注意内存消耗增加不能太多。协程的设计有几种思路，以做饭为例

1. 多个人单独完成自己的和面、切块、擀面皮、剁馅儿、包饺子、蒸饺子这一串步骤，最后把所有人蒸的饺子合并到一起；
2. 在同一时间多个人同时完成一个步骤，比如一起和面，该步骤结束后在一起切块，这样一直到最后一个步骤完成；这种方法虽然比第一种耗时更长，但可以有效的维持一致性；
3. 复合型，指的是某些阶段多个人同时完成，剩下的阶段以流水线的形式多个人自己做自己的部分

### 2.2 实现

上面的第二种办法中，因为每个人的工作效率不同，每个阶段的结束，工作快的可能需要等待工作慢的，放在协程中，这就是协程的等待，协程的等待有几种实现方式

1. time.Sleep，但这种不太好估计等待时间，最终可能导致等待的时间过长或过短；
2. 锁，使用锁来限制下一步任务的执行；
3. for 循环，使用循环来等待一个协程完成后，开始下一个

实际情况下，会使用 sync.WaitGroup 来实现，一个实际的代码例子如下

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup

	for i := 0; i < 2; i++ {
		wg.Add(1)
		go func(num int) {
			fmt.Printf("Goroutine Test %d\n", num)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
// Output:
Goroutine Test 1
Goroutine Test 0
```

WaitGroup 是 sync 包提供的一种进行协程同步的方式，用于等待一组协程的结束，类型定义为

```go
type WaitGroup struct {
    // 包含隐藏或非导出字段
}
```

使用时，应在父协程中调用 Add 方法来设定等待的协程数量，每个被等待的协程应该在结束时调用 Done 方法，同时，父协程应该调用 Wait 方法等待所有协程执行完毕。

在这三个方法使用前，要先声明 waitGroup 的变量，比如 `var wg sync.WaitGroup`，下面说明三个主要的函数

```go
func (wg *WaitGroup) Add(delta int)
```

Add方法向内部计数加上delta，delta可以是负数；如果内部计数器变为0，Wait方法阻塞等待的所有协程都会释放，如果计数器小于0，方法panic。注意Add加上正数的调用应在Wait之前，否则Wait可能只会等待很少的协程。一般来说本方法应在创建新的协程或者其他应等待的事件之前调用。

```go
func (wg *WaitGroup) Done()
```

Done 方法减少 WaitGroup 计数器的值，应在协程的最后执行。

```go
func (wg *WaitGroup) Wait()
```

Wait方法阻塞直到WaitGroup计数器减为0。

### 2.3 多协程边界问题

日常编程中，数组越界、空指针、空栈pop这些边界问题都会引起 panic，使程序终止，多协程同样存在边界问题，但通常被称为生命周期管理。

官方定义中，协程的生命周期管理是协程从创建到运行结束等全部生命历程的管理，目的是便于协程的回收利用。协程之所以需要回收利用，是因为虽然协程的的开销很小，但也是有开销的，所以一个程序能开的协程数量是有上限的，如果超过了上限，多出来的协程就需要等到其它的协程结束后才可以开始。所以要进行协程的回收利用。

协程的生命周期主要分为协程创建、协程回收和协程中断三种。协程的创建通过关键词 go 完成，协程的回收由垃圾回收机制主动完成，我们需要主动处理的，只有协程的中断。

Go 中使用 context 来处理协程的中断，这里的中断其实指的不是意外造成的中断，而是主动的取消。比如，Go 的 http 服务器一般会启动多个 Goroutine 来处理传入的请求，来进行用户的身份认证、验证相关 token 等，当一个请求被取消时，处理该请求的 Goroutine 也应该随即退出，从而便于系统释放这些 Goroutine 占用的资源。举了一个例子如下

```go
package main

import (
	"context"
	"fmt"
	"sync"
)

func main() {
	// 初始化一个context
	parent := context.Background()
	// 生成一个取消的context
	ctx, cancel := context.WithCancel(parent)
	runTimes := 0

	var wg sync.WaitGroup
	wg.Add(1)
	go func(ctx context.Context) {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Goroutine Done")
				return
			default:
				fmt.Printf("Goroutine Running Times:%d\n", runTimes)
				runTimes += 1
			}
			if runTimes > 5 {
				cancel()
				wg.Done()
			}
		}
	}(ctx)
	wg.Wait()
}
// output:
Goroutine Running Times:0
Goroutine Running Times:1
Goroutine Running Times:2
Goroutine Running Times:3
Goroutine Running Times:4
Goroutine Running Times:5
Goroutine Done
```

其中 Background 方法的作用是初始化一个默认的 context 类型并返回，context 类型的定义如下

```go
// A Context carries a deadline, cancelation signal, and request-scoped values
// across API boundaries. Its methods are safe for simultaneous use by multiple
// goroutines.
type Context interface {
    // Done returns a channel that is closed when this Context is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}
```

随后使用 WithCancel 方法返回了一个新的 Context 和 CancelFunc，事实上，所有 context.WithXXX 方法都返回这两个值。其含义是，通过这些以 With 开头的方法派生出了新的 Context，当父 Context 取消时，其派生的所有 Context 都将取消。CancelFunc 就是用来取消父 Context 用的。

在上面的例子中，将产生的新 Context 命名为 ctx，交给了 Goroutine，当协程结束时，调用了 cancel，也就是之前同时产生的 CancelFunc，这一调用会取消父 Context。然后在下一次循环中，调用接口定义中的 Done 方法返回了一个通道，然后输出 “Goroutine Done"，随即正式退出协程。
