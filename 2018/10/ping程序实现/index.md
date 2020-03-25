# ping程序实现


## 1. 设计要求

基于Raw Socket编程实现Ping的客户端程序，并分析所实现的Ping的网络攻击性。

**注：**程序已给出，只需要调试使其正常运行即可，应能完全理解程序内容

## 2. 设计思路

Ping程序的作用是测试网络的连通性，其实现主要基于ICMP的回送请求和回送应答。而且我们知道，ICMP是基于IP的一个协议，ICMP包封装在IP数据包的数据部分进行传输。

为了实现直接对IP和ICMP包进行操作，实验中使用RAW模式的SOCKET编程。因为基于UDP的socket编程和基于TCP的socket编程都无法对下一层（网络层）的数据包进行操作。

#### 1.1 所用的API函数

| 功能                     | 函数                                        |
| ------------------------ | ------------------------------------------- |
| 创建socket               | socket(af, type, protocol)                  |
| 关闭socket               | closesocket(socket)                         |
| 发送数据                 | sendto(s, buf, len, flags, to, tolen)       |
| 接收数据                 | recvfrom(s, buf, len, flags, from, fromlen) |
| 将域名翻译为IP           | gethostbyname(name)                         |
| 将IP转换为点分十进制格式 | Inet_ntoa(ip)                               |
| WSAStartupWSACleanup     | 其它socket函数使用前和使用后用这两个        |

#### 1.2 数据结构

ICMP数据头结构： 

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

IP数据包头结构：

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

#### 1.3 程序功能

A. ping ip 地址

如 ping 192.168.1.1

B. ping 域名

如 ping [www.neu.edu.cn](http://www.neu.edu.cn)

**注：**参数输入时只需要输入域名或IP地址，不需要字符`ping`

#### 1.4 平台、语言和IDE

- 平台：windows    
- 语言：C语言
- IDE：VS	

## 3. 流程介绍

![ping程序实现](/images/Goweb-ping实现/3EEkt0.png)

## 4. 代码

<script src="https://gist.github.com/shuzang/26b2052e5283fa0ff596c43fa3c52265.js"></script>
## 5. 参数及输出结果

参数

```c
www.neu.edu.cn
```

结果（在输出一些后停止）

```c
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

## 6. 安全性分析

- Sleep(1000)，删除会使被ping的目的机非常繁忙

- 发送的ICMP报文中的数据部分最大可以设置为65535-20-8个字节，这将使网络拥堵
