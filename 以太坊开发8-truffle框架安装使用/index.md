# 以太坊开发8-Truffle框架安装使用


Truffle和Ganache的安装使用

### 1. 安装Truffle

在Ubuntu18.04 下安装运行，要求Node.js版本高于v8.9.4

```bash
$ sudo apt-get install npm
$ sudo npm install npm@latest -g
$ sudo npm install n -g
$ sudo n lts
```

安装Truffle

```bash
$ sudo npm install -g truffle
```

### 2. 创建项目

以[Truffle Boxes](https://www.trufflesuite.com/boxes)中的[MetaCoin](https://www.trufflesuite.com/boxes/metacoin)项目为例

首先为 Truffle 项目创建新目录

```bash
$ mkdir MetaCoin && cd MetaCoin
```

下载MetaCoin box

```bash
$ truffle unbox metacoin

✔ Preparing to download
✔ Downloading
✔ Cleaning up temporary files
✔ Setting up box

Unbox successful. Sweet!

Commands:

  Compile contracts: truffle compile
  Migrate contracts: truffle migrate
  Test contracts:    truffle test
```

下载完成后的项目目录如下

```bash
$ tree
.
├── contracts
│   ├── ConvertLib.sol
│   ├── MetaCoin.sol
│   └── Migrations.sol
├── LICENSE
├── migrations
│   ├── 1_initial_migration.js
│   └── 2_deploy_contracts.js
├── test
│   ├── metacoin.js
│   └── TestMetaCoin.sol
└── truffle-config.js

3 directories, 9 files
```

`contracts`是solidity编写的合约存放的文件夹

- `MetaCoin.sol`创建了一种简单的代币，它引用了`ConvertLib.sol`合约
- `Migrations.sol`是一个单独的合约文件，用来管理所部署合约的状态，每个Truffle项目中都有该文件，而且通常不需要更改

`migrations`是部署脚本存放的文件夹

- `1_initial_migration.js`是`Migrations.sol`合约的迁移和部署脚本
- `2_deploy_contracts.js`是`MetaCoin.sol`合约的迁移和部署脚本
- 所有的脚本按文件名开头的序号按顺序执行

`test`是测试应用和合约的测试文件存放的目录

- `TestMetaCoin.sol`是合约形式的测试文件
- `metacoin.js`是JS脚本形式的测试文件
- 两者的功能是一样的，只是使用了不同方式做测试

`truffle.js`是Truffle的配置文件，用于配置网络信息和其它项目相关的信息

### 3. 编译

编译合约

```bash
$ truffle compile

Compiling your contracts...
===========================
> Compiling ./contracts/ConvertLib.sol
> Compiling ./contracts/MetaCoin.sol
> Compiling ./contracts/Migrations.sol
> Artifacts written to /home/shuzang/MetaCoin/build/contracts
> Compiled successfully using:
   - solc: 0.5.8+commit.23d335f2.Emscripten.clang
```

### 4. 测试

在终端中运行合约形式的测试文件

```bash
$ truffle test ./test/TestMetaCoin.sol

Compiling your contracts...
===========================
> Compiling ./contracts/ConvertLib.sol
> Compiling ./contracts/MetaCoin.sol
> Compiling ./contracts/Migrations.sol
> Compiling ./test/TestMetaCoin.sol



  TestMetaCoin
    ✓ testInitialBalanceUsingDeployedContract (83ms)
    ✓ testInitialBalanceWithNewMetaCoin (76ms)


  2 passing (6s)
```

在终端中运行JS脚本形式的测试文件

```bash
$ truffle test ./test/metacoin.js

Compiling your contracts...
===========================
> Compiling ./contracts/ConvertLib.sol
> Compiling ./contracts/MetaCoin.sol
> Compiling ./contracts/Migrations.sol



  Contract: MetaCoin
    ✓ should put 10000 MetaCoin in the first account
    ✓ should call a function that depends on a linked library (60ms)
    ✓ should send coin correctly (154ms)


  3 passing (271ms)
```

### 5. 与Ganache关联使用

为了部署合约我们需要连到区块链，Truffle提供这方面的功能，但是目前我们使用Ganache来模拟区块链做部署测试

首先在[Download](https://github.com/trufflesuite/ganache/releases)页面下载对应操作系统的Ganache软件包，Linux系统下该软件包点击即可启动，无需安装

然后编辑`truffle-config.js`文件，替换为如下内容，这些配置参数将允许Ganache以默认参数连接

```js
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*"
    }
  }
};
```

保存并关闭文件，打开Ganache

![Ganache](https://www.trufflesuite.com/img/docs/ganache/quickstart/accounts.png)

执行迁移命令，将合约部署到Ganache创建的区块链

```bash
$ truffle migrate

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      5777
> Block gas limit: 0x6691b7


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0x58a9f65031802c841183fc99a46d647870a99de5a717e27791f7ea13e9ccd47c
   > Blocks: 0            Seconds: 0
   > contract address:    0x03A587F157700FBdDff5195F22Eeab32c030424e
   > block number:        1
   > block timestamp:     1573040253
   > account:             0x6c9d11d64bDC226d3d143b228d1019cB187c962d
   > balance:             99.99477214
   > gas used:            261393
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00522786 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00522786 ETH


2_deploy_contracts.js
=====================

   Deploying 'ConvertLib'
   ----------------------
   > transaction hash:    0x1685578d778d127196ce3b7d915b44e6433049f2a250e4384f836161cba8b75c
   > Blocks: 0            Seconds: 0
   > contract address:    0x741be30b7E4F5160026A0882f15763E623Fbcd66
   > block number:        3
   > block timestamp:     1573040254
   > account:             0x6c9d11d64bDC226d3d143b228d1019cB187c962d
   > balance:             99.99185922
   > gas used:            103623
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00207246 ETH


   Linking
   -------
   * Contract: MetaCoin <--> Library: ConvertLib (at address: 0x741be30b7E4F5160026A0882f15763E623Fbcd66)

   Deploying 'MetaCoin'
   --------------------
   > transaction hash:    0x8800cc6fecf73a8d247fad63d5e739cccbd5e9f8dd350bbb709eb258396b96e4
   > Blocks: 0            Seconds: 0
   > contract address:    0xBfc5d417c17fc15E837aCA46063F6b5403ad469e
   > block number:        4
   > block timestamp:     1573040254
   > account:             0x6c9d11d64bDC226d3d143b228d1019cB187c962d
   > balance:             99.98509224
   > gas used:            338349
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00676698 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00883944 ETH


Summary
=======
> Total deployments:   3
> Final cost:          0.0140673 ETH
```

输出记录中显示了交易ID，部署的合约地址，总花费，实时状态等诸多信息。

在Ganache中点击`Transactions`按钮可以看到执行的交易的详细信息。

之后，使用Truffle控制台还可以和合约交互

```bash
$ truffle console
truffle(development)>
```

### 6. 与合约交互

truffle v5以后，控制台支持使用`async`或`await`函数，以下是一些合约交互的示例

首先部署合约并获取账户地址

```js
truffle(development)> let instance = await MetaCoin.deployed()
truffle(development)> let accounts = await web3.eth.getAccounts()
```

检查部署合约的账户余额

```js
truffle(development)> let balance = await instance.getBalance(accounts[0])
truffle(development)> balance.toNumber()
10000
```

检查余额值多少ether，合约定义1单位余额值2ether

```js
truffle(development)> let ether = await instance.getBalanceInEth(accounts[0])
truffle(development)> ether.toNumber()
20000
```

向另一个账户发送货币

```js
truffle(development)> instance.sendCoin(accounts[1], 500)
{ tx:
   '0xf392891ee029f6bc6f970b9e5ae91cfd2e92baa4e141f0a94f6c6fbf5942c8a8',
  receipt:
   { transactionHash:
      '0xf392891ee029f6bc6f970b9e5ae91cfd2e92baa4e141f0a94f6c6fbf5942c8a8',
     transactionIndex: 0,
     blockHash:
      '0x14d49b375d485ae98860383d3483174995f019f349e9fdf5fd1d669be35a4791',
     blockNumber: 6,
     from: '0x6c9d11d64bdc226d3d143b228d1019cb187c962d',
     to: '0xbfc5d417c17fc15e837aca46063f6b5403ad469e',
     gasUsed: 51072,
     cumulativeGasUsed: 51072,
     contractAddress: null,
     logs: [ [Object] ],
     status: true,
     logsBloom:
      '0x00008000000000000000000000000000000000000000000400000000000000000400000010000000000000000000000000000000000002000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000010000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000100000000000800000000000',
     v: '0x1b',
     r:
      '0x6ad89e5e88194468c9d8f82891c7839b9c4d92ee01bd689ce080e35e99e74d1d',
     s:
      '0x77f2bc1452799a94cfaec2a9400f85393794ff96b285447a3407371b2d991978',
     rawLogs: [ [Object] ] },
  logs:
   [ { logIndex: 0,
       transactionIndex: 0,
       transactionHash:
        '0xf392891ee029f6bc6f970b9e5ae91cfd2e92baa4e141f0a94f6c6fbf5942c8a8',
       blockHash:
        '0x14d49b375d485ae98860383d3483174995f019f349e9fdf5fd1d669be35a4791',
       blockNumber: 6,
       address: '0xBfc5d417c17fc15E837aCA46063F6b5403ad469e',
       type: 'mined',
       id: 'log_c14235e4',
       event: 'Transfer',
       args: [Result] } ] }
```

检查接收账户的余额

```js
truffle(development)> let received = await instance.getBalance(accounts[1])
truffle(development)> received.toNumber()
500
```

检查发送账户的余额

```js
truffle(development)> let newBalance = await instance.getBalance(accounts[0])
truffle(development)> newBalance.toNumber()
9500
```

### 7. 使用truffle develop

之前部署合约使用了Ganache，也可以使用truffle develop。这一工具是Truffle内置的测试用私链，运行在本地。

运行truffle develop

```bash
$ truffle develop
Truffle Develop started at http://127.0.0.1:9545/

Accounts:
(0) 0xfc166345efd307d6c22bde2cc8f4e4e345c49495
(1) 0x7b06844a7accc66d34294557d98224c39c496c81
(2) 0x49bb2bd5e7506708b1c458d0b6331916180c091b
(3) 0x6ee54be55b95845c3c00cf1fb34cff053b619f44
(4) 0xeb259672501d8a16c954d09a8fdf84291d67cad2
(5) 0xb924f911ae293488043f6cc1d534654213db498f
(6) 0x02ce17094c74a5062ac6a2cd432a4948b7adba26
(7) 0x2e8e85efcefb3e96f09f2ca4d3d4342ef88a0b2c
(8) 0xd773abd9401ac8973cfcf5d8957bde6121347a92
(9) 0x0fcbde05930c09ecac70f9eeddb92d8b7ddec845

Private Keys:
(0) 328523fee9c143c97a93d27c2ef67a37e57143591e584bcbbdc6de71a0f92d8e
(1) 640e66ab0c0b9e775b32d5dfc00d71decc6c96006d71d6729abe336f88ef3c57
(2) c819551e120e1de0785f4476f7221d9d656a3b7c0e0fbce1de4eef986b2384bc
(3) 648892c4e7896ab4c1d26242ecda4de7f39a46cd2822451557c2910ce7b4be7f
(4) 8902380d9dd0dd8d6d97e59c694944a95829776db1dcb61406baec815649cb03
(5) f7d2341f7fbcde003d92e1a8ff3d3972fae00aff6a7568595e7bef03490efde9
(6) e4d11c8069bfaa777f666817d2d5f6b30a33dd75fb960b6e95abfd368286ef37
(7) 7af6c87f895b9c2a661748138a3bee82aff36439bcb43346f0285aa9dfb5aa04
(8) c810feb901d3bbeb57c9fadeba4e9da6bb61d61f1512fdabf5b0363264dd6336
(9) eb21927033d555614606e545e176c9584d8d673db404b70cc8fa4124cbee5438

Mnemonic: merry wrong fruit carry rifle phrase catalog describe mail traffic rate act

⚠️  Important ⚠️  : This mnemonic was created for you by Truffle. It is not secure.
Ensure you do not use it on production blockchains, or else you risk losing funds.

truffle(develop)> 
```

以上显示了生成的十个账户和它们的私钥，可用于之后与区块链的交互。

在出现的`truffle(develop)>`提示符下，可以省略`truffle`前缀而执行truffle的相关命令，依次执行编译和迁移命令

```bash
truffle(develop)> compile

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.

truffle(develop)> migrate

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'develop'
> Network id:      5777
> Block gas limit: 0x6691b7


1_initial_migration.js
======================

   Replacing 'Migrations'
   ----------------------
   > transaction hash:    0x5cf8f2d9154c8c579535788af12bf153cba4dfe5f2e980b774f7a41cfe795089
   > Blocks: 0            Seconds: 0
   > contract address:    0x199Fb2370bb84f3eE5CacfF079D8478b67Cfba08
   > block number:        1
   > block timestamp:     1573044347
   > account:             0xFc166345EFd307D6c22bdE2cC8f4e4E345c49495
   > balance:             99.99477214
   > gas used:            261393
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00522786 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00522786 ETH


2_deploy_contracts.js
=====================

   Replacing 'ConvertLib'
   ----------------------
   > transaction hash:    0x8bb95067ae3d85e519a37fa49feb0bca0b39ecd471f6626399b75c66592d5cb7
   > Blocks: 0            Seconds: 0
   > contract address:    0x3f2dF366aC8C3356516C53B14610DE53309F7CCA
   > block number:        3
   > block timestamp:     1573044347
   > account:             0xFc166345EFd307D6c22bdE2cC8f4e4E345c49495
   > balance:             99.99185922
   > gas used:            103623
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00207246 ETH


   Linking
   -------
   * Contract: MetaCoin <--> Library: ConvertLib (at address: 0x3f2dF366aC8C3356516C53B14610DE53309F7CCA)

   Replacing 'MetaCoin'
   --------------------
   > transaction hash:    0x88213b424d793c5e8d04cce4f34789a38b5c318a3782da511eee51c73fc97da6
   > Blocks: 0            Seconds: 0
   > contract address:    0xBADc6F515922344a53675A348ccBEeec6A96a37a
   > block number:        4
   > block timestamp:     1573044347
   > account:             0xFc166345EFd307D6c22bdE2cC8f4e4E345c49495
   > balance:             99.98509224
   > gas used:            338349
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00676698 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00883944 ETH


Summary
=======
> Total deployments:   3
> Final cost:          0.0140673 ETH
```

接下来同样可以在提示符下直接进行合约交互，而不必退出使用`truffle console`

### 8. Truffle develop和Truffle console

两者都可以用于测试合约交互和手动执行交易，但Truffle develop同时还可以用来发起一个区块链

两者的使用场景为

Truffle Console

- 已经有正在使用的客户端，如Ganache或geth
- 想要部署到测试网络或以太坊主网络
- 想要使用具体的助记符或账户列表

Truffle develop

- 正在测试项目，该项目不会马上部署
- 不需要使用指定的账户，使用默认提供的账户足够
- 不想安装或管理额外的区块链客户端

### 参考

[1] Truffle Quickstart, https://www.trufflesuite.com/docs/truffle/quickstart

[2] Using Truffle Develop and The Console, https://www.trufflesuite.com/docs/truffle/getting-started/using-truffle-develop-and-the-console#truffle-develop


