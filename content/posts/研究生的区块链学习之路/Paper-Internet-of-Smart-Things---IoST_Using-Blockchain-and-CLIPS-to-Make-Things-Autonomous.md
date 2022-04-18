---
title: Internet of Smart Things-IoST
date: 2019-01-17T10:06:00+08:00
tags: [论文笔记]
categories: [研究生的区块链学习之路]
---

Author：Mayra Samaniego, Ralph Deters

Published in：2017 IEEE International Conference on Cognitive Computing (ICCC)

Date of Conference: 25-30 June 2017

会议级别：不知道

被引量：11次

keywords：IoT; Management; Blockchain; Multichain;Smart Things; Autonomy;Self-inferencing; Self-monitoring;Fog; Edge. 

<br/>

### 摘要

构成物联网的大量异构设备需要有效的资源管理。随着雾计算的出现，一些管理任务可以下移到物联网的边缘，更靠近物理设备。建立在雾网络上的区块链可以处理一些物联网的管理任务，如通信、存储和身份验证。这种情况下，以及超越了原来对物联网中Things的定义，可以称之为“Smart Things"。Smart Things提供基于CLIPS编程语言的人工智能（AI）功能，以实现自我推理和自我监控。这项工作使用私链构建工具Multichain通过读写块中信息来达成Smart Things之间的通信。本文评估了Edison Arduino板上部署的Smart Things以及雾网络上部署的的Multichain网络。

<!--more-->

### I. 引言

物联网设备从现实世界感知和捕获数据，然而，它们缺乏足够的计算资源来处理和分析这些数据，只能把它们发送到云。如今，典型的IoT系统是一个以云为中心的架构，包括Things, Services, Applications三层，其中Things的作用仅仅是一个数据收集器。云的稳健性和灵活性使得数据处理高效且可靠，然而数据流到达云端的时间可能影响建立在数据之上的决策的准确性。

云结构的显著缺点是是传感器的单一作用和数据流的传输延迟，若要构建更为先进和高级的物联网系统，就需要在Things层使物联网设备具有一定的处理能力，因此引入了物联网中自治的概念。

物联网设备计算能力的提高使得执行自治任务成为可能。实现这一目的通过在Things层创建能够自我推理和自我监督的Smart Things来实现。

物联网系统中，管理地理上分布的设备必须是低延迟的，而云存储无法处理由终端设备产生的实时数据流。使用雾计算，把管理任务下移到物联网的边缘可以提升效率和减少延迟。一些研究提出了部署在雾网络上的虚拟解决方案，如虚拟传感器和网关。然而，这些方案都偏向于虚拟化独立的组件来避免物联网设备间的实时通信。但雾计算系统中设备间实时通信的需求是恒定存在的。本文通过建立在雾网络上的区块链，实现了Smart Things的实时通信的管理，实现的能力包括：

- 去中心化的通信管理
- 低延迟通信
- 实时通信
- Time-effective event management

以下，Section II讨论物联网中的自治，Section III介绍物联网中的专家系统，Section IV讨论区块链协议，Section V解释Smart Things架构，Section VI给出实验结果和评估。最后一部分做总结。

### II. 自治

自治指的是计算机能够监督和管理自己，系统应能够在没有人参与的情况下对未知事件做出反应。根据Kephart和Chess的研究，自治系统应满足如下原则

- 自我配置与重配置
- 自我优化
- 自我修复
- 自我保护

传统互联网背景下已经有一些关于自治系统的研究。IBM有一种分层的自治计算框架。但在物联网背景下，自治系统还需要解决一些问题：

- 设备间异构
- 大量设备
- 计算能力限制
- 能耗限制
- 地理上分布
- 实时操作

更多的，根据场景的不同，物联网中的自治概念可以侧重于解决以上某种特定的问题。IBM的结构是不少研究的基础，总的来说，物联网终端设备是资源，相关的中间件是管理者，它监督设备并根据当前状态执行操作。

