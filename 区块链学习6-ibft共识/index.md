# 区块链学习6-IBFT共识


AMIS公司提出的 Istanbul Byzantine Fault Tolerance Consensus（简称IBFT或Istanbul BFT），是一个基于PBFT的交易一致性的共识。因为要考虑可能发生的异常，对共识的原始文档作一次通读，仔细理解一下。原始文档位于github [ethereum/EIPs#650](https://github.com/ethereum/EIPs/issues/650)，以下一边翻译一边阅读。

注：EIP，即Ethereum Improvement Proposal，以太坊改进建议

---

这一工作深受Clique POA的启发，并在协议层中尽可能设计相同的机制，比如验证者投票。我们遵循EIP风格，将背景和原理放在所提出的共识协议的后面供开发者阅读。这一工作也受到Hyperledger's SBFT，Tendermint，HydraChain和NCCU BFT的启发。

## 术语

- Validator：区块验证者
- Proposer：一轮共识中被选举出打包新区块的验证者。
- Round：共识轮数。一轮共识起始于Proposer打包一个新区块，结束于新区块提交或轮数改变（轮数改变可能由于验证不通过或区块更新）。
- Proposal：共识正在处理的新打包的区块。
- Sequence：新区块的序号（从创世区块起排列的一个顺序），这一数字应大于所有之前区块的序号。目前，每个区块的高度都是它的序号。
- Backlog：将来的共识信息记录在这里面。
- Round state：指定轮数和序号的共识信息，包括预准备信息、准备信息和提交信息。
- Consensus proof：提交的区块签名，每个验证者验证后都会对区块签名，可以证明区块经历了整个共识过程。
- Snapshot：验证者投票状态。

## 共识

Istanbul BFT基于 [PBFT](http://pmg.csail.mit.edu/papers/osdi99.pdf) 算法，然而，原始的PBFT需要做一些调整来适应区块链。首先，没有具体的发送请求和等待结果的`client`的概念，所有的 validator 都可以视作`clients`。其次，为了保证区块链的运行，需要在每一轮共识中持续不断的选举出 proposer 来打包新的区块，同样，每轮共识的结果是一个可验证的区块而不是文件系统的一组读写操作。

Istanbul BFT 继承了 PBFT 共识的三阶段：pre-prepare，prepare 和 commit，我们称之为预准备阶段、准备阶段和提交阶段。系统可以容忍 N 个 validator 节点的网络中F个节点错误，其中 N = 3F + 1。每一轮之前，validators会首先投票选出一个proposer，默认的选举方式是轮询。选出的proposer将会打包一个新的区块并附随 pre-prepare 消息广播出去，当接收到 pre-prepare 消息，validators 会进入 pre-prepared 状态，然后广播 prepare 消息。这一步是为了确认所有的 validators 在同一个 sequence 和同一个 round上工作。当接收到 2F + 1个 prepare 消息，validator 就会进入 prepared 状态并广播 commit 消息。这一步是为了通知其它节点它验证了新区块并且将会把新区块添加到了区块链中。最后，验证者们等待 2F + 1 个 commit 消息并进入 committed 状态，最后把区块添加到区块链末尾。

Istanbul BFT共识中的区块符合最终一致性，这也就是说所有的区块都必须位于主链中，区块链没有分叉。为了防止恶意节点生成一条和主链完全不同的链，每个验证者都要在将区块添加到区块链末尾之前附加 2F + 1 个收到的 commit 签名到区块头的 extraData 字段。因此，所有区块都是自验证的并且支持轻节点。然而，动态的 extraData字段可能造成区块哈希计算的相关问题，因为不同的验证者在同一轮共识中收到的一组 commit 消息可能不同，从而导致 extraData 字段不同，最终整个区块的哈希值不一致，为了应对这种情况，在计算区块哈希时会排除 commit 签名部分。因此，依然可以保持区块/区块哈希的一致性，并把一致性证明放在区块头。

### 1. 共识状态

Istanbul BFT是一个状态机复制算法，每个验证者为了达成区块一致都维持一个状态机副本。

状态(States)：

- new round：proposer打包新区块，验证者等待 pre-prepare 消息
- pre-prepared：验证者接收 pre-prepare 消息，广播 prepare 消息，然后等待 2F + 1 个 prepare 或 commit 消息
- prepared：验证者收到了 2F + 1 个 prepare 消息并广播 commit 消息，然后等待 2F + 1 个commit消息
- commited：验证者收到了 2F + 1 个 commit 消息，可以将新区块插入区块链末尾了
- final commited：新区块成功插入区块链末尾，验证者准备下一轮
- round change：验证者等待同一轮上的 2F + 1 个round change消息

### 2. 状态转换

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191029_状态转换图.png)

- New round —> Pre-prepared
  - proposer从交易池收集交易
  - proposer打包新区块并广播，然后进入 pre-prepared 状态
  - 每个验证者收到pre-prepare消息后，若符合如下条件，进入pre-prepared状态
    - 新区块来自有效的proposer
    - 区块头有效
    - 新区块的序号和轮数符合验证者状态
  - 验证者向其它验证者广播prepare消息
- Pre-prepared —> Prepared
  - 验证者收到 2F+1 个有效prepare消息后进入prepared状态，prepare消息有效是指符合如下条件
    - 序号和轮数匹配
    - 区块哈希匹配
    - 消息来自已知验证者
  - 验证者进入prepared状态后广播commit消息
- Prepared —> Committed
  - 验证者收到 2F+1 个有效commit消息后进入committed状态，commit消息有效是指符合如下条件
    - 序号和轮数匹配
    - 区块哈希匹配
    - 消息来自已知验证者
- Commited —> Final committed
  - 验证者附加2F+1个提交签名到extraData字段，将区块插入区块链末尾
  - 区块插入成功后验证者进入Final committed状态
- Final Commited —> New round
  - 验证者选举新的proposer并启动新一轮共识计时器

### 3. Round change工作流

- 以下三种情况将会触发 Round change
  - Round change 计时器超时
  - 无效 prepared 消息
  - 区块插入失败
- 当一个验证者发现符合以上三种情况任一种时，就会将新的轮数附加到 round change消息上广播出去，然后等待来自其它验证者的 round change消息。附加的新的轮数根据以下条件确定
  - 如果验证者收到来自其它节点的 round change 消息，它会从数量超过F+1个的轮数消息中选择最大的那个轮数
  - 否则，将 当前轮数+1 作为新的轮数
- 无论何时，验证者一旦收到同一个轮数上的F+1个 round change 消息，它就会比较收到的轮数和自己的轮数，如果收到的更大，验证者会以收到的轮数再次广播 round change 消息
- 一旦收到了同一个轮数上的 2F+1 个round change消息，验证者就会退出round change循环，选举新的proposer，然后进入new round状态
- 另一个验证者跳出轮数改变循环的情况是它通过节点同步收到了已验证的区块

### 4. Proposer选择

当前支持两种策略：round robin和sticky proposer

- round robin：每一次区块和轮数更改都会换一个proposer
- sticky proposer：只有轮数改变时才会更改proposer

### 5. 验证者列表投票

IBFT使用一个和Clique相似的验证者投票机制，每个epoch交易都会重置验证者投票，这意味着当授权或取消授权投票过程正在进行，投票过程将中止。

对所有交易区块而言

- proposer可以投一票提议更改验证者列表



未完待续



## 问题

- [Istanbul BFT's design cannot successfully tolerate fail-stop failures #305](https://github.com/jpmorganchase/quorum/issues/305)
- [Correctness Analysis of Istanbul Byzantine Fault Tolerance](https://arxiv.org/pdf/1901.07160.pdf)

- 没有激励机制
