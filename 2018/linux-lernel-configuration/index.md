# 系统移植4-内核配置


内核配置的目的主要是裁剪掉不必要的文件和目录，获得一个最适用的操作系统。可通过执行下面的命令进入配置窗口

```bash
$ make menuconfig
```

执行完毕后显示一个基于文本的图形化终端配置菜单，这是是使用最广的配置方法，如果一个.config文件已经存在，它将使用该文件设置那些默认值

## 1. 启动内核配置窗口

进入需要被配置的内核的根目录，使用 `make menuconfig` 命令启动内核配置窗口，如下图

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427162438680.png)

最上面的标题栏里，.config是核心功能列表档，我们现在就是在对这个文件做处理；x86是我的电脑CPU架构，4.1.15是我现在要配置的这个内核的版本

整个界面的用法大概是这样：

1. 左右方向键：可以移动最底下的 项目；

2. 上下方向键：可以移动上面框里的蓝色光柱选择选项，该行有箭头 (--->) 则表示该行内部还有其他细项需要来配置的意思；

3. 选定项目：以上下方向键选择好想要配置的项目之后，以左右方向键选择， 按下回车就可以进入该项目去作更进一步的细节配置；

4. 可挑选功能：在细节项目的配置当中，如果前面有 [ ] 或 < > 符号时，该项目才可以选择， 而选择使用空格键；

5. 若为 [*] <*> 则表示编译进核心；若为则表示编译成模块。

配置时可以遵循这样的原则：

1. 一定用的功能，编译进核心；
2. 未来可能用到的功能，尽量编译为模块
3. 不知道用来做什么的功能，看help也看不懂，保留默认值，或者编译为模块



## 2. 具体配置

关于内核配置主选项的记录丢失，只剩关于 General setup 部分的记录。与 Linux 最相关的程序互动、核心版本说明、是否使用发展中程序码等资讯都在该部分配置的。 这里的项目主要都是针对核心与程序之间的相关性来设计的，基本上，保留默认值即可！ 不要随便取消底下的任何一个项目，因为可能会造成某些程序无法被同时运行的困境！ 不过底下有非常多新的功能，如果有不清楚的地方，可以按`<help>`进入查阅，里面会有一些建议！ 你可以依据 Help 的建议来选择新功能的启动与否！

![General setup 选项](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_03d038acd2f0.png)

**cross-compiler tool prefix**

交叉编译工具前缀，功能是make时自动使用交叉编译器进行编译。比如在这儿填上*arm-linux*，相当与编译时使用*make CROSS_COMPILE=arm-linux-*命令。如果你不想自动使用交叉编译工具，此处应该留空

**compile also drivers which will not load**

编译驱动程序将不加载，一些驱动可能在其它的编译平台被编译，而这个编译平台跟它的期望运行平台不一样（好像就是交叉编译。。），所以即使这些驱动不能在编译平台加载运行，一些开发者仍然可能想要构建这个驱动来进行编译测试（就是看看它到底能不能编译通过，没打算让它运行么）所以，根据你的需求来选择好了，这个一般是不选。

**Local version - append to kernel release**

附加一段自定义字符串在内核的版本后面，当你使用uname命令的时候，这段字符串会显示在原本的输出后面，这个就是用来玩的吧。最大支持64个字符，看了一下我的

![uname](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_faf71b80f7fb.png)

**Automatically append version information to the version string**

这个选项会通过查找内核目录下的git标签来自动检测内核目录下是否包含git版本树信息，如果找到的话，就会在上面设置的自定义字符串后边自动添加“-gxxxxxxxx”这种格式的后缀。需要有git的支持。也是个可有可无的选项

**Kernel compression mode ( ) --->**

这儿是选择内核的压缩格式，回车进去可以看到，有Gzip,Bzip2,LZMA,XZ,LZ0,LZ4，一般情况下选哪一个都可以，如果不确定到底选哪个的话，官方推荐Gzip

![压缩格式选择](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_2852c78f6c70.png)

**Default hostname**

在这儿设置默认的主机名，在你的用户空间程序使用系统调用sethostname来修改主机名之前，内核将使用你在这儿设置的默认主机名。这儿的默认值是“(none)”，这儿保持默认就好，反正没什么意义，系统启动之后肯定要修改主机名的，那时会将主机名改为/etc/hostname下的设置

**Support for paging of anonymous memory (swap)**

通过分页、交换区等机制来实现虚拟内存，通俗的理解成是对swap区的支持也可以，总之，这个选项是你必选的，没有MMU的cpu除外

**System V IPC**

system V进程间通信支持，这个必选，这样应用程序才能使用内核提供的各种通信机制进行进程间通信。syetem V IPC包括消息队列，信号量，共享内存等

**POSIX Message Queues**

POSIX消息队列支持，POSIX消息队列是一种进程通信机制， 在POSIX消息队列中，每一个消息有一个优先级。如果你想要编译运行像mq_*这样子的函数（一般是为Solaris系统而写的程序），那么你需要选上此项

**Enable process_vm_readv/writev syscalls**

启用此选项可添加系统调用process_vm_read 和process_vm_writev，这些进程允许具有正确p的进程直接从另一进程的地址读取或写入另一个进程的地址

