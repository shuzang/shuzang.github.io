---
title: 以太坊开发7-swarm存储网络使用
date: 2019-03-28T19:59:00+08:00
lastmod: 2020-11-11
tags: [区块链]
categories: [研究生的区块链学习之路]
slug: swarm storage network using
---

说实话，swarm的资料比较难找，倒是IPFS的比较多一点。最后只在[Medium](http://medium.com/)找到了一些，本篇文章就是在实践其中的一些项目，并通过这样来学习swarm。

参考链接：[swarm官网](https://swarm-gateways.net/bzz:/theswarm.eth/)，[swarm文档](https://swarm-guide.readthedocs.io/en/latest/introduction.html)，[github项目](https://github.com/ethersphere)

还有一些帮助理解的资料：

- [关于swarm的三个理解上的问题](https://ethereum.stackexchange.com/questions/55027/swarm-in-a-private-network)

- [swarm和ipfs的比较](https://github.com/ethersphere/go-ethereum/wiki/IPFS-&-SWARM)

<!--more-->

## 1. Ethereum Swarm是什么

Ethereum swarm是一个分布式文件存储系统，开发这个项目是因为区块链中的数据存储是昂贵的，它的不同之处在于它会激励一部分参与者提供他们的存储资源，以此来维持存储网络的稳定运行，当然，目前激励机制还没有上线。

撰写本文时，找到的最新版本是POC3（Proof-of-Concept Release 3)，发布于June 21, 2018。关于该版本和该项目的详细信息见：

[Announcing Swarm Proof-of-Concept Release 3](<https://blog.ethereum.org/2018/06/21/announcing-swarm-proof-of-concept-release-3/>)

[Swarm alpha public pilot and the basics of Swarm](<https://blog.ethereum.org/2016/12/15/swarm-alpha-public-pilot-basics-swarm/>)

## 2. 安装Swarm

详细的安装方式见[这里](<https://swarm-guide.readthedocs.io/en/latest/installation.html>)，这里只介绍在Ubuntu上通过PPA安装，虽然这种方式安装的是stable版本，但版本号同样在0.3，所以就不使用更麻烦的自编译源码安装了。

```bash
$ sudo add-apt-repository -y ppa:ethereum/ethereum
$ sudo apt-get update
$ sudo apt-get install ethereum-swarm
```

安装完成后查看swarm版本

```bash
$ swarm version
Swarm
Version: 0.3.11-stable
Git Commit: c942700427557e3ff6de3aaf6b916e2f056c1ec2
Go Version: go1.10.4
OS: linux
```

## 3. 配置Swarm网络

### 3.1 第一个swarm节点

运行swarm需要以太坊账户，我们通过geth命令来创建账户，需要已经进行过[geth安装](https://github.com/ethereum/go-ethereum/wiki/Installing-Geth)。

```bash
$ mkdir swarmNode1
$ geth --datadir swarmNode1/ account new
$ export BZZKEY1="your new account address"
```

将返回的账户地址设置为环境变量`BZZKEY1`，然后启动第一个节点。

```bash
$ swarm --bzzaccount $BZZKEY1 --datadir swarmNode1/ --keystore swarmNode1/keystore --ens-api "" --bzzport 5000
```

- **bzzaccount**：设置节点账户地址
- **datadir**：设置swarm节点存储数据的文件目录
- **keystore**：账户密钥所在文件目录，设置该选项后就可以使用密码来解锁账户
- **ens-api**：将此项设置为空，swarm将不会连接到区块链，并在无区块链环境下运行
- **bzzport**：设置用来上传和下载的端口地址

运行单节点的话以上设置已经足够了，但是运行多节点的话还需要其它一些设置，并不能简单的重复第一个节点的配置过程。

### 3.2 更多swarm节点

启动多个节点的时候不能仅仅改动`--bzzport`参数，还需要改动UDP端口号，这一点文档中没有提到，启动第二个节点的示例如下：

```bash
$ mkdir swarmNode2
$ geth --datadir swarmNode2/ account new
$ export BZZKEY2="your new account address"
$ swarm --bzzaccount $BZZKEY2 --datadir swarmNode2/ --keystore swarmNode2/keystore --ens-api "" --bzzport 5500 --port 9000
```

### 3.3 连接swarm节点

现在节点都已经启动，我们需要把它们连接起来以完成彼此通信。为了完成这一点，我们需要手动地将swarmNode2的引导节点连接到swarmNode1。

首先寻找swarmNode2的引导节点地址，运行如下命令。

```bash
$ geth --exec "console.log(admin.nodeInfo.enode)" attach swarmNode2/bzzd.ipc
enode://4ae5ee37b365e316b1d2b3d07e5cb1f620919ff39b89f5640b461e64bb92cf8a2caa399548a292387c3f31741ff0e886231258a66707ce51ba5f85856790faac@127.0.0.1:9800?discport=0
```

- **exec**：执行JavaScript语句(只能结合console/attach使用)
- **bzzd.ipc**：运行swarmNode2生成的文件，结束运行该文件消失

复制结果并添加到如下命令，连接两个节点

```bash
$ geth --exec='admin.addPeer("your enode address")' attach swarmNode1/bzzd.ipc
```

`your enode address`即我们上面得到的

```bash
enode://4ae5ee37b365e316b1d2b3d07e5cb1f620919ff39b89f5640b461e64bb92cf8a2caa399548a292387c3f31741ff0e886231258a66707ce51ba5f85856790faac@127.0.0.1:9800
```

使用的时候需要去掉`?discport=0`

## 4. 测试网络连接

现在我们来测试两个节点是否已连接。使用如下命令从swarmNode1上传文件`fileToUpload.txt`，文件内容为`test file`，上传成功将返回文件哈希

```bash
$ swarm --bzzapi "http://localhost:5000" up fileToUpload.txt
82c5c438f80dc81730ab9d8aeaa8fc433b3d719590f6729872e42c6c0eed59c5
```

复制该哈希作为地址从swarmNode2查询

```bash
$ curl http://localhost:5500/bzz:/your hash comes here/
```

该条命令将返回哈希对应的文件内容`test file`

<br>

可查看[swarm文档](<https://swarm-guide.readthedocs.io/en/latest/introduction.html>)阅读更多细节