---
title: 以太坊开发4-geth console命令
date: 2019-03-08T10:27:00+08:00
lastmod: 2020-11-11
tags: [区块链]
categories: [研究生的区块链学习之路] 
slug: geth console command
---

Geth Console是一个交互式的JavaScript执行环境，其中`>`是命令提示符。在这个环境里也内置了一些用来操作以太坊的JavaScript对象，可以直接使用这些对象。这些对象主要包括：

- eth：包含一些跟操作区块链相关的方法；
- net：包含一些查看p2p网络状态的方法；
- admin：包含一些与管理节点相关的方法；
- miner：包含启动&停止挖矿的一些方法；
- personal：主要包含一些管理账户的方法；
- txpool：包含一些查看交易内存池的方法；
- web3：包含了以上对象，还包含一些单位换算的方法。

<!--more-->

## 1. console命令

### 1.1 简单启动

最简单的启动方式如下

```bash
$ geth console
```

控制台启动成功后，可以看到`>`提示符，等待输入控制台命令

```bash
> 
```

输入`exit`退出

```bash
> exit
```

### 1.2 日志控制

使用`geth console`启动会在交互界面不时出现日志提示，可以将日志输出到文件

```bash
~/Desktop$ geth console 2>>geth.log
```

此时geth.log日志文件位于命令执行时的目录下，比如我执行命令在`~/Desktop`目录，geth.log文件也在这里。可以自己手动打开文件查看日志，当然也可以新开一个终端用`tail`命令查看日志

```bash
$ tail -f geth.log
```

### 1.3 重定向日志

实际上还可以将日志输出到另一个终端，也就是重定向日志。先在2号终端输入：

```bash
$ tty
dev/pts/1
```

获取得到终端编号，然后在1号终端输入如下命令，就可以将日志输出到2号终端

```bash
$ geth console 2>> /dev/pts/1
```

不想看到日志的话也可以选择将日志输出到空的终端

```bash
$ geth console 2>> /dev/null
```

正常的不输出日志的方式实际上并不是使用如上命令，而是使用`--verbosity`来控制日志级别，此时不输出日志的命令如下

```bash
$ geth --verbosity 0 console
```

## 2. attach命令

另一个启动geth交互式JavaScript环境的方法是连接到一个geth节点

```bash
geth attach ipc:{ipc_file_path} # geth.ipc 文件路径
geth attach http://191.168.1.1:8545 # JSONRPC 的地址
geth attach ws://191.168.1.1:8546
```

注意，要连接到的节点在启动时必须开启ipc服务，不然不会产生geth.ipc文件。产生的geth.ipc文件位于存放链数据的data文件下，即`datadir`命令后面跟的文件目录，和keystort文件以及geth文件位于同一级目录。

```bash
$ tree ~/Desktop -L 2
/home/shuzang/Desktop
├── data
│   ├── geth
│   ├── geth.ipc
│   ├── history
│   └── keystore
├── genesis.json
└── geth.log

3 directories, 4 files
```

## 3. Management API

