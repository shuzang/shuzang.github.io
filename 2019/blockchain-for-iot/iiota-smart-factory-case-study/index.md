# 文献综述-区块链用于 IoT/IIoT：智能工厂案例


随着物联网的高速发展，越来越多的智能设备接入到了人们的日常生活和工业生产当中。尤其在智能工厂领域，物联网设备的使用大幅提高了工厂自动化程度，并提供了更高的容错能力。然而，现有的集中式云存储与管理方式面临很大的安全隐患和性能瓶颈。学术与工业界目前致力于将区块链技术应用于车联网、能量网络和供应链方向，而忽视了智能工厂与区块链的结合。本文从区块链和物联网的可结合性出发，具体分析了智能工厂场景区块链技术的适用性。对现有的共识算法进行分类，总结分析在智能工厂场景中可做的改进。研究工厂实时数据的存储解决方案。列举了可能面临的安全问题和攻击方式，并提出使用智能合约来完善安全机制。从而为之后的实验研究奠定了基础。

## 1. 引言

物联网是互联网概念的扩展，通过将具有独立功能的实体互联互通，可以为我们的生产生活带来巨大便利。自从物联网的概念提出以来，迅速应用到了智能家居、农业、医疗和工业等诸多领域，物联网在工业中的应用称为工业物联网。工业物联网是工业4.0的重要一部分，然而，随着终端设备数量的迅速增长，传统的基于云的中心化架构面临的部署和维护成本，向第三方支付的服务费用，以及性能、安全和隐私问题都迫切需要解决。同时，由于设备和生产过程的深度融合，恶意攻击所带来的危害也远远超出普通的互联网，甚至可能对人们的生命财产造成威胁。

区块链是解决中心化带来的单点瓶颈和安全问题的有效手段。这项技术起源于2008年中本聪提出的比特币[^nakamoto2008bitcoin]，它分布式的结构能有效避免单点问题，共识和链式结构带来的不可篡改性和通过智能合约实现的访问控制又能为物联网带来良好的安全与隐私保护。另外，智能合约还能便于设备进行一定的自动化交互。目前，许多人正在研究将其应用于物联网[^christidis2016blockchain][^ozyilma2017work][^dorri2017blockchain]。然而，要实现这一目的还需解决一些问题：

[^nakamoto2008bitcoin]:Nakamoto S. Bitcoin: A peer-to-peer electronic cash system[J]. 2008.
[^christidis2016blockchain]:Christidis K, Devetsikiotis M. Blockchains and smart contracts for the internet of things[J]. Ieee Access, 2016, 4: 2292-2303.
[^ozyilma2017work]:Özyılma K R, Yurdakul A. Work-in-Progress: Integrating low-power IoT devices to a blockchain-based infrastructure[C]. Proceeding of the Thirteenth ACM International Conference on Embedded Software. 2017.
[^dorri2017blockchain]:Dorri A, Kanhere S S, Jurdak R, et al. Blockchain for IoT security and privacy: The case study of a smart home[C]. 2017 IEEE international conference on pervasive computing and communications workshops (PerCom workshops). IEEE, 2017: 618-623.

1. 效率和安全：区块链的核心是共识算法，工作量证明(Proof of Work, PoW)则是使用最广泛的一种，它需要节点具有较高的算力，而物联网设备的相关性能往往是极为有限的。虽然消除对算力的需求可使区块链适用于物联网，但同时也失去了它带来的安全性。

2. 隐私：区块链虽然具有匿名特性，但采用如链接攻击等方式可以将行为与用户地址关联起来，从而破解用户真实身份，由于存储在区块链中的数据是公开可见的，一旦身份被获知，数据将完全泄露；物联网涉及大量的隐私数据，需要更加良好的隐私保护方案。

3. 吞吐量和数据存储：区块链共识采用洪泛的通信方式，因此具有一定的延迟，从而导致吞吐量较低。另外，网络中每个节点都要保存整个区块链的副本，由于区块链不断增长的链式结构，并不适合大量数据的存储。然而，物联网通常会产生大量实时数据，具有较高的存储和吞吐量需求。

