# 研究记录5-区块链与异常检测方向探索


对区块链进行异常检测（Anomaly detection）也是一个重要的方向，不过有的论文中也称作侵入检测（Intrusion detection），尤其是协作入侵检测（Collaborative Intrusion Detection, CID）的概念比较流行。我们虽然在搜索论文时对两种都进行了统计，但在下文说明中统一称呼为「异常检测」。

通过阅读论文，可以大致将该领域的研究分为两个方向，一个是对区块链系统进行异常检测，另一个是利用区块链系统解决异常检测领域的问题，多数为利用区块链存储异常检测的数据。

## 1. 论文查找

以下的论文是从SCI，EI，知网和arvix四个数据库检索得到的，检索的关键词有两种组合：

- `blockchain`和`anomaly`
- `blockchain`和`intrusion`

所有的论文根据数据库进行分类，但排序不分先后。关于CID，很多论文在后面添加`系统`或`网络`一词，因此全称为`CIDS/CIDN`。几篇协同入侵检测但与区块链无关的论文单列在`其它`分类中。

在每个数据库的搜索结果中，我们又将论文分为三类

1. 综述
2. 对区块链系统进行异常检测
3. 利用区块链系统解决异常检测领域的问题

下面是一些额外的说明

- 标号规则举例：SCI-1代表SCI检索结果列表的第一篇文章，同样，EI-2代表EI检索结果第二篇文章。
- 关联性说明：括号中是和研究目标的关联性级别，关联性越强，阅读优先级越高
- 分类是通过阅读各论文摘要完成的，可能出现错误，在具体阅读论文完毕后，若发现分类错误，将进行修改，添加删除线的为已确认分类的论文
- EI-11和EI-16两篇，SCI-8和SCI-9两篇都极为相似，可不必重复阅读

| 类别                             | 论文列表                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| 综述                             | -  SCI-4：比特币系统存在的安全威胁及解决方案（弱）<br>-  ~~SCI-11：区块链和入侵检测交叉综述（强）~~<br>-  EI-13：从四个维度分析区块链，其中有一个是入侵检测（中）<br>-  EI-17：详细介绍了区块链和入侵检测的情况（强） |
| 对区块链进行异常检测             | - ~~SCI-3(强)~~；~~SCI-12(强)~~；SCI-10(强)<br>- EI-1(强)；EI-2；EI-3(强)；EI-5(中)；EI-9(强)；EI-12(中)；EI-18(强)<br>- ~~arvix-1(强)~~；arvix-2；arvix-3 |
| 利用区块链解决异常检测领域的问题 | - SCI-5(中)；SCI-7<br>- EI-9；EI-10；EI-11；EI-12；EI-14；EI-15；EI-16<br>- 区块链存储异常检测的数据：  SCI-1(中)；SCI-6(弱)；SCI-8(中)；SCI-9；  EI-4(中)；EI-6(弱)；EI-7(弱)<br>- 智能合约定义异常检测算法：~~SCI-2(强)~~ |

### 1.1 论文搜索结果

#### SCI

1. Chained Anomaly Detection Models for Federated Learning: An Intrusion Detection Case Study

    攻击者越来越擅长将恶意行为数据隐藏到正常行为数据中，异常检测的时间逐渐增加，为了解决这一问题，采用联邦学习来监测数据。这种方法主要的难题在于联邦学习的过程中攻击者容易添加恶意训练样本影响本地的机器学习模型，从而规避检测，因此，作者使用区块链来存储联邦学习模型的增量数据，从而可审计模型数据的正确性。

2. ~~Collaborative Anomaly Detection on Blockchain from Noisy Sensor Data~~

    提出了一种框架用于区块链的协作异常检测，以condition-based的工业资产管理为例，从传感器数据中检测工业资产的异常。将区块链看作一个协作学习的平台而不仅仅是一个可追踪、不可变和分布式的数据管理系统，主要是把机器学习的异常检测算法定义在智能合约中，通过区块链的共识运行完成完整的异常检测过程。

3. ~~Securing Majority-Attack in Blockchain Using Machine Learning and Algorithmic Game Theory: A Proof of Work~~

    多数人攻击(majority-attack)对公链来说不是威胁，但对于私链/联盟链来讲，可能会有部分节点串通作恶。这篇文章的作者提出使用智能软件代理来监控区块链网络中利益相关者的活动，从容检测节点串通作恶。主要使用的是监督机器学习算法和算法博弈理论来阻止多数人作恶的发生。

5. Bitcoin Concepts, Threats, and Machine-Learning Security Solutions

    介绍比特币系统及其主要技术（包括区块链协议）的现有威胁和弱点，总结现有的安全研究和解决方案，提出面临的挑战和未来的研究趋势。

