# 以太坊开发1-虚拟机搭建以太坊私链


## 一、单虚拟机

最简单的方式是使用一个虚拟机，这也是在条件有限的情况下的最好选择。方法是在一台虚拟机上启用多个终端，每个终端使用不同的端口通信。步骤如下

### 1. 创建节点文件

在`~Desktop`路径下创建NodeA、NodeB和NodeC三个文件夹，代表三个节点。在三个文件夹中分别创建`genesis.json`文件

### 2. 初始化创世区块

分别执行`geth init`命令初始化创世区块

```bash
$ geth --datadir ~/Desktop/NodeA/dataA init genesis.json
$ geth --datadir ~/Desktop/NodeB/dataB init genesis.json
$ geth --datadir ~/Desktip/NodeC/dataC init genesis.json
```

### 3. 分别启动三个节点

启动NodeA

```bash
$ geth --datadir ~/Desktop/NodeA/dataA --networkid 22 --nodiscover console
```

默认的端口是30303，因为三个节点在同一个操作系统中，所以接下来启动其它两个节点时我们要指定使用其它的接口

```bash
$ geth --datadir ~/Desktop/NodeA/dataB --port 30304 --networkid 22 --nodiscover console
$ geth --datadir ~/Desktop/NodeA/dataC --port 30305 --networkid 22 --nodiscover console
```

### 4. 连接各节点

这里就使用`admin.addPeers`连接各节点就可以了。

## 二、多虚拟机快速搭建

最理想的方式是使用多台电脑，但是一般情况下实验条件不足，不过在电脑性能足够的情况下，可以开启多台虚拟机模拟这样的环境，这里的示例使用三台Ubuntu18.04 LTS系统的虚拟机。

在走了一遍完整搭建流程后，精简不需要的步骤，将快速搭建的过程总结如下，如果想要查看详细的步骤和描述信息，请看第三部分。省略的步骤包括

- 更新软件源
- 配置主机名
- 配置地址解析
- 同步时间
- 配置Golang环境
- 下载编译以太坊源码

### 正式搭建过程

1. 安装网络工具

    ```bash
   $ sudo apt-get install -y net-tools
   ```

