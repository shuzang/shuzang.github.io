# 计算机网络-ICMP协议


网络层除了主要的 IP 协议外，还有 ICMP（Internet Control Message Protocol） 协议，ping 和 traceroute 都会用到它，本文进行介绍。

<!--more-->

## 1. ICMP

网际控制报文协议 ICMP，目的是更有效的转发 IP 数据报，提高交付成功的机会。通常被主机或路由器用来报告差错情况和提供有关异常情况的报告。

ICMP 虽然是网络层协议，但其报文是作为 IP 数据报的数据部分传输的。ICMP 的报文格式如下

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20181001_epub_655484_191.jpg)

ICMP 报文有两种：差错报告报文和询问报文，报文的前四字节是统一的格式，有三个字段：类型、代码和检验和，后面四个字节与类型有关，随后就是数据部分。下表给出几种常用的报文类型

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20181001_epub_655484_192.jpg)

代码字段是为了进一步区分某种类型的几种不同情况，比如类型12，又可以分为下面两种

| 类型 | 代码 | 描述                                                       |
| ---- | ---- | ---------------------------------------------------------- |
| 12   | 0    | IP header bad (catchall error)——坏的IP首部（包括各种差错） |
| 12   | 1    | Required options missing——缺少必需的选项                   |

校验和字段用来检验整个 ICMP 报文，因为 IP 数据报首部的校验和并不检验 IP 数据报的数据部分，不能保证传输的 ICMP 报文不产生差错。

所有的 ICMP 差错报告报文中的数据字段都具有相同的格式，把收到的需要进行差错报告的 IP 数据报的首部和数据字段的前 8 个字节提取出来作为 ICMP 报文的数据字段，然后添加 ICMP 首部，就构成了 ICMP 差错报告报文。如下图

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20181001_epub_655484_193.jpg)

提取收到的数据报的数据字段前 8 个字节是为了得到运输层的端口号以及运输层报文的发送序号，这些信息对源点通知高层协议是有用的。

常用的 ICMP 询问报文有两种

1. **回送请求和回答**： ICMP回送请求报文是由主机或路由器向一个特定的目的主机发出的询问。收到此报文的主机必须给源主机或路由器发送ICMP回送回答报文。这种询问报文用来测试目的站是否可达以及了解其有关状态。
2. **时间戳请求和回答**：ICMP时间戳请求报文是请某个主机或路由器回答当前的日期和时间。在ICMP时间戳回答报文中有一个32位的字段，其中写入的整数代表从1900年1月1日起到当前时刻一共有多少秒。时间戳请求与回答可用来进行时钟同步和测量时间。

## 2. Traceroute

ICMP 的一个很有用的应用是 traceroute，用来跟踪一个分组从源点到终点的路径，Windows 系统中 命令为 tracert。原理如下：

Traceroute从源主机向目的主机发送一连串的 IP 数据报，数据报中封装的是无法交付的 UDP 用户数据报（使用非法端口）。第一个数据报 P1 的生存时间 TTL 设置为1。当 P1 到达路径上的第一个路由器 R1 时，路由器 R1 先收下它，接着把 TTL 的值减1。由于 TTL 等于零了，R1 就把 P1 丢弃了，并向源主机发送一个ICMP**时间超过**差错报告报文。

源主机接着发送第二个数据报 P2，并把 TTL 设置为 2。P2 先到达路由器 R1，R1 收下后把 TTL 减1再转发给路由器 R2。R2 收到 P2 时 TTL 为1，但减1后 TTL 变为零了。R2 就丢弃 P2，并向源主机发送一个 ICMP 时间超过差错报告报文。这样一直继续下去。当最后一个数据报刚刚到达目的主机时，数据报的 TTL 是1。主机不转发数据报，也不把 TTL 值减1。但因IP数据报中封装的是无法交付的运输层的 UDP 用户数据报，因此目的主机要向源主机发送 ICMP 终点不可达差错报告报文。

这样，源主机达到了自己的目的，因为这些路由器和最后目的主机发来的 ICMP 报文正好给出了源主机想知道的路由信息——到达目的主机所经过的路由器的IP地址，以及到达其中的每一个路由器的往返时间。下面是向 baidu.com 发出 tracert 命令后得到的结果

```powershell
> tracert www.baidu.com

通过最多 30 个跃点跟踪
到 www.a.shifen.com [61.135.169.121] 的路由:

  1     1 ms    <1 毫秒   <1 毫秒 laptop [192.168.0.1]
  2     6 ms     6 ms     3 ms  1.28.220.60.adsl-pool.sx.cn [60.220.28.1]
  3    21 ms    18 ms     5 ms  201.5.220.60.adsl-pool.sx.cn [60.220.5.201]
  4    36 ms    14 ms    13 ms  253.8.220.60.adsl-pool.sx.cn [60.220.8.253]
  5    18 ms    22 ms     *     219.158.11.113
  6    48 ms     *       26 ms  124.65.194.158
  7    86 ms    36 ms    22 ms  124.65.58.54
  8    64 ms    22 ms    21 ms  123.125.248.46
  9     *        *        *     请求超时。
 10     *        *        *     请求超时。
 11    25 ms    18 ms    75 ms  61.135.169.121

跟踪完成。
```

## 3. Ping

ICMP 的另一个重要应用是分组网间探测 PING (Packet InterNetGroper)，用来测试两个主机之间的连通性。Ping 主要使用 ICMP 回送请求和回送回答报文，这是应用层直接使用网络层 ICMP 的一个例子，没有通过运输层的 TCP 或 UDP。

