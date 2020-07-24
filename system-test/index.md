# 区块链实验9-系统测试


所有的准备工作完成后，本篇进行系统测试。测试分为三部分

1. 隐私合约及交易测试
2. 访问控制系统测试
3. 信誉系统测试

<!--more-->

## 1. 隐私功能测试

隐私合约及交易是 Quorum 自带的功能，本身不是我们实现的，因此测试只是验证该功能是否启用。合约的部署与交互使用了 Quorum for Remix 插件，测试用的合约如下

```js
pragma solidity ^0.5.4;
contract SimpleStorage {

    uint storedData;

    event Change(string message, uint newVal);

    constructor(uint initVal) public {
        emit Change("initialized", initVal);
        storedData = initVal;
    }

    function set(uint x) public {
        emit Change("set", x);
        storedData = x;
    }

    function get() view public returns (uint retVal) {
        return storedData;
    }

}
```

我们令Node2 代表农场，Node3 代表超市，假设农场部署了一个私有存储合约，状态只能被超市查看。农场部署合约传入一个初始值 50，理论上，Node2 和 Node3 可以通过调用 get() 函数获取到该数据，其它的节点无法查看该数据。

因此我们分别进入 Node3（超市） 和 Node4 的 Geth console 进行验证，如下图所示，左侧是 Node3，可以查看合约状态并获取数据，右边是 Node4，无法查看合约状态，也无法获得数据。

![私有合约状态测试](/images/区块链实验9-系统测试/私有合约状态测试.png)

从 Cakeshop 区块链浏览器可以更清楚地看到两种情况

![节点3可以查看私有合约](/images/区块链实验9-系统测试/节点3可以查看私有合约.png)

![节点4无法查看私有合约](/images/区块链实验9-系统测试/节点4无法查看私有合约.png)

## 2. 访问控制时间测试

主要测试参数为完成一次访问控制的时间，期间我们要确认信誉系统的加入是否对访问控制时间有影响，以及不同的访问控制方案是否对时间有影响。

第一部分隐私功能测试时使用了 Quorum for Remix 插件，由于该插件在 Remix 中不会返回结果，在非隐私交易时不具备优势，因此访问控制系统时间测量的预准备工作，包括合约部署和交互，使用了 Remix 自己提供的 Deploy and Run 插件，主然后利用 Web3 Provider 连接 Quorum 网络。连接端口在 geth 启动时已默认打开，我们使用的三个节点对应的 Web3 端口如下

- Node1：22000
- Node2：22001
- Node3：22002

另外，我们单独安装 Web3.js 并编写 JS 脚本来进行访问控制和时间统计

```bash
# 在用户根目录建立web3文件夹
$ mdkir web3
$ cd web3
# 在web3文件夹中本地安装 web3 1.2.8 版本，之前的版本有些部件不再维护，安装会出错，1.2.9某些功能出错
$ npm install web3@1.2.8
```

我们尝试了直接在 JS 脚本中设立循环，使用 setInterval() 函数延时 120s 发起访问请求，但这种方式得到的访问控制时间存在较大的误差，第一次循环执行结果普遍在 10ms 左右，其后迅速减少，在 1ms 和 2ms 浮动，目前不清楚原因，可能有缓存。

最终我们使用的方法是编写 Shell 脚本，在其中循环调用 JS 文件。

完成访问控制的 JS 脚本和循环调用 JS 文件的 Shell 脚本放在了 git 仓库中。执行 Shell 脚本之前，我们需要先解锁发起访问的账户，由于我们会一次性进行数十上百次测试，因此一次性将账户解锁 1小时以上，这里我们设置 4000s。解锁相应账户的命令如下，第二个参数为密码，第三个参数为时间。

```js
> personal.unlockAccount(eth.accounts[0],"",3600)
> personal.unlockAccount(eth.accounts[1],"",3600)
```

注意，设备发起访问控制时，应当首先获取目标设备绑定的访问控制合约地址，该地址可以根据设备账户在管理合约中查询得到，但这里我们为了测试方便，选择预定义，而不是每次去查询。