2. 安装geth

    ```bash
   $ sudo add-apt-repository -y ppa:ethereum/ethereum
   $ sudo apt-get update
   $ sudo apt-get install ethereum
   $ geth --help
   ```

    详细过程可以查看[Installing Go Ethereum](https://ethereum.github.io/go-ethereum/install/)

3. 创建创世区块文件

    ```bash
   $ mkdir ~/nodeA
   $ cd ~/nodeA
   $ touch genesis.json
   ```

4. 编辑`genesis.json`文件内容如下

    ```bash
   {
   "config": {
           "chainId": 72,
           "homesteadBlock": 0,
           "eip155Block": 0,
           "eip158Block": 0
       },
     "alloc"      : {},
     "coinbase"   : "0x0000000000000000000000000000000000000000",
     "difficulty" : "0x40",
     "extraData"  : "",
     "gasLimit"   : "0x2fefd8",
     "nonce"      : "0x0000000000000000",
     "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
     "parentHash" :"0x0000000000000000000000000000000000000000000000000000000000000000",
     "timestamp"  : "0x00"
   }
   ```

    之后克隆得到三个虚拟机，作为区块链的三个节点

5. 创建data文件夹供存储区块链数据

    ```bash
   $ mkdir ~/nodeA/data & cd ~/nodeA
   ```

6. 初始化创世区块

    ```bash
   geth --datadir data/ init genesis.json
   ```

7. 启动网络

    ```bash
   $ geth --datadir data/ --networkid 72 --nodiscover console
   ...
   Welcome to the Geth JavaScript console!
   ```

8. 获取节点信息

   ```bash
   > admin.nodeInfo.enode
   ```

    分别在三个节点执行上述命令，得到的三个节点的信息为：

    ```bash
    NodeA: "enode://d0d6a32cacb349c8f8c92372dc3a1e118384720636519a46c14067a3b62f6cfa8549a8766ade0f91790ebdadfb44341e03c50a171b714070527be96ed037707e@[::]:30303?discport=0"
   NodeB: "enode://d9fc148c9808fbfee7954bd3324bfdff42777de7d2545ac3c2357f05939dbd8ee153cd32320a05f4dbf18138007f743f3a2097b62e71c3c47bb3cf1559dd1328@[::]:30303?discport=0"
   NodeC: "enode://246782d2429f4697e18009505989ed82c2c0a664aab96ed80b96e28be42b6d9e2d11b27806ea43e46fddfcb3e9a41e9df4889636f51e3dadc0c937b0735d0bdc@[::]:30303?discport=0"
    ```

9. 在NodeA执行下列命令，将NodeB加入网络，NodeC同理

    ```bash
   >admin.addPeer("enode://d9fc148c9808fbfee7954bd3324bfdff42777de7d2545ac3c2357f05939dbd8ee153cd32320a05f4dbf18138007f743f3a2097b62e71c3c47bb3cf1559dd1328@[::]:30303")
   ```

    将 [::] 修改为正确节点的 IP 地址，是之前用ifconfig命令查询并记录的本地地址，不支持DNS解析的域名地址，只支持点分十进制格式的IP地址。删除` ?discport=0

10. 查看节点连接信息

     ```bash
    > admin.peers
     ```

    节点关机后，会自动被删除，节点重新启动后，会自动加入节点集群, 但是如果所有节点全部断掉，则需要重新添加。

11. 转账和挖矿测试查看下面的详细搭建过程

### 补充说明

1. `genesis.json`文件的相关说明可参见[以太坊-创世区块文件genesis.json](https://shuzang.github.io/2019/以太坊-创世区块文件genesis.json/) 

2. `geth`命令的参数说明参见[以太坊-geth命令参数说明](https://shuzang.github.io/2019/%E4%BB%A5%E5%A4%AA%E5%9D%8A-geth%E5%91%BD%E4%BB%A4%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E/)

3. 本文后半部分以`>`开头的命令是控制台命令，这是一个以太坊提供的交互式的JavaScript环境，可以在该环境中进行命令或者代码的执行操作，具体内容参见[以太坊-Geth Console命令详解](https://shuzang.github.io/2019/%E4%BB%A5%E5%A4%AA%E5%9D%8A-geth-console%E5%91%BD%E4%BB%A4%E8%AF%A6%E8%A7%A3/)

4. 本文使用了`admin.addPeer()`命令进行节点连接，但实际上还有其它方式，具体每种方式的使用参见[以太坊-节点连接到网络的几种方式](https://shuzang.github.io/2019/%E4%BB%A5%E5%A4%AA%E5%9D%8A-%E8%8A%82%E7%82%B9%E8%BF%9E%E6%8E%A5%E5%88%B0%E7%BD%91%E7%BB%9C%E7%9A%84%E5%87%A0%E7%A7%8D%E6%96%B9%E5%BC%8F/)

5. geth命令和控制台命令有时候能实现同样的功能，比如 

   ```bash
   $ geth --datadir data/ account new
   ```

   ```bash
   > personal.newAccount()
   ```

    两条命令均是实现创建新账户的功能

6. 第4条所述节点连接方式中有一种是实现创建`static-nodes.json`文件用于之后的节点识别，但该方式需要账户地址，故只能使用geth命令创建新账户，而本文所述方式可以在节点连接后，再在控制台下创建账户用于转账和挖矿的操作。

## 三、多虚拟机详细过程

主要内容来自 https://blog.51cto.com/clovemfong/2280872

使用了多个虚拟机搭建，每个虚拟机代表一个区块链节点。自编译了以太坊源码，并且由于区块链的特性，对Linux的时间做了一定的配置，同时为了多个节点间的连接，对IP地址做了配置处理。以上这些不是必须的，但能让我们思路清晰，明白构建一个区块链应该保证什么条件。

### 1. 配置操作系统环境

本文的操作环境是在虚拟机上完成，共计三台虚拟机 (配置尽量高上去，如下这个配置挖矿太耗时，这里的配置主要指内存和CPU的核，因为以太坊的挖矿算法Ethash主要和内存有关，而CPU的核多一点挖矿时可以多开几个线程)

| 主机名            | IP 地址        | 操作系统        | 内存 / GB | CPU / 核 |
| ----------------- | -------------- | --------------- | --------- | -------- |
| nodeA.shuzang.com | 172.16.222.189 | Ubuntu18.04 LTS | 2         | 2        |
| nodeB.shuzang.com | 172.16.222.190 | Ubuntu18.04 LTS | 2         | 2        |
| nodeC.shuzang.com | 172.16.222.191 | Ubuntu18.04 LTS | 2         | 2        |

> Q：linux下设置主机名的时候，为什么都要求设置成：name.domain.com这种域名的样式.
>
> A：因为linux等类unix系统都是为网络应用而开发的，主机的性能可以不需要很强大，只需要能够登录到服务器（也就是加入某个域），就可以利用服务器的资源，因此安装系统时如果主机是在某个域里，设置域名就表明该主机是这个域里的一员，通过合法的帐号和密码就可以连接登录到域服务器。如果是单机或者个人家庭网络，域名就可以随便设置了

注1：以 nodeA 为例进行相关配置，其他节点配置操作相同

注2：IP地址一栏应在虚拟机创建好后用ifconfig命令查看并记录

#### 1.1 更新软件源

根据自己需要选择是否需要更换软件源，此处用的是原生的即可。

```bash
$ sudo apt-get  update
```

#### 1.2 安装相关工具

```bash
sudo apt-get  install vim openssh-server ntp ntpdate make gcc net-tools  -y
```

#### 1.3 配置主机名

```bash
$ sudo hostname nodeA.shuzang.com
```

#### 1.4 配置地址解析

```bash
$ vim /etc/hosts
127.0.0.1       localhost
172.16.222.189  nodeA.shuzang.com
172.16.222.190  nodeB.shuzang.com
172.16.222.191  nodeC.shuzang.com
```

hostname --fqdn 验证

```bash
$ hostname -f        
nodeA.shuzang.com
```

#### 1.5 同步时间

修改时区

```bash
$ sudo timedatectl set-timezone "Asia/Shanghai" 
```

手动同步

```bash
$ sudo ntpdate time1.aliyun.com    
```

同步硬件时间

```bash
$ sudo hwclock  -w # 系统时间同步至硬件
```

手动设置完毕后，再通过如下方式进行 ntp 服务的配置，可以选择现有或者自建的时间服务器

测试连通性

```bash
$ ntpdate -q time1.aliyun.com 
server 203.107.6.88, stratum 2, offset 0.050751, delay 0.06232
22 Sep 21:04:36 ntpdate[8082]: adjust time server 203.107.6.88 offset 0.050751 sec 
```

修改配置文件

```bash
$ sudo cp /etc/ntp.conf  /etc/ntp.conf_shuzang_201809231208 #备份配置文件
$ sudo vim /etc/ntp.conf //修改配置文件
```

修改信息如下

```bash
# Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
# on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
# more information.
#pool 0.ubuntu.pool.ntp.org iburst
#pool 1.ubuntu.pool.ntp.org iburst
#pool 2.ubuntu.pool.ntp.org iburst
#pool 3.ubuntu.pool.ntp.org iburst

server time1.aliyun.com
server time2.aliyun.com
server time3.aliyun.com
server time4.aliyun.com
server time5.aliyun.com
server time6.aliyun.com
server time7.aliyun.com
```

启动 ntp 服务

```bash
$ sudo systemctl  restart ntp  #启动ntp服务
```

### 2. 配置 Golang 环境

下载 go 安装包

```bash
$ mkdir ethereum ;cd ethereum
$ wget https://dl.google.com/go/go1.11.linux-amd64.tar.gz
```

解压安装包

```bash
$ sudo tar zxvf  go1.11.linux-amd64.tar.gz   -C /usr/local/

```

配置环境变量

```bash
$  mkdir -p ~/workspace/{src,pkg,bin} 
$ sudo cp  /etc/profile /etc/profile_shuzang_201809231354
$ sudo vim /etc/profile 
# 配置文件中添加如下信息
export GOROOT="/usr/local/go"
export GOPATH="/home/ubuntu/workspace"
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOROOT/bin
```

检查配置

```bash
$ source /etc/profile
$ go env
GOARCH="amd64"
GOBIN="/home/ubuntu/workspace/bin"
GOCACHE="/home/ubuntu/.cache/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/home/ubuntu/workspace"
GOPROXY=""
......
```

### 3. 安装部署

#### 3.1 获取源代码

以太坊的源码文件网址如下

```bash
https://github.com/ethereum/go-ethereum
```

可以选择通过`git clone`或者下载`zip`包的方式进行源代码的获取, 为了方便起见，本文直接下载 zip 包上传至三台节点进行解压部署。

下载源码包

```bash
$ cd /home/ubuntu/ethereum
$ wget  https://github.com/ethereum/go-ethereum/archive/master.zip
```

解压源码包

```bash
$ unzip master.zip
$ mkdir ~/workspace/github.com/
$ mv go-ethereum-master/   ~/workspace/github.com/go-ethereum
$ cd ~/workspace/github.com/go-ethereum ; make geth  #编译安装
```

添加环境变量

```bash
$ sudo vim /etc/profile
# 添加及修改如下信息
export GETH="$GOPATH/github.com/go-ethereum/build"
export PATH=$PATH:$GOROOT/bin:$GETH/bin
```

测试生效

```bash
$ source /etc/profile
$ geth --help
```

至此环境配置完成，可以完成后进行虚拟机复制，得到三个虚拟机，然后再对主机名等进行修改，也可以等genesis.json文件创建完成后在进行虚拟机复制，然后修改相关名称。这样可以免去重复的环境配置工作，节省大量时间。

#### 3.2 创建创世区块

创建数据存储目录

```bash
$ mkdir -p /home/ubuntu/nodeA/data0
$ cd /home/ubuntu/nodeA
```

创世区块配置文件`genesis.json`

```bash
{
"config": {
        "chainId": 66,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x20000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" :"0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```

难度"difficulty"可以设置小一点，比如"0x40"，这样挖矿会容易一点，不然配置不好就只能死等。

初始化创世区块

```bash
$ geth  --datadir data0/  init genesis.json
```

![](https://s1.51cto.com/images/blog/201809/23/3cbb855f451fe1cb71b6b3efa7c598ac.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

#### 3.3 启动服务

```bash
$ geth --datadir data0/ --networkid 66 --nodiscover console
...
Welcome to the Geth JavaScript console!
```

nodiscover : 设置为不自动发现，用于控制联盟链的节点加入
![](https://s1.51cto.com/images/blog/201809/23/813b196e06db099346217fd27420ac82.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 4. 节点通信

#### 4.1 查看集群节点

```bash
> admin.peers
[]
```

目前为空状态，表明未于其他节点进行通信。

#### 4.2 获取节点信息

```bash
> admin.nodeInfo.enode
```

分别在三个节点的 console 上执行命令，显示如下:

NodeA

```bash
"enode://d0d6a32cacb349c8f8c92372dc3a1e118384720636519a46c14067a3b62f6cfa8549a8766ade0f91790ebdadfb44341e03c50a171b714070527be96ed037707e@[::]:30303?discport=0"
```

NodeB

```bash
"enode://d9fc148c9808fbfee7954bd3324bfdff42777de7d2545ac3c2357f05939dbd8ee153cd32320a05f4dbf18138007f743f3a2097b62e71c3c47bb3cf1559dd1328@[::]:30303?discport=0"
```

NodeC

```bash
"enode://246782d2429f4697e18009505989ed82c2c0a664aab96ed80b96e28be42b6d9e2d11b27806ea43e46fddfcb3e9a41e9df4889636f51e3dadc0c937b0735d0bdc@[::]:30303?discport=0"
```

#### 4.3 将 NodeB 加入 NodeA

在 NodeA 节点上执行如下命令

```bash
>admin.addPeer("enode://d9fc148c9808fbfee7954bd3324bfdff42777de7d2545ac3c2357f05939dbd8ee153cd32320a05f4dbf18138007f743f3a2097b62e71c3c47bb3cf1559dd1328@172.16.222.190:30303")
```

返回`true`表示加入成功，注意

- 将 [::] 修改为正确节点的 IP 地址，是之前用ifconfig命令查询并记录的本地地址，不支持DNS解析的域名地址，只支持点分十进制格式的IP地址
- 删除` ?discport=0`

#### 4.4 查看新加入的节点

```bash
> admin.peers
```

在 NodeA 节点上执行如下命令

![](https://s1.51cto.com/images/blog/201809/23/c6d667762a931a233142b2cd992c8432.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

通过查看 localAddress 跟 remoteAddress 查看节点关联信息。

在 NodeB 节点上执行如下命令

![](https://s1.51cto.com/images/blog/201809/23/be124927d617e9e6ea6fddab1fe5938d.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

在 NodeC 节点上执行如下命令

```bash
> admin.peers
[]
```

#### 4.5 将 NodeC 加入 NodeA 与 NodeB

在 NodeA 节点上执行如下命令

```bash
>admin.addPeer("enode://246782d2429f4697e18009505989ed82c2c0a664aab96ed80b96e28be42b6d9e2d11b27806ea43e46fddfcb3e9a41e9df4889636f51e3dadc0c937b0735d0bdc@172.16.222.191:30303")
```

在 NodeB 节点上执行如下命令

```bash
>admin.addPeer("enode://246782d2429f4697e18009505989ed82c2c0a664aab96ed80b96e28be42b6d9e2d11b27806ea43e46fddfcb3e9a41e9df4889636f51e3dadc0c937b0735d0bdc@172.16.222.191:30303")
```

#### 4.6 查看节点加入信息

在 NodeA 节点上执行如下命令

![](https://s1.51cto.com/images/blog/201809/23/81981335147af8327eab454f0d0c0aa7.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

在 NodeB 节点上执行如下命令

![](https://s1.51cto.com/images/blog/201809/23/88f203a475aef146659a9b0709892b50.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

在 NodeC 节点上执行如下命令

![](https://s1.51cto.com/images/blog/201809/23/8b778eb62c043cf3fe8868c1c518eec2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

注意: 节点关机后，会自动被删除，节点重新启动后，会自动加入节点集群, 但是如果所有节点全部断掉，则需要重新添加。

### 5. 测试

#### 5.1 转账测试

查看当前节点上的账户

```bash
> personal.listAccounts
# 或者
> eth.accounts
```

 创建新账户

```bash
> personal.newAccount("shuzang1996")
"0xcd3d95c64394452313b539a1f2de54eab2b80eed"
> personal.newAccount("shuzang1997")
"0xc4f132a71da05257a71ae5872beabd12c50dbb81"
```

查看创建的账户

```bash
> personal.listAccounts
["0xcd3d95c64394452313b539a1f2de54eab2b80eed",
"0xc4f132a71da05257a71ae5872beabd12c50dbb81"]
```

查看账户地址

```bash
> personal.listWallets
```

![](https://s1.51cto.com/images/blog/201809/23/b700aee79e063512c2f2c8f248a45919.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

查看挖矿账户

```bash
> eth.coinbase
INFO [09-23|18:17:20.246] Etherbase automatically configured       address=0xcD3d95c64394452313b539a1f2dE54Eab2B80eEd
"0xcd3d95c64394452313b539a1f2de54eab2b80eed"
```

默认情况下，平台会选择创建的第一个账户作为挖矿账户 (coinbase)，可以通过 miner.setEthbase() 进行 coinbase 账户的设置。

#### 5.2 挖矿测试

注意：在执行挖矿命令之前，务必在节点上面设置挖矿账户 (coinbase)，否则会报错`Error: etherbase missing: etherbase must be explicitly specified`

当前账户信息为

```bash
NodeA:["0xcd3d95c64394452313b539a1f2de54eab2b80eed","0xc4f132a71da05257a71ae5872beabd12c50dbb81"]
NodeB：["0x22160d844b4016ed8f45ce63c405de72b1946696","0x23e80b2f1f74fc6da4bcb015d17ed642bcc98c06"]
NodeC:
["0xa37539c0f41fea23c81f65b196df825202bc3bb5","0x3c9cdb2f29606a0b35ae95b51521a41bb5b48e27"]
```

启动挖矿

```bash
> miner.start()
INFO [09-23|18:57:41.223] Updated mining threads                   threads=6
INFO [09-23|18:57:41.223] Transaction pool price threshold updated price=1000000000
null
> INFO [09-23|18:57:41.225] Commit new mining work                   number=1 sealhash=b16ebe…9b0094 uncles=0 txs=0 gas=0 fees=0 elapsed=794.985µs
> INFO [09-23|18:57:52.671] Successfully sealed new block            number=1 sealhash=b16ebe…9b0094 hash=082473…eb94b6 elapsed=11.446s
INFO [09-23|18:57:52.671] ined potential block                  number=1 hash=082473…eb94b6
INFO [09-23|18:57:52.672] Commit new mining work                   number=2 sealhash=7a67a7…3c98aa uncles=0 txs=0 gas=0 fees=0 elapsed=1.062ms
INFO [09-23|18:57:55.115] Successfully sealed new block            number=2 sealhash=7a67a7…3c98aa hash=c5cd53…c3e66b elapsed=2.443s
INFO [09-23|18:57:55.115] ined potential block                  number=2 hash=c5cd53…c3e66b
INFO [09-23|18:57:55.116] Commit new mining work                   number=3 sealhash=cc1414…f36ad7 uncles=0 txs=0 gas=0 fees=0 elapsed=343.53µs
INFO [09-23|18:57:55.166] Successfully sealed new block            number=3 sealhash=cc1414…f36ad7 hash=765d41…03ce17 elapsed=50.270ms
INFO [09-23|18:57:55.166] ined potential block                  number=3 hash=765d41…03ce17
INFO [09-23|18:57:55.167] Commit new mining work                   number=4 sealhash=cb5122…e36b2c uncles=0 txs=0 gas=0 fees=0 elapsed=718.996µs
INFO [09-23|18:57:58.048] Successfully sealed new block            number=4 sealhash=cb5122…e36b2c hash=ab1e6d…03b40b elapsed=2.881s
INFO [09-23|18:57:58.048] ined potential block                  number=4 hash=ab1e6d…03b40b
INFO [09-23|18:57:58.049] Commit new mining work                   number=5 sealhash=0dd5ab…5d47d9 uncles=0 txs=0 gas=0 fees=0 elapsed=159.
```

注意: 尽量将虚拟机的配置调高一点，否则会出现以下这些信息，耗时非常久。

```bash
INFO [09-23|18:48:45.223] Updated mining threads                   threads=2
INFO [09-23|18:48:45.229] Transaction pool price threshold updated price=1000000000
INFO [09-23|18:48:45.230] Etherbase automatically configured       address=0x22160D844B4016eD8f45cE63C405DE72B1946696
null
> INFO [09-23|18:48:45.236] Commit new mining work                   number=1 sealhash=88e805…da6df6 uncles=0 txs=0 gas=0 fees=0 elapsed=4.290ms
INFO [09-23|18:48:48.930] Generating DAG in progress               epoch=0 percentage=0 elapsed=2.961s
INFO [09-23|18:48:51.895] Generating DAG in progress               epoch=0 percentage=1 elapsed=5.926s
INFO [09-23|18:48:54.739] Generating DAG in progress               epoch=0 percentage=2 elapsed=8.770s
..........无尽的等待.........
INFO [09-23|18:53:50.638] Generated ethash verification cache      epoch=0 elapsed=5m4.669s
INFO [09-23|18:54:08.357] Generating DAG in progress               epoch=1 percentage=0  elapsed=15.480s
INFO [09-23|18:54:19.874] Generating DAG in progress               epoch=1 percentage=1  elapsed=26.997s
INFO [09-23|18:54:29.911] Generating DAG in progress               epoch=1 percentage=2  elapsed=37.034s
..........无尽的等待.........
WARN [09-23|18:57:57.836] Discarded bad propagated block           number=1 hash=082473…eb94b6
INFO [09-23|18:57:58.398] Block synchronisation started 
INFO [09-23|18:57:58.404] Mining aborted due to sync 
INFO [09-23|18:57:58.611] Imported new state entries               count=1 elapsed=1.352ms   processed=1 pending=0 retry=0 duplicate=0 unexpected=0
INFO [09-23|18:57:58.870] Imported new block headers               count=4 elapsed=95.436ms  number=4 hash=ab1e6d…03b40b
INFO [09-23|18:57:58.889] Imported new chain segment               blocks=4 txs=0 mgas=0.000 elapsed=16.862ms  mgasps=0.000 number=4 hash=ab1e6d…03b40b cache=849.00B
INFO [09-23|18:57:58.892] Imported new block headers               count=1 elapsed=20.452ms  number=5 hash=75e595…4e4cc2
INFO [09-23|18:57:58.893] Imported new chain segment               blocks=1 txs=0 mgas=0.000 elapsed=316.348µs mgasps=0.000 number=5 hash=7
..........无尽的等待.........
INFO [09-23|19:07:35.250] Generated ethash verification cache      epoch=1 elapsed=13m42.368s
INFO [09-23|19:09:28.613] Successfully sealed new block            number=37 sealhash=95e0e0…ad356d hash=7506d6…015be8 elapsed=10m54.093s
INFO [09-23|19:09:28.614] ined potential block                  number=37 hash=7506d6…015be8
INFO [09-23|19:09:28.619] Commit new mining work                   number=38 sealhash=96b6b0…b3d462 uncles=0 txs=0 gas=0 fees=0 elapsed=662.942µs
INFO [09-23|19:12:33.104] Successfully sealed new block            number=38 sealhash=96b6b0…b3d462 hash=a91e1b…a15902 elapsed=3m4.484s
INFO [09-23|19:12:33.105] ined potential block                  number=38 hash=a91e1b…a15902
INFO [09-23|19:12:33.107] Commit new mining work                   number=39 sealhash=6bf2a3…0d8be3 uncles=0 txs=0 gas=0 fees=0 elapsed=2.170ms
```

停止挖矿

```bash
> miner.stop()
```

#### 5.3 查询余额

查询NodeA coinbase 账户(0xcd3d95c64394452313b539a1f2de54eab2b80eed)

```bash
> eth.getBalance(eth.coinbase)
180000000000000000000
```

或者

```bash
> eth.getBalance(eth.accounts[0])
180000000000000000000
```

可以对结果进行单位转换

```bash
>  web3.fromWei(eth.getBalance(eth.accounts[0]),'ether')
180
```

NodeB 查询 NodeA 的 coinbase 账户

```bash
> eth.getBalance("0xcd3d95c64394452313b539a1f2de54eab2b80eed")
180000000000000000000
```

NodeC 查询 NodeA 的 coinbase 账户

```bash
> eth.getBalance("0xcd3d95c64394452313b539a1f2de54eab2b80eed")
180000000000000000000
```

同样，在 NodeA 和 NodeC 上面同样可以查询 NodeB 上面账号地址的余额。

#### 5.4 交易转账

**场景**: NodeA(原有额度: 180) 的挖矿地址向 NodeB(原有额度: 10) 的挖矿地址转入 2 ETH

在进行交易转账之前，我们需要对交易转出方的账户进行解锁操作。NodeA 上执行如下命令，要求输入的密码是使用personal.newAccount创建账户时小括号内的参数

```bash
> personal.unlockAccount("0xcd3d95c64394452313b539a1f2de54eab2b80eed")
Unlock account 0xcd3d95c64394452313b539a1f2de54eab2b80eed
Passphrase: 
true
```

设置转账金额，将 ETH 转成 Wei 进行交易

```bash
> transferAmount= web3.toWei(2,'ether')
"2000000000000000000"
```

执行转账

```bash
>eth.sendTransaction({from:"0xcd3d95c64394452313b539a1f2de54eab2b80eed",to:"0x22160d844b4016ed8f45ce63c405de72b1946696",value:transferAmount})

INFO [09-23|20:13:20.968] Setting new local account                address=0xcD3d95c64394452313b539a1f2dE54Eab2B80eEd
INFO [09-23|20:13:20.968] Submitted transaction                    fullhash=0xd115c9ccfb1bfcc92faaf3ef6fae34ba670ef68cb03807d82d943141851985ce recipient=0x22160D844B4016eD8f45cE63C405DE72B1946696
"0xd115c9ccfb1bfcc92faaf3ef6fae34ba670ef68cb03807d82d943141851985ce"
```

查询交易池状态

```bash
> txpool.status
{
  pending: 1,
  queued: 0
}
```

执行挖矿，指定只挖一个区块

```bash
> miner.start()  ; admin.sleepBlocks(1),miner.stop()
```

![](https://s1.51cto.com/images/blog/201809/23/598f90361f820d9bd76c28352b61abbf.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

查看交易池

```bash
> txpool.status

{
  pending: 0,
  queued: 0
}
```

查看NodeB 的 CoinBase 账户余额

```bash
> web3.fromWei(eth.getBalance("0x22160d844b4016ed8f45ce63c405de72b1946696"),'ether')
12
```

## 附录

### 1. 创世区块配置文件参数说明

| 参数名     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| chainId    | 指定独立的区块链网络 ID，1-4 为保留 ID，其中 1 为主网 ID，其余为测试网 |
| alloc      | 用来设置账号以及账号的以太币数量，私有链可以不进行设置       |
| coinbase   | 用于获取挖矿奖励的矿工账号                                   |
| difficulty | 设置挖矿的难度系数                                           |
| extraData  | 附加信息，自定义即可                                         |
| gasLimit   | 设置对 gas 的消耗总量的限制, 私有链中可以填写任意数值 (大数值) |
| nonce      | 用于进行挖矿的随机数                                         |
| mixhash    | 与 nonce 配合使用，用于挖矿                                  |
| parentHash | 上一个区块的 Hash 值，由于该区块为创世区块，所以父 Hash 值为 0 |
| timestamp  | 时间戳                                                       |

data0 目录中的相关文件列表

![](https://s1.51cto.com/images/blog/201809/23/3fb7ef20a861d9fed05427bcf778cf96.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

- **chaindata : 存储区块数据**
- **keystore: 存储账户数据**

### 2. 私链服务启动常见参数说明

| 参数名     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| identity   | 用于标识区块链的名称                                         |
| init       | 用于指定创世区块的位置，并且创建创世区块以及相关目录结构     |
| datadir    | 设置当前区块链网络数据存放的位置                             |
| port       | 区块链启动的监听端口，默认为 30303                           |
| rpc        | 启用 RPC 服务，可以用于智能合约的部署与调试，默认监听 127.0.0.1, 端口为 8545 |
| rpcapi     | 设置 RPC 提供的 API 接口类型，一般为 db,eth.net.web3         |
| networkid  | 设置当前区块链的网络 ID                                      |
| nodiscover | 设置节点不被自动发现，用于控制节点的加入                     |
| console    | 启用命令行模式，用于执行命令                                 |
| help       | 帮助命令，所有参数的汇总                                     |

### 3. 控制台命令说明

以太坊提供了一个交互式的 JavaScript 环境，可以在该环境中进行命令或者代码的执行操作，其中一些常用的以太坊 JavaScript 对象可以直接被调用。

| 对象名                   | 说明                                           |
| ------------------------ | ---------------------------------------------- |
| eth                      | 操作区块链相关的方法                           |
| eth.accounts             | 查看当前系统中的所有账户地址                   |
| eth.getBalance()         | 查询账户余额，返回单位为 Wei(1 ether=10^18Wei) |
| eth.blockNumber          | 列出区块总数                                   |
| eth.getTransaction()     | 获取指定交易的状态信息                         |
| eth.getBlock()           | 获取指定区块的信息                             |
| net                      | 用于查看 p2p 网络状态                          |
| admin                    | 管理节点相关的方法                             |
| admin.peers              | 查看通讯的节点信息                             |
| admin.addPeer()          | 添加通讯 P2P 节点                              |
| miner                    | 启动 / 停止挖矿相关方法                        |
| personal                 | 用于管理账户                                   |
| personal.newAccount()    | 创建账户                                       |
| personal.unlockAccount() | 解锁账户                                       |
| txpool                   | 查看交易池信息, txpool.status()                |
| web3                     | 包含了以上对象以及单位换算等方法               |
| web3.fromWei()           | Wei 换算成 ether 或者其他单位                  |
| web3.toWei()             | 将 ether 或者其他单位换算成 Wei                |



---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2019/use-virtual-machine-builds-ethereum-private-chain/  

