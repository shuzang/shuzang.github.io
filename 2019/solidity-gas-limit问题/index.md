# Solidity-gas limit问题


在搭建的以太坊私链上进行智能合约部署时，出现了以下问题

```js
INFO [03-21|13:50:11.690] Served eth_sendTransaction               reqid=24 t=684.186µs    err="exceeds block gas limit"
Error: exceeds block gas limit undefined
```

出现该错误的原因如错误描述，是当前合约所需的gas超过了区块的最大gas。这可能与参数gasLimit有关。在创世区块的配置文件中，我们使用了默认的配置值，为`0x2fefd8`，转换为10进制即`3141592`。

注：[在线转换工具](http://tool.oschina.net/hexconvert/)

<!--more-->

### 原因查找

因为部署智能合约之前已经进行过挖矿，区块链中已有数个区块，我们查询目前的gasLimit值。

```js
>eth.getBlock(eth.blockNumber)
{
  difficulty: 131072,
  extraData: "0xd683010900846765746886676f312e3132856c696e7578",
  gasLimit: 3147727,
  gasUsed: 0,
  hash: "0x7e03472bcad02f6e85a3cdb21cfba856da58a4955dd2b6d21e3b8561446ae390",
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  miner: "0x79b43b2196723fff1485999aba45fda3e8b4df58",
  mixHash: "0x10a5130f4ea573f1f1599c11b8ade9ac3feb256c0414db1a277b7b63e8343d48",
  nonce: "0x6bba1166a347ba0f",
  number: 2,
  parentHash: "0xed8c7febfc1ab5e4e388bd886be1182635e77b0047f530c93af4eb31f898bd7c",
  receiptsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  size: 534,
  stateRoot: "0x741c086895803cc3f85c8e7fb738acfb42aa03a12a03edf246b1c14055123b78",
  timestamp: 1552396507,
  totalDifficulty: 263168,
  transactions: [],
  transactionsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  uncles: []
}
```

发现此时区块的gasLimit值为`3147727`。

查找我们部署的合约web3代码

```js
var testContract = web3.eth.contract([{"constant":true,"inputs":[{"name":"a","type":"uint256"}],"name":"multiply","outputs":[{"name":"d","type":"uint256"}],"payable":false,"stateMutability":"pure","type":"function"}]);
var test = testContract.new(
   {
     from: web3.eth.accounts[0], 
     data: '0x6080604052348015600f57600080fd5b5060a58061001e6000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c8063c6888fa114602d575b600080fd5b605660048036036020811015604157600080fd5b8101908080359060200190929190505050606c565b6040518082815260200191505060405180910390f35b600060078202905091905056fea165627a7a72305820028fd57d2fec4df9170b559fe84245ed4f81bc40f3cad3c185c8035501bdb3220029', 
     gas: '4700000'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```

发现合约所需gas为`4700000`，比gasLimit值高，所以部署失败，出现了`Error: exceeds block gas limit undefined`的错误

### 解决办法

第一种解决办法是修改`genesis.json`中的gasLimit参数，设置一个更大的值。但这样做需要重新构建网络，极为繁琐。

另一种解决办法是通过geth命令的`--targetgaslimit`参数来调整gasLimit值

```js
> --targetgaslimit 4712388
```

这里没有调整成功，提示原因是端口在运行，可能和docker有关，不知道怎么解决。

该问题最后是通过调整web3中的gaslimit值解决的，因为这个简单的智能合约怎么看都不像能消耗4700000gas的样子，果然查询之后发现只消耗100000左右，于是将web3代码中的gaslimit调整到120000，重新部署，果然成功。

一个关于gaslimit的解释见：[以太坊Block Gaslimit动态调整机制分析](https://tinycalf.github.io/2018/07/02/20180702-%E4%BB%A5%E5%A4%AA%E5%9D%8A%E5%9D%97gas%E4%B8%8A%E9%99%90%E5%8A%A8%E6%80%81%E8%B0%83%E6%95%B4%E7%A0%94%E7%A9%B6/)




