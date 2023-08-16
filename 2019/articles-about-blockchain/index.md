# 区块链方向有参考价值的文章收集


很多区块链领域极有启发性的文章或者介绍极为详细的文章都很值得保存，以前直接将文章完整的转载过来，但最近发现这种方法占用空间而且毫无意义，因此专门开一篇博客用来收集和介绍这些文章，只记录它们的链接并作简单介绍。逆序排列，最新收集的文章在最前，同样，越往前序号越大。

## 共识协议

时间：2020.01.06

文章链接：[Consensus Protocols That Meet Different Business Demands]( https://blockchain.intellectsoft.net/blog/consensus-protocols-that-meet-different-business-demands/ )

一共两篇，详细介绍了常见的各种共识协议。

## 区块链交易打包过程

时间：2019.04.03

文章链接(唐霜的个人博客)：[区块链中，交易被如何打包进区块](https://www.tangshuang.net/4097.html)

通过这篇文章要弄清楚的问题是：矿工优先打包交易费高的交易，会不会遗漏某些区块？

大部分材料都详细分析了挖矿过程，介绍了区块是如何产生的。然而，区块的产生并不是区块链的最终目的，保存交易信息才是区块链的最终目的。所以，更重要的一点是要理解，交易信息是如何被打包进区块链的。

## Hyperledger搭建

时间：2018.12.25

文章转自IBM，地址为：[英文版](https://www.ibm.com/developerworks/cloud/library/cl-model-test-your-blockchain-network-with-hyperledger-composer-playground/index.html) ,[中文版](https://www.ibm.com/developerworks/cn/cloud/library/cl-model-test-your-blockchain-network-with-hyperledger-composer-playground/index.html?ca=drs-)

Introduction:

整个教程分三部分， 第1部分学习如何在 Hyperledger Composer Playground 的本地版本中建模并测试一个简单的业务网络，第 2 部分学习如何改进和部署区块链网络，第 3 部分学习如何在计算机上安装 Hyperledger Fabric，将业务网络部署到本地实例以及与示例网络区块链应用交互。

## 比特币脚本

时间：2019.11.27

文章链接(来自CSDN)：[谈谈自己对比特币脚本的理解](https://blog.csdn.net/pony_maggie/article/details/73656597)

Introduction：

比特币脚本存在的意义是自动化的验证交易的合法性，分为锁定脚本和解锁脚本两种。假设Alice要向 bob支付0.015比特币, Alice会用到一个UTXO(假设是单输入，单输出)，这个UTXO带有一个**锁定脚本**，为交易设置“障碍”。 bob如果要接收这笔比特币(另一种说法是bob可以引用该笔输出)，就要给出一个**解锁脚本**,然后解锁脚本和锁定脚本组合后执行的结果为真才能确认交易有效。  脚本是简单的堆栈语言，是非图灵完备的，这篇文章详细解释了锁定脚本与解锁脚本的运行机理。

## 日蚀攻击

时间：2019.04.08

文章链接(来自知乎)：[比特币点对点网络中的日蚀攻击](https://zhuanlan.zhihu.com/p/42446193)

Introduction：

这篇文章是对同名论文[Eclipse Attacks on Bitcoin's Peer-to-Peer Network](https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-heilman.pdf)原理和思想的解释，实际上针对的还不是原论文，是论文作者的讲解视频。

## IoT数据

时间：2019.04.10

文章链接(来自IBM)：[了解IoT数据](https://www.ibm.com/developerworks/cn/iot/library/iot-lp301-iot-manage-data/index.html)

Introduction：

随着越来越多的事物连接到物联网，与 IoT 设备相关联的数据量及其生成的数据量（包括设备状态、元数据和传感器读数）呈指数级增长。如果 IoT 解决方案要实现价值，那么管理和了解这些数据至关重要。这篇文章介绍一些处理 IoT 数据的方法，包括存储数据、处理和分析数据以及应用规则。讲解的相当深入。

## 区块链改善学术界

时间：2019.02.26

英文原文(来自medium)：[Ideas on how to improve scientific research](https://medium.com/@barmstrong/ideas-on-how-to-improve-scientific-research-9e2e56474132)

中译文(来自知乎)：[Coinbase CEO：区块链或可改善学术届，科研将变得像编程一样开放](https://zhuanlan.zhihu.com/p/57732457)

Introduction：

科学论文之于人类持续进化的重要性不言而喻，每年，全世界范围有数百万篇论文会被发表，它们代表着人类智慧的最新成果，但其中只有一小部分真正带来了新的产品和服务，人们可真正从中获益。
然而，传统的学术环境在为人类带来创新的同时，也存在着大量的落后特性，我们不禁思考，能否改善学术界的环境，使得人类发展能够变得更快、更开放？
来自区块链独角兽公司Coinbase的首席执行官Brian Armstrong，为我们提供了一些非常棒的想法，我们或许可结合区块链token和市场上已存在的技术，使得学术研究变得像Github那样开放，人人都可参与学术研究和讨论，倘若实现这一想法，学术研究者们将变得不再苦逼，人类文明因此而将更好地发展…… 

---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2019/articles-about-blockchain/  

