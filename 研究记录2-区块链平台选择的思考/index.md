# 研究记录2-区块链平台选择的思考


首先我们就 Ethereum 等九个平台做分析比较，从而确定要用来作为实验平台的区块链。

## 1. Ethereum

### 关于存储

以太坊生态中的一些工具和技术是用来解决大数据量存储问题的，比如Swarm和IPFS。Swarm 是一个去中心化的内容存储和分发服务。 您可以将它视为 CDN，但它并不是在一家公司的服务器上托管的所有 CDN，而是通过互联网在计算机上分发。 就像运行一个以太坊节点一样，来运行一个 Swarm 节点连接到 Swarm 网络。

IPFS在2016年就正式在以太坊分叉（ETH）上推出了它的分布式文件存储网络，在概念上与Swarm完全相似。虽然与以太坊没有直接关联，但可与以太坊集成。

Swarm和ETH的区别见：[IPFS&Swarm](https://github.com/ethersphere/go-ethereum/wiki/IPFS-&-SWARM)      

在以太坊生态中，智能合约smart contract实现了分布式逻辑，Swarm实现了分布式存储，Whisper实现了分布式消息，整体结构如下所示

![YJS_20190401_以太坊生态](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20190401_以太坊生态.jpg)

参考链接：[Swarm文档说明](https://swarm-guide.readthedocs.io/en/latest/introduction.html)，[Swarm内容搜索](http://swarm-gateways.net/)，[Swarm简单入门](https://mitchellcash.com/2016/12/29/getting-started-with-ethereum-s-swarm-on-the-testnet.html)

### 关于测试

以太坊官方目前提供两种网络，同时为了测试，也可以自己搭建私有网络

**主网络**

是产生真正有价值的以太币的网络，目前区块高度超过700万，每个以太币的价值在118美刀左右。优缺点如下表

| 优点                                                 | 缺点                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| 全球化的，部署在Internet环境上                       | 任何智能合约执行都会消耗真实的以太币，也就是真实的现金。不适合开发、调试和测试 |
| 智能合约的代码，执行，区块的调用，都可以清晰的查看到 | 所有节点是全球化的，速度较慢                                 |
| 部署在主网络的智能合约，全世界任何应用都可以调用     | 对于部分商业应用来说，只需要一部分节点，例如分布式部署的10-20台服务器即可，不需要遍布全球的网络 |

**测试网络**

以太坊为了方便智能合约的开发、学习和测试，开启了几条全新的区块链，与主网络特性相同，但测试网络中的以太币价值更低，也更容易得到。这样不至于在主网络上开发出现 BUG 造成以太币的损失。主要的以太坊测试网络有：

- [Ropsten Test Network](https://ropsten.etherscan.io/)
- [Kovan Test Network](https://kovan.etherscan.io/)
- [Rinkeby Test Network](https://rinkeby.etherscan.io/)

测试网络的优缺点如下表

| 优点                                                 | 缺点                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| 合约执行不消化真实货币                               | 所有节点是全球化的，速度较慢                                 |
| 全球化，部署在Internet环境上                         | 测试网络不能作为商业应用的实际落地环境                       |
| 智能合约的代码，执行，区块的调用，都可以清晰的查看到 | 测试网络由官方提供，使用较为直接，对以太坊技术的底层实现，Geth的各种参数接口等理解会差很多 |
| 部署在测试环境上的智能合约，全世界任何应用都可以调用 |                                                              |

**私有网络**

除了官方提供的两种网络，用户还可以使用Geth工具创建自己的私有网络，用来开发、学习和测试。基本的说明见官方的github wiki: [Private network](https://github.com/ethereum/go-ethereum/wiki/Private-network)

私有网络的优点有：

- 方便开发者深入理解以太坊的技术底层
- 因为节点相对较少，速度较快
- 用户可以随时创建，随时销毁，随时重建一个以太坊网络
- 随意的增加节点数目，或者删除节点，
- 既可以在服务器上建立，也可以在自己的windows或者Mac机器上建立，
- 甚至一台机器可以建立多个节点，在一台机器上实现多节点的私有网络

缺点主要是只有在私有网络内的节点才能查看智能合约的执行、调用。目前网上关于如何构建以太坊私有网络的教程很多。但没有实践的情况下，同时也没有在教程中看到如何产生指定的交易类型的方式。

参考链接：[一个简单的以太坊私有网络搭建教程](https://medium.freecodecamp.org/ethereum-69-how-to-set-up-a-fully-synced-blockchain-node-in-10-mins-f6318d7aad40)

### 关于智能合约

以太坊是智能合约的主要代表

## 2. Hyperledger

Hyperledger是一项旨在推动跨行业开源协作的区块链技术。合作者包括金融，银行，物联网，供应链，制造和技术领域的行业领导者。[项目列表](https://www.hyperledger.org/projects)

我们从项目列表看到hyperledger的子项目中，总架构有多种，包括：Sawtooth, Iroha,  Fabric, Burrow和Indy。但目前最流行的当属Fabric，便以Fabric为焦点。

### 关于存储

首先查询了Hyperledger 是否有对IPFS的支持，然后发现有人问过这个问题，以下是回答

> **Q**：Are there plans to integrate hyperledger fabric with IPFS and BigchainDB as an alternative file storage/persistent store? Or perhaps these are already available?
>
> Nathan Aw
>
> **A**：There is no intent to integrate 'directly' with IPFS or BigchainDB. With the Fabric architecture, blocks are distributed at the node layer, and each node maintains it's own dedicated data store. This is in contrast to IPFS and BigchainDB, where data is distributed across the data store itself. 
>
> There are valid design patterns where you can integrate 'indirectly' with such distributed/decentralized data stores however. For example in your solution the client application could store data on a distributed/decentralized data store to share data with business partners, and then store a hash (and perhaps a URL to the data) in Fabric as decentralized 'proof' of the data.
>
> Dave Enyeart  
>
> [原文地址](https://lists.hyperledger.org/g/fabric/topic/17549762)

显然，Hyperledger不直接的集成IPFS等方式，但可以通过其它间接的手段来完成，这里有一个集成BigchainDB到Hyperledger Fabric的方案：[BigchainDB integrates with Hyperledger Fabric](https://blog.bigchaindb.com/bigchaindb-hyperledger-fabric-integration-4c65e5811671)

回答中提到Hyperledger自己的存储与IPFS是背道而驰的，所有我们来看以下Hyperledger的存储。fabric网络实际上有三种类型的数据存储，一种是账本本身，也就是区块链数据，是以文件形式存储；第二种是区块数据和历史数据的索引数据库；第三种是状态数据库，即存储我们在chaincode中执行的业务数据。所谓的“each node maintains it's own dedicated data store"指的就是第三者状态数据库

### 关于测试

Hyperledger是联盟链的典型代表，不过联盟链实际上就是一个范围扩大版的私链。所以我们能利用Fabric完成自己的项目。

使用的工具首选[Composer](https://www.hyperledger.org/projects/composer)，使用Hyperledger Composer完成简单的网络构建和智能合约的编写与运行后，可以继续深入的使用[Fabric](https://www.hyperledger.org/projects/fabric)实际完成项目

重要的是，Composer和Fabric都提供了许多样例场景，可以简化开发

### 关于智能合约

Hyperledger支持智能合约

## 3. Multichain 

号称90秒创建一个私有区块链， 多链(MultiChain)是一个即用型的私有区块链创建平台。隐私与控制的争议是比特币成为行业性金融机构的障碍。而用多链(MultiChain)创建的私有链则克服了这个困难。其特性包括：

- 各项参数可以完全自定义。多链是一种私链，交易和挖矿都要得到控制者的许可才能进行。
- 快速部署。二步就可以生成自已的区块链，三步就可以连接上其它区块链。
- 资产的原生支持。这与彩色币是不同的。
- 兼容比特币

采用间接的方法集成区块链和分布式的存储如IPFS应是众多链均可实现的，所以我们之后的讨论是描述这些区块链是否原生的支持。显然Multichain（多链）是不支持的。

Multichain在2018.12.19发布了2.0 beta版，有了重大改进，正好有我们需要的功能：

 - **Smart Filters**. These allow custom rules to be coded for validating transactions or data. Smart Filters are written in JavaScript and run within a deterministic version of the high-performance V8 engine that powers Google Chrome. Click for [more on Smart Filters](https://www.multichain.com/developers/smart-filters/) or a [comparison with Fabric, Ethereum and Corda](https://www.multichain.com/blog/2018/12/smart-contract-showdown/).
 - **Off-chain data**. Any item published in a MultiChain stream can optionally be stored off-chain, in order to save bandwidth and storage space. Off-chain data (up to 1 GB per item) is automatically hashed into the blockchain, with the data itself delivered rapidly over the peer-to-peer network. Click for [more about off-chain data](https://www.multichain.com/blog/2018/06/scaling-blockchains-off-chain-data/).

第一个Smart Filters是对智能合约的支持，只是换了个说法，第二个Off-chain data是解决数据存储的问题。

关于测试的问题，多链本就是为私链而生的，其各项功能显然有利于我们的实验。但这同时带来一个问题，我们虽然在私链环境下测试，因为这有利于我们对链的控制和对结果的分析，但最终的目标显然是要推广到更广的范围的，现在还不清楚Multichain支持的项目体积。    

参考链接：[Multichain](https://www.multichain.com/)，[Multichain blog](https://www.multichain.com/blog/) 

## 4. HydraChain     

HydraChain是基于以太坊的扩展，用于支持私链和联盟链的构建，100%的兼容以太坊的API和智能合约，唯一大的区别在于共识协议，原文的描述是

```bash
it relies on a registered and accountable set of validators which propose and validate the order of transactions.
```

理论上我们在以太坊部分描述的方式这边能实现，同时，相比于用geth构建私链做实验，这个专门针对私链和联盟链的项目应该拥有更好的特性。唯一的问题正如项目Issue里的第一条：`still active`


github的项目状态处于”Work in Progress"，但最近的版本记录是2016年的。。。很尴尬的情况:joy:

## 5. Openchain 

 项目社区蛮小的，网上的资料也不多，[官网](https://www.openchain.org/)描述的用例是这样的

> Openchain is a generic register of ownership. It can be modelled to work with an immense number of use cases:
>
> - Securities like stocks and bonds, commodities like gold and oil, currencies like the Dollar or even Bitcoin.
> - Titles of ownership like land titles, music or software licensing.
> - Gift cards and loyalty points.

看起来适合用于物品所有权领域，不是很适合我们的方案测试。

## 6. IBM Bluemix Blockchain  

Bluemix 是一个基于开放标准的云平台，用于构建、运行和管理应用程序和服务。​IBM Blockchain 提供私有区块链基础架构来开发受区块链支持的解决方案。Bluemix Blockchain 服务 是 Hyperledger Fabric 的一种实现。它提供了：

- 一个由 4 个对等节点组成的区块链网络
- 一个证书颁发机构服务器
- 智能合约代码（使用 Golang 开发的链代码）
- 全球/账本状态，其中包含智能合约数据的当前值（所有事务的历史记录也包含在区块链中）   

总的来说应该归到2.hyperledgrer里，以下第三个链接的项目里有一个名为marbles的子项目，好像和Composer的作用类似，还给了例子。

参考链接：[IBM Blockchain Platform](https://console.bluemix.net/docs/services/blockchain/index.html#ibm-blockchain-platform)，[IBM Bluemix Blockchain相关说明](https://www.ibm.com/developerworks/cn/cloud/library/cl-blockchain-for-cognitive-iot-apps-trs/)，[IBM Blockchain项目地址](https://github.com/IBM-Blockchain)

## 7. Chain

目标指向为金融领域，而且非开源 。官网地址https://chain.com/

## 8. IOTA        

IOTA专为物联网设计，但却不是真正的区块链结构，而是一种基于有向无环图（DAG）的称为Tangle(缠结)的结构。

参考链接：[IOTA](https://www.iota.org/)，[IOTA中国](https://www.iotachina.com/)

目前来看，IOTA和以太坊两边都有支持者，IOTA的支持者认为它解决物联网中广泛存在的微交易问题，以太坊的支持者则认为IOTA太激进，DAG 区块链还有不少问题，而以太坊则稳扎稳打，采用的技术都足够成熟，长远看更有优势。

但从我们的测试看，IOTA的文档说明很不完善，几乎找不到私人构建的方案教程。再加上DAG 区块链技术相比于传统区块链有较大差异，并不打算使用IOTA。

## 9. EOS  

EOS.IO 软件引入一种新的区块链架构设计，它使得去中心化的应用可以横向和纵向的扩展。 这通过构建一个仿操作系统的方式来实现，在它之上可以构建应用程序。 该软件提供帐户、身份验证、数据库、异步通信和跨越数百个 CPU 内核或集群的应用程序调度。 由此产生的技术是一种区块链架构，它可以扩展至每秒处理百万级交易，消除用户的手续费，并且允许快速和轻松的部署去中心化的应用

以上这段话来自[EOS.IO 技术白皮书](https://github.com/EOSIO/Documentation/blob/master/zh-CN/TechnicalWhitePaper.md)

众多的文章中介绍的EOS的特点包括：

- 支持大量的用户，可能是上亿级别的用户
- 消除手续费
- 超高性能（支持百万级TPS）
- 对IPFS的天然支持
- 使用DPoS协议，不会产生硬分叉

以及其它，目前EOS的吞吐量未达到所称的百万级，大概在12000/秒（据某个博客的数据）

## 结论

综合以上分析，我们初步选定的平台有：Ethereum，Hyperledger，Multichain，EOS。我们的目标是完成IIoT场景下访问控制的一个方案，面对的问题包括物联网设备完成PoW代价过高，实时数据（大数据量）的存储，交易确认的延迟（因为受网络拥堵情况影响，无法判断准确时间，用出块时间代表），物联网场景下并发的设备数量巨大。同时，由于要完成访问控制，还需要智能合约的支持，更由于我们需要对方案进行测试，还需要对所购建的区块链的完全控制能力，即需要选择的平台能支持私链的构建，最后，我们还要考虑该平台的流行度和社区水平，因为这决定了该平台文档的完善程度及遇到问题是否容易解决。对于所有以上所列指标，我们对四个平台总结如下表。

|                      | Ethereum | Hyperledger      | Multichain | EOS     |
| -------------------- | -------- | ---------------- | ---------- | ------- |
| 共识算法             | PoW      | PBET             | PoW        | DPoS    |
| 大数据量存储支持     | Swarm    | 本身的状态数据库 | off-chain  | IPFS    |
| 出块时间             | 15s      | 联盟链           | 私链       | 0.5s    |
| 吞吐量（每秒交易数） | 20/s     | 70-1000/s        |            | 12000/s |
| 智能合约支持         | 支持     | 支持             | 支持       | 支持    |
| 私链构建容易程度     | 中       | 中               | 高         | 中      |
| 开发容易程度         | 中       | 中               | 易         | 难      |

## 其它

### 关于通信

[What Makes a Blockchain: Protocols and the Future](https://bitsapphire.com/makes-blockchain-protocols-future/)

### 关于工具

性能比较与分析这部分内容是 2019.07-2019.08 月关注的问题，这段时期倾向于自行编写一个性能测试平台。

**blockBench**

[blockBench](https://github.com/ooibc88/blockbench) 是一个私链的基准测试框架，目前提供的测试包括 Ethereum, Hyperledger, Parity 和 Quorum。

但该项目已经很久没有维护，[melhindi/blockbench](https://github.com/melhindi/blockbench/tree/master/benchmark) 是一个 Fort 之后的仓库，相比原项目更新一点。

**caliper**

[caliper](https://github.com/hyperledger/caliper)  也是一个区块链基准测试工具，属于 Hyperledger 官方的项目，因此一直在维护，不过主要针对的也是 Hyperledger 旗下各区块链，包括 Fabric，Sawtooth等等。但因为 Ethereum 的特殊性，也有对该平台的测试。

[chainhammer](https://github.com/drandreaskrueger/chainhammer)，以太坊及其衍生的各种区块链比如 Quorum 等之间的比较，包含 TPS、blocktime、gas 和 blocksize，会自动绘制一个图表。

### 关于参考文献

1、Aad van Moorsel. [Benchmarks and Models for Blockchain](https://www.researchgate.net/publication/324235866_Benchmarks_and_Models_for_Blockchain). 2018. 主要介绍区块链基准测试应该测哪些方面

2、Anuj Das Gupta, Andrew Dickson, [Analyzing Performance in Blockchain-Based Systems](https://github.com/stratumn/performance/blob/master/Analyzing Performance in Blockchain-Based Systems.pdf). 2017.11. 介绍区块链性能分析的方法，写的很好，虽然只放在 Github 上，但如果研究这个方向，强烈推荐阅读。

3、Harry Kalodner. et.al. [BlockSci: Design and applications of a blockchain analysis paltform](https://github.com/citp/BlockSci). 2018. 声称比其它比较工具好，但没细看。唯一能确认的是，这是该领域直接相关的一篇论文。

4、Medium，[Running blockbench for ethereum](https://medium.com/@mu7eh7/running-blockbench-for-ethereum-6dca3ed44a35)，见面知意，关于以太坊的一个基准测试的文章，国内需要翻墙。

5、其它几篇论文

- [Evaluating the Efficiency of Blockchains in IoT with Simulations](https://www.scitepress.org/Papers/2017/62405/62405.pdf)
- [Untangling Blockchain: A Data Processing View of Blockchain Systems](https://www.comp.nus.edu.sg/~ooibc/blockchainsurvey.pdf)
- [BLOCKBENCH: A Framework for Analyzing Private Blockchains](https://www.comp.nus.edu.sg/~ooibc/blockbench.pdf)
- [Performance Evaluation of the Quorum Blockchain Platform](https://www.researchgate.net/publication/327570196_Performance_Evaluation_of_the_Quorum_Blockchain_Platform)