### 3.1 原理

下面是向 baidu.com 发出的 Ping 请求，计算机连续发出了四个 ICMP 回送请求报文，目标服务器收到后，就返回 ICMP 回收回答报文，往返的 ICMP 报文上都有时间戳，因此很容易得出往返时间。最后会显示出统计结果：发送到哪个机器（IP地址），发送的、收到的和丢失的分组数（但不给出分组丢失的原因），往返时间的最小值、最大值和平均值。

```powershell
>  ping baidu.com

正在 Ping baidu.com [39.156.69.79] 具有 32 字节的数据:
来自 39.156.69.79 的回复: 字节=32 时间=34ms TTL=48
来自 39.156.69.79 的回复: 字节=32 时间=54ms TTL=48
来自 39.156.69.79 的回复: 字节=32 时间=49ms TTL=48
来自 39.156.69.79 的回复: 字节=32 时间=37ms TTL=48

39.156.69.79 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 34ms，最长 = 54ms，平均 = 43ms
```

注：在 Linux 下，ping 命令会持续不断地发出 ICMP 回送请求报文，而不是发 4 个就停止，需要手动给出停止信号。

### 3.2 C语言实现

下面给出一个使用 C 语言的 Ping 命令实现，主要使用 RAW 模式的 SOCKET 编程，因为基于 UDP 的 socket 编程和基于 TCP 的 socket 编程都无法对下一层（网络层）的数据包进行操作。

**所用的API函数**

| 功能                     | 函数                                        |
| ------------------------ | ------------------------------------------- |
| 创建socket               | socket(af, type, protocol)                  |
| 关闭socket               | closesocket(socket)                         |
| 发送数据                 | sendto(s, buf, len, flags, to, tolen)       |
| 接收数据                 | recvfrom(s, buf, len, flags, from, fromlen) |
| 将域名翻译为IP           | gethostbyname(name)                         |
| 将IP转换为点分十进制格式 | Inet_ntoa(ip)                               |
| WSAStartupWSACleanup     | 其它socket函数使用前和使用后用这两个        |

ICMP数据头结构定义如下 

```c
typedef struct _ihdr
{
   BYTE i_type;      //类型
   BYTE i_code;      //代码
   USHORT i_cksum;      //校验和
   USHORT i_id;      //标识符
   USHORT i_seq;      //序号
   /* 下面的时间戳不是标准ICMP头部，这个程序里是为了容易计算世界定义的 */
   ULONG timestamp;
} IcmpHeader;
```

IP数据包头结构定义如下

```c
typedef struct iphdr
{
   unsigned char h_len : 4;     // 首部长度
   unsigned char version : 4;    // 版本号
   unsigned char tos;        // 服务类型
   unsigned short total_len;     // 总长度
   unsigned short ident;       // 标识
   unsigned short frag_and_flags;    // 标志和片偏移
   unsigned char ttl;        //跳数
   unsigned char proto;       // 协议
   unsigned short checksum;     // 校验和
   unsigned int sourceIP;      //源地址
   unsigned int destIP;        //目的地址
} IpHeader;
```

程序实现的功能包括对域名和 IP 地址发出的 Ping 命令，举例如下

```powershell
> ping 192.168.1.1
> ping [www.neu.edu.cn](http://www.neu.edu.cn)
```

程序流程如下

![ping程序实现](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20181001_ping程序.png)

代码放在了 [Github Gist](https://gist.github.com/shuzang/26b2052e5283fa0ff596c43fa3c52265.js)，测试结果如下参数

```c
> www.neu.edu.cn
32 bytes from 202.118.1.7: icmp_seq = 0.  time: 15 ms
32 bytes from 202.118.1.7: icmp_seq = 1.  time: 0 ms
32 bytes from 202.118.1.7: icmp_seq = 2.  time: 32 ms
32 bytes from 202.118.1.7: icmp_seq = 3.  time: 0 ms
32 bytes from 202.118.1.7: icmp_seq = 4.  time: 0 ms
32 bytes from 202.118.1.7: icmp_seq = 5.  time: 16 ms
32 bytes from 202.118.1.7: icmp_seq = 6.  time: 15 ms
32 bytes from 202.118.1.7: icmp_seq = 7.  time: 16 ms
32 bytes from 202.118.1.7: icmp_seq = 8.  time: 16 ms
32 bytes from 202.118.1.7: icmp_seq = 9.  time: 0 ms
32 bytes from 202.118.1.7: icmp_seq = 10.  time: 0 ms
32 bytes from 202.118.1.7: icmp_seq = 11.  time: 16 ms
32 bytes from 202.118.1.7: icmp_seq = 12.  time: 16 ms
32 bytes from 202.118.1.7: icmp_seq = 13.  time: 0 ms
32 bytes from 202.118.1.7: icmp_seq = 14.  time: 31 ms
32 bytes from 202.118.1.7: icmp_seq = 15.  time: 0 ms
32 bytes from 202.118.1.7: icmp_seq = 16.  time: 0 ms
32 bytes from 202.118.1.7: icmp_seq = 17.  time: 16 ms
32 bytes from 202.118.1.7: icmp_seq = 18.  time: 1188 ms
-    
```

注意：

- Sleep(1000)，删除会使被ping的目的机非常繁忙

- 发送的ICMP报文中的数据部分最大可以设置为65535-20-8个字节，这将使网络拥堵