6. Transaction Immutability and Reputation Traceability: Blockchain as a Platform for Access-controlled IoT and Human Interactivity

    解决通信双方的信任问题，其中一种方法是区块链异常检测与评估

7. Seeing is understanding - anomaly detection in blockchains with visualized features

    提出针对可交互性优化的用于异常检测的在线机器学习方法，将系统特性以图形方式表达，适用于任何时间序列的数据，数据存储在以太坊区块链中保证不可篡改。

9. A Collaborative Intrusion Detection Approach Using Blockchain for Multimicrogrid Systems

    实例场景：Multimicrogrid(mmg) 系统，设计了一种使用区块链的协作入侵检测（CID）用于保护该系统安全，不需要可信权威和中心服务器，同时以协作方式提高了检测准确性。它结合了周期性和触发性模式来生成CID目标，即提案生成。根据生成的提案和mmg模型，利用共识实现CID。最终的结果存在区块链上，使用激励机制促进单个微电网参与共识。最后通过一个mmg实例分析了方法有效性。

3. Designing collaborative blockchained signature-based intrusion detection in IoT environments

    应对协作入侵检测签名伪造的情况，通过区块链增量更新可信签名数据库，增强基于签名的CID的有效性。

5. CBSigIDS: Towards Collaborative Blockchained Signature-based Intrusion Detection

    和第10篇相似

6. Intrusion Detection and Mitigation System Using Blockchain Analysis for Bitcoin Exchange

    比特币使用传统入侵检测系统具有高风险。该文描述了比特币交易中的三种入侵模型，并针对每种入侵模型都提出了一种基于区块链分析的检测和缓解系统。不过这篇文章中提到的三种入侵模型有两种都是关于交易所的，第三种讨论的还是15%攻击，对加密货币型的区块链可能很有用，但对其他类型的就不怎么样了。

7. ~~When Intrusion Detection Meets Blockchain Technology：A Review~~

    区块链和侵入检测交叉综述，蛮有意思，竟然已经有人写这个了

14. ~~Anomaly Detection in Bitcoin Network Using Unsupervised Learning Methods~~

    构建了以用户为节点和以交易为节点的两个图，进行特征量提取，并使用K-means、马氏距离和无监督支持向量机三种异常检测算法，检测比特币网络中的可疑交易和可疑用户。

#### EI

1. Advise: Anomaly Detection tool for blockchain systems

    提出用于区块链系统的异常检测工具Advise，利用区块链分叉数据在网络中手机恶意请求，同时抵御eclipse攻击，该系统收集并分析恶意分叉，以构建一个威胁数据库，从而检测和预防未来的攻击。arvix的BAD: a blockchain anomaly detection solution和这个是同一篇，该文是其正式发表的名称。

2. Anomaly detection model over blockchain electronic transactions

    提出一种比特币交易异常检测模型，用支持向量机检测异常值，用k-means分组相似的异常值，通过生成检验结果评估该方案，在精度上得到了高性能的结果。

3. Blockchain-based auditing of transparent log servers

    为了保证用于密钥管理的公钥服务器的可靠性，使用了防篡改的数据结构，并在客户端之间使用了gossip协议（区块链）,但缺乏对恶意客户端的检测。该文使用以太坊区块链进行信任度审计，并提出一种轻量级异常检测算法来保护客户端。(还没具体看论文，不知道说的轻量级异常检测算法是什么)

4. Federated Blockchain-based Tracking and Liability Attribution Framework for Employees and Cyber-physical Objects in a Smart Workplace

    一个工业实例，但主要是用区块链保存异常检测数据，利用其不可篡改性提供有效监管。

5. Securing instant messaging based on blockchain with machine learning

    以网购的即时消息通信安全为例，提出一种基于国密的区块链即时通信方案。首先设计了一个消息认证模型用于避免伪造攻击和重放攻击，其次设计了一个加密哈希模式来保护消息完整性，第三，设计了一个消息加密模型来保护用户隐私。最后，提出一种基于机器学习的方法进行区块链异常检测。

6. Detecting Robotic Anomalies using RobotChain

    实例场景：使用区块链保存工厂机器人运行日志，确保这些数据不被篡改，并结合异常检测模块检测这些数据。

9. Services as Enterprise Smart Contracts in the Digital Factory

    以工厂设备预测性维护为例，使用区块链记录设备异常信息。

10. A Model for Detecting Cryptocurrency Transactions with Discernible Purpose

    通过检测加密货币钱包的元数据，识别可疑交易。采用无监督学习期望最大化算法进行数据的聚类，基于无监督学习得到的特征，使用Random Forest(RF)进行异常检测。

11. Towards blockchain-based collaborative intrusion detection systems

    利用区块链技术改进CID，提高各监视器之间的信任，并提供问责制和共识。提出了一个将区块链纳入cids领域的通用架构，并分析了实现这种架构所需做出的设计决策。

