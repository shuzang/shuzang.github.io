# 研究记录7-原始论文复现


本文是在前面搭建好的区块链实验平台基础上，对 Smart contract-based access control for the internet of things[^1] 这篇论文提出的访问控制方案进行复现，记录整个复现和测试的过程。

<!--more-->

[^1]:Zhang Y, Kasahara S, Shen Y, et al. Smart contract-based access control for the internet of things[J]. IEEE Internet of Things Journal, 2018.

## 1. 智能合约

论文中提出的访问控制系统主要包括三种合约：Register Contract（RC，注册合约），Access Control Contract（ACC，访问控制合约）和 Judge Contract（JC，判决合约）。复现的代码已放在了我的 github 仓库，仓库名为 [BBRAC:truffle-zhang](https://github.com/shuzang/BBRAC/tree/truffle-zhang)，其中 `truffle-zhang` 是其所在分支。下面介绍三种合约的作用及其中定义的功能。

### 1.1 注册合约

RC 的作用主要是注册 ACC 和 JC，在需要的时候返回它们的相关信息，使用了一个 lookupTable 来存储每个已注册合约的基本信息，这些基本信息包括：

| 字段名    | 解释                               |
| --------- | ---------------------------------- |
| scName    | 注册的合约名                       |
| subject   | subject-object 对中的 subject 地址 |
| object    | subject-object 对中的 object 地址  |
| creator   | 合约创建者地址                     |
| scAddress | 合约地址                           |

使用了一个映射结构来构造该信息表，`mapping(bytes32=>Method) public lookupTable`，以上基本信息均作为结构体 Method 的一部分，作为表的值。表的键是 bytes32 类型，是方法(method)名，这里方法的含义很模糊，作用应该是描述已注册合约的作用。RC 合约定义了如下功能：

1. methodRegister：ACC和JC注册
2. methodScnameUpdate：已注册的合约名字段更新
3. methodAcAddressUpdate：已注册的合约地址字段更新
5. methodNameUpdate：方法名更新
6. methodDelete：方法删除
7. getContractAddr：获取方法名对应的合约地址

存在的问题

1. 作为表键的方法名除了带来大量的 gas 消耗没有任何益处，反而由于处理的难题，使用内联汇编带来了大量 gas 消耗。可以用注册合约的合约地址作为键，而在 Method 结构体中新增合约描述字段
2. 结构体中合约名也没有意义，可与合约描述字段合并，或者更改为合约类型字段，用于区分 ACC 和 JC
3. 存储注册合约的 abi 没有意义，反而带来处理上的难题和大量 gas 消耗，实际上，实验中存在的 abi 只有三种，RC，ACC 和 JC，预先记录即可，如果考虑到ACC 或 JC 合约内容变动，可设置指向函数，在改变合约时将请求指向新合约。

如果解决这些问题，属于实现上的优化，目的在于减少 Gas 消耗和提高可操作性。

### 1.2 访问控制合约

ACC 的作用是对 subject 和 object 对的每个访问控制方法做描述。拥有两个表，Misbehavior表 记录行为用作判决，PolicyItem表 记录访问控制策略用于访问控制。并使用了一个事件返回访问控制相关结果。合约相关函数如下：

1. setJC：设置 JC 合约地址，用于调用 JC 合约
2. policyAdd：策略添加
3. getPolicy：策略查询
4. policyUpdate：策略权限字段更新
5. minIntervalUpdate：策略时间间隔字段更新
6. thresholdUpdate：策略阈值字段更新
7. policyDelete：策略删除
8. accessControl：实施访问控制
9. getTimeofUnblock：获取惩罚时间
10. deleteACC：合约自我销毁

ACC由 object 部署，可以控制 subject 发起的访问请求。我们假设 subject 和 object 就多条访问控制方法达成了一致，因此，一个 subject-object 对应多个ACC，但一个 ACC 只对应一个 subject-object。

### 1.3 判决合约

JC 的表结构记录了恶意行为，并拥有恶意行为判决函数，同时有一个事件返回判决基本信息，其函数如下：

1. misbeaviorJudge：进行恶意行为判决
2. getLatestMisbehavior：获取最新判决的一个恶意行为
3. deleteJC：合约自我销毁

## 2. 合约功能测试

这一部分对合约功能进行测试，由于使用 Remix 编辑器编写代码，为了操作方便，测试的过程也在 Remix  中完成。我们选择的测试方法是按照下面的步骤自己部署合约及调用相关的函数，要注意的是，Remix 提供了编写相关单元测试所需的库，可以利用这些库额外编写几个自动化的测试合约，但是，这些测试合约只能测试单个的函数，而我们需要测试的功能往往需要按顺序调用多个函数，因此不可行。

### 2.1 注册

注册新的访问控制方法/恶意行为判决方法

step1：为新方法创建新的 ACC

step2：部署新创建的 ACC 到区块链

step3：调用 RC 合约的 methodRegister 方法注册 ACC

### 2.2 更新

更新已有的访问控制方法（不觉得有存在的必要）

step1：创建新的 ACC，用于替换旧的

step2：部署新创建的 ACC

step3：调用 RC 合约的 methodUpdate 方法更新相关字段，如 ScName, ScAddress, ABI 等

step4：调用旧 ACC 的 deleteACC 方法自我销毁

### 2.3 删除

删除已有的访问控制方法

step1：调用 RC 合约的 methodDelete 方法从注册表删除方法的相关信息

step2：调用 ACC 合约的 deleteACC 方法自我销毁

### 2.3 策略管理

添加，更新和删除策略，主要调用 ACC 合约的 policyAdd, policyUpdate 和 policyDelete 方法完成

### 2.4 访问控制

假设 subject 和 object 知道它们间所有可用的访问控制方法，当前有一个 subject 想访问 object 的资源，需要执行下列步骤

step1：subject 调用 RC 合约的 getContract 方法检索用于访问控制的 ACC

step2：RC 合约返回所查询 ACC 的 address 和 ABI 给发起查询的 subject

step3：subject 调用 ACC 合约的 accessControl 方法发起访问控制，该交易将被收集到区块中，区块被确认后，accessControl 方法得以执行

step4：访问控制执行期间，ACC 调用 JC 合约的 misbehaviorJudge 方法查看该 subject 是否存在恶意行为

step5：misbehaviorJudge 完成恶意行为检测与判决，将判决结果返回给 ACC

step6：访问控制结束后，结果同时返回给 subject 和 object

## 3. 实施访问控制

功能测试完成后，我们模拟 subject 向 object 发起访问控制请求的场景，验证所设计的系统的可行性。实际实施以 pi3B+ 作为 subject，以 pi3B 作为 object，实验流程如下

1. 在台式电脑中安装[web3.js](https://web3js.readthedocs.io/en/v1.2.1/getting-started.html)，之后通过websocket远程操作树莓派

   ```bash
   # 安装npm和node
   $ sudo apt-get install npm
   # 更新npm到最新
   $ sudo npm install npm@latest -g
   # 安装node管理工具
   $ sudo npm install n -g
   # 更新node.js到最新
   $ sudo n lts
   # 授权普通用户否则npm无法对模块进行本地安装
   $ sudo chown -R $(whoami) ~/.npm
   $ sudo chown -R $(whoami) ~/.config
   # 安装web3.js
   $ npm install web3@1.0.0-beta.18
   # 查看已安装本地模块列表
   $ npm list --depth 0
   /home/shuzang/istanbul
   └── web3@1.0.0-beta.18
   ```

   不建议直接使用`npm install web3`，默认会安装web3@1.2.1版本，函数调用出现了`Transaction has been reverted by the EVM`的问题，[论坛](https://github.com/ethereum/web3.js/issues/2518)提到这是版本问题，切换了不少版本，但直到1.0.0-beta.18才执行成功。已安装高级版本情况下可以使用如下命令覆盖
   
   ```bash
   $ npm install web3@1.0.0-beta.18 --save
   ```
   
2. 在 node0 根目录启动 geth

   ```bash
   $ ps | grep geth
   $ killall -INT geth
   $ ./startall.sh
   $ geth attach data/geth.ipc
   ```

    解锁账户（之后执行操作脚本都有实现解锁账户，默认账户解锁维持5分钟）

    ```json
    > personal.unlockAccount(eth.accounts[0])
    ```

3. 使用 Remix 编译注册合约(RC)，并在`Compilation Details`中获取`WEB3DEPLOY`，复制并粘贴在 geth 控制台运行，记录返回的交易哈希和合约地址。

4. 以同样的方法部署判决合约(JC)，记录返回的交易哈希和合约地址。合约初始化需要的两个参数设置为

   - base = 2
   - interval = 3

5. 调用 RC 合约的 methodRegister 函数，注册判决合约

6. 在 object 部署 ACC 合约，合约初始化参数为 subject 节点的账户地址，记录返回的交易哈希和账户地址。

7. 调用 ACC 合约的 SetJC 函数，传入的参数为 JC 合约地址，用于进行合约调用

8. 调用 ACC 合约的 policyAdd 函数预定义一批访问控制策略，传入的参数中令时间间隔`minInterval=100`，阈值`threshold=2`

    | Resource  | Action  | Permission | ToLR                   |
    | --------- | ------- | ---------- | ---------------------- |
    | file A    | read    | allow      | 例如：2019-10-04 10:47 |
    | file A    | write   | deny       |                        |
    | program A | execute | deny       |                        |
    | ...       | ...     | ...        | ...                    |

9. 调用 ACC 合约的 getPolicy 函数验证定义的策略

10. 调用 RC 合约的 methodRegister 函数，注册访问控制合约

11. 编写 object 下的 monitor.js，用于监听访问控制请求

    ```js
    var Web3 = require('web3');
    var web3 = new Web3(Web3.givenProvider || "ws://192.168.191.4:8545");

    var rcAbi = []
    var accAbi = " "

    var rcAddr = "0xa49fe05a90c49c44b7d533c64b8cc33e5e6d582e";

    var methodName = "Access Control";
    var register = new web3.eth.Contract(rcAbi, rcAddr);
    register.methods.getContractAddr(methodName).call({
        from: "0x44d13e0c0d91a2ebe570c58cdadef2b99bf55bc1",
        gas: 10000000
    },function(error,result){
        if(!error) {
            listen(result);
        }
    });
    
    function listen(accAddr) {
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
    }
    
    ```

11. 编写 subject 下的 requester.js，用于发起访问控制。

    ```js
    var Web3 = require('web3');
    var readline = require('readline');
    var web3 = new Web3(Web3.givenProvider || "ws://192.168.191.3:8545");
    
    var rcAbi = [];
    var accAbi = [];
    
    var subject = "0x9abf7020cc405fce60fdfb84168fb9457bde52e2";
    var rcAddr = "0xa49fe05a90c49c44b7d533c64b8cc33e5e6d582e";
    
    var methodName = "Access Control";
    var register = new web3.eth.Contract(rcAbi, rcAddr);
    register.methods.getContractAddr(methodName).call({
    	from: "0x9abf7020cc405fce60fdfb84168fb9457bde52e2",
    	gas: 10000000
    },function(error,result){
    	if(!error) {
    		sendAccessControl(result);
    	}
    });
    
    function sendAccessControl(accAddr) {
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
    		var currentTime = new Date().getTime()/1000;
    		myACC.methods.accessControl("File A", "read", currentTime).send({
    				from: "0x9abf7020cc405fce60fdfb84168fb9457bde52e2",
    				gas: 10000000
    			},function(error,result){
    				if(!error){
    					currentTxHash = result
    					console.log("currentTxHash", result)
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
    }
    ```

12. 将注册合约(RC)的合约地址和 Abi，访问控制合约的Abi填充到上面两个 js 文件的rcAddr，rcAbi和accAbi变量

13. 在台式电脑中开启两个控制台界面，先后执行两个脚本。requester.js 发起的访问控制返回的结果，monitor.js 都能检测到。

    ```bash
    $ node monitor.js
    $ node requester.js
    ```
    

## 4. 附录

### 附录1 两个js脚本的算法

> Algorithm 2 Access Request JavaScript
>
> **Input**: resource, action, time
>
> **Output**: result, penalty
>
> 1: Create a RC instance register.  
> 2: Specify the access control method name method.  
> 3: (addr, abi)4— register.getContract(method).  
> 4: Create an ACC instance acc with addr, abi.  
> 5: Send a transaction containing parameters (resource, action, time) to the accessControl ABI of acc.  
> 6: while ture do  
> 7: 	if Event returnResult() is captured then  
> 8: 		(result, penalty) 4— returnResult().  
> 9: 		break.  
> 10: 	end if  
> 11: end while  
> 12: return result, penalty  
<br>

> Algorithm 3 Access Monitor JavaScript
>
> 1: Create a RC instance register.  
> 2: Specify the access control method name method.  
> 3: (addr, abi)4— register.getContract(method).  
> 4: Create an ACC instance acc with addr, abi.  
> 5: while ture do  
> 6: 	if Event returnResult() is captured then  
> 7: 		(result, penalty) 4— returnResult().  
> 8: 		Display result, penalty.  
> 9: 	end if  
> 10: end while

### 附录2 web3.js与节点交互的参考文档

https://blog.csdn.net/dieju8330/article/details/83090660

https://blog.csdn.net/dieju8330/article/details/83149164

### 附录3 可能遇到的问题及解决方案

获取节点账户列表

```js
> eth.accounts
```

解锁账户

```js
> personal.unlockAccount(eth.accounts[0])
# 可以解锁时直接添加密码和持续时间(未添加持续时间默认300s，即5分钟)
> personal.unlockAccount(address, "password", 300)
```

查看已连接节点

```js
> admin.peers
```

启动geth时即解锁账户

```js
geth --unlock "0x..." --password "密码"
```

启动geth时允许远程web3接入

```js
geth --rpccorsdomain "*"
```

remix通过web3 provider部署合约，网址栏https改为http，否则无法连接

更多关于账户的命令参见 [Managing your accounts](https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts)

### 附录4 合约部署的方式

1. 复制 Remix 中合约编译生成的 web3deploy 到 geth 控制台执行
2. 使用 Remix 的 web3 Provider 远程部署（最方便）
3. 利用 web3.js 编写 js 脚本进行部署
4. 使用 truffle

### 附录5 执行js命令的方法

 **方法1**：geth控制台执行以下命令，默认路径为执行`geth attach`的路径

```js
loadScript('requester.js')
```

 但总出现类似“err: TypeError: 'send' is not a function”的问题，无法解决，可能是`web3.js`的问题，网上有人建议换用`ethers.js`

 **方法2**：执行`geth attach`时添加参数

```js
geth --exec 'loadScript("request.js")' attach data/geth.ipc
```

使用举例见[JavaScript Console](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console)

 **方法3**：使用node直接运行

```bash
$ node requester.js
```

方法3是唯一执行成功的。
