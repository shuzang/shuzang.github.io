# 区块链实验9-系统测试


所有的准备工作完成后，本篇进行系统测试。测试分为三部分

1. 隐私合约及交易测试
2. 访问控制系统测试
3. 信誉系统测试

<!--more-->

## 1. 隐私功能测试

隐私合约及交易是 Quorum 自带的功能，本身不是我们实现的，因此测试只是验证该功能是否启用。合约的部署与交互使用了 Quorum for Remix 插件，该插件在 Remix 的插件列表中可以找到。测试用的合约如下

``` js
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

### 2.1 测试准备

第一部分隐私功能测试时使用了 Quorum for Remix 插件，由于该插件在 Remix 中无法返回执行结果，在非隐私交易时不具备优势，因此访问控制系统时间测量的预准备工作，包括合约部署和交互，使用了 Remix 自己提供的 Deploy and Run 插件，主要利用 Web3 Provider 来连接 Quorum 网络进行操作。连接端口在 geth 启动时已默认打开，我们使用的三个节点对应的 Web3 端口如下

- Node1：22000
- Node2：22001
- Node3：22002

另外，我们单独安装 Web3.js 并编写 JS 代码来进行访问控制

```bash
# 在用户根目录建立web3文件夹
$ mdkir web3
$ cd web3
# 在web3文件夹中本地安装 web3 1.2.8 版本，之前的版本有些依赖不再维护，安装会出错
$ npm install web3@1.2.8
```

为了获取足够样本进行分析，我们要进行大量的访问控制测试并获取每次的访问控制时间，先后使用的方案有三种，我们会阐述前两种方案不可行的原因

1. 在 JS 脚本中设立循环，使用 setInterval() 函数延时固定的时间发起访问控制，得到的结果中，初次访问控制的时间为 10ms 左右，其后迅速减少，在 1ms 和 2ms 左右浮动。查询后发现，Javascript 中 setInterval 函数的实质是每隔一段时间向任务队列中添加回调函数，开始执行的时间是不确定的，最后导致了时间统计的不确定性。

2. 使用 Shell 脚本编写循环，在循环中调用 JS 代码，然后使用 sleep 函数设置延时，访问时间的获取是通过 Javascript Date 对象的 getTime 方法，在发起访问控制前获取了一次时间，在获得结果后获取了第二次时间，然后求其差值。最后得到的结果发现，访问控制时间受发起访问的时机影响，呈周期性波动，也受 CPU 占用率的影响，占用率越高时间越短，显然这一结果是不合理的。关于这一次尝试的结果，可以查看第 2.5 节。

3. 使用 Linux time 命令获取执行时间，我们猜测周期性的出现是系统中其它进程的影响，为了排除它们的影响，我们使用了 Linux 的 time 命令。当测试一个程序或比较不同算法时，执行时间是非常重要的，一个好的算法应该是用时最短的。所有类UNIX系统都包含time命令，使用这个命令可以统计时间消耗。例如：

   ```bash
   [root@localhost ~]# time ls
   anaconda-ks.cfg  install.log  install.log.syslog  satools  text
   
   real    0m0.009s
   user    0m0.002s
   sys     0m0.007s
   ```

   输出的信息分别显示了该命令所花费的real时间、user时间和sys时间。

   - real时间是指挂钟时间，也就是命令开始执行到结束的时间。这个短时间包括其他进程所占用的时间片，和进程被阻塞时所花费的时间。
   - user时间是指进程花费在用户模式中的CPU时间，这是唯一真正用于执行进程所花费的时间，其他进程和花费阻塞状态中的时间没有计算在内。
   - sys时间是指花费在内核模式中的CPU时间，代表在内核中执系统调用所花费的时间，这也是真正由进程使用的CPU时间。

   shell内建也有一个time命令，当运行time时候是调用的系统内建命令，应为系统内建的功能有限，所以需要时间其他功能需要使用time命令可执行二进制文件`/usr/bin/time`。所以我们使用 `/usr/bin/time` 获取执行访问控制的时间，然后计算 user 和 sys 的和，得到的结果就是访问控制实际执行所花费的 CPU时间。

参考：[Linux time命令](https://man.linuxde.net/time)

我们首先建立 `xtime` 文件(无后缀)，将如下内容写入

```shell
#!/bin/sh
/usr/bin/time -f '%Uu %Ss %er %P' "$@"
```

`-f` 参数用于指定输出格式，`-f` 后面的几个参数说明如下

| 参数 | 描述                                                         |
| :--- | :----------------------------------------------------------- |
| %e   | real时间                                                     |
| %U   | user时间                                                     |
| %S   | sys时间                                                      |
| %P   | 进程所获取的CPU时间百分百，这个值等于user+system时间除以总共的运行时间。 |

然后建立 shell 脚本，命名为 runscript.sh，内容为

```shell
#!/bin/bash

for ((i=1;i<=500;i++));  do   
  ./xtime node requester_legal.js
  sleep 5
