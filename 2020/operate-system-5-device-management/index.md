# 操作系统5-设备管理


本篇介绍设备管理的相关内容。最近发现本科用的教材内容非常陈旧，而且不是那么浅显易懂，于是找了一本国外的教材《操作系统导论》，主要根据这个来学习。

<!--more-->

## 1. 系统架构

一个典型的系统架构如下。其中，CPU 通过某种内存总线或互联电缆连到系统内存，图像（比如显卡）或其它高性能 I/O 设备通过常规的 I/O 总线连到系统，可能是 PCI 或其衍生形式。最下面是外围总线，比如 SCSI、SATA 或者 USB，它们将最慢的设备连接到系统，包括磁盘、鼠标和其它类似设备。

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200817_epub_30179184_196.jfif" style="zoom: 67%;" />

采用这种分层架构的原因是物理布局和造价成本。

## 2. 标准设备

尽管设备可能分很多种，比如存储设备、输入输出设备、各种终端和脱机设备，但这里使用一种标准设备来介绍，这里提到的标准设备不是真实存在的，但它的组成和互操作可以代表大部分的设备。

如下图，一个标准设备主要包括两部分。一部分是对计算机其它部分展现的硬件接口，另一部分是这些设备的内部结构，一些简单的设备可能只有一个或几个芯片，但复杂一些的设备还会包括自己的 CPU 和内存。

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200817_epub_30179184_197.jfif" style="zoom:67%;" />

## 3. 设备交互

一个简化的设备接口包括三个寄存器：状态寄存器读取并查看设备当前状态，命令寄存器通知设备执行某个任务，数据寄存器将数据传给设备或从设备接收数据。通过读写这些寄存器，操作系统可以控制设备行为。

最简单的交互逻辑如下

```c
while Status == Busy 
    // wait until device is not busy
Write data to DATA register;
Write command to COMMAND register;
While Status == Busy
    // wait until device is done with your request
```

简单来说，就是操作系统不断读取状态寄存器（轮询），如果设备已经准备好了，就开始传输数据到数据寄存器，传输完成后将命令写入命令寄存器，这样设备就知道数据准备好了，开始执行命令。最后继续轮询，判断设备是否执行完命令。

这个协议简单有效，只是轮询过程比较低效，而且 CPU 需要等待设备执行，浪费大量的时间。

### 3.1 中断

中断是指计算机执行期间，由于发生了某些非预期的紧急事件，计算机暂停当前进程而执行相应的事件处理程序，执行完毕后又返回执行暂停的进程的过程。中断可能来自 I/O 设备发出的信号、外部信号（比如键盘 Esc 输入）、定时器引起的时钟中断、调试程序的断点、程序运算产生的溢出、非法指令等等。

所有需要由硬件产生的中断叫硬中断，比如上面提到的这些。还有一种叫软中断，是进程模拟用来通信的，软中断不一定需要立即执行，可以由 CPU 选择合适的时机执行。

当我们用中断来替换设备交互中的轮询过程时，可以极大提升效率。过程如下：

1. CPU 向设备发出一个请求，然后就可以让相关进程休眠，然后去执行其它任务；
2. 设备完成自身操作后，产生一个硬中断，CPU 就会去执行对应的中断处理程序，也就是唤醒睡眠的 I/O 进程来继续执行；

但是，中断的高效建立在外围设备和 CPU 处理速度差异较大的情况下，如果外设处理速度比较快，那么轮询反而比中断更好。另外一个原因是，外设的数量比较多，而且是并行工作的，可能会造成中断数量急剧增加，极端的情况，网卡每收到一个数据包产生一个中断，就会导致系统无法响应。一种解决办法是合并多个中断为一个，统一处理。

### 3.2 DMA

设备交互的简单协议还有一个问题，外设与内存的数据传输总是需要 CPU 调度，有时候对 CPU 是一个负担。解决方法是 DMA（Direct Memory Access），它的基本思想是在外设和内存之间开辟一个直接的数据通路。

DMA 工作过程如下。为了能够将数据传送给设备，操作系统会通过编程告诉 DMA控制器 数据在内存的位置，要拷贝的大小以及要拷贝到哪个设备。在此之后，操作系统就可以处理其他请求了。当 DMA 的任务完成后，DMA 控制器会抛出一个中断来告诉操作系统自己已经完成数据传输。

### 3.3 通道

DMA 方式中，数据的传送方向、存放数据的内存地址、传送的数据块长度等还是需要 CPU 来控制。通道是一种专门的硬件，可以看作是一个专管输入输出的处理机，它会替代 CPU 完整所有这些工作，并且可以控制多台外设与内存交互，因此进一步减轻了 CPU 的负担。

### 3.4 交互

系统与设备的交互主要由两种方式

1. 明确的 I/O 指令，这些指令规定了操作系统将数据发送到特定设备寄存器的方法。比如，x86 中，in 和 out 指令可以用来与设备进行交互。当需要发送数据给设备时，调用者指定一个存入数据的特定寄存器及一个代表设备的特定端口。执行这个指令就可以实现期望的行为。这些指令通常是特权指令（privileged）。操作系统是唯一可以直接与设备交互的实体。
2. 内存映射 I/O，即硬件将设备寄存器作为内存地址提供。当需要访问设备寄存器时，操作系统装载（读取）或者存入（写入）到该内存地址；然后硬件会将装载/存入转移到设备上，而不是物理内存。

## 4. 设备驱动

设备驱动程序是驱动物理设备与 DMA 控制器或 I/O 控制器等直接进行 I/O 操作的子程序的集合，负责设置相应设备有关寄存器的值，启动设备进行 I/O 操作，指定操作的类型和数据流向等。

由于所有需要插入系统的设备都需要安装相应的驱动，所有一个系统的大部分代码其实都是各种驱动程序。当然，这些驱动一般都不是激活的，只有一小部分需要在系统刚开始就连接。另外一件事，驱动程序的开发者大部分不是全职的内核开发者，所以容易出现缺陷，也是内核崩溃的主要贡献者。