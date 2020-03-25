---
title: 树莓派安装quorum节点
date: 2019-09-09
tags: [树莓派, 科研记录, 区块链]
categories: [研究生的区块链学习之路]
typora-root-url: ..\..\..\static
---

在熟悉了树莓派并和虚拟机顺利组网以后，首要面临的事情就是在树莓派中安装quorum节点，由于暂时不使用隐私保护功能，不安装隐私管理器`Tessera`或`Constellation`。需要安装的只包括：

- quorum的客户端geth
- Istanbul BFT共识配置工具istanbul-tool

### 交叉编译（2019.09.09）

[quorum项目](https://github.com/jpmorganchase/quorum)没有提供可用于arm架构的二进制包，只能自己编译。然而，在树莓派中直接进行编译存在两个问题

1. 会占用很多不必要的空间，quorum项目文件编译后大小在360M以上
2. 编译istanbul-tool依赖于golang的某些包，需要访问google，很多ip无法访问。

因此，最终选择了在PC中进行交叉编译，幸运的是，由于quorum的源项目ethereum可以交叉编译，quorum继承了交叉编译的功能。文档仍然需要看[Cross compiling Ethereum](https://github.com/ethereum/go-ethereum/wiki/Cross-compiling-Ethereum)。交叉编译依赖于名为`xgo`的包，而这个包依赖于Docker和Go，因此，交叉编译之前需要先安装它们。

*注：交叉编译在Ubuntu18.04系统下进行。*

安装golang

```bash
$ sudo snap install go --classic
$ go version
```

[安装docker(使用脚本)](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh

<output truncated>
# 想在非root用户下运行，需要将用户添加到docker group。执行如下命令
$ sudo usermod -aG docker your-user
```

下载quorum

```bash
$ git clone https://github.com/jpmorganchase/quorum.git
```

执行交叉编译

```bash
$ cd quorum
$ make geth-linux-arm-7
$ cd build/bin
# 在该目录下可以找到编译后的geth文件
```

心态爆炸，交叉编译后的geth在树莓派中无法执行，Ubuntu18.04下原本编译完直接放到/usr/local/bin下面即可使用，raspbian中当我放到同样的目录下不起作用，也不知道是交叉编译失败了还是raspbian系统不支持。考虑到raspbian基于Debian，现在不知道Debian应该把可执行文件放在哪里，网上找了很久没找到相关资料。(交叉编译其实可以，直接跳到文章最后可看到方法)

### Ubuntu mate（09.10 am）

quorum的issue中有个项目组的[回答](https://github.com/jpmorganchase/quorum/issues/661)，其中说quorum运行在树莓派中是肯定可以的，这一点终于可以放心，还推荐用Ubuntu，那就试试。

> “Yes. Whilst I haven't tried it, I'm aware that folks have done this and you can find articles on the internet describing how to do it for Ethereum (Quorum will be the same). My suggestion would be to install Ubuntu on the Rasberry and follow the normal steps for building Quorum.”

[树莓派官网](https://www.raspberrypi.org/downloads/)提供的Ubuntu可用镜像有三种：Ubuntu Mate，Ubuntu Core，Ubuntu Server。看到Ubuntu Mate的种种特性，我动心了，看起来好像是专门定制的。

[Ubuntu Mate说明及下载](https://ubuntu-mate.org/raspberry-pi/)，选择的镜像是

```
Raspberry Pi(recommended)
For aarch32(ARMv7)computers,like:
- Raspberry Pi Model B 2
- Raspberry Pi Model B 3
- Raspberry Pi Model B 3+
```

下载，镜像写入，根目录预先建立`ssh`和`wpa_supplicant.conf`文件，插入树莓派，启动运行，扫描不到ip，看起来WiFi没法自动联网。通过网线接到PC上共享网络，ssh访问被拒，接到路由器上一样不行。找资料，关于Ubuntu mate的资料比较少，最终在[官方下载页-Additional feature](https://ubuntu-mate.org/raspberry-pi/)找到一个对特性的说明，称Ubuntu mate没有像raspbian的pi账户一样预定义的用户账户，所有的配置需要在第一次启动时手动完成，ssh预先也没有安装，需要启动后自己安装`openssh-server`并启用。完了，彻底崩溃，本来没有用户账户就无法登录，连ssh都没有，第一次必须得用屏幕了。屏幕，我没有。。。

考虑到笔记本电脑上有个HDMI接口，买线总比买屏幕便宜，跑到商店买了根双头HDMI线。回来一试，没用，网上说是因为笔记本的HDMI只能输出信号，没法输入，因此不能作为HDMI屏幕使用。转眼又看到了VGA接口，这个怎么样，结果一查，HDMI转VGA也没用，笔记本的VGA同样只有输出功能，平板，手机全都不行，不能作为显示设备，最多只能用ssh连接。

台式机的显示器总行了吧，资料上说要自带电源，怕烧坏树莓派。没事，台式的显示器本来就接电源线。又去店里换一个HDMI转VGA的线，是店里唯一的线，结果是坏的，把线接到树莓派上没有反应，提示`请检查线缆`而不是`无信号输入`，拿笔记本试了一下，果然不行，完全检测不到第二屏幕。换！店里没线了怎么办，本来想换HDMI转DVI的，因为显示器后面还有个DVI接口，但店员小哥不推荐，说是用DVI的少，最后拿了HDMI转VGA母口的线，又多买了一根双头VGA线，亏到爆。

不过，总算好使了。

当在显示器上看到Ubuntu mate的界面时我是激动的，太不容易了。初始配置之后还需要进行系统安装，怪不得没法直接进入。但是路由器的WiFi接入不了，或者连接后没法上网，完全没有头绪，只好先用手机开了热点，这倒是没问题。

把之前交叉编译的`geth`文件拷贝到了Ubuntu mate，放到`/usr/local/bin`目录下，运行`geth version`测试，倒是可以了，可惜屏幕打印的文本乱码。重新启动了一下树莓派，结果无限循环启动。初始界面提示如下

```
Driver 'sdhost-bcm2835' already registered, aborting...
```

论坛上也有人遇到了这个[问题](https://ubuntu-mate.community/t/raspberry-pi-3-model-b-plus-ubuntu-mate-installation-error-driver-sdhost-bcm2835-already-registered-aborting/19300)，但从去年11月到今年5月，回帖的所有人都遇到同样的问题而没有办法解决，我已经放弃了。

### Raspbian下自编译quorum(09.10 pm)

树莓派上编译使用quorum的人不多，但编译ethereum的人绝对不少，现在想起来，终于意识到一件事，大部分人还是在raspbian系统下编译使用的，既然ethereum的`geth`客户端可以，quorum没道理不行。有可能不是系统的问题，因为raspbian和ubuntu其实都属于基于Debian的发行版，那就是交叉编译问题了。找不到哪里出的错，干脆直接在Raspbian下编译一次quorum吧，空间占用多一点就多一点，还是足够的，唯一的问题只有翻墙，但这是没办法的事情，而且`geth`的编译暂时还不需要，`istanbul-tool`才需要。

> 注：其实raspbian下翻墙试过了，我有surfshark的账号，官方也给了步骤，[How to set up Surfshark VPN on Raspberry Pi](https://support.surfshark.com/hc/en-us/articles/360013425373-How-to-set-up-Surfshark-VPN-on-Raspberry-Pi)，但最后一步连接总是出错，错误提示为
>
> TLS Error: TLS handshake failed
>
> 网上关于这个Error的问题不少，但都没起作用，就放弃了。

重新写入了之前备份的raspbian镜像（备份真的很有用，能省好多事儿）。启动树莓派，使用预定义的静态ip登录，安装go，下载github上的quorum项目(主要是这里直接下载比`git clone`快多了)。执行编译，注意使用`sudo make all`，因为编译需要分配存储空间，不给权限过不了。

树莓派卡死了。。。

重启了一次，第二次又卡死了，看来不是意外，应该是编译出了问题，果然，等了很久后，编译退出，系统正常了，但出现了错误提示，是一个存储问题。

```
 running gcc failed: fork/exec /usr/bin/gcc: cannot allocate memory
```

Ethereum的论坛有人在编译时遇到了同样的问题，[Installing geth on Raspberry Pi 3 - cannot allocate memory error](https://ethereum.stackexchange.com/questions/12222/installing-geth-on-raspberry-pi-3-cannot-allocate-memory-error)，回帖提到是因为编译所需的内存不够的缘故，建议杀掉内存占用大而且不用的进程。使用`free -h`查看

```
              total        used        free      shared  buff/cache   available
Mem:          926Mi       119Mi       575Mi       7.0Mi       231Mi       744Mi
Swap:          99Mi          0B        99Mi
```

不算少啊，700多M呢，再用`top`命令看进程，并按`M`键按内存占用排序，发现杀哪个进程都不合适。回帖中还有人提到可以调整交换空间大小，就是第二行的Swap，树莓派默认100M，可以调大点，问题的说明及解决方案见[How to set up swap space](https://raspberrypi.stackexchange.com/questions/70/how-to-set-up-swap-space)。

树莓派使用`dphys-swapfile`文件定义交换空间大小，打开配置文件

```
$ sudo nano /etc/dphys-swapfile
```

启用内容只有一行

```
CONF_SWAPSIZE=100
```

代表默认100M交换空间大小，把数值改成合适的内容，我直接改了1024。然后重新启用新的配置文件。

```
$ sudo /etc/init.d/dphys-swapfile restart
```

完成后再用`free -h`命令查看交换空间大小就变了，此时重新编译，编译速度会大大加快，等待一段时间后，编译顺利执行完毕。将编译得到的文件复制到/usr/local/bin目录下。

```bash
$ sudo cp build/bin/geth /usr/local/bin
$ sudo cp build/bin/bootnode /usr/local/bin
# 验证
$ geth version
```

成功。

### istantul-tool(09.11 am)

istantul-tool的编译毫无办法，编译过程要访问google，有些ip无法访问，但既然这里编译没有指明针对arm架构，明天把虚拟机编译好的文件拿过来试试。

Linux编译得到的istanbul文件在树莓派中无法执行，错误提示为

```
-bash: ./istanbul: cannot execute binary file: Exec format error
```

使用`file istanbul`命令查看文件信息

```
istanbul: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64                                                                                                                        .so.2, for GNU/Linux 3.2.0, Go BuildID=Whu77pcg5_4qdJyzC6lH/RiHbDbxGfs3BFqYFYhdk/Uvgfkwy9en1ShuGpCcPB/qCr7Qg3bewybrm4                                                                                                                        vmE3B, BuildID[sha1]=588353ce35513ef4a2d9695f458a338e226093b1, not stripped
```

x86-64的，看来还是架构相关，项目本身没有提供对arm的编译功能，没有办法了。不过有可能不需要在树莓派中运行，我们只需要在作为主节点的虚拟机利用它生成各节点数据，然后拷贝到树莓派中就行。

### 运行geth文件（09.11 am）

直接拷贝编译的geth文件到另一个树莓派，并使用`cp`命令复制到/usr/local/bin目录无法执行，提示

```
-bash: /usr/local/bin/geth: Permission denied
```

是因为没有执行权限，使用`chmod`命令授予权限即可顺利执行

```bash
$ cd quorum
$ sudo cp build/bin/geth /usr/local/bin
$ sudo chmod +x /usr/local/bin/geth
$ geth version
WARN [09-11|03:13:38.840] Sanitizing cache to Go's GC limits       provided=1024 updated=308
Geth
Version: 1.8.18-stable
Quorum Version: 2.2.5
Architecture: arm
Protocol Versions: [63 62]
Network Id: 1337
Go Version: go1.11.6
Operating System: linux
GOPATH=
GOROOT=/usr/lib/go-1.11
```

这样看来，交叉编译的结果也不是因为系统不支持，应该也是没有执行权限，下面是使用geth version命令测试交叉编译的geth的结果。

```bash
WARN [09-11|03:27:52.671] Sanitizing cache to Go's GC limits       provided=1024                                      updated=308
Geth
Version: 1.8.18-stable
Git Commit: 7e87e403407fcb3b3c417739eef2fe1dae923add
Quorum Version: 2.2.5
Architecture: arm
Protocol Versions: [63 62]
Network Id: 1337
Go Version: go1.12
Operating System: linux
GOPATH=
GOROOT=/usr/local/go
```

走了好多弯路。。。原来一开始的结果就可以。

