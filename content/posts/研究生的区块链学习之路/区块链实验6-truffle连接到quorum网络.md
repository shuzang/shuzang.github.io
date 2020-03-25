---
title: 区块链实验6-truffle连接到quorum网络
date: 2019-12-24
tags: [科研记录]
categories: [研究生的区块链学习之路]
slug: "Experiment 9-Connecting Truffle to Quorum"
typora-root-url: ..\..\..\static
---

本文是方案优化部分第三篇，也是最后一篇，介绍合约在Quorum区块链网络中的部署过程和访问控制的测试实现。由于论文复现的时候发现手动配置的复杂性难以言表，这次的优化实验决定使用truffle进行部署测试。

### 1. 账户设置

按设计，Raspberry Pi 3B+是lightnode1，Raspberry Pi 3B是linghtnode2。区块链网络启动后，四个validator各有10<sup>50</sup> wei(10<sup>18</sup>是 1 ether)，但是后面加入区块链网络的两台树莓派是普通节点，余额为0，所以首先由node0向两个账户分别转账1 ether。

```bash
# lightnode1账户地址为："0x77c22157a3b8840d34b8ed5975b5f2597bd6a7a2"
# lightnode2账户地址为："0xa31d40508da63fb00d7e2f4db57c3774384aa299"
```

在node0的geth console中解锁账户，并分别向lightnode1和lightnode2转账1 ether

```js
> eth.accounts
["0xbffe4ff0cbd0a7590fb71966d1e6bb1a4c2359e0"]
> eth.getBalance(eth.accounts[0])
1e+50
> personal.unlockAccount(eth.accounts[0])
Unlock account 0xbffe4ff0cbd0a7590fb71966d1e6bb1a4c2359e0
Passphrase: 
true
> 
> eth.sendTransaction({from:eth.accounts[0], to:"0x77c22157a3b8840d34b8ed5975b5f2597bd6a7a2", value:1*1e18})
"0x3ed3cbc568a64dff3c3fe4a00b87d259d2299953c47e284b19253299eb8c4725"
> eth.sendTransaction({from:eth.accounts[0], to:"0xa31d40508da63fb00d7e2f4db57c3774384aa299", value:1*1e18})
"0xc9e4193164d38d94502960fcc0d1d7c2e22a9f04307145945c4ae525c6d99aee"
> eth.getBalance(eth.accounts[0])
9.9999999999999999999999999999998e+49
```

在lightnode1和lightnode2的geth console执行下列命令查询余额，由结果可知转账成功。

```js
> web3.fromWei(eth.getBalance(eth.accounts[0]), "ether")
1
```

两台树莓派担任的角色是网关，用于管理IoT设备，因此分别在lightnode1和lightnode2中建立新账户，用来代表IoT设备，由网关向各自管理的设备转账10<sup>7</sup> wei，以供使用。

```js
# lightnode1
> personal.newAccount()
Passphrase:
Repeat passphrase:
"0x9a4aa696f85c6bf96733cc5385ccaf2b7ee13f17"
> personal.listAccounts
["0x77c22157a3b8840d34b8ed5975b5f2597bd6a7a2", "0x9a4aa696f85c6bf96733cc5385ccaf2b7ee13f17"]
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x77c22157a3b8840d34b8ed5975b5f2597bd6a7a2
Passphrase:
true
> eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:1*1e7})
"0x35bda063bc28ff3ef68b78962f427e930824390c4d1a8857c98e2dcf70485e17"
> eth.getBalance(eth.accounts[1])
10000000 

# lightnode2
> personal.newAccount()
Passphrase: 
Repeat passphrase: 
"0x016b71d115f1da36de58d2b78369fd3228bef3dd"
> personal.listAccounts
["0xa31d40508da63fb00d7e2f4db57c3774384aa299", "0x016b71d115f1da36de58d2b78369fd3228bef3dd"]
> personal.unlockAccount(eth.accounts[0])
Unlock account 0xa31d40508da63fb00d7e2f4db57c3774384aa299
Passphrase: 
true
> eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:1*1e7})
"0x96bc2b359219c72ff4f2ce910c27d4c76649685a8f9cfd1b879938317c9a1fe1"
> eth.getBalance(eth.accounts[1])
10000000
```

