# 系统移植1-移植相关知识


## 1. Linux 操作系统组成

Linux 操作系统由 Linux 内核、shell 命令解释器和应用程序3部分构成

### 1.1 shell

Linux的内核不能直接接受来自终端的用户命令，shell 为用户提供使用 Linux 操作系统的接口。在Linux 中几乎所有的操作都可以通过命令行来完成，使用 shell 编写的程序称为 shell 脚本。shell 可以作为命令语言、命令解释程序及程序设计语言，用户成功登录Linux时系统自动启用shell，当在终端输入正确的shell命令时，shell通过相应的命令和程序，通过内核执行用户需要的操作。更详细的知识可参考 [评估Linux中的shell](https://www.ibm.com/developerworks/cn/linux/l-linux-shells/index.html) 一文。

![1977年以来的Linux shell](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_41342748db2b.gif)

几种 shell 的比较表如下

| Shell                | 介绍                                                         |
| -------------------- | ------------------------------------------------------------ |
| Bourne Shell         | 编程方式占优势，但在与用户交互方面比较差                     |
| Bourne - Again Shell | 专为 Linux 而写，在 Bourne Shell 基础上增加了功能，是 Linux 默认内核 |
| C Shell              | 语法类似于 C 语言，有较高的编程能力                          |
| Tcsh                 | C shell 的扩展                                               |
| Korn Shell           | 集合了 C shell 和 Bourne shell 的优点                        |

### 1.2 Linux 内核

Linux 内核是操作系统的核心部分。它由进程管理、文件管理、存储管理、设备管理和网络管理五大部分组成。采用模块化的设计，它的功能也是通过增加和减少模块来实现的。这种设计保证系统封闭和开放与效率的平衡，避免在修剪功能时改变系统结构。

Linux 内核最注重的问题是实用和效率，其特点如下

1. 整个 Linux 内核由很多过程组成，每个过程可以独立编译，然后用连接程序将其连接在一起成为一个单独的目标程序。
2. Linux 的文件系统最大特点是实现了一种抽象文件模型——VFS。使用虚拟文件系统屏蔽了各种不同文件系统的内在差别，使得用户可以使用同样的方式访问各种不同格式的文件系统，可以毫无区别的在不同格式、不同介质的文件系统之间使用 VFS 提供的统一接口交换数据。
3. 为了保证方便的支持新设备、新功能，又不会无限扩大内核规模，Linux 系统对设备驱动或新文件系统等采用了模块化方式，用户在需要时可以动态加载，使用完毕可以动态卸载。同时对内核，用户也可以定制，选择适合自己的功能，将不需要的部分剔除出内核。这两种技术都保证了内核的紧凑性和扩展性。

## 2. u-boot

u-boot是在 PPCBOOT 和 ARMBOOT 基础上发展而来的，是一个通用引导程序，支持很多架构，这一点上篇BootLoader已经很明白。u-boot的移植过程是一个对嵌入式系统包括软硬件及操作系统加深理解的一个过程，我们通过这个过程来一点点学习。

### 2.1 常用命令

u-boot在下载模式下，提供了许多有用的命令

**环境变量类**

printenv：查看环境变量

saveenv：保存当前环境变量

setenv：设置当前环境变量

askenv：在标准输入获得环境变量

**存储类**

md：显示指定内存地址中的内容

mm：顺序显示指定地址往后内存中的内容，可同时修改，地址自动递增

mw：向内存地址写内容

nm：修改内存地址，地址不递增

mtest：简单的RAM检测

**下载类**

tftp：将内核镜像文件从PC中下载到SDRAM的指定地址，然后通过bootm来引导内核，前提是所用PC要安装设置TFTP服务

loadb：透过串口下载二进制格式的文件

loads：透过串口下载S-Record格式的文件

**启动类**

boot：预先设定的启动命令并且启动

bootm：从某个地址启动内核

bootp：通过网络使用bootp或者TFTP协议引导镜像文件

bootelf：默认从0x30008000引导elf格式的文件

bootd（=boot）：引导的默认命令，即运行u-boot中在“include/configs/”板子名.h”中设置的“bootcmd”中的命令

**Flash命令**

erase：擦除Flash内容，必须以扇区为单位进行擦除

flinfo：查看Flash的信息

help：帮助命令，用于查询u-boot支持的命令

bdinfo：查看目标系统参数和变量，目标板的硬件配置

coninfo：显示控制设备和信息

flinfo：获取Flash存储器的信息

iminfo：打印和校验内核镜像头，内核的起始地址由CFG_LOAD_ADDR指定

**Cache类命令**

icache：开启和关闭指令Cache

dcachd：开启和关闭数据Cache

**其他命令**

reset：复位CPU

run：运行已经定义好的u-boot命令

sleep：命令延时执行时间

autoscr：从内存运行脚本

base：打印或者设置当前指令与下载地址的地址偏移

cmp：对输入的两段内存地址进行比较

version：打印u-boot版本信息

**Nand相关命令**

Nand info：打印Nand Flash信息

Nand device <n>：显示某个Nand设备

Nand erase FLAddr size：FLAddr为Nand Flash起始地址，size为从中擦除数据块的大小

Nand write InAddr FLAddr size ：InAddr为写到Nand Flash中的数据在内存的起始地址

### 2.2 u-boot源代码目录结构

u-boot主要的源代码目录如下图：

![u-boot源代码目录树](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_be4a61b7d1d7.png)

主要目录的作用列一个表如下：

![主要目录作用](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_dab6588a58c1.png)

### 2.3 u-boot的编译

u-boot的源码是通过gcc和Makefile组织编译的。顶层目录下的Makefile首先可以设置开发板的定义，然后递归地调用各级子目录下的Makefile，最后把编译过的程序链接成u-boot镜像。

目录下的Makefile看不懂，等学完Makefile再回来看这个吧，这里是一篇[GNU make 中文手册](https://hacker-yhj.github.io/resources/gun_make.pdf)

#### 配置头文件

除了编译过程Makefile外，还要在程序中为开发板定义配置选项或者参数。这个头文件是include/configs<board_name>.h。<board_name>用相应的BOARD定义代替。

这个头文件中主要定义了两类变量：

- 选项：前缀是CONFIG_，用来选择处理器、设备接口、命令、属性等

- 参数：前缀是CFG_，用来定义总线频率、串口波特率、Flash地址等参数

#### 编译

根据对Makefile的分析，编译分为两步：

配置：make smdk2410_config

编译：make

编译完成后，可以得到u-boot各种格式的映像文件和符号表，如下：

![编译完成得到的文件](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_755481c360ac.png)

u-boot的3中映像格式都可以烧写到Flash中，但需要看加载器能否识别这些格式。u-boot.bin最为常用，直接按照二进制格式下载，并且按照绝对地址烧写到Flash中就可以。

#### u-boot工具

还记得上面列到的tools目录吗，这个目录下存放有u-boot的一些工具，有的工具经常被用到，下面用一个表说明几种工具的用途：

![u-boot工具](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_7fd06fbc33d3.png)

### 2.4 u-boot的移植

为了使u-boot支持新的开发板，一种简便的做法是在u-boot已经支持的开发板中选择一种和目标板接近的，并在其基础上进行修改，代码修改的步骤如下。

1. 在board目录下创建smdk2410目录，添加smdk2410.c、flash.c、memsetup.s、u-boot.lds和config.mk等；

2. 在cpu目录下创建arm920t目录，主要包含start.s、interrupts.c、cpu.c、serial.c和speed.c等文件

3. 在include/configs目录下添加smdk2410.h，它定义了全局的宏定义等。

4. 修改u-boot根目录下的Makefile文件，如下： 

   `smdk2410_config：$（@： _config=）arm arm920t smdk2410`

5. 运行 make smdk2410_config，如果没有错误，就可以开始进行与硬件相关的代码移植工作

### 2.5 u-boot的使用

1）烧写u-boot到Flash

新开发板中没有任何程序可执行，也就不能启动，需要先将u-boot烧写到Flash中

如果主板上的EPROM或者Flash能够取下来，就可以通过编程器烧写。计算机BIOS就存储在一块256KB的Flash上，通过插座与主板相连。但是多数嵌入式单板使用贴片的Flash，不能取下来烧写。这种情况可以通过处理器的调试接口，直接对板上的Flash编程。

处理器调试接口是为处理器芯片设计的标准调试接口，包含BDM，JTAG和EJTAG 3中接口标准。这3种硬件接口标准定义有所不同，但是功能基本相同，下面都统称为JTAG接口。

JTAG接口需要专用的硬件工具来连接，无论从功能、性能角度，还是从价格角度，这些工具都有很大差异。最简单方式就是通过JTAG电缆，转接到计算机并口连接。这需要在主机端开发烧写程序，还需要有并口设备驱动程序。开发板上电或复位时，烧写程序探测到处理器并且开始通信，然后把BootLoader下载并烧写到Flash中。

烧写完成后，复位开发板，串口终端应该显示u-boot启动信息。

![串口终端显示的u-boot启动信息](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_5a239c8a9488.png)



2）u-boot的环境变量

可通过printenv命令查看环境变量设置，前面介绍过这些命令

![查看环境变量](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_c5f55503ce10.png)

下表列出一些常用的环境变量的含义解释

![img](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_68caba4bb42c.png)

 u-boot的环境变量都可以有默认值，也可以修改并且保存在参数区。u-boot的参数区一般有EEPROM和Flash两种设备。

## 3. 交叉编译

因为对这方面可以说是完全不了解，所以凭现有的知识提出来的问题大概是这么几个，至于学习过程中产生的新的问题，就到时候再解决：

1、交叉编译是什么以及为什么需要交叉编译

2、怎么得到这个交叉编译环境

3、怎么使用这个交叉编译环境

### 3.1 交叉编译简介

交叉编译，就是在一种平台上（称为宿主机）开发编译，编译出来的程序，在别的平台上（称为目标机）运行，即编译的环境和运行的环境是不一样的、交叉的，而程序的调试需要通过宿主机和目标机之间的协作来交互进行。交叉编译这个概念，主要和嵌入式开发有关，英文称为cross compile。这和我们平常在X86的电脑上开发、编译可执行程序，然后直接在X86环境下运行是相对的。

一种最常见的例子就是：在进行嵌入式开发时，手上有个嵌入式开发板，CPU是arm的，然后在x86的平台下开发，比如Ubuntu环境，然后就需要在x86的平台上，（用交叉编译器）去编译写好的程序代码，生成的（可执行的）程序是放到目标开发板—arm的CPU上运行的，即在x86平台上编译，在ARM平台上运行。

至于为什么需要交叉编译，有两点

1. 嵌入式系统硬件资源的限制，比如cpu主频相对较低，内存容量较小等，相对来说，pc机的速度更快，硬件资源更加丰富，因此利用pc机进行开发会提高开发效率。这是一直以来的说法，不过，现在嵌入式的硬件主频和内存这些资源都不算小了，个人觉得可以直接放在开发板上做本地编译。
2. 嵌入式系统MCU体系结构和指令集不同，因此需要安装交叉编译工具进行编译，这样编译的目标程序才能够在相应的平台上比如：ARM、MIPS、POWEPC上正常运行。

### 3.2 交叉工具链简介

我们要完成的目标是生成可执行程序或库文件，为了达成此目标，内部的执行过程和逻辑主要包括：

1. 编译：编译的输入是程序代码，输出是目标文件，使用的工具叫编译器，常见的编译器如gcc
2. 链接：链接的输入为程序运行时所依赖的或者某个库所依赖的另外一个库（文件），链接的输出为程序的可执行文件，或者是可以被别人调用的完整的库文件。链接使用的工具叫链接器，最常见的链接器是ld

实际上，ld只是处理目标文件（二进制文件）最主要的一个工具，相关的还有很多其他工具，如as, objcopy, strip, ar等，对此，GNU官网弄出一个binutils，即binary utils，二进制工具（包），集成了所有这些和操作二进制相关的工具集合。关于这个东西，详见[GNU Binutils详解](https://www.crifan.com/files/doc/docbook/binutils_intro/release/html/binutils_intro.html)，而对于常用的列一个表给大家看一下：

![常用工具](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_f12ee86dc76e.png)

把上面设计到的一系列工具，按照对应的逻辑功能，编译、链接、后期其它处理等等，串起来，就是工具链。

而用于交叉编译的工具链就是交叉工具链。里面包含了很多工具，但是最主要的是用于编译的gcc，所以常把交叉工具链称为交叉编译器。那我们平常说的交叉编译版本的gcc，比如arm-linux-gcc，实际上指代了包含一系列交叉编译版本的交叉工具链（arm-linux-gcc，arm-linux-ld，arm-linux-as等等）

### 3.3 交叉编译器

#### 命名规则

在嵌入式开发用到交叉编译器的时候，常看到这样的名字：

```bash
arm-xscale-linux-gnueabi-gcc
arm-liunx-gnu-gcc
```

其对应的交叉编译器前缀为：

```bash
arm-xscale-linux-gnueabi-
arm-liunx-gnu-
```

命名规则为：$arch-vendor-kernel-system$，各部分解释如下

**arch**：系统架构，表示交叉编译器是用于那个目标系统架构的，哪个平台的，即用此交叉编译器编译出来的程序运行在哪种CPU上，可能是arm，x86，mips等

**vendor**：即生产厂家，提供商，表示谁做的这个交叉编译器，一般有这么几种情况，一是写成开发板的名字，比如cortex_a9；二是写成个人作者自己的名字；三是厂家的名字。

**kernel**：即用此交叉编译器编译出来的程序所运行的目标系统，主要有两种：一种是*Linux*，表示有OS（主要是Linux）的环境；另一种是*bare-metal*，表示无操作系统的环境，比如编译u-boot，运行时还没OS，比如一些跑马灯之类的小程序。

**system**：表示交叉编译器所选择的库函数和目标系统，最常见的值有gnu，gnueabi，uclibcgnueabi等。

- gnu：表示用的是glibc的意思

- eabi：embedded application binary interface，应用程序二进制接口，作用是使得程序的二进制（级别）的兼容

- uclibc：C库的一种，专门为嵌入式环境开发而编写的一个自由软件包，可以提供绝大多数标准C库的函数支持

一个简单通俗的解释为：gnu      等价于    glibc+oabi，gnueabi  等价于    glibc+eabi，uclibc    等价于    uclibc+oabi（待确认）

#### 如何获得

1）在网上寻找并下载别人已经编译好的交叉编译器

2）购买开发板时厂家会直接提供相应的交叉编译器

3）自己动手从头开始制作一个交叉编译器

4）借助一些工具来制作交叉编译器

#### 制作工具

1. [crosstool-NG](http://crosstool-ng.org/)：详细的可以看看这篇[crosstool-ng详解](https://www.crifan.com/files/doc/docbook/crosstool_ng/release/html/crosstool_ng.html)
2. [Buildroot](http://www.buildroot.net/)不仅能制作交叉工具链，而且还可以制作根文件系统rootfs。而且还支持同时编译对应的Linux内核和Uboot。
3. [crosstool](http://kegel.com/crosstool/)现在用的最多的是0.43的版本：
4. [Embedded Linux Development Kit (ELDK)](http://www.denx.de/wiki/DULG/ELDK)：也是和交叉编译相关的，提供编译好的东西供使用。
5. [OpenEmbedded](http://www.openembedded.org/wiki/Main_Page)的[BitBake](http://en.wikipedia.org/wiki/BitBake)：OpenEmbedded是一个创建嵌入式Linux的整套框架，其中包括了制作对应的交叉编译器的工具，叫做BitBake。OpenEmbedded简称OE。
6. Crossdev
7. [OSELAS.Toolchain()](http://www.pengutronix.de/oselas/toolchain/index_en.html)

---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2018/migration-related-knowledge/  

