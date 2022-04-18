---
title: Blockchain Platforms Overview for Industrial IoT Purpose
date: 2019-02-27T11:11:00+08:00
tags: [论文笔记]
categories: [研究生的区块链学习之路] 
---

Author：Nikolay Teslya; Igor Ryabchikov

Published in：2018 22nd Conference of Open Innovations Association(FRUCT)

Date of Conference: 15-18 May **2018**

会议举办地：Jyvaskyla, Finland

被引量：1次

## Abstract

如今已有很多可用区块链平台存在。但要集成到工业物联网的智能空间中，区块链平台不仅应支持代币交易，还应支持智能合约、容错共识机制，以及参与者创建和实施新区块和智能合约的地位的平等。本文分析了最常用的共识机制，公共（无权限）和私有（有权限）区块链的具体特征。还描述了满足IIoT平台开发要求的区块链平台。通过分析所得结果，可选择平台和特定模块来实现用于IIoT平台的区块链。

<!--more-->

## I. Introduction

智能工厂的内部组件之间以及和其它工厂之间的交互是工业4.0的主要问题之一。到目前为止，已有许多基于物联网的解决方案，允许将多个组件组合成单一的信息空间并在它们之间提供信息交换。工业上这种联合体正是工业物联网（IIoT，Industrial Internet of Things）的概念，即物联网用于物理、虚拟和社会工业组件在单一信息空间（也称为智能空间）中的交互。然而，生产变得越来越去中心化，一些问题随之出现，其中有几个需要强调：**需要在智能空间内组件之间以及和其它智能空间之间提供互操作性；信息空间参与者之间的信任；资源（如维护时间，能源等）和成品分发的控制**。

为了提供智能空间中组件间的互操作性，可以使用本体和本体匹配机制（称为ontology and ontology matching mechanism）,这种机制已在众多项目中做了广泛描述和使用[2-3]。组件间的信任问题由于参与者的异构变得复杂，可通过数字签名和访问控制机制解决，需要一个中心化实体来为IIoT的所有组件提供信任和访问控制。资源和成品分发的控制可以使用一个所有组件都可访问的数据库完成。这些解决方案都相当复杂并且需要复杂的基础架构才能提供容错、性能和可用性。与此同时，区块链技术的活跃发展为以上提到的问题提供了更简单的解决方案。

> [2]. A. Smirnov A. Kashevnik A. Ponomarev N. Shilov M. Shchekotov N. Teslya "Smart space-based intelligent mobile tourist guide: Service-based implementation" Conference of Open Innovation Association FRUCT pp. 126-134 2014. 
>
> [3]. A. Smirnov A. Kashevnik N. Shilov S. Balandin I. Oliver S. Boldyrev "On-the-fly ontology matching in smart spaces: A multi-model approach" Lect. Notes Comput. Sci. (including Subser. Lect. Notes Artif. Intell. Lect. Notes Bioinformatics) vol. 6294 pp. 72-83 2010.  

以上解决方案都较复杂并且需要复杂的基础架构才能提供容错、性能和可用性。与此同时，正在活跃的区块链技术为此提供了一种更简单的解决方案。

*<font color=blue>这里有两段区块链的介绍，跳过</font>*

本文的目的是分析可用的区块链解决方案，以便在工业4.0案例中实施。分析的主要问题是共识机制的实施、网络的公开性、智能合约的支持和用于在IIoT平台上提供所有这些功能的平台。这些因素可能会彼此依赖，比如，共识机制依赖于网络公开性。为了适用于IIoT平台，区块链应当支持智能合约，支持没有挖矿程序的区块生成，共识机制应当能够以少量节点运行。

本文剩余部分结构如下，Section 2描述一些关于区块链实施的进展，Section 3描述共识机制，Section 4描述公私链网络的实现以及它们的优缺点，Section 5描述一些可用于区块链和IIoT集成的区块链平台和模块

## II. State of the Art

目前已有许多区块链技术的实现，搜索显示至少存在20个平台。它们的不同主要在于共识机制、交易验证机制（可靠性和一致性的保证）和功能（例如，仅支持货币交易还是能创建智能合约的通用目的的区块链）。通用目的的区块链（支持智能合约）的情况下，可用的状态存储结构，以及限制智能合约性能的机会仍有不同。

由于区块链技术不久前才流行起来，许多项目还处于早期阶段，因此它们没有定性地描述算法来支持所声称的保证，因此不适用于实际项目的实施。已有的区块链技术的实现，尤其是共识机制的实现在[6]中得到总结。本文基本上研究了[6]中的所有共识和共识机制之外他们工作的特点，并和具体项目的需求相关联。下一部分将详细描述这些共识机制。

> [6] C. Cachin M. Vukolic Blockchain Consensus Protocols in the Wild Jul. 2017. 

## III. Consensus Mechanisms

以下是几种最流行的共识机制

- Proof-of-work (PoW) ;
- Proot-of-elapsed-time (PoET)) ;
- Proof-of-stake (PoS) ;
- Byzantine Fault Tolerance (BFT) ;
- Federated Byzantine Agreement (FBA) ;
- Various combinations of the above algorithms.

PoW和PoS不再细述，其它几种简单机翻

### A. PoET