至此账户设置完成，接下来将truffle连接到quorum网络，部署合约

### 2. 安装Truffle

在Ubuntu18.04 下安装运行，要求Node.js版本高于v8.9.4，这里全都升级到了最新

```bash
$ sudo apt-get install npm
$ sudo npm install npm@latest -g
$ sudo npm install n -g
$ sudo n lts
```

安装Truffle

```bash
$ sudo npm install -g truffle
$ truffle version
Truffle v5.1.5 (core: 5.1.5)
Solidity v0.5.12 (solc-js)
Node v12.14.0
Web3.js v1.2.1
```

### 3. 创建项目和基本配置

建立空项目

```bash
$ mkdir AC
$ cd AC
```

初始化项目

```bash
$ truffle init
✔ Preparing to download box
✔ Downloading
✔ cleaning up temporary files
✔ Setting up box
```

项目文件夹中出现相关文件说明成功，此时没有任何合约和测试代码

```bash
$ ls
contracts  migrations  test  truffle-config.js
```

修改`truffle-config.js`文件进行配置，使其关联到已建立的quorum网络

```js
// truffle-config.js
module.exports = {
  networks: {
     development: {
       host: "192.168.191.2",  // Localhost (default: none)
       port: 22000,            
       network_id: "10",       
       gasPrice: 0,
       gas: 100000000,
       type: "quorum"    
     },
     lightnode1: {
       host: "192.168.191.3",   
       port: 22000,            // Standard Ethereum port (default: none)
       network_id: "10",       
       gasPrice: 0,
       gas: 10000000,
       type: "quorum",
       from: "0x9a4aa696f85c6bf96733cc5385ccaf2b7ee13f17",    
       provider: new Web3.providers.WebsocketProvider("ws://192.168.191.3:8545")
     },
     lightnode2: {
       host: "192.168.191.4",  
       port: 22000,            // Standard Ethereum port (default: none)
       network_id: "10",       
       gasPrice: 0,
       gas: 10000000,
       type: "quorum",
       from: "0x016b71d115f1da36de58d2b78369fd3228bef3dd",
       provider: new Web3.providers.WebsocketProvider("ws://192.168.191.4:8545")    
     }
  }
};
```

建立了三个网络`development`、`lightnode1`和`lightnode2`。第一个网络`development`是node0，用来部署RC和JC，第二个网络是lightnode1，用来部署ACC，第三个网络是lightnode2，用来发起访问控制做演示。

因为在最后设置了`provider`，使用websocket进行访问，所以实际上定义的`host`和`port`两个参数是被屏蔽的，主要是因为基于http的远程连接好像已经被启用了，只能使用websocket。

参数的设置主要参考了以下两篇文档