### 2.1 无信誉系统

完整的方案中包含 MC（管理合约）、ACC（访问控制合约）和RC（信誉合约），现在，我们将 RC 从系统中移除，并删除 MC 和 ACC 中所有相关的调用。

JS 文件基本逻辑如下，在发起访问前设置开始时间，获得结果后获得结束时间，然后输出时间差，该时间差作为最终的访问控制时间。

```js
var Web3 = require('web3');
var web3 = new Web3(Web3.givenProvider || "ws://localhost:23000");

var accAbi = "这里是合约ABI"
var accAddr = "0xFe0602D820f42800E3EF3f89e1C39Cd15f78D283"

var myACC = new web3.eth.Contract(accAbi, accAddr);

var date1 = new Date();
var a = date1.getTime()

myACC.methods.accessControl("0xb7D603ef60d92cA87b9e7FfE2952F22b8433bF95","switch", "on").send({
	from: "0xfd441BCa68Ed8eCd6d5d4F5C173251a25E42316a",
	gas: 10000000,
	gasPrice: 0
}).then(function(receipt){
	if (receipt.status) {
	    console.log("Return Message: Access allow!")
        var date2 = new Date();
        var b = date2.getTime()
        var c = b-a
        console.log("Begin-End: "+ a +"-"+b)
        console.log("Time Cost: " + c + "ms")
        process.exit(0);        
	}
})
```

shell 脚本的逻辑如下

```shell
#!/bin/bash
 
echo "Note: Every 10s, send an access request"

for((i=1;i<=500;i++));  do   
  echo -e "\nThe $i-th request" 
  node requester_legal.js
  sleep 10
done  
```

合约部署时得到的结果如下

| 合约 | Gas 消耗 |
| ---- | -------- |
| MC   | 1958457  |
| ACC  | 4128313  |

合约部署完毕后定义属性和策略，最后根据情况修改 JS 和 Shell 脚本，最后得到 398 次访问时间的平均值为  4490.5ms。

### 2.2 信誉系统加入

加入信誉合约 RC 后进行测试，部署合约、注册设备、定义属性和策略。三种合约部署的 Gas 消耗分别为

| 合约 | Gas 消耗 |
| ---- | -------- |
| MC   | 2512367  |
| ACC  | 4749983  |
| RC   | 1170616  |

本次测试，我们设定两次访问请求间隔 120s，因为两次访问控制间隔少于 100s 会触发 Too frequent request 错误。这是我们自定义的一种恶意行为，为了获取足够的访问控制时间测量样本，我们设置延时。

由于我们设置了每次访问请求延时 120s，时间占用比较高，因此只测试 30 个值，测试前解锁相应的账户

```js
> personal.unlockAccount(eth.accounts[0],"",3600)
> personal.unlockAccount(eth.accounts[1],"",3600)
```

一个示例输出如下，30次测试结果的平均值为 4481ms。

```bash
$ ./runscript.sh
Note: Every 2 minutes, send an access request

The 1-th request
First Account Return Message: Access authorized
Begin-End: 1592883538991-1592883543058
Time Cost: 4067ms
...
```

### 2.3 其它方案

在下面的论文中，wang 等利用智能合约对传统 ABAC 架构进行了实现，我们认为对比该方案和我们的方案具有较大的意义。

```
P. Wang, Y. Yue, W. Sun, and J. Liu, 
“An Attribute-Based Distributed Access Control for Blockchain-enabled IoT,” 
in 2019 WiMob, Barcelona, Spain, Oct. 2019, pp. 1–6, doi: 10.1109/WiMOB.2019.8923232 .
```

鉴于作者没有提供源码，我们按照论文的描述进行了复现，然后在我们当前的实验平台下测试其访问控制时间，合法的访问测试了500次，平均时间为 4527.4ms，非法的访问测试了 500 次，平均时间为 4529.4 ms。

我们对三种情况的时间总结如下表

| 单位/ms | 无信誉系统 | 加入信誉系统 | wang的方案 |
| ------- | ---------- | ------------ | ---------- |
| 平均值  | 4490.5     | 4481         | 4529.4     |

