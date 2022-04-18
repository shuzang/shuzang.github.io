# Golang语法基础10-协程与通道


首先来回顾在操作系统中学过的一些概念。进程(processes)是程序执行的基本单位，运行在一个独立的内存地址空间中；一个进程由多个线程(threads)组成，线程的存在是为了能够同时执行多个任务，最大化利用时间，防止产生等待，线程间是共享内存地址空间的。从windows资源管理器看这一点能看的很明白，如下，每个应用程序是一个进程，Typora程序下有两个线程在同时运行。

![进程与线程](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20191216_3EkjsJ.png)

并发是建立在多线程之上的概念，将CPU的执行时间划分为许多很小的间隔，多个线程不断地切换执行，从上层看起来就像在同时执行一样，但本质上依然是线性的。并行则是程序在某个特定的事件同时运行在多个CPU上，多核处理器为并行提供了可能。因此，并发也可能是并行的。

操作系统课程中一个最主要的问题就是多线程对共享内存空间的访问，我们学到的解决方式是通过加互斥锁来实现，但在设计实现上是一个复杂的过程，非常容易出错，鉴于操作系统考试的惨痛经历，现在完全不想回忆。Go中的标准库`sync`中有一些工具可以用来实现互斥锁的相关操作，但它显然没有Go自身支持的Goroutines高效。

## 1. Goroutines

Go原生支持并发，依靠的是协程(goroutine)和通道(channel)两个概念。goroutines的概念是为了和processes、threads、coroutines等概念区别。其中coroutines也叫做协程，而且这才是常规意义下的协程，goroutines只在Go中有效。

coroutines是比线程更轻的一个概念，只使用很少的内存和资源。它对栈进行分隔，从而动态地增加或缩减内存的使用，栈的管理也是自动的，在协程退出后自动释放空间。协程可以运行在多个线程间，也可以运行在线程内，它的创建廉价到可以在同一地址空间存在100000个。这一概念也存在于其它语言(C#, Java等)中，它与goroutines的区别在于：

- Go协程意味着并行(或者可以以并行的方式部署)，协程一般不是
- Go协程通过通道来通信，协程则通过让出与恢复操作来通信

理论上，Go协程比协程更加强大。

以一个简单模型来描述goroutine：它是一个和其它协程在同一地址空间并发执行的函数。通过在函数或方法名前加上`go`关键字来创建和运行一个协程，运行结束后安静的退出(没有任何返回值)。

```go
go list.Sort() //并行的运行list.Sort，不等待
```

Go程序中必须含有的`main()`函数可以看作一个协程，尽管它没有通过`go`关键字启动，在程序初始化的过程中(`init()`函数运行)，goroutine也可以运行。

单纯的结束协程的概念是不够具体的，协程需要和通道来配合

## 2. Channel

并发编程的困难之处在于实现对共享变量的正确访问，互斥量的方式是复杂的，Go鼓励采用一种不同的方法，即在通道(channel)上传递共享值，如同Unix管道一般，通道用于发送类型化的数据，在任何给定的时间，只有一个协程可以对通道中的数据进行访问，从而完成协程间的通信，也避开了所有由共享内存导致的陷阱。这种通过通道进行通信的方式保证同步性的同时，数据的所有权也因此被传递。

这一设计理念最终简化为一句话：**不要通过共享内存来通信，而通过通信来共享内存**。

### 2.1 声明与初始化

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

### 2.2 通信操作符<-

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

### 2.3 通道阻塞

前面提到默认情况下通信是同步且无缓冲的，因此通道的发送/接收操作在对方准备好之前是阻塞的。

1. 对于同一个通道，发送操作（协程或者函数中的），在接收者准备好之前是阻塞的：如果ch中的数据无人接收，就无法再给通道传入其他数据
2. 对于同一个通道，接收操作在发送者可用之前是阻塞的（协程或函数中的）

下例中的协程在无限循环中不断地给通道发送数据，但由于没有接收者，只输出了数字0

```go
package main

import "fmt"

func main() {
	ch1 := make(chan int)
	go pump(ch1)       // pump hangs
	fmt.Println(<-ch1) // prints only 0
}

func pump(ch chan int) {
	for i := 0; ; i++ {
		ch <- i
	}
}
//Output:0
```

定义一个新的协程用来接收通道值可以持续输出

```go
package main

import "fmt"

func main() {
	ch1 := make(chan int)
	go pump(ch1)
	go suck(ch1)
	time.Sleep(1e9)
}

func pump(ch chan int) {
	for i := 0; ; i++ {
		ch <- i
	}
}

func suck(ch chan int) {
	for {
		fmt.Println(<-ch)
	}
}
```

### 2.4 信号量模式

信号量模式的意思是利用通道阻塞特性，等待所有计算完成后才让程序退出，用于并行执行函数或for循环，加快程序执行速度，一个例子瑞小安

```go
type Empty interface {}
var empty Empty
...
data := make([]float64, N)
res := make([]float64, N)
sem := make(chan Empty, N)
...
for i, xi := range data {
	go func (i int, xi float64) {
		res[i] = doSomething(i, xi)
		sem <- empty
	} (i, xi)
}
// wait for goroutines to finish
for i := 0; i < N; i++ { <-sem }
```

### 2.5 通道工厂

编程时常用一种通道工厂的模式，即不将通道作为参数传递给协程，而是用函数来生成一个通道并返回

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	stream := pump()
	go suck(stream)
	time.Sleep(1e9)
}

