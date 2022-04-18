# An ABAC  Model in RFID Systems Based on DApp for Healthcare Environments


Figueroa, Añorga, and Arrizabalaga, “An Attribute-Based Access Control  Model in RFID Systems Based on Blockchain Decentralized Applications for Healthcare Environments,” *Computers*, vol. 8, no. 3, p. 57, Jul. 2019, doi: [10.3390/computers8030057](https://doi.org/10.3390/computers8030057).

Keywords: blockchain, smart contract, RFID, ABAC, access control, IoT, healthcare

## 1. 引言

在医疗领域使用 RFID，可以追踪患者和医疗设备，更好的管理医疗资产，优化审计过程。如图1所示，一个 RFID 系统通常由四部分组成：

1. RFID 标签：包含识别数据
2. RFID 阅读器：与标签直接交互并进行信息交换
3. RFID 中间件：管理设备，管理数据（过滤、收集、整合、构建）
4. 信息管理层（业务层）：包含一些应用，如后端数据库，企业资源规划系统（ERP，enterprise resource planning），客户关系管理（CRM，customer relationship management），仓库管理解决方案（WMS，warehouse management solutions），电子产品代码追踪应用（tracking and tracing and electronic product code applications）。

![图1 RFID系统通用架构](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_RFID系统通用架构.png)

GS1（一个全球标准组织）的标准分三类：[识别]^(identify)， [捕获]^(capture)和 [共享]^(share)。RFID 标签启用电子产品代码（EPC，electronic product code）后，标签和阅读器可以执行捕获过程；识别号被编码为 GTIN（Global Trade Item Number）或被解码为 RFID 标签的 EPC 时，中间件执行识别过程；共享过程则由信息管理层完成。

GTIN 描述了一种数据结构，它使用 14 位数字并以某些组合方式进行编码，目前在条形码和 RFID 领域都有使用。一个 GTIN 号码的结构如下所示：
$$
urn:epc:id:sgtin:CompanyPrefix.ItemReference.SerialNumber
$$
GS1 标准系统用于实现可追溯性解决方案，尤其是在供应链中，如新鲜食品、健康、技术产业、运输和物流，其中 RFID 系统用于数据捕获和共享。本文以医疗行业的供应链为例，介绍一个追踪医疗资产流动的场景：医院使用大量的资产，如外科医疗器械（SMI），这些资产可以在消毒部门、手术室、实验室等区域间进行周期性的流动，器械位于错误的位置可能会危及患者的生命，缺乏详细的资产记录则可能造成资产损失。

RFID 是执行数据捕获和共享的关键技术，对任何 RFID 系统来说，安全性都至关重要，但 RFID 系统的安全威胁不同于传统的无线安全威胁，如论文[^figueroa2019comprehensive]所述，这些威胁可以分为：

1. RFID的物理组件遇到的威胁（如克隆标签、反向工程、标签修改），
2. 通信通道威胁（如窃听、浏览、重放攻击）
3. 全局系统威胁（如欺骗，DoS， 「tracing and tracking」 ）。

[^figueroa2019comprehensive]:Figueroa Lorenzo, S.; Añorga Benito, J.; García Cardarelli, P.; Alberdi Garaia, J.; Arrizabalaga Juaristi, S. A comprehensive review of RFID and bluetooth security: Practical analysis. Technologies 2019, 7, 15. 

访问控制（AC）是解决安全问题的核心，这里首先介绍本文方案使用的器械标识方法：在 GS1 的基础上，用 GTIN 标记 SMI（使用无源 RFID 标签）。编码方案如下，包括一个公司前缀（例如，医院 A:000389）、一个产品类型用以对资产进行分类（例如，剪刀：000162）以及一个序列号用以识别特定资产（例如，序列号：000169740）。
$$
01.000389.000162.000169740 \\\ Header|compPrefix|Product type|Srial Number
$$
图2用于详细说明医疗系统的工作流程。源房间（如灭菌室）将一些资产（如SMI）运送到目的房间（如0号手术室、1号手术室）。由于 $Asset_1$ 已被分配到目的房间1（例如，1号手术室），假如由于人为错误试图访问目的房间0（例如，0号手术室），其访问将被拒绝。简而言之，论文所提出的系统的目的是建立医疗资产（如SMI）访问控制系统，防止由于人为错误或外部安全威胁导致不需要的资产进入错误区域（如房间）。

![图2 Healthcare system](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_医疗系统.png)



访问控制机制通常部署在图1所示 RFID 系统的中间件部分，传统的实现通常是基于 RBAC 的中心化结构，而本文提出的方案中，利用 DApp 执行访问控制策略，智能合约用作 Dapp 和区块链间的接口，从而实现了分布式的访问控制，使允许或阻止某个资产进入某个确定的区域（如手术室）成为可能。该论文提出的方案还将将资产（如 SMI）和 GTIN 代码相关联，用于追踪资产。

作者在相关工作部分做了两个比较

1. ABAC vs. RBAC
2. Decentralized Model vs. Centralized Model

鉴于所作的比较，作者认为使用 ABAC 可以提供良好的灵活性和可扩展性，以及使用以太坊区块链提供分布式信任，实验首先在一个本地以太坊环境中实施，然后部署到 Ropsten 测试网中，最后扩展到以太坊主网络。但在扩展到主网前，需要首先数字化医疗资产。

## 2. 方案

如前所述，整个方案是一个融合多种不同技术的整体，下面首先对基本架构进行描述，从而使读者对该系统如何执行 ABAC 有一个了解。

### 2.1 分布式系统架构

基于以太坊实施的 ABAC 模型如下图所示，物理节点由 RFID Reader Control（RFID-RC），DApp和智能合约组成。当一个带有 RFID 标签的医疗器械尝试访问一个房间时，RFID-RC 发送访问请求到 DApp，DApp 查询智能合约返回与资产相关的属性（例如公司前缀，产品类型，序列号等），同时，DApp 还会从 RFID-RC 获取其它的属性如时间戳。然后，DApp 基于获取的这些属性来执行安全策略，从而决定来自标签的访问是允许还是拒绝。同时，物理节点可以通过与区块链建立新连接的方式进行复制，不影响现有节点，这体现了该系统可扩展性的优点。

![图3 基于Ethereum的分布式系统架构](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_基于Ethereum的分布式系统架构.png)

### 2.2 访问控制机制

[主体]^(subject)处理某个 [对象]^(object)的访问请求（例如，允许或拒绝访问），主要基于 ABAC  模型，大致检查四个方面：主体属性，访问控制策略，对象属性和环境信息。

尽管预期主体通常是人类，但诸如[自主服务]^(autonomous service)或应用程序等非人实体也可以作为主体这一角色。本文示例中，Reader 通过 DApp 发起对某个资产（RFID标签）的请求。为了将资产从源房间（如灭菌室）转移到目的房间（如手术室），需要建立一些边界条件。首先，如图4所示，已授权员工通过 DApp发起交易授权资产转移；其次，资产标签使用的 EPC（电子产品代码）如下表示式所示
$$
01.000389.000162.000169740 \\\ Header|compPrefix|Product type|Srial Number
$$
![图4 系统架构细节](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_系统架构细节.png)

DApp 收到访问请求后执行的过程如下所述

1. 基于 reader name（如，rdr_nm : "roomA"）和 location（如，loc：“41.40338，2.17403”) 两个属性验证主体(reader)
2. 验证其它属性，包括公司前缀（如， cmp_prf : 000389）、产品类型（如, item_ref: 000162）、指定资产的序列号（如, ser_nmb: 000169740），以及资产状态（如, st: “STERILIZED”）
3. 根据资产被送到医疗室，医疗室的阅读器收到来自资产的访问请求所经过的时间来验证环境属性。如果间隔小于10分钟（600秒），则环境条件验证通过。若在两个位置间移动资产，则发起交易设置该时间，变量 time-In（例如 time_-In:156209335）是交易完成后的时间记录，变量 time-out（例如 time-out:156209455）是 阅读器请求访问此 RFID 标签时给定的时间。

基于论文[^samarati2011access]的符号表示，我们建立了访问控制策略 C 的表达式如下

[^samarati2011access]:Samarati, P.; de Vimercati, S.C. Access Control Policies, Models, and Mechanisms. In International School on Foundations of Security Analysis and Design; Springer: Berlin/Heidelberg, Germany, 2011.

$$
if:(rdr_nm = "roomA" \cap loc = "41.40338,2.17403" \cap cmp\_pfr = 000389 \cap \\\ iem\_ref = 000162 \cap ser\_nmb = 000169740 \cap st = "STERILIZED" \cap \\\ time\_out - time\_in \lt 600), C = True  \\\ 
otherwise: C = False
$$

访问控制策略在 DApp 上执行而不是作为智能合约的一部分，主要有两个原因：一是公链存在速度限制，如果策略作为合约的一部分，会导致一定的延迟；二是公链中合约是公开的，访问控制策略将被所有人看到。

### 2.3 技术实施细节

图4表明整个系统分为三个子系统：ABAC configuration，ABAC execution 和 ETH blockchain monitoring。下表总结了每个子系统中使用的相关技术

![表4 使用的技术](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_使用的技术.png)

#### ABAC配置子系统

ABAC 配置子系统包括一个基于 React 构建的图形界面（GUI），通过浏览器交互。GUI 有两个视图，第一个视图允许授权员工向系统中添加新资产，员工将公司前缀、产品类型、资产ID（序列号）等参数输入系统，并随之生成交易存储在 ETH 区块链中。第二个视图用于转移资产，授权用户首先需要通过查询智能合约验证资产ID（序列号），这一操作通过点击 「Verify iD」按钮完成，然后就可以在两个房间转移资产，之前的资产属性如资产状态、时间戳都会更新，资产转移操作通过点击「Transfer asset」按钮完成。

![图5 ABAC配置子系统的两个视图](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_ABAC配置子系统的两个视图.png)

#### ABAC执行子系统

ABAC 执行子系统是图4中的物理节点，可以允许或拒绝对资产的访问请求，该子系统布置在每个医疗室内。执行子系统包括 RFID reader，LLRP服务器，属性解析器（AP），ABAC安全策略（ABAC-SP）和区块链接口（BI）。后三者构成 DApp，RFID Reader 和 LLRP 服务器构成 RFID-RC（RFID Reader Control）

RFID reader 附加标签的资产以及 LLRP 服务器直接交互。其中 LLRP 是 EPC Global 认可的一种协议，它是构成 reader 与其软件或控制硬件之间的接口，该协议在客户端（reader）和服务器（LLRP server）之间发送 XML 消息，论文使用开源工具 Rifidi 基于 SGTIN96 标准创建虚拟的阅读器和标签来进行概念验证。

属性解析器 AP 从LLRP服务器收到 RFID 标签的电子产品代码，然后使用一个基于Node.js库的GTIN 转换系统，将 RFID标签 的电子产品代码转换成 EPC 标签 URL。如下表达式所示，属性解析器解析得到公司前缀、产品类型和序列号等属性，同时也控制其它的属性如时间戳、reader name和位置。
$$
RFID \ Tag \ EPC: 3074257bf7194e4000001a85 \\\ 
EPC \ Tag \ URI: urn:epc:tag:sgtin-96:3.0614141.812345.6789,
$$


区块链接口构建基于Truffle框架，使用 Drizzle 库与 web3.js 服务器交互。Drizzle 是一个编写DApp 前端的前端库集合。DApp 和 RFID 部分通过执行 GET 和 POST 方法通信，例如，ABAC-SP 决定是否允许或拒绝对资产的访问，它设置一个变量，该变量通过 POST 方法发送到 LLRP 服务器。因此，LLRP 服务器发送一个 XML 消息 「keepalive」来保持与 RFID 标签的交互，或者只是断开连接。

#### 区块链监控子系统

使用了 ETH Network Stats 项目监控区块链系统的运行，如下图所示。这里有一个这篇论文实验的[视频演示](https://zenodo.org/record/3339217)

![图7 ETH监控工具](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_ETH监控工具.png)

## 3. 实验

为了证明方案的可行性，作者在两个环境中进行了实验：本地区块链和 ETH 测试链 Ropsten。

本地区块链环境下，使用如下命令部署了一个 ETH 节点，但设置为 nodiscover，因此无法连接到主网络。

```js
geth –datadir data –unlock 0x8a6d63ea98e05a550b01f8aa4a19021e43bd43f0 –networkid 123456
—-ws -wsaddr 192.168.127.95 –wsport 8546 –wsorigins "*" -rpc -rpcaddr 192.168.127.95
–rpcport 8545 –rpccorsdomain "*" –nodiscover console 2>> ETH.log,
```

测试链选择 Ropsten 而不是其它几种的原因如下：

1. 使用 PoW 共识，因此能更好的反映主网情况；
2. 可以同时使用 geth 和 parity 客户端
3. 允许添加自己的节点到测试网络，并可以从一些网址获取测试用的 ether

为了访问网络需要创建一个 Infura 项目，该项目会生成一个系统配置文件（truffle-config.js）会用到的 endpoint URL，如下所示。

```
ropsten.infura.io/v3/fa42299dbea54014801bc4145d7a1a1e,
```

### 3.1 工具

主要是 ETH Network Stats，Etherscan，Truffle Test 和 Infura Dashboard，这些工具全部被部署到本地环境及测试网络中，其中的例外是 Infura，这只是一个与测试网络有关的工具，下面是对这些工具的描述。

ETH Network Stats是一个追踪以太坊网络状态的可视化工具，使用WebSockets来接收运行节点的数据并通过界面显示，可以在本地单独运行，主要由前端的 Ethereum Network Stats 和后端的 Ethereum Network Intelligence API组成。至于 ETH Network Status 是一个类似的工具，用于 Ropsten 测试网。

Etherscan是一个监控区块链和其上交易状态的工具，作为服务器安装，但可以部署在本地。

Infura Dashboard 是为了更好地理解如何改进 DAPP 而对开发人员需求的响应，使我们能够获得有关调用 web3.js 方法的相关信息，这些方法允许与Ropsten 测试网进行某种类型的交互。

Truffle 是一个自动化测试框架，可以利用 JS 或 Solidity 编写简单易管理的测试。

| Tool          | ETH Network Stats | Ethersacn                                       | Truffle Test               | Infura Dashboard                        |
| ------------- | ----------------- | ----------------------------------------------- | -------------------------- | --------------------------------------- |
| 作用          | 网络监控          | 区块链监控                                      | 智能合约监控               | 带宽监控                                |
| 本地环境      | 本地节点网络监控  | 本地 ETH 区块链监控（如合约地址，交易和区块等） | 测试合约与本地区块链的交互 |                                         |
| Ropsten测试网 | 测试网的网络监控  | Ropsten测试网监控（如合约地址，交易和区块等）   | 测试合约与Ropsten的交互    | 允许查看每个 web3.js 方法使用的带宽行为 |

### 3.2 测量

论文对实现的每个部分都进行测试，从网络监控，节点数量和网络哈希速率等功能，到智能合约应用的延迟和每个web3.js方法的带宽消耗。下面分别介绍通过上一小节列出的工具可以监视的主要参数。考虑到与每个工具相关的主要特性，下图是这些工具使用的逻辑顺序

![图8 工具使用顺序](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_工具使用顺序.png)

ETH Network Stats 允许监控一些与 ETH 网络状态相关的参数：成功挖掘的区块数、叔块的出现、最后一个区块的挖掘时间、平均挖掘时间、平均网络哈希速率、难度、活动节点、gas price、gas limit、页面延迟、正常运行时间、节点名称、节点类型、节点延迟等。图7是一个使用示例，利用浏览器从一个本地 IP 的3000端口访问。

Ethersacn 获取与区块链相关的信息，如账户余额、账户信息、交易哈希、区块号、token类型（如Erc20）、average gas used、交易花费和交易费用等。

Infura Dashboard 允许获得的参数包括：调用的方法总数、每个方法占用的带宽、占用的总带宽。其测量的主要特征是带宽。

Truffle Test 测量的主要参数是：数据查询时间，数据插入的时间和花费，完整测试的时间，合约部署的 gas 和时间花费等。

### 3.3 结果

ETH Network State 监控网络状态，在这篇论文分析时，Ropsten 已经挖掘了5931224个区块，有14个活跃节点，平均出块时间为 14.04s，平均网络哈希速率为 120.1 MH/s，难度为 2.16 GH。这些参数可以与图7所示的本地区块链进行对比，本地区块链共挖掘了6967个区块，只有一个活动节点，平均出块时间为 27.45s，平均网络哈希速率为 142 KH/s，难度为 1.43 MH。可以看到，本地网络可用计算资源明显小于公网，这是可以预见的。

Etherscan 允许我们查看从我们的区块链测试地址发出的所有交易，可以验证的其他属性包括每个交易的状态、交易所在区块、使用的 gas 百分比（例如，平均 gas 使用量为限值的66.67%）、交易成本和费用以及 PoW 中使用的 nonce。本地网络中获得了类似的结果。

下图来自 Infura Dashboard，详细说明了为了通过智能合约与区块链交互， wpingeb3.js库 调用的主要方法以及它们所花费的带宽。该仪表盘还包括一些其它信息，如每小时带宽使用量的峰值（183.33 MB）和平均值（9.11 MB）。

![图9 Infura仪表盘检测到前五个方法的带宽使用](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_Infura%E4%BB%AA%E8%A1%A8%E7%9B%98%E6%A3%80%E6%B5%8B%E5%88%B0%E7%9A%84%E6%95%B0%E6%8D%AE.png)

使用 Truffle Test 进行的测试流程如下图所示

![图10 Truffle测试](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_Truffle测试.png)

得到的数据如下表，比较了数据插入、数据查询和完整测试的时间和 gas 消耗。百分比的计算公式如下，因为时间的不确定性，分别记录的最好和最坏时间。
$$
(Local\_network\_time/Ropsten\_network\_time) × 100
$$
![Truffle测试结果](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191111_Truffle测试结果.png)

合约迁移的延迟在本地网络中和测试网中具有很大的不同，但该参数对系统的评价不起决定性作用，因为该过程在系统部署前执行。资产属性的插入是一个主要的方法，尽管延迟是显而易见的，但不会导致访问控制策略的执行延迟。合约部署和数据插入的 gas 消耗不会因网络不同而改变。决定性的指标是数据的查询时间，由于本地网络的节点更少，数据查询的延迟也相对更小。

# 4. 总结与收获

作者利用区块链实现ABAC的思路没有值得称道的地方，但给出了针对医疗资产转移场景的一个完整方案设计，包括具体的属性管理、使用的工具、测量的参数以及详细的实验过程，这对于我们自己进行一个完整的实验是有很大借鉴意义的。

鉴于本文提到的内容，我们在实现一个基于IoT的访问控制系统时，首先要明确具体的场景并给出示例，基于智能合约实施核心功能后，要实现一个前端界面，以及使用本文提到的诸多工具测量所有相关的参数。对于我们现有的工作，欠缺的是一个前端实现和参数测量。
