# LVChain： A Lightweight and Vote-based Blockchain for Access Control in the IoT


Yu, Yue & Zhang, Sheng & Chen, Chao & Zhong, Xiaoxiong. (2018). LVChain: A Lightweight and Vote-based Blockchain for Access Control in the IoT. 870-874. 

DOI: [10.1109/CompComm.2018.8780687](https://www.researchgate.net/deref/http%3A%2F%2Fdx.doi.org%2F10.1109%2FCompComm.2018.8780687)

KeyWord: IoT, security, Blockchain, access authorization, BLE-based devies

注：插图经过重新绘制，照片来自原论文截图。

## 1. 引言

IoT与生活联系的越来越紧密，因此安全和隐私问题逐渐得到人们的关注。访问控制是安全的一个重要领域，一个完整而有效的访问控制系统应该满足机密性、完整性和可用性，并包括认证、授权和审计三个部分，这篇文章只关心授权部分。

传统授权架构是中心化的，比如著名的有XACML、OAuth和UMA，这种架构很难解决单点故障问题和提供良好的可扩展性，因此正在朝着分布式的方向演变。

作者设计了一条区块链(LVChain)来克服了上面提到的缺点，可以很好的用于蓝牙设备构成的家庭无线自组织网络。作者的主要贡献如下

1. 提出了一个新的基于区块链的分布式架构，是轻量级、可扩展和容错的；
2. 在架构中引入和实施了一个基于投票的共识算法，对计算和存储资源依赖更少；
3. 对性能进行了全面地分析，比较了本文架构、传统中心化架构和现有的分布式架构，在虚拟环境中运行了一个实验证实了本文架构在IoT环境下的可行性。

论文其余部分组织如下，第二部分为背景和相关工作，第三部分为架构总览，第四部分为性能评估和安全分析，第五部分总结全文。

## 2. 相关工作

FairAccess[^1]利用智能合约实现了基于token的访问控制，但是有较大的计算和时间开销，预设的授权规则也不可变。

[^1]:A. Ouaddah, A. Abou Elkalam, and A. Ait Ouahman, “Fairaccess: a new blockchain-based access control framework for the internet of things.” Security and Communication Networks, pp. n/a–n/a, 2017, sCN-16-0184. 

BlendCAC[^2]同样基于token，利用智能合约完成访问授权的注册、传播和撤销。该模型具有较大的计算和存储开销，并且在具有足够资源的树莓派上实现，无法代表多数IoT设备。

[^2]:R.H. Xu, Y. Chen, et al, "BlendCAC: A BLockchain-ENabled Decentralized Capability-based Access Control for IoTs." IEEE Internal Conference on Blockchain IEEE, 2018. 

ControlChain[^3]使用了四种区块链，分别负责记录设备和用户的关系、存储传感器收集的环境信息、存储授权或拒绝访问的权限信息和保存授权规则。该架构只是理论没有仿真或实现，复杂性较高而兼容性较差。

[^3]: Pinno, Otto Julio Ahlert, A. R. A. Gregio, and L. C. E. D. Bona, "ControlChain: Blockchain as a Central Enabler for Access Control Authorizations in the IoT." GLOBECOM 2017 - 2017 IEEE Global Communications Conference IEEE, 2018. 

## 3. 方案

本文的工作用于解决蓝牙设备构成的家庭无线自组织网络中的安全问题，具有如下特征：

1. 分布式结构，克服中心化结构的缺点，更好地满足IoT的开发需求；
2. 基于投票的共识，减小资源有限的IoT设备的计算压力，因此是轻量级的
3. 离线工作，因为控制和授权信息不需要通过连接的蓝牙设备扫描和广播

<img src="https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8767367/8780575/8780687/162-fig-1-source-large.gif" alt="Architecture framework" style="zoom: 50%;" />

架构总体结构如下图所示，分为三层：数据层、网络层和共识层。

数据层包括时间戳、控制信息和授权信息，存储在本地来避免泄露隐私。用户控制设备(如switch等)的行为会生成控制信息，并按时间戳顺序链接在一起，这样做同样有利于接下来的审计工作。授权信息是授权用户的身份信息，通过哈希表存储，从而加快查询速度。

网络层利用P2P协议构建蓝牙设备组成的网状网，由于蓝牙设备的广播和扫描状态，该架构是无连接的。网络中的设备结点是点对点的，一个设备接收到信息，会通过蓝牙转发和广播出去。另外，为了防止网络拥塞，限制了每个消息的转发次数，期间验证机制会验证控制信息的有效性和投票信息是否来自授权用户。

受限于蓝牙设备的计算能力，使用了基于投票的共识算法。为了减少通信开销，共识算法设计为：请求授权的用户在收到大部分授权用户的投票信息时被授权。

![架构工作流](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8767367/8780575/8780687/162-fig-2-source-large.gif)

整个架构的工作流程如上图，当一个用户尝试操作一个设备时，设备会首先根据数据层的授权信息将用户区分为授权用户和非授权用户。如果是非授权用户，设备向整个P2P网络的授权用户发送授权请求，然后授权用户进行投票，每个用户对每个请求只有一票，同意则进行投票，不同意什么都不做。接下来，在共识层中，设备接收投票回应，检查回应的有效性并计算有效投票数量。在一段确定的时间内，如果投票用户的数量超过了授权用户数量的一半，请求者被授权，其信息存储在数据层中并添加到授权信息。如果是授权用户，一方面设备转发控制请求并将控制信息按时间戳添加到区块链，另一方面如果请求针对自己，设备直接进行响应。

## 4. 性能评估

### 4.1 方案比较

论文中的架构和其它架构的比较如下表，其中(*)表示取决于证明类型和块的生成速度

|              | Scalability | Fault Tolerant | New Authorization | Get Authorization | Off-Line working |
| ------------ | ----------- | -------------- | ----------------- | ----------------- | ---------------- |
| XACML        | -           | -              | +                 | +                 | -                |
| OAuth        | -           | -              | +                 | +                 | -                |
| UMA          | -           | -              | +                 | +                 | -                |
| FairAccess   | +           | +-             | -                 | -                 | +-               |
| BlendCAC     | +           | +-             | -                 | +                 | +-               |
| ControlChain | +           | +              | -(*)              | +                 | +-               |
| LVChain      | +-          | +              | +-                | +                 | +                |

**可扩展性**：FairAccess、BlendCAC、ControlChain和本文的LVChain都是分布式结构，具有良好的可扩展性，但由于授权信息的不断增加可能对网络造成影响，LVChain的可扩展性可能略逊于其它三种。

**容错性**：分布式结构具有更好的容错性，但FairAcess和BlendCAC使用token，带有一定的中心化特征，因此略逊于另外两种。

**新授权**：评估改变一个授权的延迟。中心化架构更改授权的延迟较低，本文的LVChain使用基于投票的共识，避免和挖矿过程，延迟稍微好点。

**获取授权**：评估获取一个授权的延迟。FairAccess获取权限需要挖掘两个区块，因此比其他方案略差

**离线工作**：评估设备离线工作的可能性，中心化结构都需要稳定的连接，分布式结构都可以从本地副本查询。

### 4.2 仿真

<img src="https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8767367/8780575/8780687/162-fig-3-source-large.gif" alt="工作平台" style="zoom: 67%;" />

作者如上图所示实施了文中提出的架构，使用智能手机代表用户，配备MCU和蓝牙的智能家庭设备代表终端设备。与树莓派相比，这里设备的计算和存储能力更弱，因此能证明本文架构在资源有限的IoT环境下的适用性。软硬件具体情况如下

| 设备     | 详细信息                                                     |
| -------- | ------------------------------------------------------------ |
| IoT设备  | arm cortex-m3，maximum working frequency 48MHz, RAM 80K, ROM 256K（硬件）<br>Keil arm 5.22（软件） |
| 用户设备 | BLE 4.0， theoretical rate 60Mbps（硬件）<br>ios 8.0（软件） |

当一个未授权用户想要操作设备时，设备查找授权信息确定用户未授权，然后通过蓝牙将授权请求广播给所有授权用户，授权用户接到请求后，如下图所示做出是否同意的决策，选择「NO」不会发送任何信息，选择「YES」发送投票信息。

<img src="https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8767367/8780575/8780687/162-fig-4-source-large.gif" alt="授权请求" style="zoom:67%;" />

一系列测试的结果如下表所示，共使用了两个嵌入式设备，并使授权用户的数量逐渐增加，反映了网络扩张的过程。结果中可以看出，投票用户超过一半时用户被授权。

| The number of authorized users | The number of voting 「YES」 | The number of voting 「NO」 | The operability of the user requesting access |
| ------------------------------ | ---------------------------- | --------------------------- | --------------------------------------------- |
| 1                              | 0                            | 1                           | ×                                             |
| 1                              | 1                            | 0                           | √                                             |
| 2                              | 0                            | 2                           | ×                                             |
| 2                              | 1                            | 1                           | ×                                             |
| 2                              | 2                            | 0                           | √                                             |
| 3                              | 0                            | 3                           | ×                                             |
| 3                              | 1                            | 2                           | ×                                             |
| 3                              | 2                            | 1                           | √                                             |
| 3                              | 3                            | 0                           | √                                             |

### 4.3 安全分析

理论上，由于本文架构是分布式的，信息在每个设备中都有一个备份，单个设备的故障不会影响其它设备的正常运行。不过恶意用户不得超过一半，否则该架构难以正常运行。

## 5. 总结与启发

很多访问控制方案都是利用原来的区块链或智能合约，本文则设计了一条完全用于访问控制的新链，这也是一种思路。而且作者构建的区块链是完全建立在IoT设备上的区块链，由蓝牙设备作为节点，这种纯粹底层的区块链还没见到过。

不过，家庭自组织网络中，授权用户的数量不会太多，受到攻击时，一半以上的授权用户被控制的概率比较大，因此，这种场景使用区块链完成访问控制是否有必要值得讨论。

最后，家庭自组织网络中，蓝牙、WIFI和ZigBee三种协议都有使用，限于一种协议显然是不适合的，以设备为终端节点，需要对各种无线协议做适配，工作量大且更新繁琐，不是最好的选择。