func pump() chan int {
	ch := make(chan int)
	go func() {
		for i := 0; ; i++ {
			ch <- i
		}
	}()
	return ch
}

func suck(ch chan int) {
	for {
		fmt.Println(<-ch)
	}
}
```

### 2.6 给通道使用for循环

`for` 循环的 `range` 语句可以用在通道 `ch` 上，便可以从通道中获取值，像这样：

```go
for v := range ch {
	fmt.Printf("The value is %v\n", v)
}
```

这样的使用依然必须和通道的写入和关闭相配合， 不能单独存在

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	suck(pump())
	time.Sleep(1e9)
}

func pump() chan int {
	ch := make(chan int)
	go func() {
		for i := 0; ; i++ {
			ch <- i
		}
	}()
	return ch
}

func suck(ch chan int) {
	go func() {
		for v := range ch {
			fmt.Println(v)
		}
	}()
}
```

### 2.7 关闭通道

通道可以被显式的关闭，不过只有发送者才需要关闭通道，接收者永远不需要。

```go
ch := make(chan float64)
defer close(ch)
```

测试通道是否关闭则可以使用ok操作符

```go
v, ok := <-ch   // ok is true if v received value
```

一个完整的例子如下

```go
package main

import "fmt"

func main() {
	ch := make(chan string)
	go sendData(ch)
	getData(ch)
}

func sendData(ch chan string) {
	ch <- "Washington"
	ch <- "Tripoli"
	ch <- "London"
	ch <- "Beijing"
	ch <- "Tokio"
	close(ch)
}

func getData(ch chan string) {
	for {
		input, open := <-ch
		if !open {
			break
		}
		fmt.Printf("%s ", input)
	}
}
```

但是使用for-range读取通道是更好的办法，因为这会自动检测通道是否关闭

```go
for input := range ch {
  	process(input)
}
```

## 3. Select

 从不同的并发执行的协程中获取值可以通过关键字`select`来完成，它和`switch`控制语句非常相似，其行为像是“你准备好了吗”的轮询机制；`select`监听进入通道的数据，也可以是用通道发送值的时候。 

```go
select {
case u:= <- ch1:
        ...
case v:= <- ch2:
        ...
        ...
default: // no value ready to be received
        ...
}
```

`select` 做的事情是：选择处理列出的多个通信情况中的一个。

- 如果都阻塞了，会等待直到其中一个可以处理
- 如果多个可以处理，随机选择一个
- 如果没有通道操作可以处理并且写了 `default` 语句，它就会执行：`default` 永远是可运行的（这就是准备好了，可以执行）。

