# 区块链学习7-交易池底层实现


起源于老师提出的一个问题：区块链是如何处理同时发起的多个请求的。想了想觉得核心是交易池机制，因此准备看一下交易池的原理和实现。

<!--more-->

针对这个问题其实我们要找以下几个问题的答案：

1. 用户发起的交易请求（不论以何种方式）是如何转变为实际的交易的；
2. 产生的交易如果不能被及时处理，是如何进入交易池的；
3. 本地产生的交易和从网络中接收到的交易进入交易池是否有区别；
4. 交易池的基本数据结构是怎么样的（优先队列？）
5. 从交易池中提取交易进行打包时顺序是怎么样的（调度算法）

入手主要是通过登链社区的 [以太坊交易流程及交易池 Txpool 分析](https://learnblockchain.cn/2019/06/03/eth-txpool/) 这篇文章，通过它快速找到了源码中的相关函数，因为主要使用的是 Quorum，所以看的是 Quorum 的源码。

注：网络中很多文章提到内存池，指的也是txpool

## 1. 交易形成

我们的交易请求最终会赋值到 SendTxArgs 结构体的一个实例中

```go
type SendTxArgs struct {
 PrivateTxArgs // Quorum

 From     common.Address  `json:"from"`
 To       *common.Address `json:"to"`
 Gas      *hexutil.Uint64 `json:"gas"`
 GasPrice *hexutil.Big    `json:"gasPrice"`
 Value    *hexutil.Big    `json:"value"`
 Nonce    *hexutil.Uint64 `json:"nonce"`
 Data  *hexutil.Bytes `json:"data"`
 Input *hexutil.Bytes `json:"input"`
}
```

这个实例被传递给 `quorum/internal/ethapi/api.go` 的 `SendTransaction` 函数用来创建一个交易。创建交易的过程如下

1. 根据 From 字段找到当前账户
2. 设置交易默认参数
3. 对交易进行序列化，变为可存储和传输的形式。
4. 根据 To 字段决定是创建部署合约交易还是调用合约交易
5. 对交易进行 RLP 编码并根据之前获得的账户密钥对交易进行签名
6. 提交交易到交易池

序列化主要处理 SendTxArgs 结构中的 Data 和 Input 字段，Data 字段主要用于向前兼容，应尽量使用 Input 字段。当部署合约的时候，Input 是合约代码，当发送交易的时候，Input 是交易的内容。

## 2. 交易添加到交易池

SendTransaction 最后调用 `SubmitTransaction` 函数将交易提交到交易池，不过，更底层的调用是 `quourm/core/tx_pool.go` 的 AddLocals 函数，这里还应该提到，来自网络的交易会调用 AddRemotes 函数。

需要注意的是，调用这两个函数之前都应该验证交易的有效性。同时，这两个函数底层都调用 addTxs 函数，最终的调用是 add 函数。不过在介绍 add 函数前先了解一下交易池的结构。

交易池是一个非常复杂的结构体，但最核心的字段只有两个 `pending` 和 `queue`

```go
type TxPool struct {
    pending map[common.Address]*txList   // All currently processable transactions
 queue   map[common.Address]*txList   // Queued but non-processable transactions
}
```

[add](https://github.com/ConsenSys/quorum/blob/d51931173bde132243a87e7a2adadef4abe58470/core/tx_pool.go#L601) 函数比较复杂，但添加交易到交易池的逻辑很简单

1. 验证交易的有效性
2. 如果 nonce 已存在，且 pending 中旧交易的 price 没有新交易高，会被新交易替换掉
3. 如果 nonce 不存在，不可以替换 pending 中的任何交易，此时将新的交易插入 queue 的末尾

注：交易中的 nonce 指的是 from 账户发出交易的次数, 从0开始递增，同一账户的交易会被依次确认，所以同一个 nonce 代表是同一个交易，会优先选择 price 更高的交易。

## 3. 清理交易池

交易池是完全存在内存中的，因此有大小限制，每当超过一定的阈值就需要清理。实际实现时，pending 的缓冲区容量默认为 4096，queue 的缓冲区容量默认为 1024。

清理的时机是交易池满的时候，清理的原则是价格较低的最先清理

调用清理函数依然是在 add 函数中

## 4. 重构交易池

作为一个分布式系统，总是会出现一种情况：本地节点已经挑选好最优的交易，并准备好广播给整个网络，结果这个时候矿工已经打包好了一个区块，这时候本地节点的区块头就是旧的了，筛选好的交易也已经可能被打包，此时再广播这些交易就没了意义。

为了避免上述情况的发生，本地节点要随时监听是否有新区块产生，当监听到新区块产生这个事件后，无论是本地节点领先，还是网络上其它节点领先，都回退一个区块号，

![](https://img.learnblockchain.cn/2019/06/15596364439683.png!wl/scale/60)

本地节点回退时，把撤销的交易保持到 discarded 切片中，网络上其他节点的撤销交易保存在 `included` 切片中。

当区块号一致的时候，还需要进一步的比较区块的 `Hash` 来进一步确认区块里面的交易是否一致，如果不一致一致回退到区块 Hash 为止，回退撤销的交易依旧保存在 `discarded` 和 `included` 切片中。

等完全确认本地和网络的链没有分叉的时候，就需要比较 discarded 和 included 里面的交易，因为网络上区块的生成优先级高于本地，所以需要剔除 `discarded` 中 `inclueded` 的交易，生成 `reinject` 切片，剔除完以后还需要对 `TXpool` 按照网络新生成区块的信息设置世界状态等信息，设置完以后，重新将 `reinject` 加入 `TXpool`，加入以后在进行验证清理等流程。

## 5. 问题回答

回答文章开头提出的几个问题

1. 用户发起的交易请求（不论以何种方式）是如何转变为实际的交易的；

   所有与交易请求相关的参数被赋值到一个结构体中，然后进行序列化转变为可存储和传输的形式，最后生成交易并进行签名

2. 产生的交易如果不能被及时处理，是如何进入交易池的

   最终是调用一个 add 函数，添加到了一个队列里

3. 本地产生的交易和从网络中接收到的交易进入交易池是否有区别；

   没有区别，底层都是调用 add 函数

4. 交易池的基本数据结构是怎么样的（优先队列？）

   交易池是一个结构体，核心是 pending 和 queue 两个 map，map 的键是一个地址，值是一个交易链表形成的队列

5. 从交易池中提取交易进行打包时顺序是怎么样的（调度算法）

   price 越高优先级越大

我们可以理解为区块链底层利用交易池对并发产生的请求做了异步化，交易产生的时刻和交易被打包的时刻是随机的。

这里面我们可以视作有一个排队论的问题，相关度比较高的论文有两篇

[1] J. Li, Y. Yuan, S. Wang and F. Wang, "[Transaction Queuing Game in Bitcoin BlockChain](https://ieeexplore.ieee.org/document/8500403)," *2018 IEEE Intelligent Vehicles Symposium (IV)*, Changshu, 2018, pp. 114-119, doi: 10.1109/IVS.2018.8500403.

[2] Memon RA, Li JP, Ahmed J. [Simulation Model for Blockchain Systems Using Queuing Theory](https://www.mdpi.com/2079-9292/8/2/234#cite). *Electronics*. 2019; 8(2):234.

后注1：在实现 TXpool 的时候为了保证数据的一致性会使用大量的锁

后注2：总结以下可以发现交易池中交易的顺序与以下几方面有关

1. 交易费
2. 交易哈希（重构交易池时区块相同会进行比较）
3. 在交易池中的时间（时间过长可能会被清除）

## 6. 时间

更全面的描述可以参考 [以太坊技术与实现：交易池](https://learnblockchain.cn/books/geth/part2/txpool.html)

我们关心发起交易的时间和智能合约执行并返回结果的时间是否有区别


---

> 作者:   
> URL: https://shuzang.github.io/2020/transaction-and-txpool/  

