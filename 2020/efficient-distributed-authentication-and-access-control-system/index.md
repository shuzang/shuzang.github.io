# Efficient Distributed Authentication and Access Control System


Benhadj Djilali H., Tandjaoui D. (2019) Efficient Distributed Authentication and Access Control System Management for Internet of Things Using Blockchain. In: Renault É., Boumerdassi S., Leghris C., Bouzefrane S. (eds) Mobile, Secure, and Programmable Networking. MSPN 2019. Lecture Notes in Computer Science, vol 11557. Springer, Cham

DOI：[https://doi.org/10.1007/978-3-030-22885-9_5](https://doi.org/10.1007/978-3-030-22885-9_5)

Keywords：Internet of Things, Access controls system management, Authentication, Blockchain, Security.

## 1. Introduction

IoT现在的身份认证体系主要是基于PKI(公钥基础设施)的，以Certificate Authority(CA)作为第三方可信机构，通过数字证书来认证和管理身份。该体系有如下缺点：

1. 以CA为核心的中心化体系具有**单点故障**的问题，容易被侵入导致安全问题
2. 使用数字证书在验证过程中引入了**计算开销**，在带宽角度还有较高**通信开销**
3. 为了在消息中包含证书，每条消息长度额外增加，导致通信量大、网络拥塞和资源耗尽，

许多方案致力于减少计算和通信开销，作者提出利用区块链技术构建一个用于IoT的安全的轻量级访问控制系统，核心思路是：

1. 利用区块链存储PKI体系中的数字证书，缓解单点故障问题
2. 对每条消息中嵌入的数字证书的验证过程转换为区块链中存储的数字证书的查询过程，减少认证过程(证书交换和验证)的开销

论文其余部分组织如下：第二部分总结相关工作，第三部分描述所提出的方案，第四部分讨论和评估系统性能，第五部分总结全文。

第二部分没有实质性的东西，略过。

## 2. Proposal

作者所提出的方案是一个用于物联网网络的基于区块链的访问控制系统管理机制，设计目的是消除单点故障、减少原本的中心化PKI体系的通信和验证开销。为了实现这一目的，必须确保下面的功能集的实现：

1. IoT设备**注册**公钥
2. **验证**IoT设备公钥的成员身份
3. **查找/验证**物联网公钥的有效性。
4. 从网络中**撤销**物联网的公钥。

这些函数的细节描述分别在下面各小节中，在此之前，会先进行一次系统总览。

由于IoT设备的有限能力，作者使用椭圆曲线加密ECC和数字签名ECDSA，而不是普通的公钥加密RSA。

### 2.1 System Overview

作者的目标是保证物联网安全的同时，实现物联网的大规模部署。区块链可以为任何类型的数据提供分布式、安全和不可变的记录，如果把认证信息(比如证书)存入区块链，交换和验证这些信息就会变得不再必要。另外，区块链分布式的特性还消除了对可信第三方的需要，从而避免了单点故障发生的可能性。

下图是系统的总体结构。其中，区块链网络（这篇文章中是permissioned BC）覆盖在现有的物联网网络之上，「验证者」是城市中的一组有一定处理和存储能力的智能实体，如智能交通灯、基站、智能路灯和其它路边的单元等，负责执行共、验证交易有效性。「权威(Authorities)」类似于传统的CA，在智慧城市场景下可能是车辆部门、电子政府、制造商或公司等，作用是证明特定IoT设备是网络的一员。

在该方案中，不是将来自不同权威的证书收集到一个单独的地方进行决策，而是将所有证书推送到区块链网络，然后由验证者以完全分布式的方式进行决策。IoT设备加入/离开网络的时候，相关的准入/撤销消息以交易形式发送到区块链中，由验证者进行验证并做出最终决策。以IoT设备退出网络为例，检测到恶意行为的某个IoT设备会发送交易到区块链，然后验证者们根据预定义的规则做出从网络中删除可疑IoT设备的决策。

![General architecture of the system](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20200209_%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84)

一个IoT设备不再需要将证书附加到每条消息中，接收者只需要做一个简单的查找来检查发送者在区块链中的有效状态。通过这种方式，IoT设备间可以以最小的通信和计算开销进行相互认证。

### 2.2 Details

**IoT Device Registration**

一个 IoT 设备首先需要在网络中进行注册。以设备 IoT<sub>i </sub>为例，首先自己生成公私钥对 (pkIoT<sub>i</sub>，skIoT<sub>i</sub>) 用于签名和验证。为了在区块链中拥有一个有效的成员状态，IoT<sub>i </sub> 需要从相应的权威处获取注册证书，验证合法的情况下，权威 a<sub>j</sub> 使用其私钥 ska<sub>j</sub> 发布一个已签名的证书给IoT<sub>i</sub>，通过注册交易将证书推送到区块链网络中。证书具有如下格式，其中 cert 是生成的证书，sig是证书的签名，证书主要包含IoT设备公钥和有效期。
$$
<cert, registered, sig<ska_j, cert>
$$


**IoT Device Admission**

当区块链网络从一个权威处接收到注册交易时(可能是授权新设备或重新授权被撤销的设备)，如果来自一个经过认证的机构，则被接受。当收到关于 IoT<sub>i</sub> 足够的证书时，某个验证者就会生成一个新的准入交易，从而在区块链中将IoT设备的公钥标记为有效，其它的验证者随之会验证其有效性并将其添加到本地区块链的副本，验证有效性包括检查验证者签名和 IoT<sub>i</sub> 是否拥有足够的证书。准入交易的格式如下：
$$
< pkIoT_i, valid, sig(skm_j, pkIoT_i) >
$$
**IoT Device Authentication**

一旦 IoT<sub>i</sub> 被标记为有效，他就可以加入网络并开始发送消息。每个接收到来自 IoT<sub>i</sub> 的消息的设备都会检查该设备公钥是否在区块链中存在以及是否被标记为有效，这一步骤通过在区块链中进行简单的公钥值匹配完成。

**IoT Device Revocation**

设备 IoT<sub>j</sub> 检测到恶意行为后，必须发送一个恶意行为交易通知区块链网络，该交易格式如下
$$
< pkIoT_i, misbehavior, sig(skIoT_j, pkIoT_j) >
$$
其中 pkIoT<sub>i</sub> 是被怀疑的IoT设备的公钥，skIoT<sub>j</sub> 是检测到恶意行为的设备的私钥。如果交易来自有效的IoT设备，验证者会将交易收集到区块链中，这些交易稍后会被验证从而决定是否将被怀疑的设备从网络中删除。撤销资格的规则由权威设置，由验证者强制执行，例如，如果24小时内超过 n 个关于 IoT<sub>i</sub> 的恶意交易被收集到区块链中，则将其驱逐出网络。这一过程通过验证者发起撤销交易并广播到整个区块链实现，撤销交易的格式如下，其中skm<sub>j</sub> 是验证者的私钥
$$
< pkIoT_i, revoked, sig(skm_j, pkIoT_i) >
$$
一旦收到撤销交易，其它的验证者在验证其来源后就会将其添加到区块链。

以上提到的四种交易总结如下表

| 交易类型              | 发送者  | 交易格式                                                     |
| --------------------- | ------- | ------------------------------------------------------------ |
| 注册(Registration)    | 权威    | < cert, registered, sig(ska<sub>j</sub>, cert) >             |
| 准入(Admission)       | 验证者  | < pkIoT<sub>j</sub>, valid, sig(skm<sub>j</sub>, pkIoT<sub>i</sub>) > |
| 恶意行为(Misbehavior) | IoT设备 | < pkIoT<sub>i</sub>, misbehavior, sig(skIoT<sub>j</sub>, pkIoT<sub>j</sub>) > |
| 撤销(Revocation)      | 验证者  | <pkIoT<sub>i</sub>, revoked, sig(skm<sub>j</sub>, pkIoT<sub>i</sub>) > |

## 3. Discussion

### 3.1 存储优化

**Multiple blockchains**：不是将注册、准入、恶意行为和撤销等所有消息都保存到一条区块链中，而是不用类型的数据可以存储在不同的区块链。这种情况下，设备可以只使用准入和撤销区块链，只用这两种已经足够认证任何接收到的消息的来源，因此能节省一定的存储空间。

**Cryptographic accumulator**：思路是将一组有效的IoT设备累加到一个智能对象中，每个IoT设备在累加器中都有一个见证者可以证明它已注册。这种情况下，只需要在区块链中保存累加器即可，IoT设备只需要在消息中包含其见证者，接收者就可以通过一个简单的函数检查其成员资格。这种方法能大大减小区块链的大小，并且可以在不影响存储性能的情况下扩展到非常大的网络规模。

### 3.2 性能分析

通过使用区块链，设备消息的验证变成了一个简单的查询函数，为了比较本文方案和传统方案的验证时间，做了如下实验。

为了测试实际场景下查询函数的性能，使用比特币区块链。在比特币区块链中，为了加速访问和搜索操作，需要使用LevelDB 数据库。比特币主要使用两个数据库，一个包含所有已知区块的元数据，存在硬盘，另一个包含UTXO信息，值得注意的是，可以定制数据库从而满足特定需要。在本文方案中，加速了在区块链数据库中搜索特定交易的响应时间。通过使用LeveDB C++，可以直接访问数据库并通过交易标识符(TXID)搜索特定交易。实验使用的硬件和数据库的总结如下表所示

| 配置类型               | 详细信息                                |
| ---------------------- | --------------------------------------- |
| CPU                    | 8 ∗ Intel R Core(TM) i7-6700HQ 2.60 GHZ |
| CPU-cache              | 6144 KB                                 |
| LevelDB                | Version 1.1                             |
| Number of transactions | 36328994                                |

结果显示，1000次查询的平均查找时间为0.012ms，传统方案对于长度为256的密钥，验证时间为0.1ms，使用区块链的优势很明显，延迟几乎减少了10倍。

## 4. Idea

这篇论文的偏向是认证，没有使用任何访问控制模型，就是简单的将PKI体系中的证书保存到区块链中，并做了一定的适应性调整。性能有实际的提升，同时具有很大的可行性，能够进行大规模的实际部署。缺点在于这种方式实现的访问控制是粗粒度的，最基本的单元是IoT设备，无法具体到每个资源，而且权限区块链本身对节点准入就会进行控制，功能上可能有一定重合。

最大的启发在于没有完全抛弃原来的CA，权威依然作为区块链的一部分参与进来，对于区块链监管体系的建立有一定参考价值。



---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2020/efficient-distributed-authentication-and-access-control-system/  