使用`default`可以保证发送不被阻塞，但没有`default`的监听模式也可能被使用，通过`break`语句退出循环。一个完整的例子如下

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go pump1(ch1)
	go pump2(ch2)
	go suck(ch1, ch2)

	time.Sleep(1e9)
}

func pump1(ch chan int) {
	for i := 0; ; i++ {
		ch <- i * 2
	}
}

func pump2(ch chan int) {
	for i := 0; ; i++ {
		ch <- i + 5
	}
}

func suck(ch1, ch2 chan int) {
	for {
		select {
		case v := <-ch1:
			fmt.Printf("Received on channel 1: %d\n", v)
		case v := <-ch2:
			fmt.Printf("Received on channel 2: %d\n", v)
		}
	}
}
```

有 2 个通道 `ch1` 和 `ch2`，三个协程 `pump1()`、`pump2()` 和 `suck()`。在无限循环中，`ch1` 和 `ch2` 通过 `pump1()` 和 `pump2()` 填充整数；`suck()` 也在无限循环中轮询输入，通过 `select` 语句获取 `ch1` 和 `ch2` 的整数并输出。选择哪一个 case 取决于哪一个通道收到了信息。程序在 main 执行 1 秒后结束。 

## 4. 应用

### 4.1 惰性生成器

 生成器是指当被调用时返回一个序列中下一个值的函数，例如： 

```go
generateInteger() => 0
generateInteger() => 1
generateInteger() => 2
....
```

生成器每次返回的是序列中下一个值而非整个序列；这种特性也称之为惰性求值：只在你需要时进行求值，同时保留相关变量资源（内存和cpu）：这是一项在需要时对表达式进行求值的技术。例如，生成一个无限数量的偶数序列：要产生这样一个序列并且在一个一个的使用可能会很困难，而且内存会溢出！但是一个含有通道和go协程的函数能轻易实现这个需求。 

下例中实现了一个使用 int 型通道来实现的生成器。通道被命名为`yield`和`resume`，这些词经常在协程代码中使用。 

```go
package main

import (
    "fmt"
)

var resume chan int

func integers() chan int {
    yield := make(chan int)
    count := 0
    go func() {
        for {
            yield <- count
            count++
        }
    }()
    return yield
}

func generateInteger() int {
    return <-resume
}

func main() {
    resume = integers()
    fmt.Println(generateInteger())  //=> 0
    fmt.Println(generateInteger())  //=> 1
    fmt.Println(generateInteger())  //=> 2    
}
```

### 4.2 Futures 模式

所谓Futures就是指：有时候在你使用某一个值之前需要先对其进行计算。这种情况下，你就可以在另一个处理器上进行该值的计算，到使用时，该值就已经计算完毕了。

Futures模式通过闭包和通道可以很容易实现，类似于生成器，不同地方在于Futures需要返回一个值。

一个例子： 假设我们有一个矩阵类型，我们需要计算两个矩阵A和B乘积的逆，首先我们通过函数`Inverse(M)`分别对其进行求逆运算，再将结果相乘。如下函数`InverseProduct()`实现了如上过程： 

```go
func InverseProduct(a Matrix, b Matrix) {
    a_inv := Inverse(a)
    b_inv := Inverse(b)
    return Product(a_inv, b_inv)
}
```

在这里例子中，a和b的求逆句子的过程可以同时进行，因此换用并行计算方式如下：

```go
func InverseProduct(a Matrix, b Matrix) {
    a_inv_future := InverseFuture(a)   // start as a goroutine
    b_inv_future := InverseFuture(b)   // start as a goroutine
    a_inv := <-a_inv_future
    b_inv := <-b_inv_future
    return Product(a_inv, b_inv)
}
```

 `InverseFuture`函数以`goroutine`的形式起了一个闭包，该闭包会将矩阵求逆结果放入到future通道中： 

```go
func InverseFuture(a Matrix) chan Matrix {
    future := make(chan Matrix)
    go func() {
        future <- Inverse(a)
    }()
    return future
}
```

对计算密集型的场景，使用Futures模式是很有意义的

### 4.3 C/S模式

客户端/服务器(C/S)模式是goroutines和channels最常见的应用。

客户端(Client)可以是运行在任意设备上的任意程序，它会按需发送请求(request)至服务器。服务器(Server)接收到这个请求后开始相应的工作，然后再将响应(response)返回给客户端。典型情况下一般是多个客户端（即多个请求）对应一个（或少量）服务器。例如我们日常使用的浏览器客户端，其功能就是向服务器请求网页。而Web服务器则会向浏览器响应网页数据。 

使用Go的服务器通常会在协程中执行向客户端的响应，故而会对每一个客户端请求启动一个协程。一个常用的操作方法是客户端请求自身中包含一个通道，而服务器则向这个通道发送响应。 

下面是请求`Request`的结构，响应暗含在请求的结构里面

```go
type Request struct {
    a, b      int    
    replyc    chan int // reply channel inside the Request
}
```

服务器无限循环从`chan *Request`接收请求，为了避免某个请求长时间操作造成堵塞，为每个请求启动一个协程，然后启动`run()`函数处理该请求，处理后的值送到`chan *Reply`通道。

```go
func server(op binOp, service chan *Request) {
    for {
        req := <-service; // requests arrive here  
        // start goroutine for request:        
        go run(op, req);  // don’t wait for op to complete    
    }
}
type binOp func(a, b int) int

