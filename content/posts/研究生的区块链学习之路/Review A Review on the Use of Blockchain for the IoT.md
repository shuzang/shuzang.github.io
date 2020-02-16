---
title: A Review on the Use of Blockchain for the Internet of Things
date: 2019-01-15T19:49:00+08:00
tags: [论文笔记]
categories: [研究生的区块链学习之路]
---

### Abstract

物联网世界中，我们的许多日常物品将相互连接并与环境互动，以便收集信息并自动执行某些任务。达成这样的愿景还需要解决其它方面的问题，包括无缝身份验证，数据隐私，安全性，抗攻击的鲁棒性，易于部署和自我维护。这些特性可以通过区块链来实现。本文详细介绍了如何使区块链适应物联网的特定需求，以开发基于区块链的物联网（BIoT）应用。在描述了区块链的基础知识之后，描述了最相关的BIoT应用程序，目的是强调区块链如何影响传统的以云为中心的物联网应用程序。然后，针对影响BIoT应用程序的设计，开发和部署的许多方面，详细介绍了当前的挑战和可能的优化。最后，列举了一些建议，旨在指导未来的BIoT研究人员和开发人员在部署下一代BIoT应用程序之前必须解决的一些问题。

<!--more-->

### Introduction

首先是谈及物联网未来发展时常引用的几篇论文：

[1] Forecast: The Internet of Things, Worldwide, 2013, Gartner, Stamford, CA, USA, Nov. 2013.

