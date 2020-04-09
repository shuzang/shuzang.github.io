# 区块链与D2D


## 1. 概念引入

论文[^mach_mobile_2017]提到，一些邻近的用户设备（User Equipment, UE）组合其计算能力，在本地服务于具有更高性能需求的应用可以叫做 ad-hoc cloud。实际应用需要解决几个关键的问题：

1. 在保证处理后的数据能够返回到源 UE 的同时，在附近找到合适的计算 UE
2. 尽管没有用于促进可靠计算的控制信道，但必须启用计算 UE 间的协调
3. 考虑到电池消耗和额外的数据传输约束，必须激励计算 UE 向其它设备提供其计算能力
4. 安全和隐私问题

[^mach_mobile_2017]: MACH P, BECVAR Z. Mobile Edge Computing: A Survey on Architecture and Computation Offloading[J]. IEEE Communications Surveys & Tutorials, 2017, 19(3): 1628–1656. DOI:[10.1109/COMST.2017.2682318](https://doi.org/10.1109/COMST.2017.2682318).

## 2. 背景调查

如第一部分所述，我们主要谈论的问题被称作 ad-hoc cloud，在论文[^yaqoob_mobile_2016]中，将该方向的研究正式命名为移动自组织云（Mobile ad hoc cloud，MAC），这篇综述对2016年及之前的该方向论文进行了总结，我们总结一些可能有用的内容。

[^yaqoob_mobile_2016]:YAQOOB I, AHMED E, GANI A, 等. Mobile Ad Hoc Cloud: A Survey: Mobile Ad Hoc Cloud[J]. Wireless Communications and Mobile Computing, 2016, 16(16): 2572–2589. DOI:[10.1002/wcm.2709](https://doi.org/10.1002/wcm.2709).

本地的计算密集型应用（如游戏和高清视频）需要大量的服务与资源，利用云提供这些资源的方式叫做移动云计算（Mobile Cloud Computing， MCC）。但是，考虑到网络连接的带宽和质量，这种方式不总是可行的，这种情况下，人们提出了 MAC，利用一组移动设备的共享资源，来实现一个共同的目标。MAC 不是用来取代原有技术的，而是一种补充，当网络质量不佳、附近基站不可用或其它情况时，使用这种方式。

以实时会议（conference）出席的例子来说明：移动用户拍摄会议参与者的快照，并将捕获的人脸和存储的照片进行匹配，从而确定谁在参加哪个会话（session）；此外，出于会议议程的规划，会议组织者希望获取不同 session 中参与者的最新情况。在该场景中，移动用户可能没有足够的资源在单个设备上执行该任务，因此选择使用 MAC 满足任务执行的实时性需求。假设每个 session 有50个参与者，其中10个愿意参与 MAC，任务被分解为如下步骤

1. 检测快照中的人脸数量，并为每个人脸裁剪一个小图像
2. 分别提交每个人脸进行识别
3. 将与会者列表返回给用户

假设第1步需要15秒，第2步需要10秒，第3步需要6秒，不使用 MAC 移动用户总共需要 15+10*50+6 = 512s，如果使用 MAC，任务2被分配给 10 个 MAC 参与者（provider），时间将大幅减少，假设与所有 provider 的通信开销是3s，那么总时间为 15+3 +10 * 5+6 = 74s。

实际的过程更加复杂，面对的问题包括：[构成 MAC 的移动设备的发现]^(MAC formation)、[任务卸载]^(task offloading)，任务调度与分配，安全与隐私，[移动性和激励]^(mobility and incentives)，资源管理。按这种分类总结相关解决方案如下图

![分类解决方案](/images/总结-区块链与D2D结合背景调查/分类解决方案.png "分类解决方案")

由于通常参与 MAC 的移动节点是随机选择的，因此安全性和隐私性是一个重要问题。在 MAC 中加入任何恶意节点都会增加总体任务执行时间，从而导致性能下降。此外，位置的隐私性也是共享设备的用户的一个重要的问题。为了解决这些问题，需要合适的认证与授权机制，保护位置隐私及防止恶意节点参与 MAC。论文[^lacuesta_spontaneous_2014]提出一种可信算法，基于AES 加密保证节点间的安全通信，同时也可以管理节点的加入和退出。论文[^hammam_trust_2013]则提出一种信任管理系统，监视参与 ad hoc 网络的节点，计算其信任值并存储，从而其它移动设备可验证其信任值，信任值越高越可靠，最终达到阻止恶意节点参与的目的。除此之外，有的方案建立和分发密钥来保护通信，有的方案利用 SaaS 或 PaaS 检测异常行为。

[^lacuesta_spontaneous_2014]:Lacuesta R, Lloret J, Sendra S, Peñalver L. Spontaneous ad hoc mobile cloud computing network. The Scientific World Journal 2014; 2014: 1–19. DOI: 10.1155/2014/232419
[^hammam_trust_2013]:Hammam A, Senbel S. A trust management system for ad-hoc mobile clouds. In Computer Engineering & Systems (ICCES), 2013 8th International Conference on, IEEE, 2013

## 3. 区块链提供信任

这一部分取自论文[^qiao2019blockchain]，是一个利用区块链提供不可信 D2D 节点间信任的例子。

[^qiao2019blockchain]:Qiao G, Leng S, Chai H, et. al. Blockchain Empowered Resource Trading in Mobile Edge Computing and Networks[C]. ICC 2019 - 2019 IEEE International Conference on Communications (ICC). Shanghai, China: IEEE, 2019: 1–6. DOI:[10.1109/ICC.2019.8761664](https://doi.org/10.1109/ICC.2019.8761664).

首先，随着智能手机和 IoT 设备的增多，移动通信已成为日常生活的一部分，传统的云计算不能满足实时性的需要，D2D-ECN(device-to-device edge computing and network)被提出，目的是共享移动设备的计算和通信资源提供一个灵活的边缘计算平台。这种情况下假设希望在一个期望的时间内执行一个计算密集型的任务，可以将计算任务卸载到有可用计算资源的移动设备上执行，数据传输延迟也可以通过购买空闲通信资源减少

![本地的资源交换和任务分配](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8753818/8761046/8761664/qiao1-p6-qiao-small.gif "资源交换和任务分配")

区块链被引入用来解决分布式和点对点网络中不可信用户间的安全和可靠性问题。由于现有研究大部分使用 PoW 共识，在资源有限的 IoT 环境中不具备可行性，作者选择了一种基于信誉的共识（reputation-based consensus）[^gai2018proof]来协调设备和验证交易。整个系统的安全性靠该共识和一个信誉评估系统保证。

[^gai2018proof]:F. Gai et al., "Proof of Reputation: A Reputation-based Consensus Protocol for Peer-to-Peer Networks", *Proc. 2018 Int. Con. Dat. Sys. Adv. App. DASFAA*, Feb. 2018. ↩

作者认为现有研究没有考虑计算卸载规则对基于区块链的边缘网络的性能的影响，整个计算卸载过程可以划分为资源交易和任务分配两部分，资源交易的目的是提供高效的资源定价和分配策略，从而平衡任务所有者的[效用]^(utility)和资源所有者的收益，任务分配则是在保证较低的执行时间和系统开销。论文实际做的事有两件

1. 提出并使用智能合约实现了一个分布式的资源交易和任务分配方案，平衡资源定价与收益，最小化执行时间和系统开销；
2. 提出一个信誉评估系统并和共识配合使用保证计算卸载环境的可靠性

论文引入资源币激励用户参与，一个计算卸载过程的工作流如下图所示。设备首先在LBS中注册并被授予身份，通过{身份ID，证书，公钥，私钥，钱包地址}五元组追踪在区块链中的交易记录（资源交易和信誉度更新）；然后，任务所有者发送资源需求到LBS，资源所有者发送可用资源数量和使用限制到LBS，LBS验证证书后将这些内容写入智能合约，并在达到条件后自动触发，资源的定价和交易被设定了规则；之后，根据可用的通信和计算资源，智能合约执行任务分配算法最小化总延迟，并在准确返回计算结果后对资源提供者的信誉进行评分；这些工作完成后，任务提供者按照资源价格支付资源币；最后，任务所有者签名后广播资源交易记录和更新后的信誉度，运行 基于信誉的共识，由信誉度最高的节点将这些记录收集到区块并附加到区块链。实际过程需要经过多轮验证

![基于区块链的资源交易和任务分配系统组成和工作流](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8753818/8761046/8761664/qiao2-p6-qiao-small.gif "工作流")

这里有3个关注的地方

1. 作者使用公式推导得到了任务分配者效用和资源所有者收益的平衡函数，在**资源交易智能合约**中执行该函数得到合适的定价；

2. 作者使用公式推导最小化了任务执行的总延迟，并在**任务分配智能合约**中执行；

3. 对资源提供者的信誉度评估在得到正确的计算结果后执行，作者也提供了公式推导

整个系统的安全性通过三个方面保证：共识和信誉评估系统保证计算卸载环境可靠；智能合约保证交易记录和信誉度不被篡改；LBS保证通信安全。
