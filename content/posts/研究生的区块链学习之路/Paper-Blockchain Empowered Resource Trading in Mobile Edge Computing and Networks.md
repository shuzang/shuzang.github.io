---
title: Blockchain Empowered Resource Trading in Mobile Edge Computing and Networks
date: 2020-04-03
tags: [论文笔记]
categories: [研究生的区块链学习之路]
---

Qiao G, Leng S, Chai H, et. al. Blockchain Empowered Resource Trading in Mobile Edge Computing and Networks[C]. ICC 2019 - 2019 IEEE International Conference on Communications (ICC). Shanghai, China: IEEE, 2019: 1–6. 

DOI:[10.1109/ICC.2019.8761664](https://doi.org/10.1109/ICC.2019.8761664).

注：文中图片来自IEEE原论文

## 1. 引言

随着智能手机和 IoT 设备的增多，移动通信已经成为我们日常活动的一部分。传统的云计算不能满足实时性的需求，边缘计算和缓存技术逐渐成为现在研究的热点。这种情况下，D2D 边缘计算和网络（device-to-device edge computing and networks, D2D-ECN）被提了出来，通过为 IoT 设备提供一个灵活的边缘计算平台来共享它们的计算和通信资源（如下图所示），最终满足实时性需求。

![本地的资源交换和任务分配](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8753818/8761046/8761664/qiao1-p6-qiao-small.gif)

任务执行者希望在一段期望的执行时间内执行一个计算密集型的任务，他可以将计算任务[卸载]^(offload)给附近有可用计算资源的 IoT 设备，此外，数据传输延迟也可以通过购买空闲的通信资源来减少。

{{% admonition question "Computation-Intensive Task" false  %}}

计算密集型的任务，第一反应是深度学习或挖矿等，这里担心两个问题：

1. 应用场景是否为不可信环境，这决定了使用区块链是否有必要。
2. 这种场景可能很少，比如深度学习和挖矿专业性都太强，这种情况这种技术不会得到普及。

当前能想到的一种场景是游戏，谷歌等公司推出的基于云的游戏大大降低了个人电脑的性能需求，但严重依赖于通信情况的好坏，如果我们换个思路，游戏依然安装和运行在本地，但利用D2D网络中其它设备的计算资源，不知道是否可行。而且不确定游戏的计算任务是不是可卸载的。

{{% /admonition %}}

区块链被引入来解决分布式P2P网络中的信任问题，但是大部分边缘计算网络领域的现有研究都是用 PoW 共识，用于资源有限的物联网环境是不可行的。一些方案使用联盟链，但作者认为联盟链的本质是将挖矿过程转移到云或边缘服务器，不能从根本上解决资源消耗问题(<font color="red"> 一脸懵???</font>)。最终，作者提到一个与 DPoS 相似的基于信誉的共识[^1]比较合适，该方案是将生成新区块的权利交给信誉最高的设备。

[^1]: F. Gai, et al., “Proof of Reputation: A Reputation-based Consensus Protocol for Peer-to-Peer Networks,” in Proc. 2018 Int. Con. Dat. Sys. Adv. App., DASFAA, Feb. 2018

这些研究也没有涉及[计算卸载规则]^(computation offloading rule)对基于区块链的边缘网络性能的影响。一方面，由于缺乏信任，计算卸载的过程应该由资源交易和任务分配两阶段组成，分布式资源交易提供高效的资源定价和分配策略，用来平衡任务所有者的[效用]^(utility)和资源所有者的收益。此外，任务所有者应该由购买资源的权限，以便更好的分配计算任务。因此，分布式应用需要一个智能和轻量的计算卸载规则来保证较低的执行时间和系统开销。另一方面，信誉评估指标的设计对于保证共识和可信计算的可靠性也非常重要。

![基于区块链的资源交易和任务分配系统组成和工作流](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8753818/8761046/8761664/qiao2-p6-qiao-small.gif)

作者在这篇文章中注意力集中于设计一系列建立在轻量级资源交易和任务分配方案上的信誉评估机制，主体采用智能合约来完成。文章的贡献有三点：

1. D2D-ECN中，提出了一种分布式的资源交易和任务分配方案，方案中，邻近的 IoT 设备可以协作实施任务密集型和延迟敏感型的物联网应用。
2. 设计了一个基于博弈论的智能合约来激励设备参与资源交易。智能合约可以进行差异化定价和最优资源配置。
3. 提出了一种群体智能任务分配方案，在任务服务延迟和决策时间之间达到良好的平衡，并触发智能合约来评估物联网设备的信誉范围。

{{% admonition note "区块链的作用" false  %}}

看这个方向论文的出发点是D2D网络中的设备协作需要区块链保证可信，从上面的贡献来看，这篇论文中区块链的作用是执行任务分配、资源交易和信誉评估，并保证这些记录的不可变。确实起到了相应的作用，但我们现在需要明确的是访问控制在资源交易和保证用户对资源的控制权中是如何起作用的。

{{% /admonition %}}

## 2. 方案

总体方案分两部分，第一部分描述计算卸载过程，第二部分介绍信誉评估系统。

### 2.1 计算卸载过程

未完待续……