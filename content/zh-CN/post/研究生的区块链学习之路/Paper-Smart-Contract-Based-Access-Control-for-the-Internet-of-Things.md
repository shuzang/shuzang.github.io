---
title: Smart Contract-Based Access Control for the Internet of Things
date: 2019-05-14T19:10:00+08:00
tags: [论文笔记]
categories: [研究生的区块链学习之路] 
---

Zhang Y, Kasahara S, Shen Y, et al. Smart contract-based access control for the internet of things[J]. IEEE Internet of Things Journal, 2018.

被引：35次

## Abstract

该论文调查了物联网中的访问控制问题。提出了一个基于智能合约的访问控制框架，该框架包括多个访问控制合约（access control contracts, ACCs），一个判决合约(Judge contract, JC)，一个注册合约（register contract, RC），来实现物联网中分布式和可信任的访问控制。每个ACC为一个subject-object提供一个访问控制方法，并通过检测subject行为实现基于预定义策略的静态访问权验证和动态的访问权验证。JC则通过接收来自ACC的恶意行为汇款，进行行为判决并返回相应的处罚策略，从而实现ACCs的动态验证。RC记录ACC和恶意行为判决方法信息，并提供管理这些方法的相关功能，如注册、更新和删除。最后使用一台台式机，一台笔记本，两台树莓派，基于以太坊智能合约实现了该架构。

Section2介绍该论文考虑的物联网系统

Section4介绍基于智能合约的访问控制框架

Section5介绍该框架实验

## Introduction

IoT系统可能遭受入侵，导致相关资源（数据、服务、存储单元、计算单元等）的合法权限被获取，访问控制是阻止未授权实体对资源非法访问的主要方式。传统的IoT访问控制方案主要建立在一些著名的访问控制模型上，如RBAC、ABAC和CapBAC。RBAC方案中，访问控制基于组织中主体的角色，通过为角色关联一组访问权限，并将角色分配给主体，可以建立一个主体和访问权限间的多对多关系。ABAC的访问控制基于策略，该策略结合不同类型的属性，如主体属性、客体（实体或实体持有的资源）属性和环境属性等来表示在什么样的情况下可以授予主体权限。CapBAC中基于权能的概念授予主体访问权限，权能是可转让而不可变的权限令牌，为每个主体描述了一组访问权限。

以上的方案中，验证主体的访问权限一般有中央权威进行，存在单点故障问题。分布式 CapBAC被提出解决这一问题，其中访问权限验证由被请求的IoT对象自己来指向而不是中央权威。然而，IoT对象通常能力不足而会被入侵者轻易控制，所以它们无法作为访问权限验证实体被完全信任，因此，分布式CapBAC模型可能无法解决不可信IoT环境下的访问控制。作者提出使用区块链和智能合约来实现用于IoT的分布式和可信的访问控制，本文的访问控制框架由一个注册合约（RC）、一个判决合约（JC）和多个访问控制合约（ACC）组成，其中每个ACC提供一个subject-object对的访问控制方法，并同时实施基于预定义访问策略的静态权限验证和基于检测subject行为的动态权限验证。ACC还提供添加、更新、删除访问控制策略的功能。一旦ACC被调用，它将被区块链系统的所有参与者验证，确保访问控制的可信。为了实施动态验证，JC提供了错误行为判决方法，它接收来自ACC的关于subject的恶意行为报告，进行判决并返回相应的判决结果。为了管理访问控制和恶意行为判决方法，RC可以注册方法的相关信息，提供注册新方法、更新和删除已有方法的功能。

## System and Security Model

### A. 物联网系统结构

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/6488907/8709863/8386853/zhang1-2847705-small.gif)



- 服务器。与IoT设备/存储设备链接，为用户提供服务。交互方式包括
  1. 从传感器收集环境数据，
  2. 向执行器发送命令执行某些操作，
  3. 从存储设备查询数据或将数据存到存储设备
- 存储设备。存储对其它各方有用的数据，包括服务器运行数据，传感器收集的环境数据，用户配置文件等。
- 用户设备。例如PC，Laptop，smart phone，享受服务器提供的服务，如查询当前温度，并可从存储设备读写数据。
- IoT网关。连接大量IoT设备，作为它们的代理
- IoT设备。感知环境数据发给server/storage device，执行来自用户的命令

### B. 安全模型

典型的物联网应用中，每个节点都拥有一些其它节点需要的资源（服务、数据、存储空间等）。因此，资源所有者需要实施访问控制阻止未经授权的访问，例如，服务器需要能阻止未注册用户的访问请求，或已注册用户对未订阅服务的请求。未来阻止对存储空间和数据的非法使用，存储设备必须能限制来自未授权节点查询数据或存储数据的请求。IoT设备也必须能拒绝未授权而对其数据的检索或对执行器的控制。