**Open By Fhandle Syscalls**

如果你选上此项，意味着一个用户空间的应用程序将能够映射一个文件句柄到一个文件名，然后通过这个文件句柄来操作这个文件，这个特性对用户空间文件服务器很有用，就是说开启此选项后，可以使用句柄来追踪一个文件，这样即使文件被重命名，程序依然可以通过句柄来定位这个文件。这个选项会开启内核对open_by_handle_at()和name_to_handle_at这两个系统调用的支持

**uselib syscall**

此选项启用了uselib系统调用，这是从libc5和更早版本在动态链接器中使用的系统调用。 glibc不使用此系统调用。 如果您打算运行基于libc5或更早版本的程序，则可能需要启用此系统调用。 运行glibc的当前系统可以安全地禁用此功能。现在还有更早版本的吗，可以取消掉吧

**Auditing support**

内核审计支持，其它的一些内核子系统可能需要使用这个功能，比如SELinux

**Eanble system-call auditing support**

使能对low-overhead (低开销？) 系统调用的审计支持，这个功能可以单独使用，也可以被系统中其他的内核子系统使用，比如SELinux

**IRQ subsystem --->**

中断请求子系统，回车进去只有一个选项：

Expose hardware/virtual IRQ mapping via debugfs

这个选项可以让你通过debugfs中的irq_domain_mapping文件来获得硬件的中断请求号和Linux中的中断请求号之间的映射关系。一般只在内核调试时候使用，help解释如果你听不懂上面的那些名词，说明你不需要开启这个选项，好吧，其实我能看懂的没几个。。。

**Timers subsystem --->**

内核时钟子系统，回车进去三个选项：

1）Timer tick handling (Idle dynticks system (tickless idle))--->

定时器滴答处理程序，这儿有三个选择，下面分别介绍

第一种：Periodic timer ticks (constant rate, no dynticks)

以固定周期触发时钟中断，无论CPU是否需要，不使用动态定时器（dyniticks），这是最耗电的方式，不推荐使用。

第二种：Idle dynticks system (tickless idle)

在CPU空闲时不产生不触发的时钟中断，可以省电

第三种：Full dynticks system (tickless)

使用动态定时器，即使CPU在忙碌状态，也尽可能的关闭所有时钟中断，这个特性很早就有但是直到3.10版本才完整支持DynTicks（动态定时器），并成为内核级别的核心特性，tickless适用于CPU在同一时间内只运行一个任务，或者用户空间程序极少与内核交互的场合，但是即使你选择了tickless，仍然需要额外设置‘nohz_full=?’的内核参数才能是tickless真正生效

所以。。意思就是一般选第二个咯

2）Old Idle dynticks config

这个选项是Idle dynticks system (tickless idle)旧的入口，此选项被用来保持与旧版本的兼容，未来应该会被删除，但现在为了和上面保持一致还是选了吧

3）High Resolution Timer Suppor

高精度计时器支持，如果你的硬件不能胜任这个选项，那么开启这个选项仅仅是给内核增加体积，高精度定时器（hrtimer）是从2.6.16开始被引入内核，采用红黑树算法（传统timer使用时间轮算法），在硬中断中执行中断服务例程，它可以为我们提供纳秒级的定时精度（这么厉害），以满足对精确时间有迫切需求的应用程序或内核驱动，例如多媒体应用，音频设备的驱动程序等等，额，这个还是选着吧，说不定用板子做什么呢

**CPU/Task time and stats accounting --->**

CPU/任务 时间和状态统计

1）Cputime accounting (Simple tick based cputime accounting) --->

CPU时间统计方式，三种选择：

第一种：Simple tick based cputime acounting

最简单的基于tick统计方式，使用jiffies变量（这个变量记录了系统自启动以来经过的操作系统节拍的数目） 官方推荐：if unsure, say Y(yes)

第二种：Full dynticks CPU time accounting

利用上下文跟踪子系统，通过观察每一个内核与用户空间的边界来统计，如果你不是在帮助开发内核的动态定时器（dynticks）的话，不要选择这个

第三种：Fine granularity task level IRQ time accounting

通过读取TSC时间戳进行CPU时间统计，这是统计CPU时间的更细粒度的方式，对系统性能有显著不良影响 官方推荐：if in doubt , say No

2）BSD Process Accounting

BSD进程记账支持.用户空间程序可以要求内核将进程的统计信息写入一个指定的文件,主要包括进程的创建时间/创建者/内存占用等信息.不确定的选"N".

3）BSD Process Accounting version 3 file format

使用新的v3版文件格式,可以包含每个进程的PID和其父进程的PID,但是不兼容老版本的文件格式.比如 GNU Accounting Utilities 这样的工具可以识别v3格式

4）Export task/process statistics through netlink

通过netlink接口向用户空间导出进程的统计信息,与 BSD Process Accounting 的不同之处在于这些统计信息在整个进程生存期都是可用的.

5）Enable per-task delay accounting

在统计信息中包含进程等候系统资源(cpu,IO同步,内存交换等)所花费的时间

6）Enable extended accounting over taskstats

在统计信息中包含进程的更多扩展信息.不确定的选"N".

7）Enable per-task storage I/O accounting

在统计信息中包含进程在存储设备上的I/O字节数.