done 
```

其中，requester_legal.js 是完成访问控制的 JS 文件，接下来授予 xtime 和 runscript.sh 执行权限

```bash
$ chmod 777 xtime
$ chmod 777 runscript.sh
```

执行 Shell 脚本之前，我们需要先解锁发起访问的账户，由于我们会一次性进行 500 次测试，每次间隔 5 s，因此一次性将账户解锁 2500s 以上，这里我们设置 4000s。解锁相应账户的命令如下，第二个参数为密码，第三个参数为时间。

```js
> personal.unlockAccount(eth.accounts[0],"",4000)
> personal.unlockAccount(eth.accounts[1],"",4000)
```

执行 runscript.sh 脚本即可开始测试

```bash
$ ./runscript
0.55 0.08 5.02 12%
0.54 0.07 4.96 12%
0.54 0.11 5.00 13%
0.55 0.09 5.02 12%
...
```

所有的时间会输出到终端，如上面的格式，第一列是用户态运行时间，第二列是内核态运行时间，第三列是实际运行时间，最后一列是进程所获取的CPU时间比例。

注1：设备发起访问控制时，应当首先获取目标设备绑定的访问控制合约地址，该地址可以根据设备账户在管理合约中查询得到，但这里我们为了测试方便，选择预定义，而不是每次去查询。

注2：所有的代码文件都放在 github 仓库中。

### 2.2 无信誉系统

完整的方案中包含 MC（管理合约）、ACC（访问控制合约）和RC（信誉合约），现在，我们将 RC 从系统中移除，并删除 MC 和 ACC 中所有相关的调用。

JS 文件基本逻辑如下，可以看到，前面都是变量定义的过程，执行主体是访问控制函数，当获取到 receipt 时退出（返回的事件位于 receipt 中，可以输出确认一下，正式测试时为了输出结果的格式可以只用 receipt.status判断即可）。

```js
var Web3 = require('web3');
var web3 = new Web3(Web3.givenProvider || "ws://localhost:23000");

var accAbi = "这里是合约ABI"
var accAddr = "这里是合约地址"

var myACC = new web3.eth.Contract(accAbi, accAddr);

