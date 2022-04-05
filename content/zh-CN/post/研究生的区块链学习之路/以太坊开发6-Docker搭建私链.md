---
title: 以太坊开发6-Docker搭建以太坊私链
date: 2019-03-12T11:24:00+08:00
lastmod: 2020-11-11
tags: [区块链]
categories: [研究生的区块链学习之路] 
slug: use docker to build ethereum private chain
---

在以太坊github的[官方项目地址](https://github.com/ethereum/go-ethereum)发现其支持Docker启动，同时因为Docker相对于虚拟机的易用性，决定尝试使用Docker搭建以太坊联盟链

Docker安装部分查看自官方说明，搭建过程主要参考[简书-使用Docker搭建以太坊私有链并部署合约](https://www.jianshu.com/p/7994db7a2b89?from=singlemessage)

## 一、Docker安装

OS环境：Ubuntu 18.04 LTS(bionic)

内核版本：4.18.0-16-generic

处理器架构：amd64

官方的安装说明位于：[Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

<!--more-->

官方一共提供了三种安装方式，为了简便，我们选择直接下载`.deb`包安装

1. 前往地址[`https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/`](https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64)下载安装所需文件，包括以下三个

   ```bash
   ├── containerd.io_1.2.4-1_amd64.deb
   ├── docker-ce_18.09.3_3-0_ubuntu-bionic_amd64.deb
   └── docker-ce-cli_18.09.3_3-0_ubuntu-bionic_amd64.deb
   ```

2. 进入下载文件所在目录，执行下列安装命令

   ```bash
   $ sudo dpkg -i docker-ce-cli_18.09.3_3-0_ubuntu-bionic_amd64.deb
   $ sudo dpkg -i containerd.io_1.2.4-1_amd64.deb
   $ sudo dpkg -i docker-ce_18.09.3_3-0_ubuntu-bionic_amd64.deb
   ```

3. 通过运行`hello-world`镜像验证安装完成

   ```bash
   $ sudo docker run hello-world
   ```

    该命令会下载一个测试镜像并在容器中运行它，运行时可以从终端看到如下信息说明安装成功，运行结束自动退出。

   ```bash
   Hello from Docker!
   ```

4. 更新Docker

   通过`.deb`包安装Docker只能再次下载最新的安装包并重复执行安装步骤来更新

## 二、获取geth镜像

[docker hub](https://hub.docker.com/r/ethereum/client-go)上有现成的geth镜像，使用`pull`命令直接获取。

```bash
$ sudo docker pull ethereum/client-go
```

默认安装latest版，试运行

```bash
$ sudo docker run -it --rm -v /workspace:/workspace --entrypoint /bin/sh ethereum/client-go
```

 `-i`：打开STDIN，用于控制台交互，常与-t一起使用

` -t`：分配tty设备，支持终端登陆，默认为false，常与-i一起使用

 `--rm`：指定容器停止后自动删除容器（不支持以docker run -d启动的容器 ）

` -v`：给容器挂载存储卷，挂载到容器的某个目录，这里讲本地的/workspace挂载到了容器的/workspace目录，用来在容器和宿主机之间共享文件

 `--entrypoint`：覆盖image的入口点，ubuntu环境下docker默认入口点其实是/bin/bash，修改默认入口点是为了不让节点自动运行，稍后会对节点进行自定义配置使其成为私有链节点

查看本地镜像列表

```bash
$ sudo docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
ethereum/client-go   latest              f376688623dc        3 hours ago         42.7MB
hello-world          latest              fce289e99eb9        2 months ago        1.84kB
```

证实geth镜像获取完成，同时也能看到之前测试docker安装是否完成时获取的hello-world镜像

## 三、创建Docker网络

推荐是创建一个自有网络，然后将需要互联的容器配置到相同的网络中，此处我们创建名为“ethnet"的网络，该网络配置如下：

子网172.18.0.0/16

- IP段172.18.0.0
- 掩码255.255.0.0
- IP范围172.18.0.1~172.18.255.254
- IP广播172.18.255.255

使用如下命令创建该网络

```bash
$ sudo docker network create -d bridge --subnet=172.18.0.0/16 ethnet
```

`-d`：指定网络类型

`--subnet`：指定网段

使用如下命令查看创建的网络

```bash
$ sudo docker network ls
b88630402852        bridge              bridge              local
3fb4ed66c9e1        ethnet              bridge              local
0414b4eff7ae        host                host                local
5668e7baf5ee        none                null                local
```

其中第二个网络名为"ethnet"，正是我们创建的，其它三个是默认的网络。关于docker的几种网络的说明见[docker中的网络](https://blog.csdn.net/qxxhjy/article/details/82314128)

## 四、配置以太坊网络

运行如下命令进入一个容器

```bash
$ sudo docker run -it --rm --network ethnet --ip 172.18.0.50 -v /workspace:/workspace --entrypoint /bin/sh ethereum/client-go
```

`--network ethnet`指定了该容器加入刚才创建的ethnet网络

`--ip 172.18.0.50`指定了一个固定IP给该容器

### 1. 创建账户

首先，在容器内的/workspace目录下创建如下目录结构和文件

```bash
dapp\
dapp\miner\
dapp\data\
dapp\genesis.json
```

运行如下命令创建账户

```bash
cd /workspace/dapp/miner
geth -datadir ./data account new
```

输入两次password，获得地址，将地址记录备用。重复以上命令可创建多个账户。

### 2. 创建创世区块

编辑刚才创建的文件dapp\genesis.json内容如下

```javascript
{
  "config": {
    "chainId": 88,
    "homesteadBlock": 0,
    "eip155Block": 0,
    "eip158Block": 0
  },
  "alloc"      : {
    "0x79b43b2196723fff1485999aba45fda3e8b4df58": {"balance": "100000000000000000000"},
    "0x1ae06a8afd157b97f072a97f5c62fa836f5ef597": {"balance": "1000000000000000000"},
    "0xa75b4db0c6bfa416d544e3316d47af0fb01eb828": {"balance": "1000000000000000000"},
    "0x1a037d8e8e16a4c88e17c3d5f29ee26a9f5b2c85": {"balance": "1000000000000000000"}
  },
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x400",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000000",
  "mixhash"    :
  "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" :
  "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```

- alloc下面列举了4个账户地址，正是是上一步骤创建并记录下来的地址。
- balance是创世区块为每个账户分配的初始以太币，单位是wei。1eth=10^18wei。也就是除了第一个账户给了100eth外，其它几个账户分别只拥有1eth。

### 3. 完成以太坊网络配置

此时完成配置可使用`exit`命令退出容器，由于启动容器时加入了`--rm`参数，退出后刚才的容器会被删除，但宿主机的/workspace下的文件会被保存下来

## 五、创建以太坊网络

以上已经配置好了一个以太坊私有网络，下面开始正式创建。我们需要一个矿工节点和多个普通节点。

### 1. 创建矿工节点

矿工节点应具有如下特性：

- 它是一个容器，并且是持久的容器
- 它会自动读取genesis.json文件，并初始化以太坊网络
- 它能够连接其它节点（容器）
- 它能够接受各种rpc调用，并能够部署合约
- 它已经配置好挖矿账户，可以一键挖矿

按照以上要求我们来创建矿工节点

**创建entrypoint脚本**

创建文件：/workspace/dapp/init.sh

文件内容如下：

```bash
#!/bin/sh
geth -datadir ~/data/ init /workspace/dapp/genesis.json

if [  $# -lt 1 ]; then 
  exec "/bin/sh"
else
  exec /bin/sh -c "$@"
fi
```

该脚本的功能是让以太坊节点（容器）自动初始化以太坊网络，并且接受一个自动运行脚本作为输入

**创建自动运行脚本**

创建文件：/workspace/dapp/mine.sh，文件内容如下：

```bash
#!/bin/sh
cp -r /workspace/dapp/miner/data/keystore/* ~/data/keystore/
geth -datadir ~/data/ --networkid 88 --rpc --rpcaddr "172.18.0.50" --rpcapi admin,eth,miner,web3,personal,net,txpool --unlock "0x79b43b2196723fff1485999aba45fda3e8b4df58" --etherbase "0x79b43b2196723fff1485999aba45fda3e8b4df58" console
```

第一行命令是将刚才生成的账户私钥文件拷贝到容器的`~/data`目录下。因为/workspace是宿主目录挂载的，并不是linux文件系统，直接将datadir指定到该目录会导致geth报错。

第二行命令是启动以太坊节点的命令。 

- --networkid 88指定了networkid，这个必须与genesis.json内设置保持一致
- --rpc --rpcaddr "172.18.0.50" --rpcapi .... 这些参数表示该节点接受rpc，并且指定了rpc的协议
- --unlock "0x..." 加入该参数会需要用户输入账户密码。密码校验后会解锁该账户。账户解锁后，该节点就能使用此账户的私钥进行签名加密等动作，用以进行交易、发布合约等。
- --etherbase 参数指定了挖矿收益账户

**创建容器**

因为我们没有授予普通用户直接执行docker的权力，所以这里要先给与执行两个脚本的权力

```bash
$ sudo chmod +x init.sh mine.sh
```

创建容器

```bash
$ sudo docker run -it --name=miner --network ethnet --ip 172.18.0.50 --hostname node -v /workspace:/workspace --entrypoint /workspace/dapp/init.sh ethereum/client-go /workspace/dapp/mine.sh
```

`--name=miner`指定容器名为miner

该命令会创建一个持久化的容器，容器的entrypoint和自动运行脚本指定为刚刚我们创建的两个脚本

使用`ps`命令查看容器创建情况

```bash
$ sudo docker ps -a
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS                    PORTS               NAMES
e42f48b1435a        ethereum/client-go   "/workspace/dapp/ini…"   14 hours ago        Exited (0) 13 hours ago                       miner
0384dace397f        hello-world          "/hello"                 23 hours ago        Exited (0) 23 hours ago                       clever_cur
```

### 2. 创建普通节点

我们需要创建更多的节点来形成一个分布式的网络。

**创建自动运行脚本**

要使容器位于同一个网络，应当使他们利用同一个genesis.json初始化，即共享entrypoint脚本，只需要单独创建自动运行脚本即可

创建普通节点的自动运行脚本：/workspace/dapp/node.sh

```bash
#!/bin/sh
cp -r /workspace/dapp/miner/data/keystore/* ~/data/keystore/
geth -datadir ~/data/ --networkid 88 console
```

授予执行权限

```bash
$ sudo chmod +x node.sh
```

创建容器

```bash
$ sudo docker run -it --name=node1 --network ethnet --ip 172.18.0.51 --hostname node1 -v /workspace:/workspace --entrypoint /workspace/dapp/init.sh ethereum/client-go:v1.8.12 /workspace/dapp/node.sh
```

这里的容器名指定为node1，查看当前容器创建情况

```bash
$ sudo docker ps -a
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS                    PORTS               NAMES
23273d430443        ethereum/client-go   "/workspace/dapp/ini…"   14 hours ago        Exited (0) 13 hours ago                       node1
e42f48b1435a        ethereum/client-go   "/workspace/dapp/ini…"   14 hours ago        Exited (0) 13 hours ago                       miner
0384dace397f        hello-world          "/hello"                 23 hours ago        Exited (0) 23 hours ago                       clever_cur
```

### 3. 节点连接

以上两个容器使用`run`命令运行后都会自动进入geth的js交互式环境下，等待输入命令

```js
> 
```

查看节点已连接情况

```js
> admin.peers
[]
```

在miner容器交互界面执行下列命令查看miner节点信息

```js
> admin.nodeInfo.enode
"enode://334268de4cb42842bc1957af05bd1df7d6a1ce806279bb6637298e595c4e36794e92e6c59b7090a538120307576eb1791df61c234daa89e7acb541307a3b24da@127.0.0.1:30303"
```

在node1容器交互界面执行下列命令动态添加miner节点

```js
> admin.addPeer("enode://334268de4cb42842bc1957af05bd1df7d6a1ce806279bb6637298e595c4e36794e92e6c59b7090a538120307576eb1791df61c234daa89e7acb541307a3b24da@172.18.0.50:30303")
true
```

查看已连接节点数

```bash
> net.peerCount
2
```

查看已连接节点情况

```js
> admin.peers
[{
    caps: ["eth/62", "eth/63"],
    enode: "enode://334268de4cb42842bc1957af05bd1df7d6a1ce806279bb6637298e595c4e36794e92e6c59b7090a538120307576eb1791df61c234daa89e7acb541307a3b24da@172.18.0.50:30303",
    id: "831e42a7dfb08c648b77aa7396f17033dd103259b30791b87318b08b7ef1e67d",
    name: "Geth/v1.9.0-unstable-7504dbd6/linux-amd64/go1.12",
    network: {
      inbound: false,
      localAddress: "172.18.0.51:34858",
      remoteAddress: "172.18.0.50:30303",
      static: true,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 263168,
        head: "0x7e03472bcad02f6e85a3cdb21cfba856da58a4955dd2b6d21e3b8561446ae390",
        version: 63
      }
    }
}, {
    caps: ["eth/62", "eth/63"],
    enode: "enode://213e6b175eb2378b42d2564897d32855eae37f7960fe0a378c44f315ca178267bcff076835483274a2385952bf607b1fc1e39eda5be0b03dd1a0ec375ea5b3dc@59.66.19.211:30303",
    id: "86701d83afc9c055c682a3fbe033acf9c5a378c18e0a76c1e13d9e72aa957278",
    name: "Geth/v1.8.22-unstable-dc43ea8d/linux-amd64/go1.11",
    network: {
      inbound: false,
      localAddress: "172.18.0.51:45836",
      remoteAddress: "59.66.19.211:30303",
      static: false,
      trusted: false
    },
    protocols: {
      eth: "handshake"
    }
}, {
    caps: ["eth/62", "eth/63", "par/1", "par/2", "par/3", "pip/1"],
    enode: "enode://09f36adecd8110413be39b5bd9dfb9b06f9575d60db1ba9e4ef0796fadee0346fed412f0147136b6a557f725b0a0944dba92a885e0afdd84345fb80d469650f8@35.175.179.140:30303",
    id: "f2b63e1d6dee827ad5bbbc27341223b7463713a67d1a92c1778a5365d6353e67",
    name: "Parity-Ethereum/v2.1.4-beta-bee2cb8-20181028/x86_64-linux-gnu/rustc1.30.0",
    network: {
      inbound: false,
      localAddress: "172.18.0.51:46864",
      remoteAddress: "35.175.179.140:30303",
      static: false,
      trusted: false
    },
    protocols: {
      eth: "handshake"
    }
}]
```

然后就可以完成挖矿、转账、部署合约等其它操作了。

### 4. 退出

使用`exit`命令会直接退出geth，同时退出容器

```js
> exit
INFO [03-13|03:18:52.778] HTTP endpoint closed                     url=http://172.18.0.50:8545
INFO [03-13|03:18:52.778] IPC endpoint closed                      url=/root/data/geth.ipc
INFO [03-13|03:18:52.778] Writing cached state to disk             block=2 hash=7e0347…6ae390 root=741c08…123b78
INFO [03-13|03:18:52.778] Persisted trie from memory database      nodes=0 size=0.00B time=7.039µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [03-13|03:18:52.778] Writing cached state to disk             block=1 hash=ed8c7f…98bd7c root=b2e71c…818333
INFO [03-13|03:18:52.778] Persisted trie from memory database      nodes=0 size=0.00B time=1.037µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [03-13|03:18:52.778] Blockchain manager stopped 
INFO [03-13|03:18:52.778] Stopping Ethereum protocol 
INFO [03-13|03:18:52.778] Ethereum protocol stopped 
INFO [03-13|03:18:52.778] Transaction pool stopped 
shuzang@ubuntu:~$
```

此时使用`docker ps`命令会发现正在运行的容器列表为空，此时使用`start`命令可启动关闭的容器

```bash
$ sudo docker start -i miner
```

继续进入miner容器的geth环境，然后使用`Ctrl+P+Q`快捷键可以退出但不关闭当前容器，此时使用`docker ps`查看则发现miner容器还在运行列表

然后使用如下命令可进入容器bash环境而不进入geth环境，这种情况下便于我们进行一些需要在容器环境下执行的操作，比如编辑静态节点文件`static-nodes.json`

```bash
$ sudo docker exec -it miner /bin/sh
```

我们之前创建的脚本都是把/keystore拷贝到容器中`~/data`目录下执行的，现在来查看该目录的路径和有哪些文件

```bash
/ # cd ~/data
~/data # pwd
/root/data
~/data # ls
geth      geth.ipc  history   keystore
```

此时可使用`exit`命令退出容器，注意，因为我们这次是用`exec`命令登入的，所以退出时不关闭容器，使用`ps`命令仍能在运行容器列表中看到miner

当然，也可以使用`attach`命令而不是`exec`命令进入，但这样退出会直接退出容器，不会出现在运行列表中