目前很多研究者都致力于解决以上问题，但在工业领域，大量研究集中在车联网[^dorri2017distributed][^huang2018security]、能量网络[^li2018consortium][^liang2018distributed]和供应链[^korpela2017digital][^tian2016agrifood]方面，而较少分析和关注区块链在制造工厂内的适用性和解决方案。事实上，智能工厂作为工业4.0重要的一部分，区块链不仅在原材料和产品分销，个性化产品定制等方面可提供一定帮助，在提高工厂生产效率和数据安全方面，也是一种良好的解决方案。首先，区块链配合分布式存储系统消除对中心化的云服务商的依赖，在性能、安全和隐私方面得到改进；其次，借助智能合约，可以完成对工厂大量物联网终端设备的访问控制；并且实现对生产的动态调整，完成一定的自动化与智能化。

[^dorri2017distributed]:Dorri A, Steger M, Kanhere S S, et al. Blockchain: A distributed solution to automotive security and privacy[J]. IEEE Communications Magazine, 2017, 55(12): 119-125.
[^huang2018security]:Huang X, Xu C, Wang P, et al. LNSC: A security model for electric vehicle and charging pile management based on blockchain ecosystem[J]. IEEE Access, 2018, 6: 13565-13574.
[^li2018consortium]:Li Z, Kang J, Yu R, et al. Consortium blockchain for secure energy trading in industrial internet of things[J]. IEEE transactions on industrial informatics, 2018, 14(8): 3690-3700.
[^liang2018distributed]:Liang G, Weller S R, Luo F, et al. Distributed blockchain-based data protection framework for modern power systems against cyber attacks[J]. IEEE Transactions on Smart Grid, 2018.
[^korpela2017digital]:Korpela K, Hallikas J, Dahlberg T. Digital supply chain transformation toward blockchain integration[C]//proceedings of the 50th Hawaii international conference on system sciences. 2017.
[^korpela2017digital]:Tian F. An agri-food supply chain traceability system for China based on RFID & blockchain technology[C]//2016 13th international conference on service systems and service management (ICSSSM). IEEE, 2016: 1-6.

区块链和制造工厂结合的相关研究[^lin2018bsein][^kapitonov2018robonomics]，和智能家居场景[^xue2018private][^qu2018hypergraph][^xu2018building]具有一定的相似性。这两个场景的研究通常集中于细粒度的访问控制和安全框架，然而，大部分并不深入，只是区块链和分布式存储技术的简单结合。另外，工业物联网同普通的物联网在诸多方面有着区别，如设备的自动化，低时延高可靠性的要求。但是，两者都更适合使用私有区块链而非公有区块链。

[^lin2018bsein]:Lin C, He D, Huang X, et al. BSeIn: A blockchain-based secure mutual authentication with fine-grained access control system for industry 4.0[J]. Journal of Network and Computer Applications, 2018, 116: 42-52.
[^kapitonov2018robonomics]:Kapitonov A, Berman I, Bulatov V, et al. Robonomics Based on Blockchain as a Principle of Creating Smart Factories[C]//2018 Fifth International Conference on Internet of Things: Systems, Management and Security. IEEE, 2018: 78-85.
[^xue2018private]:Xue J, Xu C, Zhang Y. Private Blockchain-Based Secure Access Control for Smart Home Systems[J]. KSII Transactions on Internet & Information Systems, 2018, 12(12).
[^qu2018hypergraph]:Qu C, Tao M, Yuan R. A hypergraph-based blockchain model and application in Internet of Things-enabled smart homes[J]. Sensors, 2018, 18(9): 2784.
[^xu2018building]:Xu Q, He Z, Li Z, et al. Building an Ethereum-Based Decentralized Smart Home System[C]//2018 IEEE 24th International Conference on Parallel and Distributed Systems (ICPADS). IEEE, 2018: 1004-1009.

本文旨在总结物联网和区块链结合的众多已有方案，分析它们的优势与不足，并在制造工厂场景下，分析各方面可做的改进，存储方案的选择和访问控制方案的设计。从而提高工厂的生产效率，降低其成本，为下一步的概念验证和实际的大规模实施做理论准备。

本文其余部分安排如下。第二部分介绍相关共识方案并分析工厂场景下共识可做的优化。第三部分介绍待选的存储方案，并比较它们在工厂场景的适用性。第四部分详述可能可能面临的攻击和访问控制的方案，同时包括对隐私问题的讨论。第五部分讨论其它的补充机制。总结在最后的第六部分。