可以看到，三种方案的平均访问控制时间很相似，没有较大的区别。另外，我们还验证了三种方案下访问请求通过和被拒绝两种情况，时间同样没有差别，也在 4500ms 左右。

三种方案的主要区别在于合约逻辑的不同，包括循环的执行、合约间的相互调用次数，尤其是 wang 的方案合约间相互调用比较多。从结果来看，访问控制的时间显然不受这些因素影响。为了验证该结果，我们写了两个关于存储的合约进行测试，合约内容如下

```
pragma solidity >=0.4.22 <0.7.0;

/**
 * @title Storage
 * @dev Store & retreive value in a variable
 */
contract Storage {
    Helper public h;
    uint256 num;
    
    constructor(address _h) public {
        h = Helper(_h);
    }
    
    function store(uint256 _num) public {
        num = _num;
    }
    
    function get() public view returns (uint256){
        return num;
    }
    
    function retreive() public view returns (uint256){
        uint number;
        for (uint i = 0; i < 500; i++) {
            uint t = h.getNumber();
            number = number + t;
        }
        return number;
    }
}

contract Helper {
    function getNumber() public pure returns (uint256);
}
```

调用的 Helper 合约具体内容如下

```
pragma solidity >=0.4.22 <0.7.0;

contract Helper {
    function getNumber() public pure returns (uint256){
        return 23;
    }
}
```

我们调用 Storage 合约中的 store 函数查看对合约状态进行更改时的时间消耗，调用 get 函数查看不更改合约状态时的时间消耗，调用 retreive 函数查看多次调用其它合约时对时间的影响。得到的结果如下

1. 调用的函数对合约状态有影响和没影响两种情况下时间消耗是不同的；
2. 同种情况下（都更改合约状态或不更改），合约逻辑对时间消耗没有影响，这里的合约逻辑只循环次数和合约调用次数，在测试中，调用其它合约1次和调用500次结果没有区别。

### 2.4 意外

我们在测试时发现一个意外情况，访问时间会受发起访问的时机的影响，假设 t 为两次访问的间隔，那么区别如下

|          | t % 5 == 0 | t % 5 == 1 | t % 5 == 2 | t % 5 == 3 | t % 5 == 4 |
| -------- | ---------- | ---------- | ---------- | ---------- | ---------- |
| 结果趋向 | 4500ms     | 3500ms     | 2500ms     | 6500ms     | 5500ms     |

三种方案都会出现如上结果，事实上，访问时间不仅收发起访问控制的时机影响，为了排除后台进程占用对时机的影响，我们特地利用程序分别使 CPU 占用保持在 50%，80%和100%重新进行了测试，不同的CPU占用率下，访问时间也不同，具体如下

|             | t % 5 == 0 | t % 5 == 1 | t % 5 == 2 | t % 5 == 3 | t % 5 == 4 |
| ----------- | ---------- | ---------- | ---------- | ---------- | ---------- |
| CPU占用20%  | 4500ms     | 3500ms     | 2500ms     | 6500ms     | 5500ms     |
| CPU占用50%  | 4300ms     | 3300ms     | 2300ms     | 6300ms     | 5300ms     |
| CPU占用80%  | 4100ms     | 3100ms     | 2100ms     | 6100ms     | 5100ms     |
| CPU占用100% | 3950ms     |            |            |            |            |

最后，我们特定将 CPU 限制在了单核单线程，得到的结果没有区别。

将所有的结果总结如下

1. 对合约的调用操作，分为不改变合约的状态和会改变合约的状态两种情况，这两种情况调用产生的时间消耗不同；
2. 访问时间属于更改合约状态这种情况，此时，访问时间不因程序逻辑的变化而变化，意思是循环数量、合约间相互调用的数量，都不会对访问消耗时间产生影响；
3. 访问通过和被拒绝属于合约逻辑问题，同样不对访问时间产生影响；
4. 访问时间受网络情况影响，包括发起访问的时机和后台CPU的占用率；

## 3. 信誉系统功能测试



未完待续...