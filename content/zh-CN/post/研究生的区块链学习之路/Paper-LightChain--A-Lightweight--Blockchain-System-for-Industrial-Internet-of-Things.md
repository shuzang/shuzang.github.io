---
title: LightChain:A Lightweight Blockchain System for Industrial Internet of Things
date: 2019-03-19T11:00:00+08:00
tags: [论文笔记]
categories: [研究生的区块链学习之路] 
---

Author： Yinqiu Liu, Kun Wang, Yun Lin, and Wenyao Xu

Published in：IEEE Transactions on Industrial Informatics

State：Early Access

Index Terms：Blockchain, Industrial Internet of Things, Distributed System, Consensus Mechanism, Data Filter.

## Abstract

虽然区块链和IIoT之间的结合得到了广泛的关注，但区块链的高资源需求和IIoT设备的有限性能之间的矛盾还无法较好的解决。一方面，由于公钥结构、默克尔树和PoW等数学概念的引入，部署区块链需要巨大的算力；另一方面，全节点应该能同步大量的区块数据和处理P2P网络中的大量交易。IIoT设备难以承受其对存储容量和带宽的占用。本文中，我们提出了名为**LightChain**的轻量级区块链使其适用于IIoT场景，提出了一个名为**Synergistic Multiple Proof（SMP）**的共识机制来促进IIoT设备间的合作，提出了一种称为**LightBlock（LB）**的轻量级数据结构，用于简化广播内容。此外，还设计了一种 **Unrelated Block Offloading Filter (UBOF)**以避免分类帐的无限增长，同时不影响区块链的可追溯性。实验表明，LightChain可以将计算成本降低39.32％，将块生成速度提高74.06％。在存储和网络使用方面，降幅分别为43.35％和90.55％。

<!--more-->

## 一、Introduction

IIoT被大量采用，但有如下问题：

- 拥有大量分散的设备，面对DDoS攻击是脆弱的
- 中心化的管理结构无法自我认证，会产生隐私泄露问题
- LPWAN的发展使大量IIoT设备在地理上是分离的，中心化服务的花费难以承受

最近大家都在研究如何将区块链部署在IIoT来解决上述问题。

- 基于区块链不可否认和难以篡改的特性，在P2P网络中共同维持工业信息，可以实现数据追溯，并达成在非信任环境中的价值传递
- 区块链能提供分布式的域名服务，有助于解决当前DNS的漏洞，诸如DDoS攻击和DNS欺骗

已有区块链和IoT/IIoT结合的例子如Ruffchain等。然而，IIoT设备无法满足区块链的高资源需求。有很多方案来优化区块链的资源消耗问题，如：

- Ehmke[23]，区块链协议PoP，允许不必下载整个区块而验证交易

- Dorri[24]，私有不可变分类账，中心化的管理

- Li[25]，DAG

- Zamani[26]，分片（Sharding）

- Bitcoin-NG，leadership selection

- Multichain，cross-chain mechanism

但它们都只优化一方面。

将区块链部署在IIoT场景面临的关键问题包括：

- 打包区块有奖励，所以算力高的节点会持续增加算力，所有节点陷入算力竞争，最后由于马太效应，导致算力集中
- 为了实现分布式的一致性，区块链需要参与者保存网络中产生的大量数据，所有的数据保存在本地并不断增长而没有减少，会占用大量存储空间
- IIoT场景异构网络很常见，在高吞吐量的情况下，资源有限的节点无法支持区块链相关的操作

本文为了解决上述问题，本文提出一种轻量级区块链，资源问题和解决方案如下：

- 算力—通过新的共识方案SMP
- 存储空间—检测不相干块（UB），通过一个过滤器（UBOF）过滤它们，减少存储占用
- 网络资源—广播数据结构LightBlock而不是整个块，减少广播时通信的冗余

<br>

## 二、Proposal

### 1. Framework

整个方案分四层，如图1所示，从上到下依次是：API层，LightChain层，Cache层，Storage层。

图1. Four-layer framework of LightChain

![Four-layer framework of LightChain](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9424/8736410/8664132/wang1-2904049-large.gif)

各层功能如下：

API层：提供各种操作的请求接口

LightChain层：即普遍意义上的区块链，包括共识机制等部分

Cache层：用了加速对调用操作的响应

Storage层：提供持久化的存储，通常只由资源富裕的节点提供此服务

如图1，以具体的工厂实例来说明，传感器发起的交易请求通过局域网发送到Node1，Node1的API层处理这些请求并发送到区块链（即LightChain层），在LightChain层完成验证。受限于存储问题，将数据存储分为两类，完整的区块链数据会存在云端数据库作为备份，Node1会对数据进行过滤缓存有效数据。

### 2. LightChain Layer

LightChain层的结构如图2，

图2. Architecture of LightChain layer，由Conchain，Webchain和Chainbase三部分组成。类似于C/S架构，Conchain/Webchain是Client端。模块间通信通过本地Socket完成。

![Architecture of LightChain layer](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9424/8736410/8664132/wang2-2904049-large.gif)

Webchain：将API层发送的操作类型和JSON流转换称预定义的信息类型和二进制流

Conchain：在本地挖到新块或接受到块验证请求后，将消息发送到Chainbase

- 共识机制—减少资源消耗
- P2P网络—减少广播冗余

Chainbase：管理本地交易池，验证交易，拥有添加或过滤Cache layer数据缓存的权限

## 三、Evaluation

构建实验网络时模拟了IIoT的实际场景。主要考虑了IIoT场景的以下特征

- 网络异构及地理分离
- 资源限制
- 复杂的网络拓扑

使用阿里云的10台ECS构建了一个P2P网络，ECSs作为全节点（矿工节点）工作，并且各ECS具有不同的CPU和内存，部署在亚洲多个地点，算力级别在MH/s，符合IIoT设备能力，每个ECS拥有自己的公网IP，同一地点的ECS在同一网段。

开发了一个自动交易生成器，控制几个交易并间隔固定的时间提交随机交易，这里的账户作为P2P网络中的轻量级节点。

每个矿工连接到一个交易生成器，设置两个参数，一个控制账户数量，决定P2P网络的规模；一个用于控制交易生成的间隔。

### 1. 安全分析

分析对几种攻击的抵御能力：

- 双花攻击
- 无厉害关系（PoS中同时在多个fork上投票以获取最大利益）

### 2. 结果分析

共识：

- CPU使用率：减少
- 块生成速度：增加
- 算力消耗：减少

网络：通信冗余减少

存储：过滤区块，存储减少

## 四、Conclusion

为了使区块链适用于IIoT场景，提出共识算法SMP减少算力消耗，提出数据结构LB减少通信冗余，过滤区块减小存储负担。并通过实验证明了该方案的优越性。