## 2. 共识

区块链技术的核心是共识，广义上讲，它是一组参与各方能达成一致所需遵守的规则。在区块链中，共识确保了节点和交易的可信，并解决了双花问题。迄今为止，已提出了种类繁多的共识协议，它们达成一致的速度有快有慢，有的同时适用于公有和私有区块链，有的则只适用于私有区块链 ,有些是通用类型的共识，有些只针对特定的需求和场景。以下我们先介绍各种不同类型的共识，然后再根据工厂场景进行挑选，并提出可做的改进。

### 2.1 共识分类

自区块链第一次出现[1]以来，使用最广泛的共识机制一直是工作量证明，代表性的应用有比特币、以太坊和莱特币等。PoW中，为了添加新块，节点必须证明自己做了一定的工作，这一工作一般是大量的哈希运算，无论是计算依赖（Bitcoin PoW）还是存储依赖（Ethash），都需要消耗电力资源，带来资源浪费和高成本，但却可以在无信任网络中实施。PoW共识中的交易速度较慢，另外，还需要激励机制来鼓励矿工参与维护网络。

与PoW不同，权益证明（Proof-of-Stake, PoS）的核心是在网络中拥有更多权益的人具有更小的攻击可能性，这个权益指节点持有的代币数量及持有代币的时间。PoS消除了对算力的需求，但可能出现最富者支配记账权的情况，同时，还存在“无厉害关系(nothing at stake)”问题，即在出现分叉的情况下，权益持有者有动机在分叉形成的两条链上都下赌注，更有可能出现双花问题。PoS的代表性应用主要是Cardano、NXT、Tezos和未来的以太坊。

委托权益证明(Delegate Proof of Stake, DPoS)会选举一定数量的节点进行交易验证和区块添加，投票的权重取决于节点的权益大小，并且，投票选举的验证者出现“作恶”情况也可以通过重新投票随时进行替换。DPoS能以更高的速度形成区块，并且在单位时间处理大量的交易，共识运行的效率更高。Daniel Larimer在2014年设计了它，并在BitShares中首次使用，后来又在Steemit和EOS项目中使用该共识，其它如TRON也使用DPoS方案。以上三种共识是公有区块链中最常使用的共识，但它们也适用于私有区块链。

在私有区块链中，由于部分节点可以相互信任，参与验证的节点数减少，从而交易和区块生成的速度可以加快。拜占庭容错(Byzantine Fault Tolerance, BFT)正是针对这种场景提出的共识。BFT可以容忍小于1/3个恶意或无效节点的存在。事实上，BFT是一个大的分类，许多共识都可以划归到BFT的范围内，主要包括授权拜占庭容错(Delegated Byzantine Fault Tolerance, DBFT)，实用拜占庭容错(Practical Byzantine Fault Tolerance, PBFT)，联邦拜占庭协议(Federated Byzantine Agreement, FBA)和其它的一些共识协议。我们将它们适用的区块链类型和主要应用案例总结如表3.1所示。

表2.1 BFT类共识

| 公有/私有区块链                        | 私有区块链                                                   |
| -------------------------------------- | ------------------------------------------------------------ |
| DBFT：NEO, TON<br>FBA：Stellar, Ripple | PBFT：Hyperledger, Chain  <br>SIEVE：Hyperledger beta  <br>Round Robin：Multichain, Tendermint  <br>Loopchain Fault  Tolerance(LFT)：ICON  Cross-Fault  <br>Tolerance(XFT)：Hyperledger  beta |

除以上几种主流的共识外，还有众多用于特定任务或特定场景的共识如Proof-of-Activiy(PoA)，Proof-of-Elapsed-Time(PoET)，Proof-of-Authority(PoA)和Proof-of-Burn等。以上共识建立的区块链都是链式结构，实际上，为了改善吞吐量等问题，人们还提出了图状结构的“区块链”，如IOTA的DAG区块链和HashGraph。而无论是链式还是图状的区块链共识，都消除或部分消除了PoW对算力的依赖问题，并且在吞吐量、扩展性等方面进行了相关的改进，使其适用于特定的或更广泛的领域。同时，由于技术的飞速发展，提出时间较晚的共识往往具有更多更好的特性。

