# 操作系统6-文件管理


本篇介绍文件管理相关的内容。包括硬盘结构、文件和目录的概念，以及一个文件系统涉及的其它知识。

<!--more-->

## 1. 硬盘结构

硬盘是计算机最主要的外部存储设备，尽管常见的存储设备还有光盘、U盘等，但最常用的还是硬盘。目前，我们所见的硬盘主要有机械硬盘和固态硬盘两类，前者使用磁性盘片来存储数据，后者使用闪存颗粒存储数据。

机械硬盘主要的组成包括磁盘片、主轴、磁盘臂、磁头等。如下图所示，硬盘中有大量的磁盘片，所有的盘片通过主轴连接在一起，主轴连接到一个电机，以恒定的速率旋转。每个磁盘片有两面，数据在这两面上持久存储，通过磁头可以读取表面的数据。磁盘转动时，磁头所走过的路径会形成一个圆形，叫做磁道，每个磁盘片都有数百上千的磁道，另外，人们把所有磁盘片的相同磁道称作一个柱面。最后，为了便于读写，人们还将每个磁盘片划分为一些相等大小的扇区，一个扇区 512 字节。不过，操作系统读取硬盘的时候，不会一个个扇区地读取，这样效率太低，而是一次性连续读取多个扇区，即一次性读取一个"块"（block）。这种由多个扇区组成的"块"，是文件存取的最小单位。"块"的大小，最常见的是4KB，即连续八个 sector组成一个 block。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200818_1597710757496.jpg)

固态硬盘不再使用盘片，而是使用某种存储芯片作为存储介质，基本原理是对电荷的存储。固态硬盘在大部分方面都比机械硬盘表现好。固态硬盘已经没有了磁道、柱面等概念，物理单位是一个个的闪存颗粒，每个闪存颗粒由成千上万大小相同的块（Block）组成，块的大小一般为数百 KB 到数 MB，每一个块的内部又分为若干个大小相同的页（Page），页的大小一般为 4KB 或者 8KB。页是基本的读写单位，块是数据擦除的基本单位。