func run(op binOp, req *Request) {
    req.replyc <- op(req.a, req.b)
}
```

但以上过程仅仅是对客户端的请求是并发处理的，实际上，server本身也是以协程方式启动的

```go
func startServer(op binOp) chan *Request {
    reqChan := make(chan *Request);
    go server(op, reqChan);
    return reqChan;
}
```

startServer会在`main`函数中被调用，下面是`main`函数调用示例，一共发送了100个请求到服务器进行处理， 只有它们全部被送达后我们才会按相反的顺序检查响应： 

```go
func main() {
    adder := startServer(func(a, b int) int { return a + b })
    const N = 100
    var reqs [N]Request
    for i := 0; i < N; i++ {
        req := &reqs[i]
        req.a = i
        req.b = i + N
        req.replyc = make(chan int)
        adder <- req  // adder is a channel of requests
    }
    // checks:
    for i := N - 1; i >= 0; i-- {
        // doesn’t matter what order
        if <-reqs[i].replyc != N+2*i {
            fmt.Println(“fail at”, i)
        } else {
            fmt.Println(“Request “, i, “is ok!”)
        }
    }
    fmt.Println(“done”)
}
//Output:
Request 99 is ok!
Request 98 is ok!
...
Request 1 is ok!
Request 0 is ok!
done
```

完整的程序如下，因为`server`在`main`函数返回后是被强制结束的，下面还针对这一点做出改进，方法是提供一个退出通道给server

```go
package main

import "fmt"

type Request struct {
	a, b   int
	replyc chan int // reply channel inside the Request
}

type binOp func(a, b int) int

func run(op binOp, req *Request) {
	req.replyc <- op(req.a, req.b)
}

func server(op binOp, service chan *Request, quit chan bool) {
	for {
		select {
		case req := <-service:
			go run(op, req)
		case <-quit:
			return
		}
	}
}

func startServer(op binOp) (service chan *Request, quit chan bool) {
	service = make(chan *Request)
	quit = make(chan bool)
	go server(op, service, quit)
	return service, quit
}

func main() {
	adder, quit := startServer(func(a, b int) int { return a + b })
	const N = 100
	var reqs [N]Request
	for i := 0; i < N; i++ {
		req := &reqs[i]
		req.a = i
		req.b = i + N
		req.replyc = make(chan int)
		adder <- req
	}
	// checks:
	for i := N - 1; i >= 0; i-- { // doesn't matter what order
		if <-reqs[i].replyc != N+2*i {
			fmt.Println("fail at", i)
		} else {
			fmt.Println("Request ", i, " is ok!")
		}
	}
	quit <- true
	fmt.Println("done")
}
```