该论文使用下面论文中的访问控制矩阵抽象了访问控制问题。

> R. S. Sandhu and P. Samarati, “Access control: Principle and practice,” sIEEE Commun. Mag., vol. 32, no. 9, pp. 40–48, Sep. 1994.

其中，定义希望访问其它节点资源的subject集合S，持有资源的object集合O。每个属于集合O的object(称为o)，都有资源集R0（例如，文件和程序）。每个属于资源集R0的资源r0都与访问权集合Ar0（如读、写、执行）关联。对于每个subject（称s）和给予它的资源r0，构成映射G(s,r0)，该映射属于访问权集合Ar0。

访问控制矩阵只给了一个抽象的定义，该论文中使用访问控制列表实现，列表中每条包括subject，object的资源，subject执行的操作，以及对操作的许可（allow或deny）。主要目的是基于该模型处理物联网中的访问控制问题。

## Access Control Framework

### A. 智能合约系统

每个ACC实现一对节点的访问控制，JC进行恶意行为判决，RC存储JC和ACCs信息并提供管理合约的功能。如下图所示。

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/6488907/8709863/8386853/zhang3-2847705-small.gif)

#### 1. ACC

ACC由object部署，这个object想要控制来自subject的访问请求，假设一个subject-object对就访问控制方法达成一致，并且每个访问策略由一个ACC实现，则该subject-object对与多个ACCs关联，但一个ACC仅可以和一个subject-object对关联。框架中，为了控制来自subject的访问请求，每个ACC的实施不只是通过检查预定义策略进行的静态访问权验证，而且检查subject行为进行动态验证。

一个ACC例子如下表，每行都是一个确定的（resource, action）对，Permission字段可用于静态验证，ToLR（Time of Last Request）字段可用于动态验证，如subject短时间发起大量请求的恶意行为。

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/6488907/8709863/8386853/zhang.t1-2847705-small.gif)

为了记录subject的恶意行为，ACC还维持一个恶意行为列表如下

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/6488907/8709863/8386853/zhang.t2-2847705-small.gif)



Misbehavior字段也可能描述恶意行为的细节，便于JC进行判决。ACC还提供一些函数接口如下以便管理策略和实施访问控制

- policyAdd()
- policyUpdate()
- policyDelete()
- accessControl()：接受访问控制请求并返回结果和惩罚。
- setJC()
- deleteACC()：自我销毁

只有ACC创建者可以添加新策略，更新和删除已有策略，设置JC和删除ACC。

#### 2. JC

JC接收到ACC发来的恶意行为报告是，基于subject的恶意行为历史来基于惩罚，并将判决结果返回ACC。一个JC示例如下，Object是遭受恶意行为的节点：

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/6488907/8709863/8386853/zhang4-2847705-small.gif)

JC还接收misbehaviorJudge()和deleteJC()两个函数接口进行恶意行为报告和自毁。

#### 3. RC

RC管理访问控制和恶意行为判断方法。因此，RC维护一个查找表，该表注册所需信息以查找和执行所有方法，查找表示例如下：

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/6488907/8709863/8386853/zhang.t3-2847705-small.gif)

JC的subject和object字段留空。依据查找表，RC提供以下函数接口

- methodRegister()
- methodUpdate()
- methodDelete()
- getContract()：接收MethodName并返回合约地址和和合约ABIs

### B. 架构主要功能

包括注册、更新和删除访问控制方法；注册和更新恶意行为方法；添加、更新和删除ACC策略，进行访问控制等。基本都是通过调用相应的合约函数实现。

## Case Study

### A. 软硬件

两台电脑，两套树莓派。电脑作为用户设备，树莓派作为IoT网关。在树莓派间考虑访问控制，一个作为subject，一个作为object。每个设备上都运行geth，并配置所有设备形成私有区块链网络。两台电脑能力较强所以作为矿工。

### B. 实现

主要是ACC、JC、RC合约逻辑

### C. 实验

就是进行各种调用查看结果

### D. 花费与开销

主要考虑了各操作的gas消耗和执行时间。

## Conclusion

本文研究了物联网中的访问控制问题，为此我们提出了一个基于智能合约的架构来实现分布式和可信赖的访问控制。该框架包括用于系统中多个subject - object对的访问控制的多个ACC，一个用于在访问控制期间判断subject的恶意行为的JC，以及一个用于管理ACC和JC的RC。还为物联网系统中的访问控制提供了案例研究，其中包括一台台式计算机，一台笔记本电脑和两台Raspberry Pi。案例研究证明了所提架构在实现物联网分布式和可信赖访问控制方面的可行性