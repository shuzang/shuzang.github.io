---
title: An Attribute-Based Distributed Access Control for Blockchain-enabled IoT
date: 2020-04-22
tags: [论文笔记]
categories: [研究生的区块链学习之路] 
---

P. Wang, Y. Yue, W. Sun, and J. Liu, “An Attribute-Based Distributed Access Control for Blockchain-enabled IoT,” in *2019 International Conference on Wireless and Mobile Computing, Networking and Communications (WiMob)*, Barcelona, Spain, Oct. 2019, pp. 1–6, doi: [10.1109/WiMOB.2019.8923232](https://doi.org/10.1109/WiMOB.2019.8923232).

第一作者是西电的，国家自然科学基金项目成果，研究方向完全一致，都是利用智能合约实现 ABAC 模型完成物联网访问控制。可以看的出来，这篇论文也深受 Zhang[^zhang2019smart] 的影响，参考的文献全都看过，由于方向一致，这是必然的事情。因此，这里记的笔记不包括引言、相关工作、实验等部分，仅仅理解和阐述作者的架构描述，并分析其优缺点，将优点吸纳到我们当前的方案。

[^zhang2019smart]:Y. Zhang, S. Kasahara, Y. Shen, X. Jiang, and J. Wan, “Smart contractbased access control for the internet of things,” IEEE Internet of Things Journal, vol. 6, no. 2, pp. 1594–1605, 2019.

## 1. 系统架构

用来说明方案的 IoT 场景是 smart home，如下图所示。作者将设备分为三类

1. 有足够计算和存储能力的节点，比如 server 和 desktop，这部分作为区块链全节点；
2. 只有有限的计算和存储能力，比如 mobile phone 和 smart TV，这部分作为轻节点；
3. 计算和存储能力高度有限，比如传感器，这些设备称为超轻量级节点，由所连接的网关作为代理。

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8913409/8923119/8923232/wang1-p6-wang-small.gif)

Server 负责存储 IoT 设备产生的数据，包括传感器收集的环境信息、运行过程产生的日志文件，同时向设备提供服务，因此也会发送一些命令到设备从而控制设备的执行。全节点或轻节点通过有线或 Wi-Fi 连到网络，维持区块链的运行，保存所有或部分访问控制信息，执行访问控制。超轻量级节点通过 Bluetooth、Wi-Fi、ZigBee 等技术连到网关，从而连到网络，不存储访问控制信息，只通过网关发起访问控制请求或获取访问控制结果。

## 2. 访问控制架构

核心是 ABAC 模型，如下图所示，收到访问控制请求后根据主体属性（Subject Attribute, SA）、客体属性（Object Attribute, OA）和环境属性（Environment Attribute, EA）执行预定义的策略，从而得到结果。

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8913409/8923119/8923232/wang2-p6-wang-small.gif)

作者使用了一个 Subject Contract（SC）、一个 Object Contract（OC）、一个 Access Control Contract（ACC）和多个 Policy Contracts（PC）来实现该模型，各部分介绍如下。

### 2.1 Subject Contract

由于设备可以作为 subject 发起访问，也可以作为 object 提供资源，subject 访问不同的 object 时属性还有可能不同，作者将主体属性划分为两部分：

1. Manufacturer Attribute（MA）：由制造商在出厂时设置的属性，无法二次修改，主要包括一些设备的基本信息，比如 MAC 地址和序列号。
2. Setting by Object Attribute（SOA）：如果设备 o 作为 object 设置了针对 subject s 的 SOA，意味着该属性只在 s 访问 o 时生效，其它 subject 访问 o 不生效。

SC 负责管理合法制造商的账户、IoT 设备账户和设备的主体属性信息，属性以键值对的形式定义，如下所示
$$
[name_1:value_1] [name_2:value_2] ……  [name_n:value_n]
$$
一个 MA 的示例为 $[type:remotecontrol][mac:00efefefefef]$，

一个 SOA 的示例为 $[group:owner][role:children]$

SC 提供了如下功能

1. addmanufacturer()：只能由 SC 所有者调用，传入制造商账户地址，将该地址代表的制造商加如合法制造商列表
2. addsubject()：只能由合法制造商列表中的成员调用，负责注册新的 IoT 设备
3. addobattr()：由 object 调用，设置针对某个 subject 的 SOA，接收设备地址和一个描述属性的字符串
4. delemanufacturer()，deleteobattr()：如函数名

### 2.2 Object Contract

OC 负责管理每个设备的 object attributes（OA），和主题属性的结构定义相同，一个 OA 示例为$[type:TV][location:living]$。OC 提供的函数功能有：

