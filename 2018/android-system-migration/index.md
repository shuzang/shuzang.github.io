# 系统移植3-Android系统移植


采用Android6.0（棉花糖），编译环境为Ubuntu14.04和opoenjdk-7-jdk

## 1. 获取源码

首先参阅Android网站[source.android.com/source/initializing.html](http://source.android.com/source/initializing.html)安装依赖

除Android网站要求的软件包，还需要安装如下软件包：

```bash
$ sudo apt-get install uuid uuid-dev
$ sudo apt-get install zlib1g-dev liblz-dev
$ sudo apt-get install liblzo2-2 liblzo2-dev
$ sudo apt-get install lzop
$ sudo apt-get install git-core curl
$ sudo apt-get install u-boot-tools
$ sudo apt-get install mtd-utils
$ sudo apt-get install android-tools-fsutils
$ sudo apt-get install openjdk-7-jdK
```

获取安卓源码

```bash
$ cd ~
$ mkdir myandroid
$ mkdir bin
$ cd myandroid
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
$ ~/bin/repo init -u https://android.googlesource.com/platform/manifest -b android-6.0.1_r22
$ ~/bin/repo sync # This command loads most needed repos. Therefore, it can take several hours to load.
```

获取内核源码

```bash
$ cd ~/myandroid
$ git clone git://git.freescale.com/imx/linux-2.6-imx.git kernel_imx
# the kernel repo is large. Therefore, this process can take a while.
$ cd kernel_imx
$ git checkout m6.0.1_2.1.0-ga
```

获取u-boot源码

```bash
$ cd ~/myandroid/bootable
$ mkdir bootloader
$ cd bootloader
$ git clone git://git.freescale.com/imx/uboot-imx.git uboot-imx
$ cd uboot-imx
$ git checkout m6.0.1_2.1.0-ga
```

为标准Android源码包patch

## 2. 编译

前面步骤执行完后，执行下列命令开始编译

```bash
$ cd ~/myandroid
$ source build/envsetup.sh
$ lunch sabresd_6dq-user
$ make 2>&1 | tee build-log.txt
```

make命令执行完后，默认在myandroid/out/target/product/sabresd_6dq中生成输出，我们使用boot-imx6dl.img，u-boot-imx6dl.imx和system.img文件。此时得到的system.img文件的格式并不是我们想要的，要转换成raw格式

System.img经常以两种格式出现：raw和sparse。

一种是raw ext4 image，即raw image，使用file命令可查看它：

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427163333049.png)

其特点是完整的ext4分区镜像（包含很多全零的无效填充区），可以直接使用mount进行挂载，因此比较大（一般1G左右）。

另一种是sparse ext4 image，即simg，使用file命令查看它：

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427163351924.png)

就是说是一个非常普通的dat文件。由于它将raw ext4进行稀疏描述，因此尺寸比较小（没有全零的无效填充区，一般在300到500M之间）。

我们进行的编译会默认的生成sparse格式的system.img，因此需要进行转换，而Android本身提供了源代码工具进行两者的转换，位于

```bash
system/core/libsparse/simg2img.c // 将sparse image转换为raw image； 
system/core/libsparse/img2simg.c // 将raw image转换为sparse image
```

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427163428279.png)

转换的命令如下

```bash
$ simg2img <sparse_image_files> <raw_image_file>
# 使用示例
$ simg2img system.img system.raw.img
```

## 3. 分区及烧录

分区与格式化

```bash
$ sudo ./fsl-sdcard-partition.sh -f imx6dl /dev/sdX
# fsl-sdcard-partition.sh为分区脚本，
$ sudo mkfs.fat -n BOOT /dev/sdX1
```

烧录uboot

```bash
$ sudo dd if=u-boot-imx6dl.imx of=/dev/sdX bs=512 seek=2;sync
```

烧录内核

```bash
$ cp boot-imx6dl.img /media/登录名/BOOT/boot-imx6dl.img
```

烧录系统

```bash
$ sudo dd if=system.raw.img of=/dev/sdX5;sync
```

连线，上电，进行串口调试，设置参数如下

```bash
$ setenv bootargs 'console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,bpp=32,if=RGB666 video=mxcfb1:dev=ldb,bpp=32,if=RGB666 video=mxcfb2:off video=mxcfb3:off vmalloc=320M androidboot.console=ttymxc0 consoleblank=0 androidboot.hardware=freescale cma=384M'
$ setenv bootcmd 'fatload mmc 1:1 12000000 boot.img;boota 12000000'
$ saveenv
```

此时系统应可以启动成功

## 4. 使用界面

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427163821116.png)

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427163831726.png)

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_image-20200427163844988.png)

## 5. Android 项目源码管理

做 Android 移植需要了解的

AOSP："Android Open-Source Project"，中文意为"Android 开放源代码项目"。

AOSP项目由不同的子项目组成,为了方便进行管理,Google采用Git对AOSP项目进行多仓库管理

多仓库项目管理的原理是这样的（该部分学习甚至图片都是借用网上某文章）：

我们有个非常庞大的项目Pre,该项目由很多个子项目R1,R2,...Rn等组成,为了方便管理和协同开发,我们为每个子项目创立自己的仓库,整个项目的结构如下:

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_clip_image002.jpg)

将一个项目Pre进行分库后会遇到这么一个问题:如果我们想要创建Pre分支来做feature开发,这就意味着,我们需要到每个子项目中分别创建对应的分支,这个过程如果纯粹靠手工做,那简直是个灾难,利索当然我们会想写个自动化处理程序(我们假设这个工具叫做RepoUtil)来帮助我们解决这个问题.这个RepoUtil也会有版本管理之类的需求,因此我们也用Git对其管理,并为其创建对应的仓库.此时整个项目的结构如下:

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_clip_image004.jpg)

这里RepoUtil知道整个项目Pre下的每个子项目(即维护子项目的列表),同时需要提供对这些子项目的管理功能,比如统一创建分支等.但是从"单一职责"角度来看,RepoUitl这个工具的功能过于复杂,我们完全可以将维护子项目列表这个功能抽取出来作为一个新项目sub_projects,因为子项目也会变化,因此,为其创建对应的仓库,并用Git管理,这样的化,RepoUtil只需要通过简单的对ub_projects进行依赖即可,此时整个项目的结构如下:

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180411_clip_image006.jpg)

AOSP项目结构和我上文的描述非常类似.repo工具对应RepoUtil,mainfest对应sub_projects.

总结一下:repo就是这么一种工具,由一系列python脚本组成,通过调用Git命令实现对AOSP项目的管理.