PoET类似于PoW，但对于随机块创建，解决方案不是基于资源密集型任务，而是基于特殊硬件（特别是在Hyperledger Sawtooth中）它使用称为Intel Software Guard Extensions（SGX）的特定Intel CPU指令集，允许应用程序在受保护的环境中运行可信代码。它允许等待一段随机时间，并且由于加密签名，可以证明用户已经做出了期望。此解决方案允许降低块创建的成本（不需要花费资源来计算数学问题），但它有一个明显的缺点 - 网络的性能取决于硬件或虚拟环境性能。如果硬件被黑客入侵，网络将失去效率（这足以提醒英特尔处理器的最后两个漏洞 - 幽灵和熔化，由于补丁导致系统性能下降到2-11％）和操作机制变得众所周知。此外，更不用说硬件开发商和制造商将有可能通过硬件架构中的后门来影响网络。此外，由于块生成的可能性取决于参与者拥有的英特尔处理器的数量，因此仍存在资金投入影响的问题。该协议在Hyperledger Sawtooth区块链网络中实现

### B. BFT

拜占庭容错是信息系统容忍一个或多个组件故障的属性。它起源于拜占庭将军问题，该问题假定需要在三位将军之间达成共识，这是其中一人可能向其他人发送冲突信息的条件。基于BFT的共识机制用于私链网络（在其成员彼此已知的网络中）。在区块链网络中，当少于三分之一的节点出现故障时，该机制允许在条件下达成共识，包括干扰整体目标的恶意节点。违反条件可能导致缺乏进度或分支（例如，如果超过三分之二的节点是恶意协同工作的节点）。与之前描述的协议相比，该协议的特点是它不允许回滚状态 - 接受的有效块是最终的并且不能被替换。但这也有一个缺点 - 只有部分节点的性能才能实现进展，而上述机制即使只运行一个节点也能运行。在将数量条件（超过2/3）应用于参与者权重之和而不是其数量的意义上，存在将该机制与利益证明相结合的实施方式。此机制在许可节点中最受欢迎。该机制用于各种私有区块链网络实现 -  Hyperledger Fabric [17]，Tendermint [18]，Corda [19]，Exonum [20]等。

### C. FBA

类似于BFT，但允许每个参与者在达成共识时指出他们自己的可信参与者列表，而不是共享共同的假设。这种机制在Ripple 和Stellar 项目中实现

<br>

## IV. Permissioned and Permissionless Blockchain Platforms

可以将区块链分为公链和私链，两种网络的对比如表1所示

表1. 

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8457083/8468260/8468276/Tes-table-1-source-large.gif)

除了共识机制外，区块链平台之间的另一个区别是功能。许多实现仅针对参与者之间的资产（货币或对象）的审计和转移，包括MultiChain，Chain Core等。为了开发共同的智能空间，有必要实现一个通用目的的区块链平台（支持智能合约）。

区块链和IIoT集成的架构在[1]中提出，它允许通过共享信息空间（智能空间）将小型和弱能力设备（如传感器）和强大的计算单元联合起来，并通过将签名的共享信息记录到区块链中来提供它们之间的信任。在目前的研究状态下，传感器仅用于提供信息而无需自己进行签名。此外，由于较弱的计算能力，IIoT的这些部分不能参与共识。总结以上，IIoT和区块链技术的整合应基于许可链技术进行实施，从而提供每秒超过2500次交易的高性能，并允许为小组参与者的工作创建私人智能空间。

> [1] N. Teslya I. Ryabchikov "Blockchain-Based Platform Architecture for Industrial IoT" Proceeding of the 21st conference of FRUCT Association pp. 321-329 2017.

## V. Blockchain Platforms and Modules for Integration with Industrial IoT

### A. Platforms

作者把范围缩小到了Corda, Hyperledger Fabric, Tendermint和Symbiont四种平台。然后详细的讨论了Corda和Hyperledger。

### B. Modules

除了使用现有的区块链平台，还可以使用现有的共识机制创建自己的区块链平台。可以通过一组库实现。这里介绍两个：**BFT-SMaRt**和**Tendermint**。然后就是详细的介绍。

<br>

## Conclusion

本文提供的分析表明，区块链平台有很多种实现方式。它们的区别在于很多因素，最重要的是块创建协议，块添加的共识，智能合约的支持（环境，编程语言，功能）。要在工业物联网和区块链平台之间建立连接，需要创建将IoT和区块链功能联合起来的模块。该模块将允许在IIoT的智能空间中存储信息，并在区块链中使用签名事务复制它。此外，还可以创建可在适当条件下自动处理的合约。这种集成的最佳平台是Hyperledger Fabric / Burrow。它们提供容错共识机制以及用于创建和处理智能合约的内置基础架构。可以使用这些平台创建两种类型的区块链网络 - 私有和公共。

此外，通过在IIoT基础设施上创建自己的区块链网络，可以深入集成IIoT和区块链。为此目的，本文提供了用于创建自己的区块链功能的库列表。这些库为各种共识机制提供了现成的解决方案，但开发人员应该为智能合约创建自己的基础架构，例如通过容器技术。

未来的工作将集中在第一个案例上，开发IIoT模块和私有区块链集成。此外，第二种情况将部分用于实现共识机制的深度集成，以便在所有参与者之间提供更高的块添加和共享速度。

> 总结：
>
> 区块链用于IIoT目的包括三点，1）工厂内各组件间，工厂间的交互；2）组件间的信任；3）资源和成品分发
>
> 主要研究问题为：共识的比较与选择，区块链类型选择，是否支持智能合约，平台比较选择
>
> 结论为应选择私链平台，并且使用私链的共识，平台应支持智能合约