2. On Blockchain Architectures for Trust-Based Collaborative Intrusion Detection

    考虑使用区块链技术解决CID系统的信任管理问题，提出使用区块链保护CID对等节点间共享信息的完整性，增强其问责机制，通过阻止内部攻击来确保协作，称为信任链。提出了一种结合了PoS和PoW的共识，使协作的CID节点能维持该链。

3. Enhancing challenge-based collaborative intrusion detection networks against insider attacks using blockchain

    计算机网络中入侵变得越来越复杂，协同入侵检测被用于这种情况，但内部攻击是这种抵御手段的主要威胁，基于挑战的信任管理可以帮助抵御内部攻击，但高级内部攻击仍然具有较高威胁，使用区块链可以增强基于挑战的入侵检测系统抵御高级内部攻击的鲁棒性。（和EI-18非常相似）

5. DeepCoin: A Novel Deep Learning and Blockchain-Based Energy Exchange Framework for Smart Grids

    构建了一个基于区块链的智能电网能量框架，并使用递归神经网络来检测该框架中的网络攻击和欺诈交易，实质是一种基于深度学习的入侵检测系统，研究了该入侵检测系统在三种数据集上的性能

6. Review paper on untwist blockchain: a data handling process of blockchain systems

    从四个维度分析区块链，其中包括入侵检测。并提出Blockbench，一个用来理解公链和私链性能的标准框架。

8. Probabilistic Blockchains: A Blockchain Paradigm for Collaborative Decision-Making

    建立了一个概率区块链的新模式，允许建立高效和分布式的风险评估和决策，可用于入侵检测。并且介绍并分析了概率区块链在计算机网络入侵检测系统中的应用。

9. Enhancing Medical Smartphone Networks via Blockchain-Based Trust Management Against Insider Attacks

    针对医疗物联网场景，由于其分布式的特性，内部攻击具有较大的安全威胁，使用区块链技术可以增强信任管理能力。文章主要针对一个医疗智能手机网络，将区块链用于增强基于贝叶斯推理的信任管理方案检测恶意节点的有效性，但摘要里没谈到具体如何使用区块链增强信任管理，是不是用来存储恶意检测数据。

10. Towards Blockchained Challenge-Based Collaborative Intrusion Detection

    协同入侵检测系统/网络（CIDS/CIDN）被广泛地用于保护分布式网络资源，检测可能的威胁。但检测系统/网络容易受到内部攻击，因此需要集成信任机制。基于挑战的信任机制是有用的，但仍然容易受到高级内部攻击影响，使用区块链可以在多方面增强基于挑战的信任机制的鲁棒性，比如增强内部节点的检测和识别不真实的输入等。（和EI-13非常相似）

11. Information security model of block chain based on intrusion sensing in the IoT environment

    详解介绍区块链和入侵检测的情况，并把入侵检测用于区块链信息安全模型，结果说明了具有较高检测效率和容错性。

12. DOORchain: Deep Ontology-Based Operation Research to Detect Malicious Smart Contracts

    区块链系统的一个重要问题是确保交易是安全的而不是恶意的，这篇论文提出结合深度学习、ontology和运筹学三种方式用于区块链中的入侵检测，准确度得到了提高。

#### 知网

1. 基于人工免疫的比特币快捷交易异常检测模型

    主要应对比特币中的双花攻击。在每个传统比特币节点中加入免疫检测模块进行抗原提取、并利用检测器进行异常检测,在威胁控制中心动态演化检测器并分发免疫疫苗以便有效的进行防御。

2. 基于免疫的区块链eclipse攻击的异常检测

    应对eclipse攻击，提出一种基于免疫的区块链eclipse攻击的新型检测模型,建立了该模型的架构,给出了模型中各要素的形式定义及各模块的执行流程。

#### arvix

1. ~~CIoTA: Collaborative IoT Anomaly Detection via Blockchain~~

    利用区块链对资源有限的物联网设备做协作异常检测，检测模式是一种扩展的马尔可夫模型，各节点在本地运行一个模型，然后通过运行共识维持一个结合模型。该方案具有高可扩展性和对攻击的弹性，无需人工干涉，是一个通用模型，作者还用一个48个树莓派的集群以智能摄像头和智能灯泡为例做了实验。

2. Anomaly Detection in the Bitcoin System - A Network Perspective

3. AI-enabled Blockchain: An Outlier-aware Consensus Protocol for Blockchain-based IoT Networks

#### 其它

1. trust management for host-based collaborative intrusion detection
2. a decentralized Bayesian attack detection algorithm for network security
3. Bayesian decision aggregation in collaborative intrusion detection network
4. robust and scalable trust management for collaborative intrusion detection
5. a trust-aware, P2P-based overlay for intrusion detection