更大关于固态硬盘的知识可以参考 [知乎，详解固态硬盘的有趣知识及其底层原理](https://zhuanlan.zhihu.com/p/114237145)

## 2. 文件与inode

可参考：[阮一峰，理解inode](http://www.ruanyifeng.com/blog/2011/12/inode.html)

文件（file）是一个抽象的概念，最底层就是一串二进制0和1，高一点可以看作一个字节序列，其中每个字节都可以读取或写入。在更高一点的层次，我们用一个叫做 inode 的结构记录关于文件的一些元信息，这个结构的定义可以举个例子

```c
struct stat {
    dev_t st_dev;         // ID of device containing file
    ino_t st_ino;         // inode number
    mode_t st_mode;       // protection
    nlink_t st_nlink;     // number of hard links
    uid_t st_uid;         // user ID of owner
    gid_t st_gid;         // group ID of owner
    dev_t st_rdev;        // device ID(if special file)
    off_t st_size;        // total size, in bytes
    blksize_t st_blksize; // blocksize for filesystem I/O
    blkcnt_t st_blocks;   // number of blocks allocated
    time_t st_atime;      // time of last access
    time_t st_mtime;      // time of last modification
    time_t st_ctime;      // time of last status change
};
```

我们可以看到主要包含的内容有：inode 号、保护信息（即文件权限）、链接数、文件拥有者的用户 ID 和所在的组 ID、文件总字节数、I/O 读取的单个 Block 大小、底层数据块的位置、实际信息等。

使用 `stat` 命令可以读取到这些信息

```bash
$ stat foo
  文件：foo
  大小：6         	块：8          IO 块：4096   普通文件
设备：805h/2053d	Inode：5767595     硬链接：1
权限：(0664/-rw-rw-r--)  Uid：( 1000/ shuzang)   Gid：( 1000/ shuzang)
最近访问：2020-08-22 19:00:39.660722364 -0700
最近更改：2020-08-22 19:00:29.121819131 -0700
最近改动：2020-08-22 19:00:29.121819131 -0700
创建时间：-
```

注意，inode 对底层数据块的引用一般是通过指针完成的，每个指针指向一个磁盘块，为了包含更多的数据，指针可以指向另一个完全由指针组成的块，使用间接指针的方式扩大包含的数据量。通常来讲，inode 中会设置一定数量的直接指针和一个间接指针。

### 2.1 inode大小

inode 本身的存储也占据硬盘空间，所以硬盘至少会有两个区：数据区存放文件数据，inode 区存放所有 inode。不过我们习惯将存放 inode 的区域称作 inode 表。 

由于 inode 的内容不多且比较固定，所以其大小也是固定的， 一般是128字节或256字节。

使用 `df` 命令可以查看每个硬盘分区的 inode 总数和使用情况

```bash
$ df -i
文件系统         Inode 已用(I) 可用(I) 已用(I)% 挂载点
udev            493235     477  492758       1% /dev
tmpfs           500276     938  499338       1% /run
/dev/sda5      6520832  233720 6287112       4% /
tmpfs           500276       1  500275       1% /dev/shm
tmpfs           500276       5  500271       1% /run/lock
tmpfs           500276      18  500258       1% /sys/fs/cgroup
/dev/loop0        4338    4338       0     100% /snap/code/39
/dev/loop2       12796   12796       0     100% /snap/core/9804
/dev/loop3       10775   10775       0     100% /snap/core18/1885
/dev/loop5       10756   10756       0     100% /snap/core18/1880
/dev/loop1       12862   12862       0     100% /snap/core/9665
/dev/loop4       24339   24339       0     100% /snap/gnome-3-34-1804/33
/dev/loop6       24339   24339       0     100% /snap/gnome-3-34-1804/36
/dev/loop8       10206   10206       0     100% /snap/go/6123
/dev/loop12        463     463       0     100% /snap/snapd/8542
/dev/loop9       62342   62342       0     100% /snap/gtk-common-themes/1506
/dev/loop11      15827   15827       0     100% /snap/snap-store/467
/dev/loop13        465     465       0     100% /snap/snapd/8790
/dev/loop10      15827   15827       0     100% /snap/snap-store/433
/dev/sda1            0       0       0        - /boot/efi
tmpfs           500276      89  500187       1% /run/user/1000
/dev/loop14       4303    4303       0     100% /snap/code/40
/dev/loop15      10206   10206       0     100% /snap/go/6274
```

### 2.2 inode号

每个 inode 都有一个 inode 号，它被操作系统用来识别文件。

我们应该清除的是，文件名只是为了方便用户使用，操作系统对文件的操作本质是通过 inode。举个例子，当我们打开文件的时候，系统首先通过文件名找到 inode 号，然后通过 inode 号获取 inode 信息，最后根据 inode 信息找到文件数据所在的 block，读取数据。

通过 inode 号找 inode 的例子如下图，假设 inode 表大小为 20KB，由 80 个 inode 组成，inode 区域从 12KB 开始（即超级块从 0KB 开始，inode 位示图在 4KB 位置，数据位示图在 8KB位置）。要读取 inode 号 32，文件系统首先计算 inode 区域偏移量（32×inode大小，即 8192），将它加上磁盘 inode 表的起始地址（12KB），从而得到希望的 inode 块的正确字节地址：20KB。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200818_epub_30179184_249.jfif)

使用 `ls -i` 命令可以查看文件名对应的 inode 号

```bash
$ ls -i foo
5767595 foo
```

## 3. 目录

目录也是一种文件，但它的内容比较具体，存储的是它所包含的文件的文件名和对应的 inode 号。该内容用抽象数据结构来表达如下

```c
struct dirent {
    char d_name[256];     // filename
    ino_t d_ino;          // inode number
    off_t d_off;          // offset to the next dirent
    unsigned short d_reclen;    // length of this record
    unsigned char d_type;       // type of file
}
```

举个例子，foo 文件的 inode 号为 10，则目录的内容中会有一条 （foo, 10） 的记录。但我们要记得，目录也是一个文件，因此目录也有自己的 inode。

注1：目录的层次结构从根目录（/）开始，由根文件系统产生。

注2：移动文件或重命名文件，只是改变文件名，不改变 inode 号。

注3：通常来说，系统无法从inode号码得知文件名

## 4. 软链接和硬链接

### 4.1 硬链接

一般情况下，文件名和 inode 号之间是一一对应的关系，但我们可以令多个文件名指向同一个 inode 号。这种情况就叫**硬链接**。硬链接的本质是增加了一个对应关系，没有改变 inode 和底层数据，这意味着我们使用不同的文件名访问到的是同一个文件，修改和删除数据也会影响到所有文件名。

Linux 中使用 `ln` 命令建立硬链接。

inode 结构中有一项叫做链接数，每建立一个硬链接，该值就会加一，反过来，删除一个文件名，该值就会减一，只有减到零的情况下，系统才会回收该 inode 号和对应的数据。

系统创建目录时，默认会生成两个目录项："."和".."。前者的inode号码就是当前目录的inode号码，等同于当前目录的"硬链接"；后者的inode号码就是当前目录的父目录的inode号码，等同于父目录的"硬链接"。所以，任何一个目录的"硬链接"总数，总是等于2加上它的子目录总数（含隐藏目录）。

硬链接的缺点是不能创建目录的硬链接，也不能链接到其它文件系统。

### 4.2 软链接

软链接，也叫符号链接，软链接可以为目录创建，也可以链接到其它文件系统。因为软链接的实质类似于指针，加上存在文件 B，为 B 建立软链接 A，实际上是建立了一个新的文件，A 有自己的 inode，只不过 A 的内容是 B 的路径。当我们访问 A 时，会自动导向文件 B。

因此，软链接 A 是依赖于 B 存在的，如果删除了文件 B，打开文件 A 就会提示指向的文件或目录不存在。

使用 `ln -s` 命令创建软链接。

## 5. 文件系统实例

文件系统是操作系统中与管理文件有关的软件和数据，负责文件的建立、删除、读写、修改、复制以及按名存取和存取控制。

我们假设磁盘的基本读写单位是 4KB，每 4KB 为一块，一个磁盘可以分为若干块（假设为 N），这些块的地址从 0 到 N-1，下面的介绍以一个只有 64 块的磁盘为例。

### 5.1 基本结构

首先，磁盘中主要存放的一定是 **用户数据**，我们将存放用户数据的磁盘区域称为数据区域，假设将 64 个块的最后 56 个专门留给它们。

此外，文件系统还必须记录每个文件的基本信息，就是 inode，所有的 inode 存放在一个统一的区域，叫做 **inode 表**。由于 inode 大概 128 或 256 字节，所以不需要预留很大的空间，一个 4KB 的块就可以存放 16 个 inode，这里我们在 64 个块中留了 5 个块来存放 inode。

然后，我们还需要一个结构来记录 inode 和数据块空闲还是已分配。可选的方法很多，比如空闲块链表，将所有的空闲块链接在一起，当需要空闲块时从链头开始摘取并修改头指针，回收空闲块时插入链表尾。但通常使用的是一种简单有效的结构：**位示图**（inode）。位示图的原理是划分一部分空间，将每个比特位对应一个块，如果该位为0，说明对应的块空闲，如果该位为 1，说明对应的块已分配。数据区域和 inode 表通常使用不同的位示图。

最后还有一个结构叫做 **超级块**，包含文件系统的基本信息，比如有多少个 inode 和数据块、inode 表的起始位置、文件系统的类型等。

所以基本的结构有五部分，如下图：超级块、inode 位示图、数据区域位示图、inode表、数据区域。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200818_epub_30179184_248.jfif)