### 2.2 共识需求及改进

目前已存在不少专门针对物联网开发的区块链平台，市场占有率较高的几位分别是：使用DAG结构的IOTA；使用PoS协议的量子链QTUM；使用DPoS的EOS；使用PoW和PoS混合的沃尔顿链Waltonchain以及使用权威证明(Proof of Authority, PoA)的VeChain。这些平台的市场占有情况说明了它们所使用的共识都是一定程度上适用于物联网的，基于这些协议的特性，以及制造工厂的相关背景，我们总结单个制造工厂中的共识需要的改进：

**资源限制**：尽可能减少对资源的依赖是首要需求，这方面可做的改进主要包括1）共识本身。消除PoW共识对算力的严重依赖，实际上，PoW之后的共识普遍都达成了这一点。但除这种办法外，有一些研究者仍然在PoW上做努力[^xiong2018when]，他们引入了“银行“的思想，使资源有限的设备可以向附近算力较强的设备“借”算力，从而完成挖矿操作。这种方案虽然消除了物联网设备的资源限制问题，但没有解决电力浪费问题，在工业领域，这种电力浪费带来的成本是不想要的。2）加密算法。为了安全性进行签名使用的加密算法也需要进行哈希运算，从而带来对算力的一定需求，所以选择加密算法时应在安全性的基础上选择算力需求较低的算法。[^fernandez2018review]中，研究者认为RSA和ECC（椭圆曲线加密）都具有一定的不足，而AES(Advanced Encryption Standard)是更好的选择，建议使用Simon、Scrypt或X11这样的新算法，但是对IoT设备使用的哈希算法还应做进一步的分析。3）P2P通信，P2P通信是共识过程不可缺少的一部分，但需要对终端设备持续供电，这一机制在一定程度上是与物联网的节能策略相背离的，对资源造成了一定的浪费，故需要对通信机制进行一定的优化。可以通过广播精简的数据结构而不是整个区块来减少广播时的通信冗余。

[^xiong2018when]:Xiong Z, Zhang Y, Niyato D, et al. When mobile blockchain meets edge computing[J]. IEEE Communications Magazine, 2018, 56(8): 33-39.
[^fernandez2018review]:Fernández-Caramés T M, Fraga-Lamas P. A Review on the Use of Blockchain for the Internet of Things[J]. IEEE Access, 2018, 6: 32979-33001.

**可扩展性**：可扩展性主要包括两方面，1）物联网终端设备的巨大数目及其不断增长的趋势，要求共识必须具有良好的可扩展性，能够支持设备动态的加入退出。[^novo2018blockchain]中通过分离物联网网络和区块链，只将访问控制相关的交易记载在链中的方式，提供了较高的可扩展性；2）小型的交易可能会增加与通信相关的能耗，而载荷过大的交易物联网设备将可能无法处理，故交易和区块的大小需要能根据物联网网络的带宽限制进行调整。鉴于此，在区块链中普遍应用的全节点和轻节点方案基础上，Slock.it又提出了两种节点类型，从而填补了该方面的空缺。

[^novo2018blockchain]:Novo O. Blockchain meets IoT: An architecture for scalable access management in IoT[J]. IEEE Internet of Things Journal, 2018, 5(2): 1184-1195.

**吞吐量（TPS）**：TPS即每秒处理的交易数，是区块链的吞吐量单位，物联网由于其巨大的设备数目和复杂的运行环境，需要较高的TPS，比特币中TPS值为7，以太坊为15，都难以满足要求，采用BFT共识的区块链这一参数值普遍较高，可以达到10000。区块链平台EOS据称可以达到百万级。而采用DAG的IOTA则是实时的。实际上，大多数的平台都在致力于吞吐量的提高，比较著名的例子是以太坊的分片方案，比特币的扩容方案，以及闪电网络、雷电网络等链下解决方案。

吞吐量的问题一部分是共识过程的复杂性带来的，主要是通信及交易确认的延迟，所以使用速度较快的哈希算法如scrypt而不是SHA-256能带来一定的优化，另一部分原因是任务的单进程处理，分片方案的实质就是提高并发程度。