该部分翻译自：[management API](https://github.com/ethereum/go-ethereum/wiki/Management-APIs)

除官方 [DApp APIs](https://github.com/ethereum/wiki/wiki/JSON-RPC) 接口外，go-ethereum还支持额外的一些Management API。与DApp API类似，这些也是使用JSON-RPC提供的，并遵循完全相同的规范。这些API正是应用在Geth提供的控制台中。

除了官方公布的DApp API（如eth，shh，[web3](https://web3js.readthedocs.io/en/1.0/getting-started.html)）之外，额外的Management API列表如下：

- `admin`: Geth node management
- `debug`: Geth node debugging
- `miner`: Miner and [DAG](https://github.com/ethereum/wiki/wiki/Ethash-DAG) management
- `personal`: Account management
- `txpool`: Transaction pool inspection

| admin                                                        | debug                                                        | miner                                                        | personal                                                     | txpool                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| addPeer                                                      | backtraceAt                                                  | [setExtra](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#miner_setextra) | [ecRecover](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_ecrecover) | [content](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#txpool_content) |
| [datadir](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#datadir) | blockProfile                                                 | [setGasPrice](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#miner_setgasprice) | [importRawKey](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_importrawkey) | [inspect](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#txpool_inspect) |
| [nodeInfo](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#admin_nodeinfo) | [cpuProfile](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_cpuProfile) | [start](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#miner_start) | [listAccounts](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_listaccounts) | [status](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#txpool_status) |
| [peers](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#admin_peers) | [dumpBlock](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_dumpblock) | [stop](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#miner_stop) | [lockAccount](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_lockaccount) |                                                              |
| [setSolc](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#admin_setcolc) | [gcStats](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_gcStats) | [getHashrate](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#miner_gethashrate) | [newAccount](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_newaccount) |                                                              |
| [startRPC](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#admin_startrpc) | [getBlockRlp](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_getblockrlp) | [setEtherbase](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#miner_setetherbase) | [unlockAccount](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_unlockaccount) |                                                              |
| [startWS](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#admin_startws) | [goTrace](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_goTrace) |                                                              | [sendTransaction](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_sendtransaction) |                                                              |
| [stopRPC](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#admin_stoprpc) | [memStats](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_memStats) |                                                              | [sign](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_sign) |                                                              |
| [stopWS](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#admin_stopws) | [seedHash](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_seedhash)[sign](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal_sign) |                                                              |                                                              |                                                              |
|                                                              | setBlockProfileRate                                          |                                                              |                                                              |                                                              |
|                                                              | [setHead](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_sethead) |                                                              |                                                              |                                                              |
|                                                              | [stacks](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_stacks) |                                                              |                                                              |                                                              |
|                                                              | [startCPUProfile](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_startCPUProfile) |                                                              |                                                              |                                                              |
|                                                              | [startGoTrace](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_startGoTrace) |                                                              |                                                              |                                                              |
|                                                              | [stopCPUProfile](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_stopCPUProfile) |                                                              |                                                              |                                                              |
|                                                              | [stopGoTrace](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_stopGoTrace) |                                                              |                                                              |                                                              |
|                                                              | traceBlock                                                   |                                                              |                                                              |                                                              |
|                                                              | traceBlockByNumber                                           |                                                              |                                                              |                                                              |
|                                                              | [traceBlockByHash](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_blockbyhash) |                                                              |                                                              |                                                              |
|                                                              | [traceBlockFromFile](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_traceblockfromfile) |                                                              |                                                              |                                                              |
|                                                              | [traceTransaction](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_tracetransaction) |                                                              |                                                              |                                                              |
|                                                              | [verbosity](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_verbosity) |                                                              |                                                              |                                                              |
|                                                              | [vmodule](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_vmodule) |                                                              |                                                              |                                                              |
|                                                              | writeBlockProfile                                            |                                                              |                                                              |                                                              |
|                                                              | writeMemProfile                                              |                                                              |                                                              |                                                              |

### 3.1 Admin

`admin`API给了我们几种非标准的RPC方法，允许我们对Geth实例进行细粒度的控制，包括但不限于网络节点和RPC的端点管理。

#### admin.addPeer(url)

该方法请求添加一个新的远程节点到跟踪的静态节点列表中，节点将始终尝试保持与这些节点的连接，如果远程节点断开，则每隔一段时间尝试重连一次。

```javascript
> admin.addPeer("enode://a979fb575495b8d6db44f750317d0f4622bf4c2aa3365d6af7c284339968eef29b69ad0dce72a4d8db5ebb4968de0e3bec910127f134779fbcb0cb6d3331163c@52.16.188.185:30303")
true
```

####  admin.datadir

用于查找正在运行的Geth节点当前用于存储的数据库的绝对路径

```javascript
> admin.datadir
"/home/Karalabe/.ethereum"
```

####  admin.nodeInfo

查询当前网络正在运行的Geth节点的所有已知信息，包括节点作为P2P网络参与者的本身的信息和它自己运行的应用协议（e.g. eth,les,shh,bzz）的专有信息。

```javascript
> admin.nodeInfo
{
  enode: "enode://44826a5d6a55f88a18298bca4773fca5749cdc3a5c9f308aa7d810e9b31123f3e7c5fba0b1d70aac5308426f47df2a128a6747040a3815cc7dd7167d03be320d@[::]:30303",
  id: "44826a5d6a55f88a18298bca4773fca5749cdc3a5c9f308aa7d810e9b31123f3e7c5fba0b1d70aac5308426f47df2a128a6747040a3815cc7dd7167d03be320d",
  ip: "::",
  listenAddr: "[::]:30303",
  name: "Geth/v1.5.0-unstable/linux/go1.6",
  ports: {
    discovery: 30303,
    listener: 30303
  },
  protocols: {
    eth: {
      difficulty: 17334254859343145000,
      genesis: "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3",
      head: "0xb83f73fbe6220c111136aefd27b160bf4a34085c65ba89f24246b3162257c36a",
      network: 1
    }
  }
}
```

####  admin.peers

和admin.nodeInfo相似，但查询的是所有连接的远程节点的信息

```Javascript
> admin.peers
[{
    caps: ["eth/61", "eth/62", "eth/63"],
    id: "08a6b39263470c78d3e4f58e3c997cd2e7af623afce64656cfc56480babcea7a9138f3d09d7b9879344c2d2e457679e3655d4b56eaff5fd4fd7f147bdb045124",
    name: "Geth/v1.5.0-unstable/linux/go1.5.1",
    network: {
      localAddress: "192.168.0.104:51068",
      remoteAddress: "71.62.31.72:30303"
    },
    protocols: {
      eth: {
        difficulty: 17334052235346465000,
        head: "5794b768dae6c6ee5366e6ca7662bdff2882576e09609bf778633e470e0e7852",
        version: 63
      }
    }
}, /* ... */ {
    caps: ["eth/61", "eth/62", "eth/63"],
    id: "fcad9f6d3faf89a0908a11ddae9d4be3a1039108263b06c96171eb3b0f3ba85a7095a03bb65198c35a04829032d198759edfca9b63a8b69dc47a205d94fce7cc",
    name: "Geth/v1.3.5-506c9277/linux/go1.4.2",
    network: {
      localAddress: "192.168.0.104:55968",
      remoteAddress: "121.196.232.205:30303"
    },
    protocols: {
      eth: {
        difficulty: 17335165914080772000,
        head: "5794b768dae6c6ee5366e6ca7662bdff2882576e09609bf778633e470e0e7852",
        version: 63
      }
    }
}]
```

#### admin.setSolc(path)

当调用`eth_compileSolidity`RPC方法时用来设置Solidity编译器路径，不设置的话，默认路径是`/usr/bin/solc`

```javascript
> admin.setSolc("/usr/bin/solc")
"solc, the solidity compiler commandline interface
 Version: 0.3.2-0/Release-Linux/g++/Interpreter

 path: /usr/bin/solc"
```

####  admin.startRPC(host, port, cors, apis)

开启一个基于JSON RPC的HTTP Webserver来处理客户端请求，参数含义如下：

- `host`: network interface to open the listener socket on (默认`"localhost"`)
- `port`: network port to open the listener socket on (默认`8545`)
- `cors`: [cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) header to use (默认 `""`)
- `apis`:通过此接口提供的API模块 (默认 `"eth,net,web3"`)

返回一个布尔值表示HTTP RPC监听是否打开，任何时候只允许一个HTTP端点保持活动

```javascript
> admin.startRPC("127.0.0.1",8545)
true
```

#### admin.startWS(host, port, cors, apis)

和RPC相似，不同的是开启的是WebSocket Websever。Example

```Javascript
> admin.startWS("127.0.0.1", 8546)
true
```

#### admin.stopRPC()

和RPC开启对应，关闭HTTP Webserver

```Javascript
> admin.stopRPC()
true
```

####  admin.stopWS()

和WebSocket开启对应，关闭WebSocket Webserver

```Javascript
> admin.stopWS()
true
```

> Q：HTTP和WebSocket有啥区别
>
> A：[HTTP协议和WebSocket协议（一）](https://www.cnblogs.com/Jamie1032797633/p/9342957.html), [HTTP协议和WebSocket协议（二）](https://www.cnblogs.com/Jamie1032797633/p/9343000.html)

### 3.2 Debug

提供一些非标准的RPC方法，运行我们在运行时检查、调试和设置某些调试标志

这部分的话从上面表格也看出来了，方法比较多，而且是调试用，就不细列了，用的时候查就行了。

### 3.3 Miner

允许我们远程控制节点的挖矿操作并进行各种挖矿相关的设置

- miner.setExtra(string)：设置一个矿工挖矿时可以包含的额外数据，最大32字节
- miner.setGasPrice(number)：设置挖矿时接受的最低Gas价格，低于此价格的交易不会被收集到区块
- miner.start(number)：以给定数量的线程启动CPU挖矿进程，如果需要生成新的DAG
- miner.stop()：停止CPU挖矿操作
- miner.setEtherbase(address)：设置挖矿奖励账户地址

### 3.4 Personal

管理私钥

#### personal.listAccounts

返回密钥库中所有密钥对应的以太坊账户地址

```javascript
> personal.listAccounts
["0x5e97870f263700f46aa00d967821199b9bc5a120", "0x3d80b31a78c30fc628f20b2c89d7ddbf6e53cedc"]
```

####  personal.newAccount()

生成新的私钥并将其存储在密钥库目录中。密钥文件使用给定的密码加密。返回新帐户的地址。未提供参数时，会在控制台提示输入密码。

```javascript
> personal.newAccount()
Passphrase: 
Repeat passphrase: 
"0x5e97870f263700f46aa00d967821199b9bc5a120"
```

也可以直接提供参数作为密码

```javascript
> personal.newAccount("h4ck3r")
"0x3d80b31a78c30fc628f20b2c89d7ddbf6e53cedc"
```

####  personal.unlockAccount(address, passphrase, duration)

使用密码解密密钥库中某个地址对应的账户。使用JavaScript控制台时，密码短语和解锁持续时间都是可选的。如果密码短语未作为参数提供，控制台将以交互方式提示输入密码短语。

解锁持续时间默认为300秒，geth退出持续时间也会结束。帐户解锁时可以与eth_sign和eth_sendTransaction一起使用。

```javascript
> personal.unlockAccount("0x5e97870f263700f46aa00d967821199b9bc5a120")
Unlock account 0x5e97870f263700f46aa00d967821199b9bc5a120
Passphrase: 
true
```

提供密码字段和持续时间作为参数

```javascript
> personal.unlockAccount("0x5e97870f263700f46aa00d967821199b9bc5a120", "foo", 30)
true
```

如果不想将密码字段作为参数显式输入，但又想设置持续时间，如下

```javascript
> personal.unlockAccount("0x5e97870f263700f46aa00d967821199b9bc5a120", null, 30)
Unlock account 0x5e97870f263700f46aa00d967821199b9bc5a120
Passphrase: 
true
```

#### personal.sendTransaction(tx, passphrase)

验证给定的密码短语并提交交易。该交易与eth_sendTransaction的参数相同，并包含from地址。如果验证了密码短语可以用来解锁`tx.from`的账户，签名并发送到网络上。该帐户未在节点中全局解锁，不能在其他RPC调用中使用。

```javascript
> var tx = {from: "0x391694e7e0b0cce554cb130d723a9d27458f9298", to: "0xafa3f8684e54059998bc3a7b0d2b0da075154d66", value: web3.toWei(1.23, "ether")}
undefined
> personal.sendTransaction(tx, "passphrase")
0x8474441674cdd47b35b875fd1a530b800b51a5264b9975fb21129eeb8c18582f
```

#### personal.sign(message, account, [password])

```javascript
> personal.sign("0xdeadbeaf", "0x9b2055d370f73ec7d8a03e965129118dc8f5bf83", "")
"0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
```

#### personal.ecRecover(message, signature)

返回与某个私钥相关联的地址，该私钥用于`personal.sign`中计算签名

```javascript
> personal.sign("0xdeadbeaf", "0x9b2055d370f73ec7d8a03e965129118dc8f5bf83", "")
"0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b"
> personal.ecRecover("0xdeadbeaf", "0xa3f20717a250c2b0b729b7e5becbff67fdaef7e0699da4de7ca5895b02a170a12d887fd3b17bfdce3481f10bea41f45ba9f709d39ce8325427b57afcfc994cee1b")
"0x9b2055d370f73ec7d8a03e965129118dc8f5bf83"
```

####  personal.lockAccount(address)

从内存中删除具有给定地址的私钥。该帐户不能再用于发送交易。

#### personal.importRawKey(keydata, passphrase)

将给定的未加密私钥（十六进制字符串）导入密钥存储区，并使用密码短语对其进行加密。返回新帐户的地址

### 3.5 Txpool

给我们访问交易池和排队队列中待处理交易的权限，交易池中是所有挂起的交易

#### txpool.content

返回交易池和待处理交易队列中的内容

```javascript
> txpool.content
{
  pending: {
    0x0216d5032f356960cd3749c31ab34eeff21b3395: {
      806: [{
        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
        blockNumber: null,
        from: "0x0216d5032f356960cd3749c31ab34eeff21b3395",
        gas: "0x5208",
        gasPrice: "0xba43b7400",
        hash: "0xaf953a2d01f55cfe080c0c94150a60105e8ac3d51153058a1f03dd239dd08586",
        input: "0x",
        nonce: "0x326",
        to: "0x7f69a91a3cf4be60020fb58b893b7cbb65376db8",
        transactionIndex: null,
        value: "0x19a99f0cf456000"
      }]
    },
    0x24d407e5a0b506e1cb2fae163100b5de01f5193c: {
      34: [{
        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
        blockNumber: null,
        from: "0x24d407e5a0b506e1cb2fae163100b5de01f5193c",
        gas: "0x44c72",
        gasPrice: "0x4a817c800",
        hash: "0xb5b8b853af32226755a65ba0602f7ed0e8be2211516153b75e9ed640a7d359fe",
        input: "0xb61d27f600000000000000000000000024d407e5a0b506e1cb2fae163100b5de01f5193c00000000000000000000000000000000000000000000000053444835ec580000000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        nonce: "0x22",
        to: "0x7320785200f74861b69c49e4ab32399a71b34f1a",
        transactionIndex: null,
        value: "0x0"
      }]
    }
  },
  queued: {
    0x976a3fc5d6f7d259ebfb4cc2ae75115475e9867c: {
      3: [{
        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
        blockNumber: null,
        from: "0x976a3fc5d6f7d259ebfb4cc2ae75115475e9867c",
        gas: "0x15f90",
        gasPrice: "0x4a817c800",
        hash: "0x57b30c59fc39a50e1cba90e3099286dfa5aaf60294a629240b5bbec6e2e66576",
        input: "0x",
        nonce: "0x3",
        to: "0x346fb27de7e7370008f5da379f74dd49f5f2f80f",
        transactionIndex: null,
        value: "0x1f161421c8e0000"
      }]
    },
    0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a: {
      2: [{
        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
        blockNumber: null,
        from: "0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a",
        gas: "0x15f90",
        gasPrice: "0xba43b7400",
        hash: "0x3a3c0698552eec2455ed3190eac3996feccc806970a4a056106deaf6ceb1e5e3",
        input: "0x",
        nonce: "0x2",
        to: "0x24a461f25ee6a318bdef7f33de634a67bb67ac9d",
        transactionIndex: null,
        value: "0xebec21ee1da40000"
      }],
      6: [{
        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
        blockNumber: null,
        from: "0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a",
        gas: "0x15f90",
        gasPrice: "0x4a817c800",
        hash: "0xbbcd1e45eae3b859203a04be7d6e1d7b03b222ec1d66dfcc8011dd39794b147e",
        input: "0x",
        nonce: "0x6",
        to: "0x6368f3f8c2b42435d6c136757382e4a59436a681",
        transactionIndex: null,
        value: "0xf9a951af55470000"
      }, {
        blockHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
        blockNumber: null,
        from: "0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a",
        gas: "0x15f90",
        gasPrice: "0x4a817c800",
        hash: "0x60803251d43f072904dc3a2d6a084701cd35b4985790baaf8a8f76696041b272",
        input: "0x",
        nonce: "0x6",
        to: "0x8db7b4e0ecb095fbd01dffa62010801296a9ac78",
        transactionIndex: null,
        value: "0xebe866f5f0a06000"
      }],
    }
  }
}
```

#### txpool.inspect

和`txpool.content`相似，不同的是只列出所有交易摘要，而`txpool.content`是列出前几个交易的详细内容

```javascript
> txpool.inspect
{
  pending: {
    0x26588a9301b0428d95e6fc3a5024fce8bec12d51: {
      31813: ["0x3375ee30428b2a71c428afa5e89e427905f95f7e: 0 wei + 500000 × 20000000000 gas"]
    },
    0x2a65aca4d5fc5b5c859090a6c34d164135398226: {
      563662: ["0x958c1fa64b34db746925c6f8a3dd81128e40355e: 1051546810000000000 wei + 90000 × 20000000000 gas"],
      563663: ["0x77517b1491a0299a44d668473411676f94e97e34: 1051190740000000000 wei + 90000 × 20000000000 gas"],
      563664: ["0x3e2a7fe169c8f8eee251bb00d9fb6d304ce07d3a: 1050828950000000000 wei + 90000 × 20000000000 gas"],
      563665: ["0xaf6c4695da477f8c663ea2d8b768ad82cb6a8522: 1050544770000000000 wei + 90000 × 20000000000 gas"],
      563666: ["0x139b148094c50f4d20b01caf21b85edb711574db: 1048598530000000000 wei + 90000 × 20000000000 gas"],
      563667: ["0x48b3bd66770b0d1eecefce090dafee36257538ae: 1048367260000000000 wei + 90000 × 20000000000 gas"],
      563668: ["0x468569500925d53e06dd0993014ad166fd7dd381: 1048126690000000000 wei + 90000 × 20000000000 gas"],
      563669: ["0x3dcb4c90477a4b8ff7190b79b524773cbe3be661: 1047965690000000000 wei + 90000 × 20000000000 gas"],
      563670: ["0x6dfef5bc94b031407ffe71ae8076ca0fbf190963: 1047859050000000000 wei + 90000 × 20000000000 gas"]
    },
    0x9174e688d7de157c5c0583df424eaab2676ac162: {
      3: ["0xbb9bc244d798123fde783fcc1c72d3bb8c189413: 30000000000000000000 wei + 85000 × 21000000000 gas"]
    },
    0xb18f9d01323e150096650ab989cfecd39d757aec: {
      777: ["0xcd79c72690750f079ae6ab6ccd7e7aedc03c7720: 0 wei + 1000000 × 20000000000 gas"]
    },
    0xb2916c870cf66967b6510b76c07e9d13a5d23514: {
      2: ["0x576f25199d60982a8f31a8dff4da8acb982e6aba: 26000000000000000000 wei + 90000 × 20000000000 gas"]
    },
    0xbc0ca4f217e052753614d6b019948824d0d8688b: {
      0: ["0x2910543af39aba0cd09dbb2d50200b3e800a63d2: 1000000000000000000 wei + 50000 × 1171602790622 gas"]
    },
    0xea674fdde714fd979de3edf0f56aa9716b898ec8: {
      70148: ["0xe39c55ead9f997f7fa20ebe40fb4649943d7db66: 1000767667434026200 wei + 90000 × 20000000000 gas"]
    }
  },
  queued: {
    0x0f6000de1578619320aba5e392706b131fb1de6f: {
      6: ["0x8383534d0bcd0186d326c993031311c0ac0d9b2d: 9000000000000000000 wei + 21000 × 20000000000 gas"]
    },
    0x5b30608c678e1ac464a8994c3b33e5cdf3497112: {
      6: ["0x9773547e27f8303c87089dc42d9288aa2b9d8f06: 50000000000000000000 wei + 90000 × 50000000000 gas"]
    },
    0x976a3fc5d6f7d259ebfb4cc2ae75115475e9867c: {
      3: ["0x346fb27de7e7370008f5da379f74dd49f5f2f80f: 140000000000000000 wei + 90000 × 20000000000 gas"]
    },
    0x9b11bf0459b0c4b2f87f8cebca4cfc26f294b63a: {
      2: ["0x24a461f25ee6a318bdef7f33de634a67bb67ac9d: 17000000000000000000 wei + 90000 × 50000000000 gas"],
      6: ["0x6368f3f8c2b42435d6c136757382e4a59436a681: 17990000000000000000 wei + 90000 × 20000000000 gas", "0x8db7b4e0ecb095fbd01dffa62010801296a9ac78: 16998950000000000000 wei + 90000 × 20000000000 gas"],
      7: ["0x6368f3f8c2b42435d6c136757382e4a59436a681: 17900000000000000000 wei + 90000 × 20000000000 gas"]
    }
  }
}
```

#### txpool.status

和前两个相比更简单，只返回交易数目

```javascript
> txpool.status
{
  pending: 10,
  queued: 7
}
```

### 3.6 net和web3

这两个也有使用，但是没找到Javascript控制台下命令格式的具体说明

## 四、常用命令

- personal.newAccount()：创建账户；
- personal.unlockAccount()：解锁账户；
- eth.accounts：枚举系统中的账户；
- eth.getBalance()：查看账户余额，返回值的单位是 Wei（Wei 是以太坊中最小货币面额单位，类似比特币中的聪，1 ether = 10^18 Wei）；
- eth.blockNumber：列出区块总数；
- eth.getTransaction()：获取交易；
- eth.getBlock()：获取区块；
- miner.start()：开始挖矿；
- miner.stop()：停止挖矿；
- web3.fromWei()：Wei 换算成以太币；
- web3.toWei()：以太币换算成 Wei；
- txpool.status：交易池中的状态；
- admin.addPeer()：连接到其他节点；