[2]White Paper: [Cisco Visual Networking Index: Global Mobile Data Traffic Forecast Update, 2016–2021](https://www.cisco.com/c/en/us/solutions/collateral/service-provider/visual-networking-index-vni/mobile-white-paper-c11-520862.html). San Jose, CA, USA, Mar. 2017.

除此之外，还有IDC和Statista的统计数据

[3] IDC, [Worldwide Internet of Things Forecast, 2018–2022](https://www.idc.com/getdoc.jsp?containerId=US44281718)

[4] statista, [Internet of Things (IoT)](https://www.statista.com/study/27915/internet-of-things-iot-statista-dossier/)

目前，大多数物联网解决方案依赖于集中式的server-client架构，通过Internet连接到云服务器。这种方式虽然现在运行的很好，但以上提到的物联网未来设备的大量增长表明必须提出新的架构。过去有提案建议使用分布式架构来创建大型点对点无线传感器网络，但这种架构在隐私和安全方面有所不足，直到区块链技术的到来。因此，如图1所示，在过去的几年中，物联网时代前的封闭和集中式架构逐渐向物联网时代开放的接入以云为中心的替代方案发展，下一步是在多个同行之间分配云功能，其中区块链技术可以帮助。

区块链技术能够跟踪，协调，执行交易并存储来自大量设备的信息，从而能够创建不需要集中式云的应用程序。像IBM这样的公司更进一步，将区块链作为一种使未来物联网民主化的技术[12]，因为它解决了当前大规模采用的关键挑战：

- 由于与集中式云和服务器群的部署和维护相关的成本，许多物联网解决方案代价依然昂贵。当供应商不建立这样的基础设施时，成本来自中间商。
- 当必须将定期软件更新分发给数百万个智能设备时，维护也是一个问题。
- 斯诺登事件后，物联网设备用户很难信任设备提供商，因为他们通常会向某些机构（即政府，制造商或服务提供商）提供设备访问和控制权，允许他们收集并分析用户数据。因此，隐私和匿名应成为未来物联网解决方案的核心。
- 封闭源代码缺乏信任。为了增加信任和安全性，透明度至关重要，因此在开发下一代物联网解决方案时应考虑开源方法。值得注意的是，开源代码和封闭代码一样容易受到漏洞和攻击的影响，但是，由于许多用户可以不断监视它，因此不太容易受到第三方的恶意修改

其他作者此前曾就区块链在不同领域的应用进行过调查。例如，在[25]中，对区块链和智能合约的基础知识进行了广泛的描述，并对BIoT解决方案的应用和部署进行了很好的概述。但是这篇文章并没有深入探讨理想的BIoT体系结构的特性，也没有深入研究创建BIoT应用程序的可能优化。另一项工作[26]，作者提供了关于体系结构和区块链中涉及的不同机制的一般性评论，尽管它没有关注其在物联网中的应用。同样，在[27]和[28]中，研究人员对区块链进行了概述，但他们强调其应用于不同的大数据领域和多个工业应用。最后，值得一提的是[29]和[30]中提出的系统评价，它们分析了文献中论文在提议使用区块链时所涉及的主题。

[25] K. Christidis,  Blockchains and smart contracts for the Internet of Things,2016.
[26] Z. Zheng, An overview of blockchain technology: Architecture, consensus, and future trends, Jun. 2017,
[27] E. Karafiloski, Blockchain solutions for big data challenges: A literature review, Jul. 2017, 
[28] T. Ahram,  Blockchain technology innovations, Jun. 2017
[29] M. Conoscenti, Blockchain for the Internet of Things: A systematic literature review, Nov./Dec. 2016
[30] J. Yli-Huumo,  Where is current research on blockchain technology?—A systematic review,2016.

### Blockchain Basics

在深入研究如何在物联网应用中使用区块链的细节之前，必须首先强调区块链并不总是每个物联网场景的最佳解决方案。传统数据库或基于定向非循环图（[DAG](https://www.iota.org/)）的分类帐可能更适合某些物联网应用。具体而言，为了确定区块链的使用是否合适，开发人员应确定IoT应用程序是否需要以下功能：

去中心化，P2P通信，支付系统，公共的序列交易记录，鲁棒的分布式系统，微交易的集合

以上每种特性在文中都有具体的解释。下图显示了一个通用流程图，允许根据物联网系统的特征确定所需的区块链类型。

![流程图](https://user-images.githubusercontent.com/26682846/54514418-0edf9800-4995-11e9-9fa6-7a55e2803da3.png)

### BIoT Applications

区块链应用始于比特币(blockchain 1.0)，然后演变为智能合约(Blockchain 2.0)，后来转向司法、效率和协调的应用(blockchain 3.0)。

智能合约主要是从以太坊起源，而除加密货币和智能合约外，区块链技术可以应用于涉及物联网应用的不同领域，最相关的如下图所示：

![区块链在物联网中应用](https://user-images.githubusercontent.com/26682846/54514436-1c951d80-4995-11e9-967c-1ca6c8c49024.png)

以及包括区块链用于农业物联网、能量领域，医疗领域，改进物联网低安全级别

### Design of an Optimized Blockchain for IoT Applications

区块链可以用于物联网，但一开始并没有针对物联网设计，所以应用时应进行一定的构成调整。这种工作已有一些人做，他们研究了不同场景下的BIoT性能，分析了影响因素，但主要关注的是共识算法的影响。

比如PBFT是否是大规模点对点网络的性能瓶颈，以及PoW和BFT的可扩展性，结论是PoW扩展性是差的，但与BFT结合可能会稍微有所改善。

除了共识算法，区块链中其它元素也可以用于物联网，下面具体的讨论每一种元素在物联网中可能的优化：

1. Architecture

2. Cryptographic Algorithms

   物联网设备通常无法提供非对称的加密算法需要的算力和能耗，RSA不行，椭圆曲线勉强可以。同时IoT应用的散列函数也必须是安全，快速且应该消耗尽可能少的能量。普通区块链使用SHA-256，实际上在物联网里不行，建议使用AES，如Simon

3. Message Timestamping

   为了跟踪区块链的修改，交易必须都经过签名和加盖时间戳。加盖时间戳应以同步方式执行，因此通常使用时间戳服务器。比特币使用一个相对的时间顺序，即当前块必定在上一个块之后。没有其它有用的方案。

4. Consensus Mechanisms, Mining and Message Validation

   共识的理想情况是一人一票，多数票决定，但只能在受控环境实现。PoW使得区块链吞吐量，可扩展性和能耗受影响，物联网中不可取。

   目前已有的替代方案有：

   - PoS，DPoS
   - TaPoS是PoS变体。在PoS中只有一些节点参与达成共识，但在TaPoS中，所有生成事务的节点都有助于网络的安全性
   - Proof-of-Activity (PoA)
   - PBFT
   - DBFT，BFT的变体
   - Ripple
   - Stellar Consensus Protocol (SCP)
   - BFTRaft ,是基于Raft算法的BFT共识方案
   - Sieve，IBM提出，已由Fabrc实现
   - Tendermint
   - Bitcoin-NG
   - Proof of Burn(PoB)，燃烧证明
   - Proof-of-Personhood (PoP) 

   最后，控制用户访问的私有区块链降低了Sybil攻击的可能性，因此它们不需要挖矿和共识算法，激励机制也可以取消

5. Blockchain Updating/Maintenance and Protocol Stack

   利用区块链做物联网设备的软件更新，这种场景之前已经见过

   关于协议栈，一些人建议对OSI进行一定的更改使其适应区块链。如“Internet of Money” (IoM)，它提出了一个运行在TCP/IP上的五层结构如下

   ![货币互联网对OSI的改动](https://user-images.githubusercontent.com/26682846/54514461-2e76c080-4995-11e9-995a-026d3a0420a9.png)

### Current Challenges for BIoT Applications

1. Privacy

   目前提出的多数方案都有对资源的要求，因此，对资源受限的物联网作用有限

2. Security

3. Energy Efficiency

   物联网终端资源有限，区块链有两方面消耗能源，一是挖矿，二是P2P通信。现在有一些办法，一些人认为挖矿消耗的算力可以用于其它的事情，不至于白白浪费，或者改用PoS等不消耗算力的方法

   关于P2P通信，它们对于区块链通信必不可少，区块链接收的更新越多，通信的能量消耗就越多。为了减少更新次数，mini-blokchains 可以允许物联网节点直接与区块链交互，因为它们只保留最新的事务并降低整个节点的计算要求

   哈希算法也可以使用能量消耗更小的方法

4. Throughput and Latency

   区块链的交易确认延迟比较高

5. Blockchain Size,Bandwidth and Infrastructure

   随着区块链边长，节点需要的存储也变大，区块链压缩技术应研究，对于物联网节点来说，可以作为轻量级节点运行，因为大量数据是无关的，但这种方法要求区块链体系中必须存在有些能力比较强的节点维持链，但这又会带来一定的集中化。还有一种办法是迷你区块链

6. 其它

### 进一步的挑战和建议

- 复杂的技术挑战
- 互操作性和标准化
- 区块链基础设施
- 组织，治理，监督，法律
- 快速现场测试


