---
title: A Blockchain based access control for IoT
date: 2020-02-06
tags: [论文]
categories: [研究生的区块链学习之路]

---

Imen Riabi, Yosr Dhif, Hella Kaffel Ben Ayed, Khaled Zaatouri. A Blockchain based access control for IoT[C]. International Wireless Communications & Mobile Computing Conference (IWCMC), 2019.

DOI: [10.1109/IWCMC.2019.8766506](https://doi.org/10.1109/IWCMC.2019.8766506)

Keywords: Access control, Blockchain, Smart Contract, Internet of Things

## 1. Introduction

作者的考虑主要基于三点

1. 传统中心化的访问控制带来的单点故障和可扩展性问题；
2. 资源有限的IoT设备需要轻量级访问控制方案(对CPU、内存、功耗的低需求)
3. 对低延迟的需求

整篇文章其余部分的组织结构为：第二部分为相关工作，第三部分为区块链技术介绍(阅读时略过)，第四部分为区块链安全机制介绍，第五部分阐述提出的访问控制模型，最后实施提出的方案和总结全文。

## 2. Related Works

说明已有物联网访问控制方案不合适的原因

**RBAC**：the Role based access control，基于角色的访问控制，物联网环境高度动态且用户数量巨大，纯RBAC无法应对。

**ABAC**：the Attribute based access control，规则的数量随着用户、属性的增长迅速增加，不适用于物联网高度动态和实时的环境。

**Cap-BAC**：Capability based access control，主要指OAuth-IoT，主要的问题是中心化结构带来的可扩展性问题和高延迟以及不支持可移动性。

## 3. BC Security Mechanisms

该部分讨论区块链的安全机制从而评估其安全级别。区块链提供的安全服务和对应的实现机制如下表所示：

| 区块链安全服务 | 使用的机制                   |
| -------------- | ---------------------------- |
| 完整性         | 哈希函数                     |
| 交易真实性     | 数字签名                     |
| 机密性         | 非对称加密                   |
| 可用性         | 多个副本分布于整个网络       |
| 匿名性         | 公钥用作节点地址             |
| 可追溯性       | 所有交易记录在区块链中       |
| 防篡改         | 需要大量的算力才能破坏区块链 |

## 4. The Proposal

选择的模型是 Capability-BAC 和 Identity-BAC 的结合，使用Capability-BAC提供的token用于向请求者授权，使用Identity-BAC 提供的ACL记录subjects列表和它们的访问权限，资源所有者在智能合约中注册从而形成ACL，目的是利用区块链和智能合约取代传统的中心化授权服务器。一个传统访问控制和基于区块链的访问控制角色对应表如下

| 传统访问控制 | 基于区块链的访问控制     |
| ------------ | ------------------------ |
| 资源所有者   | 区块链中的资源所有者节点 |
| 请求者       | 区块链中的请求者节点     |
| 授权服务器   | 区块链中的矿工节点       |

### 4.1 Actor of the solution

进一步介绍上表提到的角色

- 资源所有者：区块链中拥有资源的节点，部署智能合约并在智能合约中定义其资源的访问控制列表，接收来自区块链中其它节点(请求访问资源的节点)的注册请求。
- 矿工：区块链中专门计算的节点，管理来自请求者的授权请求，授予对资源具有特定访问权限的token。矿工是授权服务器的替代，可以基于资源所有者已部署的智能合约中的ACL生成token。
- 请求者：区块链中想要以指定权限访问特定资源的节点，希望获取对应的访问token。

### 4.2 Proposed mechanisms

该方案中使用智能合约存储和管理ACL，每个资源所有者在自己部署的智能合约中定义资源相关的ACL，这些ACL被矿工用于验证请求者的访问权限，从而生成访问token并给予对应的请求者。节点间的通信使用区块链本身的交易机制。

### 4.3 Registration process

一个新的资源请求者在智能合约中注册的时序图如下

{{< mermaid >}}
sequenceDiagram
    participant Resource Requester
    participant Resource Owner
	participant ACL Smart Contract
	Resource Requester ->> Resource Owner: 1. Registration Request Transaction(resource privilege)
	Resource Owner ->> Resource Requester: 2. Registration Response Transaction
	alt if Registration Response == True
		Resource Owner ->> ACL Smart Contract: 3. AddToACL(@RR, resource, privilege)
	end
{{< /mermaid >}}

资源请求者在对资源发起访问控制前，必须先在智能合约中定义的ACL里进行注册，然后才能向矿工申请到访问用的token。为了实现这一点，资源请求者发送一个注册请求到资源所有者来申请对特定资源的访问权限，如果资源所有者通过该请求，就会将请求者加入到合约中的ACL中，并返回一个接受注册请求的交易，如果拒绝请求，就会返回一个拒绝注册请求的交易。

### 4.4 Authorization process

一旦请求者插入到资源所有者的ACL中，他就会发送一个授权请求交易到矿工，其中包含资源与权限。矿工通过智能合约中定义的ACL验证其是否真的拥有对所请求资源的权限。如果有权限，就生成一个token，包含请求者地址、资源、权限和表示token生命周期的时间戳。矿工对token进行签名并加密，利用授权响应交易将其发送给请求者节点。如果没有权限(智能合约返回无效请求)，矿工就发送一个拒绝授权响应交易给请求者的地址。接收到token后，请求者发送包含token的访问请求交易到资源请求者的地址。最后，资源所有者解密token，验证是否被miner签名，如果是，访问通过，否则拒绝。

## 5. Implementation

使用V2X(Vehicle to everything)通信系统验证方案的可行性，使用该系统意味着和汽车通信的对象可能是其它连接的汽车、基础设施或任意连接到汽车网络环境的对象。此外，还提供了驾驶员辅助系统ADAS作为资源提供，此时，连接的汽车是资源所有者，任何已连接的对象都可以是资源请求者。

使用以太坊平台和其上的智能合约环境实施该实验，使用Truffle框架编译、测试和部署合约到实际的区块链环境，使用Geth作为命令行工具操作区块链节点，使用Node.js编写代码调用web3.js API和区块链节点通信。

实验使用了电脑和树莓派，两者安装以太坊节点，电脑中的节点可作为矿工或资源请求者，树莓派中的节点作为汽车(资源所有者)。一旦存储ACL的智能合约部署完成，资源请求者就可以从矿工请求访问token。

实验实现了以下接口用于同私链中的解决方案交互：

- 汽车节点接口：该接口显示所连接的汽车节点的地址和汽车节点部署的智能合约的地址，另外，还会显示汽车所有者可执行的操作列表，包括检查上一个请求、设置新的ACL属性、响应注册或访问请求、退出列表。

  ![Connected car node interface](https://user-images.githubusercontent.com/26682846/73933416-5c4f5580-4917-11ea-9472-1f6729e715cc.png)

- 请求者节点接口：该接口询问请求者的账户地址和密码，验证后显示资源请求者可执行的操作列表，包括检查之前接收到的交易，发送注册、新的token或访问请求，退出列表。

  ![Requester node interface](https://user-images.githubusercontent.com/26682846/73933411-5a859200-4917-11ea-82aa-ac18b5d7decb.png)

- 矿工接口：该接口显示节点地址和矿工可执行的操作列表，包括检查之前的请求、响应对token的请求和退出列表。

  ![miner interface](https://user-images.githubusercontent.com/26682846/73933415-5bb6bf00-4917-11ea-9ac4-1578c14dbae5.png)

授权过程被划分为两阶段，第一阶段基于智能合约中的ACL验证和授权请求，第二阶段基于已获得的token验证访问请求。

- 接受授权请求：矿工基于ACL中定义的权限进行授权，发送对应的token

  ![Accepting an authorization request](https://user-images.githubusercontent.com/26682846/73933417-5ce7ec00-4917-11ea-84a5-78e8d4a43852.gif)

- 接受访问请求：资源所有者验证token有效后授予访问权限

  ![Accepting an access request](https://user-images.githubusercontent.com/26682846/73933418-5d808280-4917-11ea-87b9-5cf5cc6f0d06.gif)

## 6. Discussing

该方案中访问需要经过2~3次通信，延迟可能比较大，同时，使用ACL定义访问控制权限，每个访问请求都需要资源所有者处理，人工的话工作量可能比较大。