**激励机制**：比特币和以太坊等公链场景下，由于要鼓励矿工进行挖矿，不断地添加新块到区块链中，需要激励机制的存在。在车联网、能量网络等工业领域，激励机制也有某种程度的适用性，然而在单个工厂场景中，运行节点的成本由工厂负责，即使没有激励机制，也能得到很好的运行。在这种情况下，可以考虑取消激励机制的存在，或者将基于货币的激励机制更改为基于信誉积分的激励机制。

从以上几方面进行优化的共识一定程度能更适应单个工厂场景，然而，如TCP/IP协议的出现一般，并不是最完善的技术最适合，以太坊、EOS、IOTA等平台的方案已经对物联网具有极好的适应性，首要和初步的考虑仍然是使用它们来进行方案的设计与实验，只有进行到方案优化阶段，我们才考虑从上述提到的诸多方面做改进。

## 3. 存储

区块链本身面临着存储问题。随着时间的推移，新的区块不断的添加到区块链的末尾，这使得区块链的体积不断增长，带来的问题是参与共识的矿工初始需要同步更多的内容，致使节点参与挖矿的门槛不断提高。目前采用的轻节点是一种解决方案，但这种情况下区块链无法缺少一定量的全节点，从而会导致某种程度的中心化，另一种选择是使用mini-blockchain[^bruce2014mini][^franca2015homomorphic]，这种方式只有新用户添加到区块链时，区块链才会增长。最后如[^kim2019storage]等人在研究的区块链压缩方案也是一种思路。

[^bruce2014mini]:Bruce J D. The mini-blockchain scheme[J]. White paper, 2014.
[^franca2015homomorphic]:França B F. Homomorphic mini-blockchain scheme[J]. 2015.
[^kim2019storage]:Kim T, Noh J, Cho S. SCC: Storage Compression Consensus for Blockchain in Lightweight IoT Network[C]//2019 IEEE International Conference on Consumer Electronics (ICCE). IEEE, 2019: 1-4.

而对物联网来说，要和区块链结合，必须解决不断产生的庞大数据的存储问题。因为虽然区块链可以防止数据篡改和单点故障，但使用区块链直接存储物联网数据，会面临机密性和可扩展性问题。

**机密性**：数据不可变性是区块链的优势，它给了区块链高稳健性，但不是所有的数据都需要不可变的存储，如用户的个人资料，更改后先前的数据仍会永远留在区块链中，任何人都能看到。同样，许多场景中数据最好只对一部分节点可见。解决办法是对原始数据进行加密再存储，在物联网场景下，也可以使用智能合约进行访问控制，从而把数据访问权只开放给授权节点。

**可扩展性**：如前所述，区块链的数据在不断增长，增加了节点参与的难度。在物联网场景将加剧这一问题，一种解决办法是数据仍然进行传统的中心化存储，而把数据哈希存储在区块链中[^ali2018secure]，这样可以保证数据的完整性，但缺失了解决单点问题的初衷。更合适的办法是采用分布式存储，Wulf Kaal将其划分为六种情况[^wulf2017how]，我们认为其划分有部分重复和缺失，故重新总结如下：

[^ali2018secure]: Ali S, Wang G, Bhuiyan M Z A, et al. Secure Data Provenance in Cloud-Centric Internet of Things via Blockchain Smart Contracts[C]//2018 IEEE SmartWorld, Ubiquitous Intelligence & Computing, Advanced & Trusted Computing, Scalable Computing & Communications, Cloud & Big Data Computing, Internet of People and Smart City Innovation (SmartWorld/SCALCOM/UIC/ATC/CBDCom/IOP/SCI). IEEE, 2018: 991-998.
[^wulf2017how]:Wulf Kaal. How can blockchain be used as a database to store data[Online]. https://www.quora.com/How-can-blockchain-be-used-as-a-database-to-store-data. 2017.

1. 传统的分布式存储：由于关系数据库对大数据存储的支持不是很好，故大数据的存储一般使用NoSQL数据库，[^siddiqa2017bigdata]对市场上出现的大数据存储技术做了全面的调查，关注点主要在于分布式存储技术，大部分都以NoSQL的格式进行存储，包括Google File System(GFS)，Hadoop Distributed File System(HDFS)，BigTable，HBase，MongoDB等，它们速度快、可扩展、容错、支持丰富的查询语言，但它们声称的容错指的是分布式本身带来的容错性能，一个节点被毁，因为副本数据的存在并不会对整个系统造成影响，而缺乏对恶意攻击的抵御性能，集群中的所有节点相互之间完全信任，任何一个恶意节点都能毁掉整个数据库。

   IPFS：建立在BitTorrent协议和分布式哈希表（Distributed Hash Table）基础之上的内容寻址服务，它成功的解决了大数据量的问题，但作为一个目标是下一代Web的系统，上传的所有数据将无法得到控制，这是企业及工厂所有者们不想看到的。