可以通过 `df` 命令查看文件系统，然后使用 `sudo dumpe2fs /dev/dsa*` 查看超级块的信息

```bash
shuzang@ubuntu:~$ df
文件系统           1K-块     已用     可用 已用% 挂载点
udev             1972940        0  1972940    0% /dev
tmpfs             400224     1892   398332    1% /run
/dev/sda5      102168536 10744048 86191592   12% /
...

$ sudo dumpe2fs /dev/sda5
dumpe2fs 1.45.5 (07-Jan-2020)
Filesystem volume name:   <none>
Last mounted on:          /
Filesystem UUID:          45e4318b-0433-4e09-a0ea-48e29ac60801
Filesystem magic number:  0xEF53
Filesystem revision #:    1 (dynamic)
Filesystem features:      has_journal ext_attr resize_inode dir_index filetype needs_recovery extent 64bit flex_bg sparse_super large_file huge_file dir_nlink extra_isize metadata_csum
Filesystem flags:         signed_directory_hash 
Default mount options:    user_xattr acl
Filesystem state:         clean
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              6520832
Block count:              26082560
Reserved block count:     1304128
Free blocks:              22924783
Free inodes:              6286837
First block:              0
Block size:               4096
Fragment size:            4096
Group descriptor size:    64
Reserved GDT blocks:      1024
Blocks per group:         32768
Fragments per group:      32768
Inodes per group:         8192
Inode blocks per group:   512
Flex block group size:    16
Filesystem created:       Tue May 19 03:52:48 2020
Last mount time:          Fri Aug 21 01:35:42 2020
Last write time:          Fri Aug 21 01:35:35 2020
Mount count:              7
Maximum mount count:      -1
Last checked:             Tue May 19 03:52:48 2020
Check interval:           0 (<none>)
Lifetime writes:          26 GB
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)
First inode:              11
Inode size:	          256
...
```

### 5.2 挂载

磁盘通常会划分为不同的分区，每个分区可能使用不同的文件系统，挂载就是将该文件系统粘贴到整个操作系统的文件目录树上，挂载的位置是一个路径，进入该路径就相当于进入了该文件系统。

### 5.3 文件读取

已知文件名，读取文件的过程我们大致已经熟悉，即通过文件名获得 inode 号，通过 inode 号获得 inode 信息，通过 inode 信息读取数据。

但这里还不清楚的一点是如何通过文件名找到 inode。文件系统采取的方式是遍历路径名，所有的遍历都从文件系统的根开始，即 `/` 。文件系统第一次读取根目录的 inode，根目录的 inode 号是众所周知的，一般是 2，因此文件系统就会读入 inode 号为2的块。一旦读入 inode，文件系统就可以查找指向数据块的指针，数据块中包含了根目录的内容。根目录的内容中存放了所有子文件的文件名和 inode 号的对应，所以根据路径可以找到下一级目录或文件的 inode 号，如果是目录，就继续遵循上面的过程进行递归的读取，最后读取得到文件的数据。

最后要注意的是，简单的读取并不会对位示图进行改变，只有创建或删除文件数据时才会改变位示图的内容。