myACC.methods.accessControl("这里是传入的参数列表").send({
	from: "这里是发起访问的账户地址",
	gas: 10000000,
	gasPrice: 0
}).then(function(receipt){
	if (receipt.status) {
	    console.log(receipt)
        process.exit(0);        
	}
})
```

合约部署时得到的结果如下

| 合约 | Gas 消耗 |
| ---- | -------- |
| MC   | 1958457  |
| ACC  | 4128313  |

合约部署完毕后定义属性和策略，最后根据具体情况修改 JS 和 Shell 脚本，500 次访问时间的平均值为  0.62682s，最大值为 0.99s，最小值为 0.57s。

### 2.3 信誉系统加入

加入信誉合约 RC 后进行测试，部署合约、注册设备、定义属性和策略。三种合约部署的 Gas 消耗分别为

| 合约 | Gas 消耗 |
| ---- | -------- |
| MC   | 2512367  |
| ACC  | 4749983  |
| RC   | 1170616  |

注意，我们在访问控制合约初始化时设定了两次访问控制请求间隔应不少于 100s，否则会触发 Too frequent request 错误。这个设定会极大的延长我们测试的总时间，所以我们把这个时间间隔重新设定为 4s，这样 shell 脚本中的 sleep 5 就不会触发错误。

测试前解锁相应的账户

```js
> personal.unlockAccount(eth.accounts[0],"",3600)
> personal.unlockAccount(eth.accounts[1],"",3600)
```

500 次测试结果的平均值为 0.66736s，最大值为 2.71s，最小值为 0.55s。

### 2.4 wang的方案

在下面的论文中，wang 等利用智能合约对传统 ABAC 架构进行了实现，我们认为对比该方案和我们的方案具有较大的意义。

```
P. Wang, Y. Yue, W. Sun, and J. Liu, 
“An Attribute-Based Distributed Access Control for Blockchain-enabled IoT,” 
in 2019 WiMob, Barcelona, Spain, Oct. 2019, pp. 1–6, doi: 10.1109/WiMOB.2019.8923232 .
```

鉴于作者没有提供源码，我们按照论文的描述进行了复现，然后在我们当前的实验平台下测试其访问控制时间，500 次测试的平均值为 0.69348s，最大值为 1.96s，最小值为 0.6s。

我们对三种情况的时间总结如下表

| 单位/ms | 无信誉系统 | 加入信誉系统 | wang的方案 |
| ------- | ---------- | ------------ | ---------- |
| 平均值  | 626.82     | 667.36       | 693.48     |
| 最大    | 990        | 2710         | 1960       |
| 最小    | 570        | 550          | 600        |

可以看到，信誉系统的加入使得访问控制的平均时间增加了约6%，主要是因为执行访问控制时需要和信誉合约进行交互。而 wang 的方案相比于加入信誉系统的方案平均时间增加了约 4%，相对于没有信誉系统的方案增加了约 11%。

三种方案的主要区别在于合约逻辑的不同，包括循环的执行、合约间的相互调用次数，尤其是 wang 的方案合约间相互调用比较多，所以平均时间要更多。

### 2.5 意外

我们在使用 JavaScript Date对象测试时发现一个意外情况，访问时间会受发起访问的时机的影响，假设 t 为两次访问的间隔，那么区别如下

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

1. 对合约的调用操作，分为不改变合约的状态和会改变合约的状态两种情况，这两种情况调用产生的时间消耗不同；
2. 访问时间属于更改合约状态这种情况，此时，访问时间不因程序逻辑的变化而变化，意思是循环数量、合约间相互调用的数量，都不会对访问消耗时间产生影响；
3. 访问通过和被拒绝属于合约逻辑问题，同样不对访问时间产生影响；
4. 访问时间受网络情况影响，包括发起访问的时机和后台CPU的占用率；

### 2.6 其它

最后我们看一下更改合约状态的函数和不更改合约状态的函数执行时间是否有差别。我们写了两个关于存储的合约进行测试，合约内容如下

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

## 3. 信誉系统功能测试

如上篇所述，信誉系统的测试需要发起持续不断的合约调用，具体来说，就是在随机的时间调用随机的合约函数，从而验证奖励、惩罚、容忍、报警四大功能。

随机的时间间隔使用泊松分布生成，具体方法在 [另一篇文章](https://shuzang.github.io/generate-random-timings-for-a-poisson-process/) 中介绍。在生成的随机时间点具体执行哪个合约调用，我们进行了如下考虑

首先，会影响信誉值的行为包括

| 事件                             | 所属合约及函数         | 合法/恶意 | 事件编号 |
| -------------------------------- | ---------------------- | --------- | -------- |
| Attribute add                    | MC:addAttribute        | 合法      | 1        |
| Device manager update            | MC:updateManager       | 合法      | 2        |
| Device customed attribute update | MC:updateAttribute     | 合法      | 2        |
| Device delete                    | MC:deleteDevice        | 合法      | 3        |
| Attribute delete                 | MC:deleteAttribute     | 合法      | 3        |
| Resource attribute add           | addResourceAttr        | 合法      | 1        |
| Policy add                       | ACC:addpolicy          | 合法      | 1        |
| Device manager update            | ACC:updateManager      | 合法      | 2        |
| Resource attribute update        | ACC:updateResourceAttr | 合法      | 2        |
| Resource attribute delete        | ACC:deleteResourceAttr | 合法      | 3        |
| Policy delete                    | ACC:deletePolicy       | 合法      | 3        |
| Policy item delete               | ACC:deletePolicyItem   | 合法      | 3        |
| Access authorized                | ACC:accessControl      | 合法      | 4        |
| Blocked end time not reached     | ACC:accessControl      | 恶意      | 0        |
| Policy check failed              | ACC:accessControl      | 恶意      | 0        |
| Too frequent access              | ACC:accessControl      | 恶意      | 1        |
| Both above two                   | ACC:accessControl      | 恶意      | 1        |

1. 以上影响信誉值的行为间是有先后关系的，比如，update、delete 都必须在 add 之后完成，access control 发起的条件是相关的属性和策略已定义等。如果我们完全随机的产生事件，很可能某些行为无法执行。

   解决办法：忽略行为间的先后关系，因为如果存在显式的先后关系，一定会被我们定义的 require 机制给阻止，不会导致系统出错。

2. 当确定了具体的行为，调用时如何传入参数。因为属性和策略的增删改一定会影响访问控制的结果，这样的话，访问成功还是失败完全不受我们控制。

   解决办法：去除会影响结果的行为，因为同一类行为对信誉值的影响是相同的，所以每种行为我们只需要保留一个就可以。比如，合法行为中，Attribute add、Resource attribute add、Policy add 三者等价，我们只保留 Attribute add 一个行为，产生的所有 add 类行为都转换为对 addAttribute 一个函数的调用。最后我们保留下来的行为包括

   | 事件                             | 所属合约及函数     | 合法/恶意 | 事件编号 |
   | -------------------------------- | ------------------ | --------- | -------- |
   | Attribute add                    | MC:addAttribute    | 合法      | 1        |
   | Device customed attribute update | MC:updateAttribute | 合法      | 2        |
   | Attribute delete                 | MC:deleteAttribute | 合法      | 3        |
   | Access authorized                | ACC:accessControl  | 合法      | 4        |
   | Policy check failed              | ACC:accessControl  | 恶意      | 0        |
   | Too frequent access              | ACC:accessControl  | 恶意      | 1        |

   为了排除仅剩的这些行为间的相互影响，我们规定，access control 将依赖于已定义好的属性和策略，新添加/更改/删除属性不会对访问结果产生影响。

3. 每一种行为出现的概率可能是不一致的。比如，update 出现的次数可能多一点，add 和 delete 会少一点，access control 发起的频率可能是最高的。但是，在这里，6种不同的行为我们都以同样的概率来考虑，我们用 0-5 一共 6 个整数表示 6 种不同的事件，然后随机生成这个范围内的一个整数，从而确定某一时刻要调用的函数。

使用一个文本文件存放输入参数，第一列是随机时间，第二列是随机事件，利用 Bash 读取每一行，然后在具体的时间调用指定的 JS 脚本完成这一过程。记录的结果包括从信誉合约检测到的各种值。