1. addobattr()：接收设备地址和一个描述属性的字符串，设置客体属性
2. deleteobattr()：接收设备地址，删除对应的属性
3. getattr()：接收设备地址，获取对应的属性

### 2.3 Policy Contract

每个用户创建自己的 PC，并在 ACC 中和用户的设备进行绑定，因此，一个 PC 可能对应多个 IoT 设备，但只有一个所有者且只有所有者可以添加或删除策略。

[策略]^(policy) 和 [规则]^(rule) 在 PC 中是不同的，策略由如下五个字段定义
$$
resource, action,duty,rule,algorithm
$$
其中，$duty$ 是实施完访问控制需要做的事；一个策略可能包含多个规则，一个 $rule$ 由 $SA,OA,EA,resource,action,result$ 六部分组成；$algorithm$ 用来在规则产生矛盾时进行判定；返回的结果有两种：$allow$ 和 $deny$。一个示例为

> policy: [resource:switch] [action:on] [duty:record] [algorithm:denyoverrides] 
>
> rule1: 
>
> ​	subject attribute: [group:owner] [role:parent] [type:remotecontrol]
>
> ​	object attribute: [type:TV] [location:livingroom]
>
> ​	environment attribute: [time: 21:00 - 23:00]
>
> ​	result: allow

PC 提供如下函数功能：

1. addpolicy()：添加新策略到用户策略集，接收四个参数：resource, action, duty 和 algorithm
2. addrule()：添加新规则到策略，接收六个参数：SA, OA, EA, resource, action 和 result，通过资源和操作，可以找到相应的策略并将规则添加到规则列表
3. delepolicy()，delerule()：如函数名

### 2.4 Access Control Contract

ACC 用来确定请求是否符合用户自定义的策略，最终会返回相应的结果并执行预定的 $duty$，如记录访问历史到区块链等，历史记录结构示例如下

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8913409/8923119/8923232/wang.t1-p6-wang-small.gif)

PC 提供的函数功能如下

1. Initialization()：为了和 SC，OC 交互，记录它们的合约地址

2. setobjectpolicyaddress()：负责将 PC 地址绑定到 IoT 设备地址，接收这两个地址作为参数

3. accesscontrol()：执行访问控制，接收 subject address、object address 、resource 、action 四个参数，与其它几个合约交互获取相应的属性信息和策略信息，然后根据策略中每个规则进行判决并记录，如果满足规则，获取 $allow$ 或 $deny$ 两个结果之一，如果不满足，返回 $NotAplicable$，最后利用 $algorithm$ 处理冲突得到最终结果。如果设置了 $duty$ 字段，那么执行该字段描述的任务。该函数的算法伪代码如下

   ![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8913409/8923119/8923232/wang.al1-p6-wang-large.gif)

## 3. 总结与收获

这篇论文基本可以看作传统的 XACML 的架构使用智能合约的实现，比如 SC 和 OC 用来维护属性信息，相当于策略信息点（PIP）；PC 维护策略信息，相当于策略管理点（PAP），ACC接收并执行访问控制策略，相当于策略实施点（PEP）和策略决策点（PDP）的结合。这种结构将各部分功能进行了良好的划分，确保了低内聚高耦合，所以现在应该去深入理解一下传统 ABAC 模型，了解其缺点后，再讨论是像这篇论文这一原样实现还是做一些改变。

在 Ouaddah 的论文中评价了 ABAC 模型的优劣[^ouaddah2016access]。他认为 ABAC 模型的优点是较好的互操作性和细粒度的访问控制，缺点是较为复杂和非用户驱动，这里介绍一下两个缺点：

1. **Complexity**：属性语义的诠释，属性的可信度，表达基于属性的授权请求和响应的语法定义，这些都是导致 ABAC 复杂的原因。另外，XACML 的复杂性常导致用户避免使用它转而使用更传统的方法。这种复杂性还阻碍了它在日常场景中的应用，例如，可穿戴娱乐物联网应用领域。只有在需要高度互操作性和细粒度表达的应用中才适合使用这种模型。
2. **Not User-driven**：尽管 XACML 和 ABAC 是完善而精确的策略描述方法，XACML 策略的结构是复杂的。用户必须深入理解 XACML 才能熟练地写下详细地策略，这使得 XACML 难以掌握和使用。这种方式地隐私管理不支持以[本机方式]^(native way)与用户交互，为了让用户参与策略制定过程，需要一个用户驱动的隐私管理器。

[^ouaddah2016access]:A. Ouaddah, H. Mousannif, A. A. Elkalam, and A. A. Ouahman, “Access control in The Internet of Things: Big challenges and new opportunities,” *Computer Networks*, vol. 112, pp. 237–262, 2016.

可以思考如何在区块链中改善这两者。