2. 去中心化的云文件存储：将存储系统托管在用户的计算机上，而不是在云端的服务器，由节点用户出租和购买存储空间来维持该系统。这样的项目包括Sia，Storj，PPIO，Ethereum Swarm。不需要保持在线就能分享文件。上传的文件能得到很好的安全保护，存储可靠，速度快，容量大。解决了传统物联网中心化存储单点故障问题，但在费用上是不可预知的，并且，在维持区块链的同时维持该存储系统，需要一些中间件来维持，显得有些笨拙和不方便。
3. Ties DB： TiesDB 继承了传统 NoSQL 分布式数据库的大部分功能，并增加了拜占庭容错和激励。有了这些功能，它可以成为公共数据库，并通过智能合约在 Ethereum 和其他区块链上启用功能丰富的应用程序。任何用户都有数据库写入权限，但被其账户地址唯一识别，所有的请求都进行签名。记录创建之后，创建者则成为记录的所有者一起被记录。之后，记录只能被记录所有者修改。每个人都可以阅读所有的记录，因为数据库是公开的。根据请求和复制检查所有的权限。额外的权限可以通过智能合约管理。

4. 巨链数据库BigChainDB：BigChainDB 的目标是将NoSQL数据库的主要优点与区块链技术的优点结合起来，如表3.1[^bigchainDB]所示 ，实际上，结合的正是MongoDB和Tendermint(一种BFT)共识

[^siddiqa2017bigdata]:Siddiqa A, Karim A, Gani A. Big data storage technologies: a survey[J]. Frontiers of Information Technology & Electronic Engineering, 2017, 18(8): 1040-1070.
[^bigchainDB]:BigchainDB. https://www.bigchaindb.com/features/[Online]. 

表3.1 Bitcoin，分布式数据库，BigchianDB特性比较

|              | 比特币区块链 | 分布式数据库 | BigchianDB |
| ------------ | ------------ | ------------ | ---------- |
| 不可变性     | Yes          |              | Yes        |
| 去中心化     | Yes          |              | Yes        |
| 网络上的资产 | Yes          |              | Yes        |
| 高吞吐量     |              | Yes          | Yes        |
| 低延迟       |              | Yes          | Yes        |
| 丰富的权限   |              | Yes          | Yes        |
| 查询功能     |              | Yes          | Yes        |

物联网和区块链结合的实验系统中，IPFS[^liu2017blockchain][^yu2017ethdrive]、SGS[^ayoade2018decentralized]、BigchainDB[^roman2018iot]和Swarm[^ozyilmaz2019designing]都有研究者使用过，其结合方式均为将物联网产生的原始数据存入这些存储系统，再将数据哈希存入区块链，这一方案会面临一个新的问题，即需要维持区块链和存储两套P2P系统，然后利用中间件进行交互，这种结构是复杂的不易维护且笨重的，Multichain中的off-chain结构通过将存储节点和区块链节点合并，解决了这一问题。另外，在工厂场景中，我们需要的是一个可以在本地部署的存储系统，如Storj这种分布式的云是不适用的，但Ethereum Swarm例外，它是一个分布式的云文件系统，但也可以在本地部署。

