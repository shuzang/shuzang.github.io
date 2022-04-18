# 以太坊开发2-创世区块文件genesis.json


## 文件说明

关于`genesis.json`的官方说明位于两处

- [Private network](https://github.com/ethereum/go-ethereum/wiki/Private-network)
- [Connecting to the network](https://github.com/ethereum/go-ethereum/wiki/Connecting-to-the-network)

每条区块链都以创世区块开头，而`genesis.json`正是创世区块的配置文件，它是区块链最重要的识别标志之一。实际上，每条区块链的创世区块文件都是唯一的，如果两条机器启动Geth时所选用的创世区块文件不同，就无法被识别为同一条区块链的成员。因此，同一条联盟链/私链中的所有节点必须使用同一份创世区块文件进行初始化配置。

## 标准示例

一个创世区块文件`genesis.json`的标准示例如下：

```bash
{
    "config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "nonce": "0x000000000000002a",
    "difficulty": "0x020000",
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "coinbase": "0x0000000000000000000000000000000000000000",
    "timestamp": "0x00",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "extraData": "0x",
    "gasLimit": "0x2fefd8"
    "alloc": {
        "7df9a875a174b3bc565e6424a0050ebc1b2d1d82": { "balance": "300000" },
        "f41c74c9ae680c1aa78f42e5647a62f353b7bdde": { "balance": "400000" }
    }
}
```

##　参数说明

关于示例文件中各参数说明如下。首先config中的内容是区块链相关的基本配置参数。

- `ChainId` - identifies the current chain and is used for replay protection. You should set it to a unique value for your private chain.
- `homesteadBlock` - your chain won't be undergoing the switch to Homestead, so leave this as `0`.
- `eip155Block` - your chain won't be hard-forking for these changes, so leave as `0`.
- `eip158Block` - your chain won't be hard-forking for these changes, so leave as `0`.

最重要的是链编号`ChainId`，用于标识区块链，关于它更详细的说明见[EIP-155](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md)，Ethereum Improvement Proposals ([EIPs](https://github.com/ethereum/EIPs))是以太坊改进建议，描述了以太坊平台的标准，包括核心协议规范，客户端API和智能合约标准。

- **mixhash** - 一个 256 位的哈希证明，与 nonce 结合使用，证明已经对该块进行了足够的计算：工作量证明（PoW）。 nonce 和 mixhash 的组合必须满足黄皮书 4.3.4  Block Header Validity，(44)中描述的数学条件。它允许验证块确实已经加密地挖掘，因此，从这方面来说，它是有效的。

- **nonce** - 一个64位的哈希证明，与mixhash结合使用，证明在该块上进行了足够的计算：工作量证明（PoW）。  nonce 和 mixhash 的组合必须满足黄皮书 4.3.4  Block Header Validity，(44)中描述的数学条件。它允许验证块确实已经加密地挖掘，因此，从这方面来说，它是有效的。 nonce 是加密安全的挖掘工作证明，证明在确定该令牌值时已经花费了特定量的计算。 （黄皮书, 11.5. Mining Proof-of-Work )。

- **difficulty** -初始挖矿难度，可以根据前一个块的难度和时间戳来调整。难度越高，Miner 必须执行更多计算才能发现有效块。此值用于控制区块链的区块生成时间，将块生成频率保持在目标范围内。在测试网络上，我们将此值保持为较低的值以避免在测试期间等待，因为在区块链上执行交易需要发现有效区块。

- **alloc** - 以太坊账户信息，可以留空，等待部署完成后再启动以太坊创建账户；也可以预先配置好以太坊账户及其余额。这里的账户余额以wei为单位。

- **coinbase** - 挖矿收益账户，是一个160 位地址。它们是采矿奖励本身和合约交易执行退款的总和。通常在规范中命名为 “beneficiary”，有时在在线文档中称为 “etherbase”。可以设置为零地址，留待运行以太坊挖矿之前再设置。

- **timestamp** - 标量值等于此块开始时 `Unix time（）` 函数的合理输出。该机制在块之间的时间方面强制实施稳态。最后两个块之间的较小周期导致难度级别的增加，从而导致找到下一个有效块所需的额外计算。如果周期太大，则减少了难度和到下一个块的预期时间。时间戳还允许验证链内的块顺序（黄皮书，4.3.4。（43））。

- **parentHash** - 整个父块头的 Keccak 256 位哈希（包括其 nonce 和 mixhash）。指向父块的指针，从而有效地构建块链。创世区块中其值为0，实际上创世区块没有这个参数也可以。

- **extraData** - 可选字段，但最多 32 字节，to conserve smart things for ethernity。 

- **gasLimit** - 每个区块所消耗的gas限制。自己做测试时需要设置的高一点，以避免在测试期间受到此阈值的限制。注意：这并不表示我们不应该关注合约的汽gas消耗量。

原始的英文说明如下：

**mixhash** A 256-bit hash which proves, combined with the `nonce`, that a sufficient amount of computation has been carried out on this block: the Proof-of-Work (PoW). The combination of `nonce`and `mixhash` must satisfy a mathematical condition described in the Yellowpaper, 4.3.4. Block Header Validity, (44). It allows to verify that the Block has really been cryptographically mined, thus, from this aspect, is valid.

**nonce** A 64-bit hash, which proves, combined with the mix-hash, that a sufficient amount of computation has been carried out on this block: the Proof-of-Work (PoW). The combination of `nonce`and `mixhash` must satisfy a mathematical condition described in the Yellowpaper, 4.3.4. Block Header Validity, (44), and allows to verify that the Block has really been cryptographically mined and thus, from this aspect, is valid. The `nonce` is the cryptographically secure mining proof-of-work that proves beyond reasonable doubt that a particular amount of computation has been expended in the determination of this token value. (Yellowpager, 11.5. Mining Proof-of-Work).

**difficulty** A scalar value corresponding to the difficulty level applied during the `nonce` discovering of this block. It defines the mining Target, which can be calculated from the previous block’s difficulty level and the timestamp. The higher the difficulty, the statistically more calculations a Miner must perform to discover a valid block. This value is used to control the Block generation time of a Blockchain, keeping the Block generation frequency within a target range. On the test network, we keep this value low to avoid waiting during tests, since the discovery of a valid Block is required to execute a transaction on the Blockchain.

**alloc** Allows defining a list of pre-filled wallets. That’s an Ethereum specific functionality to handle the “Ether pre-sale” period. Since we can mine local Ether quickly, we don’t use this option.

**coinbase** The 160-bit address to which all rewards (in Ether) collected from the successful mining of this block have been transferred. They are a sum of the mining reward itself and the Contract transaction execution refunds. Often named “beneficiary” in the specifications, sometimes “etherbase” in the online documentation. This can be anything in the Genesis Block since the value is set by the setting of the Miner when a new Block is created.

**timestamp** A scalar value equal to the reasonable output of Unix `time()` function at this block inception. This mechanism enforces a homeostasis in terms of the time between blocks. A smaller period between the last two blocks results in an increase in the difficulty level and thus additional computation required to find the next valid block. If the period is too large, the difficulty, and expected time to the next block, is reduced. The timestamp also allows verifying the order of block within the chain (Yellowpaper, 4.3.4. (43)).

**parentHash** The Keccak 256-bit hash of the entire parent block header (including its `nonce` and `mixhash`). Pointer to the parent block, thus effectively building the chain of blocks. In the case of the Genesis block, and only in this case, it’s `0`.

**extraData** An optional free, but max. 32-byte long space to conserve smart things for ethernity. :)

**gasLimit** A scalar value equal to the current chain-wide limit of Gas expenditure per block. High in our case to avoid being limited by this threshold during tests. Note: this does not indicate that we should not pay attention to the Gas consumption of our Contracts.