## 2. 提出问题

在论文的阅读过程中，主要考虑以下几个问题：

- 区块链系统中存在哪些类型的异常
  - 多数人攻击：私链或联盟链有应对此种攻击的必要，通过回报函数判断是否有攻击倾向
  - 金融网络中的盗窃或非法交易，通过无监督学习检测可疑用户和交易
  - 异常不只是链上的交易数据，还有链下设备的行为，设备可能本身就一直提交错误数据
- 如何检测可能的异常
- 强化学习是否可应用到这种场景
- 额外的组件添加到区块链系统是否影响可信度

## 3. 后续进展

结合之前阅读的论文，当前检测的异常有：

1. 恶意用户及与恶意用户有关的盗窃资金的恶意交易
2. 多数人攻击中有倾向勾结获取有利于自身利益的节点
3. 具体的恶意行为定义，如设备请求的资源量、设备请求的频率、设备通信目标或通信手段的改变等。

当前检测的手段有：

1. 机器学习：K-means聚类，马氏距离（Mahalanobis distance），无监督支持向量机
2. 扩展的马尔可夫模型

当前关于异常检测的思路三个

1. Anomaly Detection in Bitcoin Network Using Unsupervised Learning Methods一文以区块链账户为节点，以账户交易为边，构建了用户图模型，从中提取特征向量，以此向量为基础，通过K-means聚类，马氏距离，无监督支持向量机三种方法检测异常。但这篇论文针对的重点是货币型区块链，从构建的用户图提取的特征向量都是与货币有关的，检测的是对加密货币的盗窃行为。在我们的实验中，如果以区块链账户为节点，由于每个账户代表的是IoT设备或其它实体，发起的交易一般与访问控制相关，因此用户图提取的特征向量将是访问控制场景下的恶意用户和恶意访问控制，由此进行异常检测。这一思路的检测是非实时的。
2. CIoTA Collaborative IoT Anomaly Detection via Blockchain中使用扩展的马尔可夫模型做设备节点的异常检测，但矩阵的值是设备状态转变的概率。但如果矩阵中的值不填充概率，而是根据奖励函数填充设备信用的变化，也可以最终得出异常。而且，该论文在本地定义检测模型，又通过共识运行维持一个结合的检测模型，是通过对共识机制的调整进行实现，不具备适用性。如果将检测模型定义在智能合约中，很多设计可以得到简化。这一思路的检测是一定程度的实时检测。
3. 预定义各种行为和节点信用，每种行为都会引起信用变化，信用的具体化可以使用合约构建内部货币系统或其它方式，信用降到0拒绝节点的任何行为。异常检测算法则可以使用合约实现或定义在共识过程中。

之后要做的事（按优先级排序）

1. 总结异常检测的参数（检测量）
   - 整个体系哪些地方可能出现异常
   - 分层讨论，共识层、网关层和设备层每一层可能的异常行为
2. 寻找如何进行异常模拟或生成
3. 学习异常检测算法

### 3.1 异常智能合约

两类异常检测系统：detection of anomalies on BC and BC helps anomaly detection.

当前考虑的系统是BC based access control in IIoT. 常规意义的异常检测有大量可用的算法，但需要找到有效的监测参数。
其实区块链的验证机制有抑制异常智能合约的功能。以我们的系统为例，我有以下几个问题：

1. 对矿工的奖励机制是什么？
   IBFT中没有矿工，只有验证者，没有奖励机制。若确实需要一套奖励机制，可以利用智能合约实现一套货币系统。
2. 有什么样的典型异常智能合约？
   智能合约的内容在部署前所有参与者应达成共识，部署后所有参与者可以验证合约未经修改，智能合约一旦部署无法修改。关于智能合约可能出现的异常包括：
   - 智能合约源代码在编写的时候出现的错误和漏洞，在部署后被人发现并利用。这种漏洞因合约内容而异，主要依赖于在合约部署前进行安全性检查
   - 对合约的调用问题，比如节点短时间发起过多请求
   - 拒绝执行合约通过的访问控制请求，或者返回假的执行结果
   - 注册不存在的设备或恶意设备，并关联相关的合约
   - 私自对部署的合约代码进行修改，从而避过合约内置的安全检查或谋取利益。
3. 除了阻止异常智能合约生成，能否发现并锁定企图制造异常智能合约的用户？
   网关部署的访问控制合约需要在注册合约中进行注册，在注册时进行安全性检查，比对合约ABI是否被篡改。
4. 如果问题3的回答是可行，谁负责组织实施发现和锁定操作？
   注册合约，通过对ABI进行检查确认合约内容未经篡改。不过实际上ABI由网关提交，网关有可能部署恶意合约但提交正常合约的ABI。


---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2019/blockchain-and-anomaly-detection/  