[^liu2017blockchain]:Liu B, Yu X L, Chen S, et al. Blockchain based data integrity service framework for IoT data[C]//2017 IEEE International Conference on Web Services (ICWS). IEEE, 2017: 468-475.
[^yu2017ethdrive]: Yu, X. L., Xu, X., & Liu, B. (2017). EthDrive: A peer-to-peer data storage with provenance. In CAiSEForum-DC (pp. 25–32).
[^ayoade2018decentralized]:Ayoade G, Karande V, Khan L, et al. Decentralized iot data management using blockchain and trusted execution environment[C]//2018 IEEE International Conference on Information Reuse and Integration (IRI). IEEE, 2018: 15-22.
[^roman2018iot]:Román V, Ordieres-Meré J. [WiP] IoT Blockchain Technologies for Smart Sensors Based on Raspberry Pi[C]//2018 IEEE 11th Conference on Service-Oriented Computing and Applications (SOCA). IEEE, 2018: 216-220.
[^ozyilmaz2019designing]:Ozyilmaz K R, Yurdakul A. Designing a Blockchain-Based IoT With Ethereum, Swarm, and LoRa: The Software Solution to Create High Availability With Minimal Security Risks[J]. IEEE Consumer Electronics Magazine, 2019, 8(2): 28-34.

综上所述，在单个工厂场景中，我们需要一个分布式的，可在本地部署的文件系统，同时，因为和区块链主体的分离，存储系统本身也需要高吞吐量、低延迟、不可变的安全特性等，多级权限不需要由存储系统提供，而由区块链的访问控制实现，最后，还需要支持在异构的物联网终端设备上运行。传统的分布式存储缺乏不可变特性和相关安全机制，IPFS和去中心化的云存储除Swarm外无法在本地部署，Multichain不支持并且在可预期的未来也不会支持在各种异构设备上运行，因此，我们需要将BigchainDB、TiesDB或Swarm与区块链深度结合，在保证原有特性的同时，减小双系统部署的复杂性，并提供异构系统的支持，从而为区块链和工厂进一步结合铺平道路。

## 4. 安全和隐私

区块链的价值与安全性随着用户数量的增加而增加，而物联网多数节点为自动化的终端设备，少数节点为人工控制，因此需要单独考虑。

### 4.1 常见安全问题

同互联网面对种种安全和隐私威胁相似，物联网面临同样的问题，并且随着物联网渐渐融入生活的方方面面，设备遭到安全攻击或用户隐私泄露带来的危害也更大。原因有很多方面，包括：（1）大多数通信都是无线的，这使得系统更容易受到攻击如身份欺骗，消息窃听，消息篡改和其他安全问题，以及（2）多种类型的设备在能量，内存和处理能力方面的资源有限，这阻碍了它们实施高级安全解决方案。

区块链与物联网的结合解决了诸如数据完整性方面的问题，但仍存在未解决的安全问题，并且区块链也带来了一些其本身面临的安全问题。下面介绍这些安全问题和可能面对的攻击。

**管理者**：区块链中交易和区块的添加依赖于矿工的工作，矿工伪造交易会被发现，但其拒绝添加交易的行为却可能带来严重后果。相似的，与物联网结合时，终端设备身份需要在链中验证和确认，如果身份提供者负责授权，那么它也可以阻止，[^kravitz2017securing]使用非对称密钥提供了一种解决方案。[^liu2017blockchainbased]提出了一种基于云的物联网应用的数据完整性框架

[^kravitz2017securing]:D. W. Kravitz and J. Cooper, ‘‘Securing user identity and transactions symbiotically: IoT meets blockchain,’’ in Proc. Global Internet Things Summit (GIoTS), Geneva, Switzerland, Jun. 2017, pp. 1–6
[^liu2017blockchainbased]:B. Liu, X. L. Yu, S. Chen, X. Xu, and L. Zhu, ‘‘Blockchain based data integrity service framework for IoT data,’’ in Proc. IEEE Int. Conf. Web Services, Honolulu, HI, USA, Jun. 2017, pp. 468–475.

**女巫攻击(Sybil attack)**：节点通过伪造多个身份来进行攻击，骗取权限或数据等，可以通过身份认证解决，即每个物联网设备的唯一标识和不可替代，这一点一般通过密钥分发完成。

**欺骗性攻击(spoofing attack)**：伪装成合法用户以利用其权限

**消息替换(message substitution)**：在信息传输过程中对其进行替换，而不被接收者发现。

**DoS(Denial of Service)或DDoS(Distributed DoS)攻击**：短时间发起大量请求，使网络拥塞，Ali的方案中用CH控制密钥列表来识别消息，避免DDoS带来的问题

**重放攻击(Message Replay Attack)**：用户请求被攻击者获取，然后重新发给服务器，从而达到认证通过的目的，加密解决不了该问题，一般通过时间戳或随机数解决。