ADEPT PoC是物联网自治系统的另一个方案。和IBM的架构不同，它是混合和去中心化的，ADEPT使用Telehash用于点对点通信，使用BitTorrent用于分布式文件共享，使用Ethereum区块链用于自治设备的协作比如存储设备配置和身份认证。

*ADEPT PoC：The Autonomous Decentralized Peer-to-Peer Telemetry Proof of Concept*

### III. 专家系统

专家系统的概念都熟悉。这里主要是介绍CLIPS是一种开发专家系统的编程语言，由NASA开发，用C编写完成，多平台支持。

*CLIPS：The C' Language Integrated Production System*

### IV. 区块链

熟悉，略过

### V. Smart Things 架构

使用ADEPT PoC方案构建的自治系统由两部分组成：Smart Things和区块链协议。

#### Smart Things

一个Smart Things是一个软件定义的物联网设备，能够分析当前状态，监视可能的改变和推断知识。主要目标是通过直接在物联网设备上开发人工智能（AI）功能，在Things层中保持监控和决策。一个Smart Thing由三部分组成：

1. Reader

   作用是从环境中感知数据，然后发给Self-C. resource

2. Self-Inferencing resource

   集成了基于CLIPS的专家系统，在开始时会声明一组初始规则和事实(fact)，之后基于这些预先配置的规则进行AI推理。接收到的数据会转换为事实然后插入到系统的事实列表，系统分析这些事实并执行操作，最后把分析结果发给Self-monitoring resource.

3. Self-Monitoring Resource

   从Self-Inferencing resource接收结果并评估。根据专家系统的规则和推理过程决定是否把数据发送到区块链从而分发到Things层。

以上三部分都使用Go语言编程。使用RESTful微服务进行通信。

#### 私链构建工具Multichain

使用Multichain管理Smart Things间的通信，因为是私链，所以只有预先注册的成员能访问区块链。Multichain执行的共识算法是轮询（Round Robin, RR）调度算法。每个块必须有创建者签名。块的创建者必须等一个固定的时间才能创建新块。

Multichain部署在雾网络中，靠近Things层。和Smart Thing中的Self-Monitoring Resource交互。它接收并存储数据，并扩散到整个网络的所有节点，所以节点能够实时的获知决策结果。

### VI. 实验与评估

评估Smart Things和Multichain区块链

#### 评估Smart Things

使用Edison Arduino开发板评估Smart Things的性能。在该板上运行Smart Thing的三部分组件。从环境中获取温度值1000并发送到专家系统。

每个请求都会使用AES进行加密，由专家系统解密数据，把数据转化成fact，插入face列表，推理，分析，最后把结果发给Self-Monitoring resource。

在实验中加入不同的延迟间隔来测试不同请求级别下的Smart Things性能。这个延迟间隔指的是数据发送的延迟。结果表明，不同的延迟并不会影响Smart Things的性能。Smart Things的平均响应时间是1.7ms，所有的实验中，响应时间都大于1.2ms。加密和解密时间包含在内，这也说明了该系统中提出的Smart Things的良好性能。

#### 评估Multichain

评估部署在雾网络中的Multichain的性能，实验中的Multichain有三个节点。开发板中的Smart Things做完推理分析后，发送1000个结果给区块链，每个请求都包含成员的身份验证信息，消息使用AES加密，同样，加密和解密的时间包含在实验结果中。

Multichain的平均响应时间是389ms。延迟不影响性能。这说明了区块链能够存储数据并分发到所有节点。同时，Multichain对请求处理成功的概率是98.47%。

<br/>

### 结论和未来工作

自治由Smart Things完成，Smart Things间的实时通信由区块链完成。总的来说，这项研究对物联网系统做了如下贡献：

- 设计开发Smart Things，可实时完成自我推理和自我监督
- 在CLIPS上设计和开发专家系统，来执行数据分析和自我推理
- 设计并实现了集成区块链的雾网络，可与Smart Things实时通信，并将管理任务分配到Things层的边缘。

未来的工作侧重于评估不同开发板的Smart Things:flushed:





