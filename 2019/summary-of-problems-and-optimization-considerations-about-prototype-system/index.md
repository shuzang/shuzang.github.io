# 研究记录8-原始论文问题总结及优化考虑


本文介绍在论文复现过程中发现的一些问题和自己产生的一些想法。与原论文[^1]设计架构的对比可以查看本文最后的对比表，[点这里](#总结) 直接跳转。

[^1]:Zhang Y, Kasahara S, Shen Y, et al. Smart contract-based access control for the internet of things[J]. IEEE Internet of Things Journal, 2018.

## 1. 应用场景思考

考虑一个高层次的抽象结构，IBFT 共识最初是应对银行场景提出的，具有更高的可管理性和吞吐量，更低的延迟和最终一致性，但没有考虑验证者的可扩展性，而且 Quorum 是基于以太坊开发的并支持智能合约，这些特性是适合于工业场景的。

[IBFT共识](https://github.com/ethereum/EIPs/issues/650) 维持的核心是验证者，能容忍1/3以下的验证者节点故障（或恶意节点），因此验证者的数量不可少于4个，没有设置上限，但一般不会太多。区块链支持普通节点加入，数量不做限制，但普通节点对区块链的维持不起作用，不影响新区块的生成和区块链的延长。

基于以上的描述，考虑可能场景如下

1. 同一企业旗下位于不同地理位置的工厂间（参考 [Ali Dorri](https://ieeexplore.ieee.org/document/7917634) 在多个智慧家庭间设置区块链）。每个独立的工厂拥有一个验证者节点维持共识，工厂内的网关作为普通节点存在，IoT 设备受网关管理。
2. 工厂、设备供应商（维修方）、材料供应商、其它合作商、监管机构等构成小型联盟（参考 [Dennis Miller](https://ieeexplore.ieee.org/document/8378971) 和 [Barco You](https://arxiv.org/ftp/arxiv/papers/1809/1809.06551.pdf)）。每一方拥有一个验证者节点维持共识，各自都可以添加网关作为普通节点。

以上场景描述中以每一个参与方拥有一个验证者节点为例，但具体每一方分配多少验证者节点数量可以更好的保证安全性，可以做进一步分析。另外需要注意的是，超过2/3的节点串通就可以控制整个共识过程，少于2/3但大于1/3的节点串通可以导致共识永远无法达成。因此，多数人攻击可能是一个潜在的安全问题，[Roberto Saltini](https://arxiv.org/pdf/1901.07160.pdf) 最近（8月份）分析了IBFT的安全性并给出了改进建议。

注1：IBFT共识中没有矿工，也没有激励机制。交易、部署和调用智能合约虽然都需要 Gas，但 Gasprice 是0，不会真的消耗掉。因此，在我们的架构中，网关作为普通节点加入时，需要被分配一定的代币用于之后的操作，这一机制可通过智能合约完成，向每个新注册的设备账户转移一定数量的代币（做不到，因为调用智能合约首先需要用于余额，但是就是因为没有余额才想要调用智能合约获取余额，这是一个死循环，只能通过其他方式获取余额）。

注2：Quorum是联盟链，因此本身带有准入控制，不是所有人都可以加入。另外，Quorum还支持隐私交易。

## 2. 基本架构设计

这里对改进的架构做一次完整描述。整个架构自上而下可分为三层：共识层，网关层，设备层。前两层的所有节点构成区块链。以下对每层做解释

1. 共识层。联盟中的每个成员运行一个验证者节点，这些验证者节点共同维持区块链的存在，所使用的共识是IBFT。除此之外，访问控制三个合约中的注册合约和判决合约都由验证者节点部署。

2. 网关层。工厂或其它成员的网关设备，作为普通节点加入区块链，它们不参与共识过程，但拥有发起交易，部署和访问智能合约的能力（本质就是一个轻节点）。网关的核心作用是管理下一层的 IoT 设备，所有与网关连接的 IoT 设备都会由网关代理建立一个区块链账户，并调用注册合约进行注册，记录设备的基本信息、访问控制的优先级和归属权。IoT 设备的所有行为都与该区块链账户相关联，当设备发起访问控制请求，设备关联的区块链账户是发送者，当设备收到访问控制请求，接收者也是该账户。但这些操作都由网关代理完成。网关本身也需要进行注册，节点的第一个账户代表网关本身，其它账户分别与每个管理的 IoT 设备相关联。

    共识层的验证者节点和网关层的普通节点一同构成区块链。

3. 设备层。所有最下层的 IoT 设备，包括各种传感器/执行器甚至连到网关的智能手机等终端设备共同构成设备层。IoT 设备本身不作为区块链普通节点存在，所有行为由归属的网关代为管理。一个网关一般管理多个 IoT设备，但一个 IoT 设备只从属于一个网关，不能在区块链中重复注册。

采用这种分层管理的原因主要是常规的传感器/执行器的能力不足以作为区块链节点存在，即使能力足够，有限的能力也往往要用在本身的工作上。另一方面，这种分层管理方式还可以保证 IoT 设备不会直接暴露在网络中，减小了攻击可能性。虽然这同时带来了另一个问题，即作为核心的网关设备被攻破，其管理的所有设备都将失联。但可以注意到的一点是，IoT 设备的注册信息在区块链中始终存在，因此当判断网关失联或变成恶意节点后，可以将它所管理的设备的归属权移交其它网关。

智能合约中的归属权移交容易实现，但实际的移交却困难重重，这里分两种情况

1. 传感器/执行器与网关有线连接，这种情况下，即使合约中的归属权移交，也没有任何意义；
2. 传感器/执行器通过 Lora, ZigBee, Wifi，NB-IoT 等无线通信技术连接到网关，这种情况下，一方面不一定存在新的可连接网关，一方面如何控制 IoT 设备断开原网关重连到指定新网关上较难实现；
3. 设备即使移交成功，管理需要的密钥也较难转移。

普通情况下，基本的管理单位是设备，当设备退出网络可以删除其注册信息，当检测到设备恶意行为可以对其关联的账户进行处罚。但从另一方面讲，网关的沦陷可能意味着其所管理的设备全部沦陷，因此可以直接设置对网关账户的处罚，而对网关账户的处罚将级联作用于归属该网关的所有设备。因为网关沦陷后，如果设备不物理重连到其它网关，即使在区块链中移交了归属权，也无法继续进行操作，或者说控制设备完成访问控制。

理论上边缘设备的使用是不冲突的，边缘服务器本身可以作为普通节点加入，与一般的网关同属网关层，边缘服务器对资源进行分配时一般是容器虚拟化的方式，普通 IoT 设备对边缘服务器资源的访问控制请求将由边缘服务器关联到分配了资源的容器上。所以从这个角度看，验证者、网关节点等直接运行区块链节点似乎不是最好的选择，可以以容器方式运行。

## 3. 访问控制方案

当前架构中实现的访问控制的主体是智能合约，所有的操作通过智能合约完成。共有四种智能合约，注册合约、访问控制合约，判决合约和存储合约。

1. 注册合约。注册合约的作用是注册访问控制合约、判决合约、网关和网关管理的 IoT 设备，并提供对注册信息的管理（增删查改）。访问控制合约和判决合约在部署后应当立即调用注册合约进行注册，IoT 设备加入网络后应当由网关代理调用注册合约进行注册，声明设备基本信息、设备优先级，设备关联的区块链账户和所归属的网关。注册合约由验证者节点部署。
2. 访问控制合约。每个设备注册时都需要同时部署一个与之关联的访问控制合约，其中定义相关的访问控制策略，也定义恶意行为判定的策略，检测到的恶意行为会记录到判决合约并从中获取判决结果。访问控制合约中只能预定义少量的针对固定设备的访问控制策略，大量的访问控制请求处理通过设备优先级比较完成。执行操作的访问控制请求在链下进行，被请求的网关收到请求后，查询合约确认操作权限，确认拥有权限后被请求网关控制设备执行操作，获取执行结果并返回给请求者。读取数据的查询请求则由请求者直接发起，从调用访问控制合约验证读取权限后，从注册合约获取设备存储合约的地址，然后调用设备存储合约，获得数据哈希，根据数据哈希从存储平台中获得数据。写操作一般是向存储中写入设备数据，这一权限应当只属于设备本身和设备归属的网关。
3. 判决合约。判决合约的作用是记录恶意行为和做出判决，向访问控制合约返回判决结果。由于可能用到记录的这些恶意行为，由验证者节点部署一个判决合约最合适。但如果不需要，可以像访问控制合约一样一个设备关联一个判决合约，
4. 存储合约。以上流程中会涉及数据的写操作，传感器收集的数据源源不断地传输给网关，网关应当在本地进行处理并将数据存储到存储平台（Swarm 或 IPFS），获取返回的所存数据的数据哈希，在调用访问控制合约验证写操作权限后，将数据哈希存储到存储合约中。

访问控制的流程值得仔细考虑，比如网关如何将执行结果返回给请求者，初步的想法是链下进行，或者在合约中设置一个状态量，执行成功后将状态量置为真，然后智能合约主动将结果告知请求者。

访问控制可能发生在同一工厂的设备间，用于获取某些需要的资源或请求执行一些操作，也可能发生在设备维修商对工厂的设备，目的是检测设备情况，同时，监管机构也可能不时调用相关数据进行验证。同一企业位于不同地方的工厂间也可能相互访问。

### 3.1 关于属性集的思考

ABAC 中有四种实体可以拥有属性，分别是主体(subject)、客体(object)、动作(action)和环境(environment)。在我们的方案中，所有的区块链账户背后都是一个设备（可能是IoT设备、网关或服务器等），每个设备都可能是发起访问控制的主体(subject)。设备拥有资源，比如数据、计算能力、存储能力等，每个资源是接收访问控制的客体(object)。动作(action)是可执行的操作，比如读、写、执行等。环境(environment)是一些独立于资源的需要满足的环境条件，比如两次访问控制请求的间隔，请求的频率等。这四者都需要定义与存储其属性集。我们来考虑几种属性集分别应该定义在哪种合约中，又分别应该按什么样的格式来定义和管理。大致的想法有以下三种：

1. 属性集全部定义在 RC 中。由于 RC 的作用本来就是合约管理，似乎再添加属性集管理的功能也无可厚非，但这样带来的问题一是 ACC 进行访问控制时所有读取属性的操作都要跨合约进行，二是管理权问题，需要严格划分谁来定义和管理属性，三是类似于环境和操作属性等严格与设备相关，每个设备的这些属性都有可能不同，统一定义到 RC 中不好区分。
2. 属性集全部定义在 ACC 中，这样带来的问题是策略判决时访问主体(subject)的属性要先调用 RC 获取 subject的合约地址，再访问 subject 关联的 ACC 获取它的属性，流程繁琐
3. 主体(subject，也即设备)的属性集定义在 RC 中，其它属性集定义在 ACC 中，独立维护主体的属性集的好处是调用属性不再需要两道流程，同时资源、操作、环境这些属性集由各自的管理者定义和管理，有助于根据具体的情况进行调整。

一个可参考的实现是 wang 的论文[^wang_2016_blockchain]。

[^wang_2016_blockchain]:P. Wang, Y. Yue, W. Sun, and J. Liu, “An Attribute-Based Distributed Access Control for Blockchain-enabled IoT,” in 2019 International Conference on WiMob, Barcelona, Spain, Oct. 2019, pp. 1–6, doi: 10.1109/WiMOB.2019.8923232. 

## 4. 仿真实验

实验总体会涉及两部分，耦合程度并不高，第一部分是区块链平台搭建和智能合约实现。区块链平台选择了Quorum，这是一个基于以太坊开发的联盟链平台，提供了更高的可管理性、吞吐量，降低了延迟，提供了隐私交易的功能。由于 Quorum 保留了以太坊的智能合约，访问控制的基本功能都通过智能合约实现，合约的编写，编译和安全检查使用在线编辑器 Remix 完成，合约的部署和测试则使用 Truffle 框架完成。网关与合约的交互主要通过 web3.js 库来完成，区块链的节点使用 geth 客户端，由于官方没有提供 arm 架构下的客户端，我们自行进行了编译。

第二部分是网关和 IoT 设备。在IIoT场景下，机械臂是最常见的终端 IoT 设备，但受限于实验条件，我们无法将其用于实验。其它容易获得且能代表一个典型场景的设备暂时也无法找到，因此目前考虑能完成通用操作的设备，比如任意的传感器和执行器。终端 IoT 设备从普遍意义上可以分为传感器和执行器两种，传感器收集并传输数据，执行器执行操作。由于当前的架构设计中采用分层管理的方式，网关作为区块链节点，终端 IoT 设备由网关控制，我们需要一套包括网关、传感器、执行器在内的设备做实验模拟，可考虑的有三种

1. 树莓派+arduino+传感器/执行器
2. 树莓派+Lora通信+arduino+传感器/执行器
3. Beaglebone+CC1350+Sub1GHz通信+CC1350+传感器/执行器

目前已有的传感器有音量传感器、灰尘传感器，执行器有全彩LED。另外 CC1350 LunchPad 内置了温度传感器和LED。传感器数据的处理或执行命令的传递使用 Python 编写程序完成。

一些两部分之间的交互流程，如从合约调用的结果中获取访问控制命令传输给 IoT 设备，或者预处理 IoT 设备传输的数据交给网关，都通过 node.js 调用 web3.js 库来完成。

## 5. 安全分析

将区块链中的异常分两种，一种是意外发生的异常，无法确定异常类型；另一种是主动发起的攻击。

按架构中的三个层次分别总结各层中可能出现的异常（安全隐患）

### 5.1 共识层

共识层的主体是验证者，它们有可能破坏整个区块链的正常运行，可能的恶意行为（来自[#650-Faulty node](https://github.com/ethereum/EIPs/issues/650)）有

- NotBroadcast：验证者不广播共识过程中的消息
- SendWrongMsg：验证者发送错误的消息
- ModifySig：验证者修改消息签名
- AlwaysPropose：验证者不断地打包新区块并广播
- AlwaysRoundChange：验证者收到消息时总是发送`Round change`消息
- BadBlock：验证者打包的区块内容具有非法交易

以上是在共识运行过程中验证者可能做出的破坏。同时，承载验证者节点的设备可能在运行过程中宕机从而导致节点失联。

另外，联盟的部分成员可能串通作恶，为自己谋取利益，当串通的节点数超过总节点数的1/3，区块链将不再安全。

### 5.2 网关层

网关层的普通节点可能的恶意行为包括

- 持续发起访问控制请求
- 拒绝执行已验证的访问控制请求
- 返回虚假的设备执行结果
- 部署错误的访问控制合约
- 注册不存在的IoT设备

### 5.3 设备层

设备层与区块链无关，其通讯链路的安全只能依靠传统方案保证。

除以上所述的具体恶意行为，传统的攻击方式，包括 DDoS、女巫、中间人、日蚀等各种各样的攻击手段应分别讨论。

## 6. 合约中存在的问题

以下是我们在论文复现中发现的合约实施过程中出现的问题，或者想到的一些需要实施的合约改进。

### 6.1 设备注册

之前已经讨论过设备注册功能，这一功能应当在合约实施中添加。

设备注册这一功能将依旧由注册合约（RC）完成，注册的设备信息由结构体 devices 统一描述，包括设备ID、设备类型，设备说明、设备优先级、设备关联的区块链账户，设备归属的网关账户、设备关联的访问控制合约地址（如果采用单独的存储合约，还会包括设备关联的存储合约地址）等字段，其中，设备关联的区块链账户地址与设备唯一关联，可以使用 mapping 结构作为键来使用。

如之前所述，设备的注册将由网关代理完成，每个设备对应一个唯一的区块链账户，应进行静态检查避免重复注册。未注册的设备无法完成任何操作，而区块链中发起对设备的请求时，应当同时包括请求者(subject)、设备(object)、设备代理网关(manager)三个角色的账户地址。

访问控制的双方不做限制，可能是机构双方，可能是机构对设备，也可能是设备对设备。

判决合约的主键应更改为 IoT 设备关联的区块链账户。区块链账户的恶意行为应该折算计入网关设备的恶意行为。

### 6.2 合约warning

合约编译的过程中产生了不少的warning，这些warning虽然不影响合约的部署和使用，但总令人担心具有某些安全隐患，尤其是我们实现的合约本身就与安全息息相关。我们之前从中发现了关于过量 Gas 消耗和内联汇编(inline assembly)的问题，并致力于对其进行改进，同时，对合约中的某些设计我们也持怀疑态度，但很多问题在逐渐深入的了解过程中都被发现其实并没有太大的影响。即使如此，我们依然在此对各个warning做一下分析。

<font color="red">这部分所作的改进可以精简一下，作为自己的一部分工作。</font>

#### 内联汇编

在注册合约(RC)和访问控制合约(ACC)中，使用了 methodName 和 resource 两个字段作为索引值，用来查询方法、策略和恶意行为的相关信息。由于这两个字段长度的不确定性，将其定义成了string类型，但mapping的键无法使用动态长度(dynamically-sized)的string作为类型，因此将其定义成了固定长度(fixed-sized)的byte32类型。

```js
struct Method{
    string scName;       //contract name
    address subject;     
    address object;      
    address creator;     //the peer(account) who created and deployed this contract
    address scAddress;   //the address of the contract
}

mapping(bytes32=>Method) public lookupTable;
```

为了实现完整的功能，需要在每个函数中都事先将string转换为bytes32类型，这一功能通过stringToBytes32函数实现，而该函数完成类型转换使用了内联汇编(inline assembly)，并由此引发了waring.

> browser/RC.sol:23:9:CAUTION: The Contract uses inline assembly, this is only advised in rare cases.
> Additionally static analysis modules do not parse inline Assembly, this can lead to wrong analysis results.

我们当前所担心的，是内联汇编会不会引起Gas的大量消耗，以及会不会带来安全隐患两个问题。[Solidity文档](https://solidity-cn.readthedocs.io/zh/develop/assembly.html)中对内联汇编的解释一定程度能解答我们的问题。

solidity的内联汇编是为了实现更细粒度的控制，通过汇编风格的代码直接从底层访问以太坊虚拟机。内联汇编程序使用`assembly{...}`来标记，在大括号中书写汇编代码。我们的类型转换代码如下

```js
function stringToBytes32(string memory _str) public pure returns (bytes32 result) {
    bytes memory tempBytes = bytes(_str);
    if(0==tempBytes.length){
        return 0x0;
    }
    assembly{
        result := mload(add(_str,32))
    }
}
```

可见，函数中的汇编代码只是单纯的使用了`add`和`mload`操作码，而智能合约的所有代码最终都会转化为这种底层的操作码，可见不会引起Gas的额外消耗。文档中称「内联汇编是一种在底层访问以太坊虚拟机的语言。这抛弃了很多 Solidity 提供的重要安全特性」。这说明了使用内联汇编可能会带来一些安全上的问题，但我们暂时并不清楚是什么。至于编译时弹出的警告，注意是由于编译器无法对汇编语句进行相关的检查，所以对我们进行提示。

但这一警告同时也提示我们要明确自己在做什么，因为除非必须，否则最好不要使用内联汇编。从这一点看，我们回到使用内联汇编的出发点，string类型转换为bytes32类型，但我们发现这种转换其实是没有必要的，因为转换过程中超出bytes32大小的部分依然会被截断，那么为什么一开始就定义为bytes32类型，因为同样的条件下bytes32比string使用更少的gas。唯一的理由是基于可读性的考虑，在合约调用传入参数时可以直接将双引号或单引号包围的字符串类型参数传入，我们认为，基于这样的理由在合约中完成类型转换是不必要的，类型转换的工作应该在调用合约的脚本中实现。因此关于内联汇编的warning得出结论

<font color="#FF0000">删除合约中的类型转换函数，将类型转换操作迁移到调用合约的脚本中完成，同时将合约中相关的string类型调整为bytes32类型</font>

关于如何在脚本中进行类型转换，有两个参考：

[stackoverflow—Pass parameter as bytes32 to Solidity Smart Contract](https://stackoverflow.com/questions/50728855/pass-parameter-as-bytes32-to-solidity-smart-contract)

[csdn—web3j中字符串如何转换Bytes32](https://blog.csdn.net/mongo_node/article/details/81094602)

#### Gas需求无限

这一warning是我们判断合约Gas消耗过量的主要依据。

> Gas requirement of function <function_name>() high: infinite. If the gas requirement of a function is higher than the block gas limit, it cannot be executed. Please avoid loops in your functions or actions that modify large areas of storage(this includes clearing or copying arrays in storage)

这一warning占到总warning的1/3到一半，可以说是最大的问题了。但[网上找到的回答](https://ethereum.stackexchange.com/questions/37321/gas-requirement-of-function-function-name-high-infinite)中发现这个warning其实并不重要，因为很多情况都会引起 `infinite` 的警告，但这并不意味着代码中存在无限循环或者说代码不正确。在我们的程序中，导致该 warning 的三个个核心问题是内联汇编的存在，string 类型的定义以及调用其它的合约，Remix无法预测gas消耗时会提示 `infinite`，string 作为动态类型无法预测，内联汇编的语句无法在编译时进行检查，合约无法获得另一个合约的代码，因此这三种情况都会做出这样的提示。

在上一节中我们已经决定撤销在合约中进行类型转换，因此内联汇编将随之消失，而合约的调用是必需的操作，我们只要知道不会造成影响就好。

关于如何消除这一警告的一个解释是 [stackExchange—How to get the cost (in gas) of the non-constant function call?](https://ethereum.stackexchange.com/questions/27695/how-to-get-the-cost-in-gas-of-the-non-constant-function-call)

至于string类型的使用，让我们继续考虑上一小结，考虑一个相关的问题：[Use string type or bytes32](https://ethereum.stackexchange.com/questions/11556/use-string-type-or-bytes32)。答案很清楚，对长度超过32字节的任意字符串数据，使用string类型，否则就使用合适固定长度的bytes，无论是bytes32还是其它。

考虑我们的合约，注册合约(RC)中定义了string类型的scName，意为合约名。根据我们已掌握的知识，普通的ASCII码如英文字母和数字等都只占据1个字节，汉字一般占据3个字节(当然，我们传参一般不用汉字)。原作者给出的示例合约名为`ACC`和`Judge`，以我们编写的代码中定义的合约名来看，`Judge`和`AccessControlMethod`两个合约名都不超过32个字节，因此，合约名字段可以不适用string类型。同时基于合约名字段的有效性考虑，我们打算将其更改为合约类型字段，共分`JC`和`ACC`两种合约类型，更具有实际意义。因此最后的决定为

<font color="#FF0000">注册合约中的合约名字段更改为合约类型，同时字段类型由string调整为bytes32</font>

判决合约(JC)中定义操作、恶意行为为 string 类型，恶意行为的长度确实是不固定的，可能超过32字节，但操作一般情况只包括`read`，`write`和`execute`三种，即使存在其它操作描述，一般不超过32字节，故将其定义为byte32类型。最后的决定为

<font color="#FF0000">判决合约中操作字段类型由string调整为bytes32</font>

访问控制合约(ACC)中除与JC相同的两个字段外，错误信息描述同样使用string定义，但它同样是有意义的。而权限字段只有 `allow` 和 `deny` 两种选择，故将其定义为bytes32类型。最后的决定为

<font color="#FF0000">访问控制合约中操作字段和权限字段由string调整为bytes32</font>

以上之所以统一更改为bytes32类型而非根据长度调整是为了转换方便。

#### 删除不彻底

RC和ACC中都会涉及`delete`操作，用来删除映射表中无效的方法或策略。但也因此引发了如下warning

> browser/RC.sol:52:9:Using delete on an array leaves a gap. The length of the array remains the same. If you want to remove the empty position you need to shift items manually and update the length property.

这是因为`delete`操作的本质是对变量赋初值，并不清理内存，因为隐射与数组比较大时，删除操作之后调整长度和内存将会非常消耗gas，没有意义。所以这里就不管这个警告了。

#### 变量名相似

因所起变量名相似引起的警告，不做修改。

> Register.methodRegister(bytes32,bytes32,address,address,address,address) : Variables have very similar names _subject and _object.

当前三个合约中相似的变量名包括`object`和`subject`，`key`和`res`，`ToLR`和`NoFR`，注意区分就好，而且剩下的warning中一半以上都是这个。

#### 函数使用

我们在合约中使用了自我销毁函数销毁不用的合约，以及使用异常函数进行判断，因此出现该警告，可以无视。

> browser/JC.sol:63:9:Use of selfdestruct: can block calling contracts unexpectedly. Be especially careful if this contract is planned to be used by other contracts (i.e. library contracts, interactions). Selfdestruction of the callee contract can leave callers in an inoperable state.
>
> Use assert(x) if you never ever want x to be false, not in any circumstance (apart from a bug in your code). Use require(x) if x can be false, due to e.g. invalid input or a failing external component.

#### 重入

任何从合约 A 到合约 B 的交互都会将控制权交给合约 B， 这使得合约 B 能够在交互结束前回调 A 中的代码，从而引发[重入攻击](https://solidity-cn.readthedocs.io/zh/develop/security-considerations.html#id4)，因此出现该warning，可以使用“检查-生效-交互”（Checks-Effects-Interactions）模式避免重入。

我们在合约代码中添加了条件判定，但该模式没有被分析器检查到，这里不用担心。

#### 其它

原论文中作者统计的 ACC，RC 和 JC 部署的Gas需求分别为2 543 479, 1 559 814和1 380 781。经过我们调整后，三种合约部署的Gas需求减少到了2112988，759800和1019782。但这种比较似乎没有必要，因为之后会添加不少功能，会进一步增加Gas需求。

原架构作者的示例实验使用的是以太坊和工作量证明共识，我们考虑场景的限制，在第一阶段实验换用了 Quorum 和 IBFT 共识。在 Quorum 中，对 Gas 有一定的调整，[#Issue38](https://github.com/jpmorganchase/quorum/issues/38)详细的说明了这一部分改动。主要是Quorum 中交易不消耗 Gas，但这并不是说没有 Gas 的存在，只是将 gasPrice 设为了0。因此，部署和交易前，账户中依然需要一定量的Gas做支持，否则无法执行，而 Quorum 中没有挖矿这种稳定的代币来源，因此，新加入的节点只能由区块链初始化时预定义的节点发送一定的货币来获得 Gas，从而开启交易。但同时这也意味着，在 Quorum 中，减少 Gas 消耗的必要性并没有在采用工作量证明的以太坊中强。

<font color="#FF0000">需要向每个新加入的节点发送足以发起访问控制交易或部署合约的货币</font>

一些未完成也没有必要完成的

- get 类函数用 event 代替：引文 get 类函数不改变合约状态，没有换用 event 的必要性。
- send,call 和 event 自带的一些返回值可不必重复定义：send 和 call 没有返回多少有用的信息，event 返回的可能重复的是合约地址，但其实也没什么用。

### 6.3 自主决策

访问控制的权限校验严重依赖于预定义的策略，当设备数量过多时，人工预定义策略是一个极为庞大的工作。而在无法检测到相关策略的情况下权限申请会被直接驳回，由此可能引发错误。这一机制对可扩展性的支持也不是很友好。

当前思路是在设备注册时即对设备资源优先级进行预定义，当接收到访问控制请求时，如果未查询到定义的访问控制策略，则根据设备的优先级进行判定，最后做出决策。这种判定表的设计在强访问控制（MAC）中有参考。

### 6.4 授权时间

目前的架构设计没有权限授予时间的概念，所有的授权都是一次性的，下次访问要重新发起访问控制，在很多情况下并不合适。

思路是添加权限授予时间字段，默认一次性授权，根据情况可以自定义授权时间，在授权时间范围内不必重新发起请求。

### 6.5 隐私

基于场景中可能的隐私需求，利用 Quorum 的隐私交易管理器实现。

更新：实际上有大量的论文自己设计某种算法来保证隐私，也许我们不应当自我限制必须用隐私管理器解决。

## 7. 实验中存在的问题

- ~~实验中 ACC 对 JC 合约的调用不成功，应修复合约间调用出错的问题~~
- ~~使用虚拟机为实验带来了高延迟~~
- ~~合约功能测试可以通过测试合约完成，或利用Remix完成，或者在测试链中测试~~
- 使用 truffle 框架简化合约部署、脚本执行等相关操作(truffle 框架具有自己的限制)
- ~~网关设备调整为普通节点，在区块链启动之后加入，并预先分配一定的代币~~
- 通过构建前端页面形成完整访问控制 Dapp，将一些涉及多个操作的合约交互流程自动化(但估计很难有精力完成)——确实没有精力，也没有能力。
- 网关与 IoT 设备的连接，来自 IoT 设备的数据如何返回到区块链，来自区块链的控制命令如何交给 IoT 设备，这两种交互的实施（属于对访问控制系统的应用，没有实现）
- 利用相关工具对合约代码进行安全性检查（好用的工具都收费了）
- ~~利用 Ethscan 等工具监测区块链获取相关数据~~（找了一个区块链浏览器）

当前工作的关键词

## 8. 总结

<span id="总结">总结</span>一下所作的调整和改进

1. 使用联盟链 Quorum 和联盟链共识 IBFT，增强了可管理器、吞吐量，减小了延迟，可以进行隐私交易
2. 分层管理的结构具体化，设备由网关代理并在区块链中注册，读数据、写数据、执行请求等流程和场景具体化
3. 基本的管理单位由资源调整为设备，核心的访问控制结构变成 RC+ACC+JC+SC，分别指注册合约、访问控制合约、判决合约和存储合约。
4. 网关故障时的归属权移交机制（不可行）
5. 使用强访问控制对原先的访问控制列表进行补充（未实现）
6. 授权时间调整为动态（未实现）

其它的一些想法

1. 访问控制的论文再总结一次，归纳发展情况和不足，提出我们当前方案的改进
   - 设备可以阻止别人的未授权访问，当若设备本身已被控制，我们的方案无法解决，所以假设设备、网关是安全的，不考虑内部攻击，只考虑通过访问控制阻止未授权访问。访问控制的策略由资源所有者定义
   - 原方案存在的问题：过于分散。每个 subject-resource 对都要定义一个 ACC，但不是所有的 subject 都会对每个资源进行访问，全部定义会有冗余，这是访问控制列表方法的缺点。何时布置 ACC 合约，如何授予权限都需要管理员管理，工作量极大。
2. 权限转移，不只是权限所有者，也可能是被授予权限的人  [Blockchain Based Access Control](https://link.springer.com/chapter/10.1007%2F978-3-319-59665-5_15)
3. 当前的行为检测是绝对的，但很多情况下根本无法判断某个行为是好的还是坏的，只能打分来进行量化，基于分数来判断行为是否出格

## 9. 相关资料

[1]  李晓峰, 冯登国, 陈朝武, et al. 基于属性的访问控制模型[J]. 通信学报, 2008(04):95-103.  [链接]( http://www.cnki.com.cn/Article/CJFDTotal-TXXB200804018.htm )

[2]  基于属性的访问控制关键技术研究综述[J]. 计算机学报, 2017(7).  [链接](http://kns.cnki.net//KXReader/Detail?TIMESTAMP=637085548841398750&DBCODE=CJFD&TABLEName=CJFDLAST2017&FileName=JSJX201707013&RESULT=1&SIGN=Xd7ex2falR%2bt%2bAGP4Ai5jpxADxs%3d )

[3] Vincent C. [Attribute-Based Access Control](https://ieeexplore.ieee.org/document/7042715)

[4] NIST Guide to ABAC Definition and Considerations：[链接](https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.sp.800-162.pdf)

[5] [SCI 数据库查询页面](http://apps.webofknowledge.com/Search.do?product=UA&SID=7B86yaCsiTIZKDa1PZT&search_mode=GeneralSearch&prID=afa0ae1b-f225-42f0-b473-fc1243fa3746)

[6] 访问控制模型简单介绍：https://blog.csdn.net/LngZd/article/details/100781310

[7] Golang实现的各种访问控制模型的库，[官网](https://casbin.org/docs/zh-CN/abac)，[Github](https://github.com/casbin/casbin)





---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2019/summary-of-problems-and-optimization-considerations-about-prototype-system/  