- [Truffle Configuration](https://www.trufflesuite.com/docs/truffle/reference/configuration)
- [Building Dapps for Quorum:Private Enterprise Blockchains](https://www.trufflesuite.com/tutorials/building-dapps-for-quorum-private-enterprise-blockchains)

### 5. 部署

将RC，ACC和JC三个合约放入`contracts`文件夹，然后在项目根目录执行`truffle compile`编译命令，编译所有合约，不过编译命令可以不执行，因为下面的`truffle migrate`无论是否执行过编译都会再检查一遍，如果编译过，就忽略，如果没有编译，会自动执行编译命令。

在`migrations`文件夹新建文件`2_deploy_contracts.js`，用于部署合约，编写内容如下

```js
var Register = artifacts.require("Register");
var Judge = artifacts.require("Judge");
var AccessControl = artifacts.require("AccessControl");

module.exports = function(deployer, network) {
  if (network == "lightnode1") {
    deployer.deploy(AccessControl, Register.address, Judge.address);
  } else {
    deployer.deploy(Register);
    deployer.deploy(Judge, 2, 3);
  }
};
```

首先从node0部署RC和JC两个合约，`truffle migrate`命令默认连接`truffle-config.js`配置中的`development`网络，我们之前已将该网络设置为node0的ip和端口。

注：所有`truffle migrate`和`truffle exec`命令执行前都要先对相应的账户解锁，否则无法成功

```bash
$ truffle migrate
Compiling your contracts...
===========================
> Compiling ./contracts/ACC.sol
> Compiling ./contracts/JC.sol
> Compiling ./contracts/Migrations.sol
> Compiling ./contracts/RC.sol
> Artifacts written to /home/shuzang/AC/build/contracts
> Compiled successfully using:
   - solc: 0.5.12+commit.7709ece9.Emscripten.clang

Starting migrations...
======================
> Network name:    'development'
> Network id:      10
> Block gas limit: 0x6dcd11a0


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0xefb249903f558ccd7b6326f73a886356b55377a4262b34b81f5f9f2940b4347e
   > Blocks: 0            Seconds: 4
   > contract address:    0x4EC4F8BA5aEcA93955f67CFA58dbe4C57b21b37c
   > block number:        2922
   > block timestamp:     0x5e02bcb5
   > account:             0xbfFe4ff0cBd0A7590Fb71966D1E6bb1a4c2359e0
   > balance:             99999999999999999999999999999998
   > gas used:            263741
   > gas price:           0 gwei
   > value sent:          0 ETH
   > total cost:          0 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:                   0 ETH


2_deploy_contracts.js
=====================

   Deploying 'Register'
   --------------------
   > transaction hash:    0xa49e8d423980248e9c03eb52ecd3b46c15209e867285d2bc718a20620f3addd2
   > Blocks: 0            Seconds: 4
   > contract address:    0x8980FC2bBD25958d0c72F5ba5fa3e5faF1A48c05
   > block number:        2924
   > block timestamp:     0x5e02bcbf
   > account:             0xbfFe4ff0cBd0A7590Fb71966D1E6bb1a4c2359e0
   > balance:             99999999999999999999999999999998
   > gas used:            3227866
   > gas price:           0 gwei
   > value sent:          0 ETH
   > total cost:          0 ETH


   Deploying 'Judge'
   -----------------
   > transaction hash:    0xaf538d559ecc72949d40bbc4d1dde67dfa535c7d5a78267c4de414b51fef4bf9
   > Blocks: 0            Seconds: 4
   > contract address:    0x2C2Fb0DD2440e72318Fb018f923F78Ff86541D08
   > block number:        2925
   > block timestamp:     0x5e02bcc4
   > account:             0xbfFe4ff0cBd0A7590Fb71966D1E6bb1a4c2359e0
   > balance:             99999999999999999999999999999998
   > gas used:            1349320
   > gas price:           0 gwei
   > value sent:          0 ETH
   > total cost:          0 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:                   0 ETH


Summary
=======
> Total deployments:   3
> Final cost:          0 ETH
```

然后部署ACC，`truffle migrate`命令指定连接网络`lightnode1`，`--f`指定部署脚本，否则因为之前该脚本已成功执行会略过，而又因为没有新的脚本而没有任何操作。`truffle-config.js`配置中的`lightnode1`网络已设置为raspberry pi 3B+的ip和端口，默认账户设置为第二个账户，也就是新建的用于表示IoT设备的账户。

```bash
$ truffle migrate --network linghtnode1 --f 2
Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'lightnode1'
> Network id:      10
> Block gas limit: 0x67a09e29


2_deploy_contracts.js
=====================

   Deploying 'AccessControl'
   -------------------------
   > transaction hash:    0xf49cca809ad273812de75225dcfe50b624bfe2a93fd2b44653491e8ee0edeb04
   > Blocks: 1            Seconds: 4
   > contract address:    0x05455fa63e5a7cb6575D75c99855cF3A1Adc72b1
   > block number:        3160
   > block timestamp:     0x5e02c15b
   > account:             0x9A4aa696F85C6bF96733Cc5385cCaf2b7ee13f17
   > balance:             0.00000000001
   > gas used:            5263918
   > gas price:           0 gwei
   > value sent:          0 ETH
   > total cost:          0 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:                   0 ETH


Summary
=======
> Total deployments:   1
> Final cost:          0 ETH

```

### 6. 合约交互

尝试了三种方式，但最后只有 truffle console 这种方式真正完成了。

#### ~~6.1 truffle-contract~~

truffle使用[truffle-contract](https://github.com/trufflesuite/truffle/tree/master/packages/contract)接口(原文是contract abstraction，合约抽象)来交互，truffle develop的控制台、migrate写的部署脚本和基于JS的单元测试等都用的是这个抽象接口。

主要方式是编写JavaScript代码，使用提供的抽象接口可以调用已部署合约的函数，无论是发起交易改变合约状态还是仅仅调用获得返回结果。

在项目根目录(和`trufffle-config.js`同文件夹)创建文件`JCRegister.js`，用于注册判决合约，文件内容如下

```js
var Register = artifacts.require("Register");
var Judge = artifacts.require("Judge");

module.export = function(done) {
    console.log("Getting deployed Register contract...")
    Register.deployed().then(function(instance) {
        console.log("Register Judge contract...");
        return instance.contractRegister("Judger", "JC", "0xbffe4ff0cbd0a7590fb71966d1e6bb1a4c2359e0", Judge.address);
    }).then(function(error, result) {
        if(!error) {
            console.log("Transaction:", result.tx);
            console.log("Finished!");
        }
        else
            console.log(error);
    }).catch(function(e) {
        console.log(e);
        done();
    });
};
```

解锁node0账户，然后执行下列命令

```bash
$ truffle exec JCRegister.js
```

`truffle exec`执行错误，错误信息如下

```bash
$ truffle exec JCRegister.js
Using network 'development'.

TypeError: fn is not a function
    at Object.exec (/usr/local/lib/node_modules/truffle/build/webpack:/packages/require/require.js:124:1)
    at /usr/local/lib/node_modules/truffle/build/webpack:/packages/core/lib/commands/exec.js:89:1
    at processTicksAndRejections (internal/process/task_queues.js:93:5)
Truffle v5.1.5 (core: 5.1.5)
Node v12.14.0

```

#### ~~6.2 web3.js~~

所以还是使用web3.js吧，在用户根目录建立web3文件夹，本地安装web3.js模块

```bash
$ cd ..
$ pwd
/home/shuzang
$ mkdir web3 && cd web3
$ npm install web3
$ npm list --depth 0
/home/shuzang/web3
└── web3@1.2.4
```

创建文件`1_Register_JC.js`用于注册判决合约，命名方式参考了truffle的命名方式文件内容如下，ABI过长，本文以省略号代替。

```js
var Web3 = require('web3');
if(typeof web3 !=='undefined'){ //检查是否已有web3实例
    web3=new Web3(web3.currentProvider);
}else{
    //否则就连接到给出节点
    web3=new Web3();
    web3.setProvider(new Web3.providers.WebsocketProvider("ws://localhost:8545"));
}

var rcAbi = [...]

web3.eth.getBlock(0, function(error, result){
	if(!error)
		console.log("connection succeed");
	else
		console.log("something wrong, connection failed");
});


var rcAddress = "0x8980FC2bBD25958d0c72F5ba5fa3e5faF1A48c05";
var jcAddress= "0x2C2Fb0DD2440e72318Fb018f923F78Ff86541D08";

var register = new web3.eth.Contract(rcAbi);
register.options.address=rcAddress;

register.methods.contractRegister("Judger", "JC", "0xbffe4ff0cbd0a7590fb71966d1e6bb1a4c2359e0", jcAddress).send({
	from: "0xbffe4ff0cbd0a7590fb71966d1e6bb1a4c2359e0",
	gas: 10000000
}, function (error, result){
	if(!error){
		console.log('Transaction: ' + result);
		console.log('Finished!');
	}
	else
		console.log(error);
 })

 register.methods.getContractAddr("Judger").call({
	from: "0xbffe4ff0cbd0a7590fb71966d1e6bb1a4c2359e0",
	gas: 10000000
}, function (error, result){
	if(!error){
        console.log('Judge contract address:' + result);
	}
	else
		console.log(error);
 })
```

web3.js执行合约交易不成功，全部陷在交易池了

```js
> txpool.status
{
  pending: 5,
  queued: 0
}
```

日志记录里提示

```bash
VM returned with error                   err="evm: execution reverted"
```

不明白为什么执行不了

#### 6.3 truffle console

truffle的文档里提到与已部署的合约交互可以使用truffle console，故尝试

首先注册判决合约，需要在node0中进行，进入`development`网络

```bash
$ truffle console
```

命令执行完毕后进入`truffle console`控制台，注册合约并查询合约地址进行验证

```js
truffle(development)> Register.deployed().then(function(instance) {instance.contractRegister("Judger", "JC", "0xbffe4ff0cbd0a7590fb71966d1e6bb1a4c2359e0", Judge.address);})
undefined
truffle(development)> Register.deployed().then(function(instance) {return instance.getContractAddr("Judger");})
'0x2C2Fb0DD2440e72318Fb018f923F78Ff86541D08'
```

退出重新执行`truffle console`命令，进入`lightnode2`网络

```bash
truffle(development)> .exit
$ truffle console --network lightnode2
```

进入`lightnode2`的`truffle console`控制台，注册设备相关属性，然后查询属性进行验证

```js
truffle(lightnode2)> let instance = await Register.deployed()
undefined
truffle(lightnode2)> instance.subjectRegister("0x016b71d115f1da36de58d2b78369fd3228bef3dd", "0xa31d40508da63fb00d7e2f4db57c3774384aa299", "thermostat", "subject")
...
truffle(lightnode2)> instance.getAttribute("0x016b71d115f1da36de58d2b78369fd3228bef3dd", "deviceType")
'thermostat'
truffle(lightnode2)> instance.getAttribute("0x016b71d115f1da36de58d2b78369fd3228bef3dd", "deviceRole")
'subject'
```

退出`lightnode2`的控制台，进入`lightnode1`的控制台

```bash
truffle(lightnode2)> .exit
$ truffle console --network lightnode1
```

首先在RC中注册ACC

```js
truffle(lightnode1)> Register.deployed().then(function(instance) {instance.contractRegister("Temperature_Sensor1", "ACC", "0x9a4aa696f85c6bf96733cc5385ccaf2b7ee13f17", AccessControl.address);})
undefined
truffle(lightnode1)> Register.deployed().then(function(instance) {return instance.getContractAddr("Temperature_Sensor1");})
'0x05455fa63e5a7cb6575D75c99855cF3A1Adc72b1'
```

然后在ACC中注册资源属性，并查询属性进行验证

```js
truffle(lightnode1)> let instance = await AccessControl.deployed()
undefined
truffle(lightnode1)> instance.resourceAttrAdd("data", "currentTemperature", "23")
truffle(lightnode1)> instance.getResourceAttr("data", "currentTemperature")
'23'
```

接下来针对已注册的属性，在ACC中设置访问控制策略，并查询策略进行验证

```js
truffle(lightnode1)> instance.policyAdd("data", "read", "subject", "deviceType", "=", "thermostat")
truffle(lightnode1)> instance.getPolicy("data", "read", "deviceType")
Result {
  '0': 'subject',
  '1': 'deviceType',
  '2': '=',
  '3': 'thermostat',
  _attrOwner: 'subject',
  _attrName_: 'deviceType',
  _operator: '=',
  _attrValue: 'thermostat'
}
```

最后发起访问控制请求和监听依然使用web3.js完成，切换到web3目录下，首先新建`test.js`测试是否可使用，测试脚本内容如下

```js
var Web3 = require('web3');

if(typeof web3 !=='undefined'){ //检查是否已有web3实例
    web3=new Web3(web3.currentProvider);
}else{
    //否则就连接到给出节点
    web3=new Web3();
    web3.setProvider(new Web3.providers.WebsocketProvider("ws://localhost:8545"));
};

var connect = function() {
    web3.eth.getBlock(0, function(error, result){
        if(!error)
            console.log("connection succeed");
        else
            console.log("something wrong, connection failed");
    });
}


var getAccount = async function() {
    await connect();
    var account0;
    web3.eth.getAccounts(function(error, result){
        if(!error){
            account0=result[0];
            //console.log(account0);
            console.log("accounts:"+result);
        }
        else{
            console.log("failed to get Accoutns");
        }
    });
}


getAccount().then(function() {
    web3.eth.getBalance("0xbffe4ff0cbd0a7590fb71966d1e6bb1a4c2359e0").then(function(balance) {
        console.log('balance:',balance);
        console.log("test passed!");
    })
})
```

测试结果如下，说明没有问题

```bash
$ node test.js
connection succeed
accounts:0xbfFe4ff0cBd0A7590Fb71966D1E6bb1a4c2359e0
balance: 99999999999999999999999999999998000000000000000000
test passed！
```

在web3目录下新建`requester.js`文件和`monitor.js`文件，前者用来发起访问控制请求，后者用来监听访问控制触发的事件

`requester.js`的内容如下，通过websocket连接lightnode2，发起访问控制请求

```js
var Web3 = require('web3');
var readline = require('readline');
var web3 = new Web3(Web3.givenProvider || "ws://192.168.191.4:8545");

var accAbi = [...];

var accAddr = "0xb29094a4DE9c2E22b598b39fE38860b9117340A6"
var myACC = new web3.eth.Contract(accAbi, accAddr);


var previousTxHash = 0;
var currentTxHash = 0;

var rl = readline.createInterface({
	input: process.stdin,
	output: process.stdout,
	prompt: 'Send access request?(y/n)'
});

rl.prompt();
rl.on('line',(answer) => {
	if('y' == answer) {
	myACC.methods.accessControl("data", "read").send({
			from: "0xbd93271c5d2ccacdc307d1825614d5557ad6e0fd",
			gas: 10000000,
			gasPrice: 0
		},function(error,result){
			if(!error){
				currentTxHash = result
				console.log("currentTxHash: ", result)
			}
		})

	myACC.events.ReturnAccessResult({
			fromBlock: 0
		}, function(error, result){
		if(!error) {
			if(previousTxHash != result.transactionHash && currentTxHash == result.transactionHash) {
				console.log("Contract: "+result.address);
				console.log("Block Number: "+result.blockNumber);
				console.log("Tx Hash: "+result.transactionHash);
				console.log("Block Hash: "+result.blockHash);
				console.log("Time: "+result.returnValues._time);
				console.log("Message: "+result.returnValues._errmsg);
				console.log("Result: "+result.returnValues._result);
				if (result.returnValues._penalty > 0) {
					console.log("Requests are blocked for " + result.returnValues._penalty +"seconds!")
				}
				console.log('\n');
				previousTxHash = result.transactionHash;
				rl.prompt();
			}
		}
	})
	}
	else{
	console.log("access request doesn't send!")
	rl.prompt();
	}
}).on('close',() =>{
	console.log('All actions had executed!');
	process.exit(0);
});
```

`monitor.js`的内容如下，由lightnode1发起，用于监听返回的事件

```js
var Web3 = require('web3');
var web3 = new Web3(Web3.givenProvider || "ws://192.168.191.3:8545");

var accAbi = [...];
var accAddr = "0xb29094a4DE9c2E22b598b39fE38860b9117340A6";
var myACC = new web3.eth.Contract(accAbi, accAddr);

myACC.events.ReturnAccessResult({
	fromBlock: 0
}, function(error, result){
		if(!error) {
			console.log("Contract: "+result.address);
			console.log("Block Number: "+result.blockNumber);
			console.log("Tx Hash: "+result.transactionHash);
			console.log("Block Hash: "+result.blockHash);
			console.log("Time: "+result.returnValues._time);
			console.log("Message: "+result.returnValues._errmsg);
			console.log("Result: "+result.returnValues._result);
			if (result.returnValues._penalty > 0) {
				console.log("Requests are blocked for " + result.returnValues._penalty +"seconds!")
			}
			console.log('\n');
		}
})
```

实验结果如下

![request access authorized](/images/区块链实验6-truffle连接到quorum网络/81cdmj.png)
![monitor access authorized](/images/区块链实验6-truffle连接到quorum网络/81c6pT.png)
![request blocked](/images/区块链实验6-truffle连接到quorum网络/81gC38.png)
![monitor request blocked](/images/区块链实验6-truffle连接到quorum网络/81gK3T.png)

### 7. 错误测试

之前的实验验证的是访问权限被授予的情况，现在测试被拒绝的情况

在lightnode2建立新的IoT设备账户，这次代表摄像头(Camera)，

```bash
> personal.newAccount()
Passphrase: 
Repeat passphrase: 
"0x42b97b26ed5f53693bcc9b58ad8c724718ea0a15"
> personal.listAccounts
["0xa31d40508da63fb00d7e2f4db57c3774384aa299", "0x016b71d115f1da36de58d2b78369fd3228bef3dd", "0x42b97b26ed5f53693bcc9b58ad8c724718ea0a15"]
> personal.unlockAccount(eth.accounts[0])
Unlock account 0xa31d40508da63fb00d7e2f4db57c3774384aa299
Passphrase: 
true
> eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[2],value:1*1e7})
"0x9f36de89d44db23e63e4fa0d3135d1c17e800cca73f0661ae276f20c0e3d1902"
> eth.getBalance(eth.accounts[2])
10000000
```

修改`truffle-config.js`文件中lightnode2网络的执行账户为新建立的账户

```js
...
lightnode2: {
  host: "192.168.191.4",  
  port: 22000,            // Standard Ethereum port (default: none)
  network_id: "10",       
  gasPrice: 0,
  gas: 10000000,
  type: "quorum",
  from: "0x42b97b26ed5f53693bcc9b58ad8c724718ea0a15",
 ...
```

进入`lightnode2`的truffle console

```bash
$ cd AC
$ truffle console --network lightnode2
```

注册新设备的属性

```js
truffle(lightnode2)> Register.deployed().then(function(instance) {instance.subjectRegister("0x42b97b26ed5f53693bcc9b58ad8c724718ea0a15","0xa31d40508da63fb00d7e2f4db57c3774384aa299", "camera", "subject");})
undefined
truffle(lightnode2)> Register.deployed().then(function(instance) {instance.getAttribute("0x42b97b26ed5f53693bcc9b58ad8c724718ea0a15","deviceType");})
'camera'
truffle(lightnode2)> Register.deployed().then(function(instance) {instance.getAttribute("0x42b97b26ed5f53693bcc9b58ad8c724718ea0a15","deviceRole");})
'subject'
```

建立`requester2.js`，内容和`requester.js`相似，只是发起访问控制的是`lightnode2`新建立的`camera`设备账户，也就是说与requester.js的不同仅在于发起访问控制的账户

错误测试的结果如下

![request static check failed](/images/区块链实验6-truffle连接到quorum网络/81gQvF.png)
![monitor static check failed](/images/区块链实验6-truffle连接到quorum网络/81g8b9.png)

### 8. 总结

总结一下本篇中需要做的事情

1. node0部署RC，获取RC合约地址
2. node0部署JC，传入参数base=2、interval=3，获取JC的合约地址
3. JC合约在RC中注册
4. node0分别转给lightnode1和lightnode2 1 ether
5. lightnode1新建IoT设备账户，从第一个账户向该账户转入1000 0000wei
6. lightnode2新建两个IoT设备账户，从第一个账户分别向这两个账户转入1000 0000wei
7. lightnode2的两个IoT设备账户在RC中注册设备属性(事实上所有节点都应注册设备属性，这里是因为实验只需要它们两个发起访问控制)
8. lightnode1的IoT设备账户部署ACC，传入RC和JC的合约地址，获取ACC的合约地址
9. lightnode1在RC中注册ACC
10. lightnode1在ACC中注册资源属性，设置访问控制策略
11. lingtnode2的两个IoT设备通过调用ACC向lightnode1的IoT设备发起访问控制