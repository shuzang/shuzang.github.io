---
title: A Blockchain based access control for IoT
date: 2020-02-06
tags: [论文笔记]
categories: [研究生的区块链学习之路]
toc: true 
---

Imen Riabi, Yosr Dhif, Hella Kaffel Ben Ayed, Khaled Zaatouri. A Blockchain based access control for IoT[C]. International Wireless Communications & Mobile Computing Conference (IWCMC), 2019.

DOI: [10.1109/IWCMC.2019.8766506](https://doi.org/10.1109/IWCMC.2019.8766506)

Keywords: Access control, Blockchain, Smart Contract, Internet of Things

注：本文图片来自原论文。

## 1. 引入

作者的考虑主要基于三点

1. 传统中心化的访问控制带来的单点故障和可扩展性问题；
2. 资源有限的IoT设备需要轻量级访问控制方案(对CPU、内存、功耗的低需求)
3. 对低延迟的需求

文章其它部分的组织结构为：第二部分为相关工作，第三部分为区块链技术介绍(阅读时略过)，第四部分为区块链安全机制介绍，第五部分阐述提出的访问控制模型，第六部分通过实验实施提出的方案，最后总结全文。

## 2. 相关工作

该部分说明了已有物联网访问控制方案不合适的原因

**RBAC**：the Role based access control，物联网环境高度动态且用户数量巨大，纯RBAC无法应对。

**ABAC**：the Attribute based access control，规则的数量随着用户、属性的增长迅速增加，不适用于物联网高度动态和实时的环境。

**Cap-BAC**：Capability based access control，主要指OAuth-IoT，主要的问题是中心化结构会带来可扩展性问题和高延迟，同时该模型不支持可移动性。

## 3. 区块链安全机制

该部分讨论区块链的安全机制从而评估其安全级别，区块链提供的安全服务和对应的实现机制如下表所示：

| 区块链安全服务 | 使用的机制                   |
| -------------- | ---------------------------- |
| 完整性         | 哈希函数                     |
| 交易真实性     | 数字签名                     |
| 机密性         | 非对称加密                   |
| 可用性         | 多个副本分布于整个网络       |
| 匿名性         | 公钥用作节点地址             |
| 可追溯性       | 所有交易记录在区块链中       |
| 防篡改         | 需要大量的算力才能破坏区块链 |

## 4. 方案

作者选择将 Capability-BAC 和 Identity-BAC 两个模型相结合，利用token向请求者授权(Capability-BAC)，利用访问控制列表ACL记录请求者和对应的访问权限(Identity-BAC)。资源所有者在智能合约中存储ACL，资源请求者发起请求从而逐步填充ACL的内容，从而令区块链替代传统的中心化授权服务器。

### 4.1 Actor

方案中涉及的角色如下

- 资源所有者：区块链中拥有资源的节点，部署智能合约并在智能合约中定义ACL，接收来自区块链中其它节点(请求访问资源的节点)的注册请求。
- 矿工：区块链中有一定计算能力的节点，替代传统的授权服务器，对来自请求者的请求进行管理，基于资源所有者部署的智能合约中的ACL生成和授予token。
- 请求者：区块链中想要以指定权限访问特定资源的节点，希望获取对应的访问token。

一个传统访问控制和基于区块链的访问控制角色对应表如下

| 传统访问控制 | 基于区块链的访问控制     |
| ------------ | ------------------------ |
| 资源所有者   | 区块链中的资源所有者节点 |
| 请求者       | 区块链中的请求者节点     |
| 授权服务器   | 区块链中的矿工节点       |

### 4.2 Proposal

作者提出的方案使用智能合约存储和管理ACL，每个资源所有者在自己部署的智能合约中定义和资源相关的ACL，这些ACL被矿工用于验证请求者的访问权限，从而生成访问token并给予对应的请求者。节点间的通信使用区块链本身的交易机制。

**注册过程**

一个新的资源请求者在智能合约中注册的时序图如下

mermaid图
sequenceDiagram
    participant Resource Requester
    participant Resource Owner
	participant ACL Smart Contract
	Resource Requester ->> Resource Owner: 1. Registration Request Transaction(resource privilege)
	Resource Owner ->> Resource Requester: 2. Registration Response Transaction
	alt if Registration Response == True
		Resource Owner ->> ACL Smart Contract: 3. AddToACL(@RR, resource, privilege)
	end


资源请求者在对资源发起访问控制前，必须先在智能合约中定义的ACL里进行注册，然后才能向矿工申请到访问用的token。为了实现这一点，资源请求者发送一个注册请求到资源所有者来申请对特定资源的访问权限，如果资源所有者通过该请求，就会将请求者加入到合约中的ACL中，并返回一个接受注册请求的交易，如果拒绝该请求，就会返回一个拒绝注册请求的交易。

**授权过程**

一旦请求者收到资源所有者返回的接受注册请求的交易，就发送一个授权请求交易到矿工，其中包含资源与权限。矿工通过智能合约中定义的ACL验证其是否真的拥有对所请求资源的权限。如果确认拥有权限，就生成一个token，包含请求者地址、资源、权限和表示token生命周期的时间戳，矿工对token进行签名并加密，利用授权响应交易将其发送给请求者节点。如果验证后发现没有权限(智能合约返回无效请求)，矿工就发送一个拒绝授权响应交易给请求者的地址。接收到token后，请求者发送包含token的访问请求交易到资源请求者的地址，资源所有者解密token，验证是否被矿工签名，如果是，访问通过，否则拒绝访问请求。

## 5. 仿真

作者使用V2X(Vehicle to everything)通信系统验证方案的可行性，使用该系统意味着和汽车通信的对象可能是其它汽车、基础设施或任意连接到汽车网络的其它对象。将驾驶员辅助系统ADAS作为资源，汽车作为资源所有者，任何已连接的对象都可以是资源请求者。

选择以太坊平台进行实验，使用Truffle框架编译、测试和部署智能合约，使用Geth作为客户端操作区块链节点，使用Node.js编写代码调用web3.js API和区块链节点通信。

实验使用的设备为电脑和树莓派，两者均安装以太坊节点，电脑中的节点作为矿工和资源请求者，树莓派中的节点作为资源所有者(汽车)。一旦存储ACL的智能合约部署到区块链，资源请求者就可以从矿工申请访问token。

作者实现了以下接口用于交互：

- 资源所有者节点接口：运行后显示所连接的汽车节点的地址和汽车节点部署的智能合约的地址，另外，还会显示汽车所有者可执行的操作列表，包括检查以前的请求、设置新的ACL属性、响应注册或访问请求、退出列表。

  ![Connected car node interface](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8761262/8766347/8766506/riabi2-p6-riabi-large.gif)

- 请求者节点接口：该接口要求请求者输入自己的账户地址和密码，验证通过后显示资源请求者可执行的操作列表，包括检查之前接收到的交易，发送注册请求、申请token的请求或访问请求，退出列表。

  ![Requester node interface](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8761262/8766347/8766506/riabi3-p6-riabi-small.gif)

- 矿工接口：该接口显示节点地址和矿工可执行的操作列表，包括检查之前的请求、响应对token的申请和退出列表。

  ![miner interface](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8761262/8766347/8766506/riabi4-p6-riabi-small.gif)

授权过程被划分为两阶段，第一阶段矿工基于智能合约中的ACL验证和授予token，第二阶段资源所有者基于验证token有效性授予访问权限。

1. 矿工基于ACL中定义的权限进行授权，发送对应的token

   ![Accepting an authorization request](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8761262/8766347/8766506/riabi5-p6-riabi-small.gif)

2. 资源所有者验证token有效后授予访问权限

   ![Accepting an access request](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8761262/8766347/8766506/riabi6-p6-riabi-small.gif)

## 6. 总结与启发

作者确实利用区块链设计了一个完整可行的访问控制方案，解决了单点故障和可扩展性问题。在该方案中，作者将矿工纳入权限授予过程，保证了权限的不可篡改，和区块链结合的较为深入，这是一个亮点。但依然存在以下两个问题：

1. 请求者发起一次访问请求需要经过2~3次通信过程。当第一次发起对某个资源的访问请求时，需要首先向资源所有者发起注册请求并获取回应，然后向矿工发起请求获取token，最后再一次向资源所有者发起请求获取权限，总计3次通信过程，之后每次发起请求，依然需要获取token和获取权限两次通信。由于方案中通信的实质是区块链中的交易，而交易打包到区块并经过验证拥有一段确认时间，多次往返通信会造成一个较大的延迟，不利于物联网环境中的实际操作。
2. 资源所有者利用ACL定义请求者对资源的访问权限，从方案设计来看，每次有新的请求都需要资源所有者主动识别和确认是否授权，物联网环境设备数量较大，因此单位时间产生的访问请求量级也比较大，这种授权方式工作量较大，不利于操作。

