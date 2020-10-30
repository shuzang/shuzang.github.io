# 符合泊松分布的事件模拟到达时间生成


我们要进行的仿真是在随机的时间执行随机的事件，这个时间就叫做事件到达时间。根据已有知识，随机的事件到达时间应该符合泊松分布，事件到达时间的间隔符合指数分布，实现时通常采用生成到达时间间隔的方式。这里的实现翻译了文章 [How to Generate Random Timings for a Poisson Process](https://preshing.com/20111007/how-to-generate-random-timings-for-a-poisson-process/)，使用的语言是 Go。

<!--more-->

事件的发生是随机的，但是从总体上看，事件以平均的速率发生，这就是泊松过程。举个例子，USGS 预计每年全世界大约发生 13000 场 4 级以上的地震，这些地震发生的时间是随机的，但一定在 13000 场左右。

统计学中有大量的函数和方程用于建模泊松过程，这篇文章介绍了一种其中一种函数，并给出了一个实现程序。

## 1. 指数分布

如果每年 13000 场地震，那么平均 40 分钟一场地震，所以定义变量 $\lambda = \frac{1}{40}$，称为速率参数，这是一个频率的衡量：单位时间（地震的例子里是分钟）发生事件（地震）的平均速率。

因此，接下来的问题是，下一分钟发生地震的概率是多少？下一个十分钟呢？这里有一个众所周知的函数，称为 指数分布的累积分布函数（cumulative distribution function for the exponential distribution），该函数看起来如下：
$$
F(x) = 1 - e^{-\lambda x}
$$


![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/exponential-curve.png)

其含义是，随之时间的流逝，在世界上某个地方发生地震的可能性不断增大，这里「指数」的含义是指数衰减，随着时间流逝，不发生地震的可能性逐渐趋近于0，相应的，发生至少一场地震的可能性也趋向于1。

插入一些值，我们发现：

- 下一分钟发生地震的可能性为 $F(1) \approx 0.0247$，该值无限接近于 $\frac{1}{40}$，这个我们预设的地震频率，但不相等；
- 下一个十分钟发生地震的可能性为 $F(10) \approx 0.221$

特别的，下一个 40 分钟发生地震的可能性为 $F(40) \approx 0.632$，因此，40分钟的间隔内很可能发生地震，但不绝对。

## 2. 编写仿真

现在，假设我们要模拟游戏引擎或其他某种程序中地震的发生。首先，我们需要弄清楚每次地震的开始时间。

一种方法是循环，每隔 X 分钟之后，在 0 到 1 之间采样一个随机浮点值。如果该数字小于 $F(X)$，则开始地震。X 可以是一个小数值，因此可以每分钟采样几次，甚至每秒采样几次。只要随机数生成器是统一的并且提供足够的数值精度，这一个方法就会很好用。但是，如果打算以 $λ=\frac{1}{40}$ 每秒进行 60 次采样，随机数生成器需要至少18位精度，标准 C运行时库并不总是提供这一精度。

另一种方法是回避整个采样策略，只需编写一个函数即可确定下一次地震的确切时间。此函数应返回随机数，但不是大多数生成器生成的统一类型的随机数，而是以遵循指数分布的方式生成随机数。

Donald Knuth 在 「The Art of Computer Programming」一书的 3.4.1(D) 一节描述了一种生成这种值的方法，只需在 y 轴上选择介于 0 和 1 之间的均匀分布的随机点，然后在 x 轴上找到相应的时间值即可。例如，如果我们从下图 y 轴选择 0.2 点，那么到下一次地震的时间将是 64.38 分钟。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/inverse-lookup.png)

由于指数函数的反函数是 ln，写这个程序很简单，其中 U 是 0 到 1 之间的随机值：

## 3. 实现

```go
package main

import (
	"fmt"
	"math"
	"math/rand"
	"time"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	for i := 0; i < 5; i++ {
		fmt.Println(nextTime(1 / 40.0))
	}

}

func nextTime(rateParameter float64) float64 {
	return -math.Log(1.0-rand.Float64()) / rateParameter
}
// Output:
3.645968256349058
21.416099701223878
27.140451644356354
132.53700107810388
10.94869965544849
```

经测试，该函数返回的平均时间确实为40

```go
package main

import (
	"fmt"
	"testing"
)

func TestNextTime(t *testing.T) {
	var sum float64
	for k := 0; k < 10; k++ {
		for i := 0; i < 1000000; i++ {
			sum += nextTime(1 / 40.0)
		}
		fmt.Println(sum / 1000000)
		sum = 0
	}
}
// Output:
=== RUN   TestNextTime
39.936436485414866
40.073299195147676
40.02405410596529
39.984823394877324
39.970452381128254
40.05045384327815
39.94419161580051
40.038542654941246
39.983753932119754
40.029867240804506
--- PASS: TestNextTime (0.42s)
PASS
ok  	github.com/shuzang/test	0.652s
```

实际上，Go 在 math/rand 库中本身就提供了一个生成符合指数分布的随机数的函数，叫做 `rand.ExpFloat64()`。实现的算法使用的是 Marsaglia 和 Tsang 在 2000 年发布的论文 [The Ziggurat Method for Generating Random Variables](https://www.jstatsoft.org/v05/i08/paper)

## 4. 其它仿真器

[The One](http://akeranen.github.io/the-one/) 是一个 opportunistic Network Environment simulator，可以设置一个仿真的 IoT 网络，参数包括网络中设备数目、带宽、通信到达时间等，使用不同的模型生成随机的运动和通信，并将过程可视化。