**日蚀攻击**：区块链中通过隔离节点伪造安全的通信环境

工厂场景与比特币系统或其它的系统相反，节点的身份（即匿名性）是不被需要的，反而应该被验证；而数据的隐私通过完整的访问控制来实现，确保什么人什么时间可以访问什么数据。

但当多个工厂间或与用户通过跨链等技术连接时，需要考虑更多隐私方面的问题，但这不是我们现在需要考虑的。[^cryptonote2018]中提出的环签名方案保证只有交易双方知道身份。同态加密和零知识证明可以解决交易隐私问题，但解决方法都是资源密集型的，在资源受限的IoT设备上适用性有限。

[^cryptonote2018]:CryptoNote’s. Accessed: Apr. 10, 2018. [Online]. Available: https://cryptonote.org

### 4.2 访问控制

论文[^ouaddah2017access] 中提出，认证和访问控制是解决物联网中安全与隐私问题的主要手段，而任何有效的访问控制系统都应满足三个安全需求：机密性、完整性和可用性。机密性确保只有授权用户才能获得数据。完整性确保数据未经修改，物联网的场景中，将完整性分为两方面：1) 交易完整性：在网络传输过程中，交易信息不得被修改，核心是通信过程。2) 数据完整性：数据在整个生命周期内保持一致性和可信赖性。确保只有授权用户才能修改存储的数据。可用性意味着合法用户需要时可以访问数据。

[^ouaddah2017access]:Ouaddah A, Mousannif H, Elkalam A A, et al. Access control in the Internet of Things: Big challenges and new opportunities[J]. Computer Networks, 2017, 112: 237-262. ↩

完整的访问控制包括认证、授权和问责。[34]提供了关于授权的参考模型，将其分为目标、模型、架构、机制四层，从而提供一个完整的构思和实现思路。

物联网和区块链结合拥有了区块链的安全特性，机密性通过访问控制和加密技术实现，完整性由数字签名和区块链的不可变特性带来，可用性则基于分布式系统的基本特性，少量节点的受损不会导致数据的损坏不可访问。

工厂场景中提到访问控制的含义有两层，一是对存储的数据的权限的访问控制，这一部分在上面提到的多数方式中都能得到良好的实现，而对数据存入存储网络还是直接存入区块链可由矿工节点执行，存入区块链的数据由智能合约进行合适的存储；二是对设备权限的访问控制，如前所述，我们需要对更下层的，包括物联网网关甚至传感器的控制。

关于区块链和物联网结合，现有的访问控制的实现方式有两种，一种是利用提出的区块链架构实现访问控制，Ali在[4]中提出的架构中，通过簇头（Cluster Head）对节点进行分组管理，维护密钥列表，从而实现访问控制；一种是利用智能合约实现访问控制。[7]中使用名为Energy aggregators(EAG)的中间实体管理与能源交易有关的事件，并为IIoT节点提供无线通信服务，通过一定程度的中心化实现访问控制；Oscar在[18]中将物联网系统与区块链系统分离，只将访问控制策略定义在区块链中，通过Managers来注册设备和访问策略定义等，从而实现设备管理和访问控制；[^hashemi2016world]定义了一种基于区块链的多级机制来改进访问管理，该机制将指定功能，访问列表和访问权限。

[^hashemi2016world]:S. H. Hashemi, F. Faghri, P. Rausch, and R. H. Campbell, ‘‘World of empowered IoT users,’’ in Proc. IEEE 1st Int. Conf. Internet-Things Design Implement. (IoTDI), Berlin, Germany, Apr. 2016, pp. 13–24.

## 5. 其它

最后，区块链网络可能需要以下机制来补充其功能，并且这些机制也应是分布式的。

- 指向区块链中资源的DNS服务，用于查找相应的区块

- 安全的通信和文件交换，区块链中的消息通信是公开的，如果需要两个节点间专用的通信，需要telehash, Whisper之类的协议，不同工厂的区块链的交互需要跨链机制的支持，另一方面，传感器数据的存储可能使用私有区块链，而货币和服务的流通可能使用比特币或以太坊。

## 6. 总结

我们讨论了单个工厂内使用区块链的可能性，以及为了提高工厂生产效率和安全，在共识、存储、安全和隐私的各个方面可以做的种种改进，从而为下一步的研究做准备。


