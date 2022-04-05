---
title: 系统移植2-Debian系统的移植
date: 2018-04-11
lastmod: 2020-04-27
tags: [linux]
categories: [爱编程爱技术的孩子]
autoCollapseToc: true
slug: Migration of Debian system 
---

Debian 系统的移植总分四部分：u-boot的编译与烧录，Linux内核的编译与烧录，Debian 基本根文件系统的制作、配置与烧录，开发板设置。

## 1. u-boot的编译

对应于EMSYM的blurr开发板的u-boot项目使用GitHub进行开源维护，[下载地址](https://github.com/EMSYM/U-boot)，我的编译环境为Ubuntu16.04 LTS系统

### 1.1 下载源码

方法一：在链接打开后的项目界面依次选择clone or download—>Download ZIP，将源码下载到PC中相应的文件夹（记得解压.....）

方法二：采用git命令（须事先安装git）

```bash
$ git clone https://github.com/EMSYM/U-boot.git
```

注：该链接在选中clone or download后可看到

### 1.2 分支切换

在下载好的u-boot项目目录下打开虚拟终端，创建并切换分支

```bash
$ git checkout -b v4.1 origin/blurr-4.1.15
```

![创建并切换分支](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_d1bb18bcb403.png)

### 1.3 安装依赖

gcc-arm-linux-gnueabi是我们用到的交叉编译器

```bash
$ sudo apt install gcc-arm-linux-gnueabi
$ sudo apt-get install build-essential gcc
```

注：Ubuntu缺省情况下，并没有提供C/C++的编译环境，因此还需要手动安装。如果单独安装gcc以及g++比较麻烦，幸运的是，为了能够编译Ubuntu的内核，Ubuntu提供了一个build-essential软件包。因为依赖关系的问题，安装了该软件包，编译c/c++所需要的软件包也都会被安装。因此如果想在Ubuntu中编译c/c++程序，只需要安装该软件包就可以了。

![build-essential的依赖关系](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_8135c2d310f2.png)

### 1.4 开始编译

指定核心类型

```bash
$ ARCH=arm
```

生成配置文件（用到Makefile，参阅[GNU make中文手册](https://hacker-yhj.github.io/resources/gun_make.pdf)）

```bash
$ make mx6dl_blurr_defconfig
```

![生成配置文件](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_c22b1be42612.png)

指定交叉编译前缀，编译u-boot

```bash
$ make CROSS_COMPILE=arm-linux-gnueabi- 
```

![编译通过](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_967c5a8d8bba.png)

执行完上述步骤后，编译即可成功

## 2. Linux内核编译

同样，对应于blurr开发板的Linux内核项目也在GitHub上开源维护，[项目地址](https://github.com/EMSYM/linux)

![github上的工程](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_fd26ed77a3ab.png)

### 2.1 源码下载

下载源码的手段与u-boot类似。这里要提醒的是，最好直接Download ZIP，不要clone，因为下载ZIP只有100多M，clone会有一个多G。而这两种手段得到的源码对之后的编译没有影响。（记得解压zip文件）

### 2.2 安装压缩工具lzop

```bash
$ sudo apt-get install lzop
```

lzop是最后生成时要用到的一个压缩工具，当没有安装此工具就开始编译，过程中会出现一个lozp：not found的错误

![lozp：not fountd](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_5665b6610702.png)

### 2.3 编译

进入下载好的Linux项目主目录，逐步执行如下命令

```bash
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabi-
$ make blurr_defconfig
$ make
```

前两句的意思是指定芯片的架构以及交叉编译器前缀，然后就开始编译了（之前编译u-boot时已装好交叉编译器），可参考 [export命令的介绍](http://blog.csdn.net/wl_fln/article/details/7258294)

![四条指令](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_57171204afb8.png)

会有一些warning，但不碍事

![warning](https://picped-1301226557.cos.ap-beijing.myqcloud.com/ad7eeaabb67f.png)

要等很久.....

![编译成功](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_cc3a1d8404d3.png)

完成如上步骤后即可编译完成，我们需要编译得到的zImage文件和imx6dl-blurr.dtb文件位置如下：

- zImage：linux-blurr 4.1.15—>arch—>arm—>boot

- imx6dl-blurr.dtb：linux-blurr 4.1.15—>arch—>arm—>boot—>dts

zImage位置如下图

![zImage](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_beb1c3962a3d.png)

imx6dl-blurr.dtb直接从上面目录里的dts点进去

![imx6dl-blurr.dtb](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_6488bacf69de.png)

## 3. Debian 根文件系统的制作与配置

该部分内容主要参考官方文档 [EmDebian CrossDebootstrap](https://wiki.debian.org/EmDebian/CrossDebootstrap)，方法为QEMU/debootstrap的方法。使用debootstrap工具完成根文件系统的制作，使用QEMU模拟器完成配置工作。

### 3.1 基本说明

制作根文件系统的工具常用Debootstrap和Multistrap，制作流程大概分为四个步骤：

1、从源里下载需要的.deb软件包

2、解压它们到相应的目标目录

3、用chroot命令修改该目标目录为根目录

4、配置脚本，完成安装

通常，使用debootstrap、cdebootstrap或multistrap完成第一二阶段的工作，QEMU完成第三四阶段的工作。

### 3.2 工具软件安装

使用如下的命令安装所需要工具软件

```bash
$ sudo apt-get install binfmt-support qemu qemu-user-static debootstrap
```

debootstrap是根文件系统制作工具，qemu是模拟器，是为了在宿主机上模拟开发板的环境

接下来的步骤需要以root身份执行，因为debootstrap的工作以及chroot到建好的新系统的目录都需要root权限

### 3.3 根文件系统制作

首先选择要引导的目标架构和Debian版本（例如stable、testing或sid）,我们选择开发板对应的arm架构，版本选择Debian9.1,叫做stretch。

创建一个目录文件，制作好的根文件系统将放在这个目录下。需要提示以后所有对根文件系统的修改都局限在这个目录里，不会影响到宿主机，所以不用担心搞毁你的系统。为了介绍清楚，我们采用“debian_armhf_stretch”作为目录名，但它其实就是rootfs，如果你这样命名的话

```bash
$ sudo mkdir debian_armhf_stretch
```

制作根文件系统，需要运行debootstrap

```bash
$ sudo debootstrap --foreign --arch armhf stretch debian_armhf_stretch http://ftp.debian.org/debian/
```

其中`debian_armhf_stretch`是创建的目录，`armhf`是目标架构，`http://ftp.debian.org/debian`是Debian镜像，必需的.deb包将从这里下载。可以随意选择自己喜欢的镜像，只要它有我们要用于的目标架构。[可用的Debian镜像列表](http://www.debian.org/mirror/list)，[debootstrap命令说明](https://linux.cn/man/man8/debootstrap.8.html)

会运行两次，第一次是从网上下载，第二次就是在debian_armhf_stretch目录下生成bin、sbin这些Linux文件系统目录了。

### 3.4 QEMU 配置

根文件系统已经创建完成。默认情况下，debootstrap创建的是一个非常小的系统，所以可能需要扩展一下，这个放在后面的配置新系统。

先复制“qemu-arm-static” 到刚构建的根文件系统中。为了能chroot到目标文件系统，针对目标CPU的qemu模拟器需要从内部访问

```bash
$ sudo cp /usr/bin/qemu-arm-static  debian_armhf_stretch/usr/bin
```

接下来运行debootstrap的第二个阶段来解压步骤3中安装的所有软件包

```bash
$ sudo DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot debian_armhf_stretch debootstrap/debootstrap --second-stage
```

该命令意思是设置一些环境变量,然后切换根目录到debian_armhf_stretch（这个操作是chroot做的，这个命令很有意思），执行目录debian_armhf_stretch/debootstrap下的命令: debootstrap --second-stage。终端上最后会打印 `I: Base system installed successfully.`，就说明成功了。

切换到qemu

```bash
$sudo chroot debian_armhf_stretch
```

不过如果这样的话之后的提示都是英文的，想显示中文需要把上面的命令这样输入

```bash
$ sudo LANG=C.UTF-8 chroot debian_armhf_stretch
```

## 4. 新系统配置

刚刚创建的新系统需要一些简单的调整以便于你运用它做一些特殊的工作。下面的步骤需要以root权限执行

1. 手动在/etc/apt/sources.list里面添加如下条目

   ```bash
   deb http://ftp.debian.org/debian/ stretch main contrib non-free
   deb-src http://ftp.debian.org/debian/ stretch main contrib non-free
   ```

   源的选择是随便的，只要它支持目标平台就行，不过国内[中科大的源](https://lug.ustc.edu.cn/wiki/mirrors/help/debian)好用一点，这时候就可以像往常一样使用apt-get install 命令安装其它软件包了。

2. 安装xorg和KDE，KDE新版本叫plasma

   ```bash
   $ sudo apt-get install xorg
   $ sudo apt-get install plasma-desktop
   ```

   ![plasma-desktop](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_ff04dfdcef82.png)

   这句执行也是真的久，不要着急，中间可能会自动进入一些配置界面，根据选项自己选就行，如果乱码的话可以选第一个，其实上面设置成中文就没乱码了。

3. 安装终端konsole，安装浏览器qupzilla（这是因为Debian9以下不支持chrome），安装文件管理器dolphin，网络管理器plasma-nm

   ```bash
   $ sudo apt-get install konsole
   $ sudo apt-get install qupzilla
   $ sudo apt-get install dolphin
   $ sudo apt-get install plasma-nm
   ```

4. 打开etc文件夹中的fstab文件，在末尾添加

   ```bash
   /dev/sda / ext4 defaults 0 1
   ```

5. root用户下输入命令

   ```bash
   chown root:root /usr/bin/sudo
   chmod 4755 /usr/bin
   ```

## 5. 烧录工作

u-boot，Linux内核和根文件系统都是需要烧录的，下面介绍如何进行这部分工作。首先将SD卡格式化，准备好要烧录的文件

- u-boot：u-boot.imx
- 内核：zImage和imx6dl-blurr.dtb
- 根文件系统：debian_armhf_stretch

### 5.1 设置SD卡

SD卡需要创建2个分区，并且在第一个分区之前预留一段空间用于保存u-boot.imx；然后第一个分区设置为fat格式，用于保存内核编译成的两个文件；第二个分区设置为ext4格式，用于保存根文件系统。

#### 格式化

在分区之前，先将SD卡格式化（可能Windows下格式化方便一点），首先在Linux系统下插入SD卡，查看挂载位置，使用[fdisk](http://www.linuxidc.com/Linux/2012-06/61872.htm)命令

```bash
$ sudo fdisk -l
```

这个命令作用为查看磁盘使用情况的，最后那个即为我们新挂载的SD卡

![查看挂载地址](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_e2b342bb2c91.png)

从结果输出中可以看到SD卡，挂载位置是/dev/sdb，输入命令

```bash
$ sudo fdisk /dev/sdb
```

该命令含义为进入分割硬盘模式，这个命令执行结束就可以操作分区了。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_0b65803507c5.png)

#### 删除原有分区

输入m可以获取帮助，显示所有可用命令。先用d 命令删除原来的分区，提示Selected partition，在后面输入要删除的分区号，回车，分区即可删除成功

![删除分区](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427153230942.png)

输入命令p查看此时的分区信息，已变成一整块的磁盘未分区，接下来开始分区

#### 重新分区

n命令新建分区，第一次输入p分割为主分区，分区号为1，分区的起始位置和结束位置自己算好填上去，注意分区必以sector为单位，一个sector 为512 bytes，所以不要输不是512整数倍的数。

n命令新建分区，第二次输入e分割分区，分区号为2，起始位置接着上一个分区的结束位置。

![分区](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_bd06ff3756c8.png)

p命令查看分区信息，看是否符合预想。w命令保存退出。弹出重新插入SD卡或者重新启动Ubuntu系统，之后再次查看分区信息，这是为了更新分区表给内核，否则看不到分好的区。

![再次查看](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_b3f7856930ad.png)

格式化新建的分区

```bash
$ sudo mkfs.ext4 /dev/sdb2
```

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_81816c5f55ca.png)

然后对两个分区格式化

输入t，选择分区1，将分区1格式化为fat，这里输入6为FAT16，c为FAT32

输入t，选择分区2，将分区2格式化为ext4格式，输入83即可

注：这里输入83格式化结束最后查看分区其实显示的是83 Linux，这就已经是ext4格式了，不要觉得自己错了，因为Linux的默认格式就是ext4

w命令保存

![格式化分区](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_823a55d4bfef.png)

### 5.2 烧录

#### u-boot

执行如下命令，把u-boot弄到SD卡开头预留的那段空间

```bash
$ sudo dd if=u-boot.imx of=/dev/sdb  bs=512 seek=2 ;sync
```

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427154449772.png)

[dd命令的解释](http://blog.csdn.net/pugu12/article/details/47047341)，至于为什么要seek=2，跳过头两个块我现在也不懂

#### 内核

把编译好的内核的两个文件zImage和imx6dl-blurr.dtb复制到第一个分区，可以直接鼠标操作

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427154551645.png)

#### 根文件系统

先 fdisk -l 查看第二个分区的位置，是/dev/sdb2。把第二个分区挂载到/mnt下，这里执行成功后把文件放到/mnt下就相当于放到了U盘第二个分区

```bash
$ sudo mount /dev/sdb2 /mnt
```

然后就在rootfs的当前目录用cp命令复制，意思是将根文件系统的所有内容复制到挂载的目录

```bash
$ sudo cp -rf rootfs/.  /mnt
```

执行完有必要的话取消挂载

```bash
$ sudo umount /mnt
```

## 6. 开发板设置

确保zImage文件和imx6dl-blurr.dtb文件都已放入SD卡第一个分区，也就是FAT格式的分区中；文件系统解压到SD卡第二个分区，也就是ext4格式的分区中

注：若之前已经进行到屏幕上出现两只企鹅，此时只需要将解压好文件系统的SD卡插入开发板，上电即可进入命令行界面，若无法进入命令行界面则继续进行下列步骤，

1. 靠近 HDMI接口的地方有个开关，拨动开关到boot一端，设置八位红色开关为0100001

   ![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427155128647.png)

2. 将SD卡插入SD3插槽

3. 下好窗口调试软件putty（windows或Linux下都可），其它串口调试软件也行，用数据线连接开发板和PC（调试），开发板与屏幕，开发板与电源，开发板与键盘

4. 打开设备管理器，找到这个设备，这时候还没有驱动，所以右键更新驱动程序，在网络上查找，第一次可能一直是寻找中，那就关掉重新打开。直到驱动装好，从设备管理器里就可以看到相应的COM口。

5. 打开putty，选择serial，将COM1改成上面设备管理器里看到的COM号，speed改为115200，然后最下面点open

   ![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427155359810.png)

6. 此时看到串口输出（若没有，请按开发板HDMI口旁边的RESET按钮），按回车进入调试模式，输入如下命令命令

   ```bash
   setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk2p2 init=/sbin/init'
   saveenv
   #加载镜像
   fatload mmc 1:1 12000000 zImage; #将SD卡分区一的镜像加载到内存地址0x12000000
   fatload mmc 1:1 11000000 imx6dl-blurr.dtb;
   #加载Device Tree文件
   #启动镜像
   bootz 12000000 - 11000000
   ```

   ![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427155557593.png)

开发板上电，如果SD卡中此时只有u-boot和内核，屏幕上会有两只企鹅，

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_stickpicture.png" alt="显示企鹅" style="zoom:33%;" />

如果根文件系统也已经放进去了，那就会出现登录选项，连接键盘与开发板，输入root，这个是拿到的这个根文件系统就是这个名字，之后即可使用键盘输入各种命令与开发板交互。在命令行使用如下命令可以启动 GUI，然后就进入KDE桌面环境了，Debian系统哦

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427160233842.png)

注：开发板获得读写权限的方法如下，如果模拟器中已完成此工作则不需要

```bash
$ mount rw -o remount /
```

## 7. 使用界面

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427160245617.png)

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427160256212.png)

## 8. 参考

1. [debian下为arm开发板创建基于debian或emdebian的根文件系统](http://www.cnblogs.com/qiaoqiao2003/p/3738552.html)

   这篇文章主要是提供一个思路，用工具来制作Debian的根文件系统。

2. [创建基于arm的debian文件系统](http://blog.csdn.net/luoqindong/article/details/42737879)

   主要是因为上面制作的根文件系统直接弄到板子上没网，所以就没法用板子的命令行安装软件，只好借助模拟器在PC上装好一些东西，比如GUI和其它一些软件。这篇文章提供一些模拟器QWMU使用的借鉴

3. [Emdebian](http://www.emdebian.org/emdebian/flavours.html)

   这个就没什么了，主要是区别一下我们用到的嵌入式Debian的源码版本