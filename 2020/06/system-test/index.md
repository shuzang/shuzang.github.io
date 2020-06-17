# 区块链实验9-系统测试


万事俱备，只待测试，本篇进行系统测试。

<!--more-->

测试分为三部分

1. 隐私合约及交易测试
2. 访问控制系统测试
3. 信誉系统测试

各部分都不太一样，下面分节进行介绍

## 1. 隐私合约及交易测试

隐私合约及交易是 Quorum 自带的功能，因此测试起来比较简单。我们令Node2 代表农场，Node3 代表超市，假设农场部署了一个私有存储合约，状态只能被超市查看。测试用的合约如下

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

农场部署合约传入的初始值为 50，理论上，Node2 和 Node3 可以通过调用 get() 函数获取到该数据，其它的节点无法查看该数据。

合约的部署与交互使用了 Quorum for Remix 插件

分别进入 Node3（超市） 和 Node4 的 Geth console 进行验证，如下图所示，左侧是 Node3，可以查看合约状态并获取数据，右边是 Node4，无法查看合约状态，也无法获得数据。

![私有合约状态测试](/images/区块链实验9-系统测试/私有合约状态测试.png)

从 Cakeshop 区块链浏览器可以更清楚地看到两种情况

![节点3可以查看私有合约](/images/区块链实验9-系统测试/节点3可以查看私有合约.png)

![节点4无法查看私有合约](/images/区块链实验9-系统测试/节点4无法查看私有合约.png)

## 2. 访问控制测试

访问控制测试主要分为三部分

1. 当前系统的访问控制时间；
2. 删除信誉系统后的访问控制时间，用来对比信誉系统的添加对访问控制时间的影响；
3. 传统 ABAC 架构利用智能合约实现的访问控制时间，用来对比我们实现的系统的时间性能。

### 2.1 访问控制时间测量

隐私交易时使用了 Quorum for Remix 插件，由于该插件在 Remix 中不会返回结果，在非隐私交易时不具备优势，因此访问控制系统时间测量的预准备工作，包括合约部署和交互，使用了 Remix 自己提供的 Deploy and Run 插件，主要利用 Web3 Provider 连接 Quorum 网络。连接端口已默认打开，我们使用的三个节点对应的端口如下

- Node1：22000
- Node2：22001
- Node3：22002

合约部署完成、设备注册完成、属性定义完成及策略定义的完整记录在仓库中，完成后，可以正式开始测试。

不过首先记录三种合约部署的 Gas 消耗

- MC：2512367 gas
- RC：1170616 gas
- ACC：4749983 gas

因为我们要进行时间的计算和一些复杂的交互操作，单独安装使用 Web3.js 来进行访问控制

```bash
# 在用户根目录建立web3文件夹
$ mdkir web3
$ cd web3
# 在web3文件夹中本地安装 web3 1.2.8 版本，之前的版本有些部件不再维护，安装会出错，1.2.9某些功能出错
$ npm install web3@1.2.8
```

我们第一次尝试了直接在 JS 脚本中设立循环，使用 setInterval() 函数延时 120s 发起访问请求，但这种方式第一次访问控制的时间和之后的时间存在较大的差距，第一次普遍在 10ms 左右，其后迅速减少，在 1ms 和 2ms 浮动，目前不清楚原因，可能有缓存，因此采用编写 Shell 脚本循环调用 JS 脚本的方式。

注：这里之所以延时 120s，是因为两次访问控制间隔少于 100s 会触发 Too frequent request 错误。这是我们自定义的一种恶意行为，为了获取足够的访问控制时间测量样本，我们设置延时。

完成访问控制的 JS 脚本和循环调用 JS 脚本的 Shell 脚本也放在了仓库中。执行 Shell 脚本之前，首先解锁发起访问的账户，由于我们设置了延时 120s，计划测量 30 个值，因此一次性将账户解锁 1小时以上，这里我们设置 4000s。

首先测试合法的访问控制，解锁相应的账户。

```js
> personal.unlockAccount(eth.accounts[0],"",3600)
```

得到 30 个合法访问时间记录，30次平均访问时间为 10.9 ms，最坏时间为 26ms

```bash
shuzang@ubuntu:~/network/web3$ ./runscript.sh requester_timeCostTest.js
Note: Every 2 minutes, send an access request

The 1-th request
Return Message: Access authorized
Begin-End: 1592384473006-1592384473018
Time Cost: 12ms

The 2-th request
Return Message: Access authorized
Begin-End: 1592384600573-1592384600584
Time Cost: 11ms

The 3-th request
Return Message: Access authorized
Begin-End: 1592384726041-1592384726059
Time Cost: 18ms

The 4-th request
Return Message: Access authorized
Begin-End: 1592384850653-1592384850661
Time Cost: 8ms

The 5-th request
Return Message: Access authorized
Begin-End: 1592384975578-1592384975589
Time Cost: 11ms

The 6-th request
Return Message: Access authorized
Begin-End: 1592385100854-1592385100863
Time Cost: 9ms

The 7-th request
Return Message: Access authorized
Begin-End: 1592385225599-1592385225608
Time Cost: 9ms

The 8-th request
Return Message: Access authorized
Begin-End: 1592385350523-1592385350534
Time Cost: 11ms

The 9-th request
Return Message: Access authorized
Begin-End: 1592385475584-1592385475592
Time Cost: 8ms

The 10-th request
Return Message: Access authorized
Begin-End: 1592385600576-1592385600588
Time Cost: 12ms

The 11-th request
Return Message: Access authorized
Begin-End: 1592385725534-1592385725560
Time Cost: 26ms

The 12-th request
Return Message: Access authorized
Begin-End: 1592385850536-1592385850546
Time Cost: 10ms

The 13-th request
Return Message: Access authorized
Begin-End: 1592385975510-1592385975521
Time Cost: 11ms

The 14-th request
Return Message: Access authorized
Begin-End: 1592386100563-1592386100574
Time Cost: 11ms

The 15-th request
Return Message: Access authorized
Begin-End: 1592386225578-1592386225589
Time Cost: 11ms

The 16-th request
Return Message: Access authorized
Begin-End: 1592386350536-1592386350545
Time Cost: 9ms

The 17-th request
Return Message: Access authorized
Begin-End: 1592386475546-1592386475557
Time Cost: 11ms

The 18-th request
Return Message: Access authorized
Begin-End: 1592386600561-1592386600570
Time Cost: 9ms

The 19-th request
Return Message: Access authorized
Begin-End: 1592386725653-1592386725663
Time Cost: 10ms

The 20-th request
Return Message: Access authorized
Begin-End: 1592386850526-1592386850539
Time Cost: 13ms

The 21-th request
Return Message: Access authorized
Begin-End: 1592386975544-1592386975556
Time Cost: 12ms

The 22-th request
Return Message: Access authorized
Begin-End: 1592387100602-1592387100611
Time Cost: 9ms

The 23-th request
Return Message: Access authorized
Begin-End: 1592387225590-1592387225601
Time Cost: 11ms

The 24-th request
Return Message: Access authorized
Begin-End: 1592387350529-1592387350539
Time Cost: 10ms

The 25-th request
Return Message: Access authorized
Begin-End: 1592387475655-1592387475663
Time Cost: 8ms

The 26-th request
Return Message: Access authorized
Begin-End: 1592387600541-1592387600552
Time Cost: 11ms

The 27-th request
Return Message: Access authorized
Begin-End: 1592387725572-1592387725580
Time Cost: 8ms

The 28-th request
Return Message: Access authorized
Begin-End: 1592387850520-1592387850529
Time Cost: 9ms

The 29-th request
Return Message: Access authorized
Begin-End: 1592387975833-1592387975843
Time Cost: 10ms

The 30-th request
Return Message: Access authorized
Begin-End: 1592388100622-1592388100631
Time Cost: 9ms

```

完成后进行非法访问的测试，解锁账户

```js
> personal.unlockAccount(eth.accounts[1],"",3600)
```

非法访问测试30次的平均访问时间 16.7 ms，最坏时间 119ms

```bash
shuzang@ubuntu:~/network/web3$ ./runscript.sh requester_timeCostTest.js
Note: Every 2 minutes, send an access request

The 1-th request
Return Message: Policy check failed
Begin-End: 1592388934582-1592388934592
Time Cost: 10ms

The 2-th request
Return Message: Policy check failed
Begin-End: 1592389061921-1592389061939
Time Cost: 18ms

The 3-th request
Return Message: Policy check failed
Begin-End: 1592389185602-1592389185611
Time Cost: 9ms

The 4-th request
Return Message: Policy check failed
Begin-End: 1592389310527-1592389310536
Time Cost: 9ms

The 5-th request
Return Message: Policy check failed
Begin-End: 1592389435521-1592389435532
Time Cost: 11ms

The 6-th request
Return Message: Policy check failed
Begin-End: 1592389560571-1592389560583
Time Cost: 12ms

The 7-th request
Return Message: Policy check failed
Begin-End: 1592389685567-1592389685578
Time Cost: 11ms

The 8-th request
Return Message: Policy check failed
Begin-End: 1592389810541-1592389810550
Time Cost: 9ms

The 9-th request
Return Message: Policy check failed
Begin-End: 1592389935607-1592389935617
Time Cost: 10ms

The 10-th request
Return Message: Policy check failed
Begin-End: 1592390060589-1592390060598
Time Cost: 9ms

The 11-th request
Return Message: Policy check failed
Begin-End: 1592390185527-1592390185539
Time Cost: 12ms

The 12-th request
Return Message: Policy check failed
Begin-End: 1592390310539-1592390310552
Time Cost: 13ms

The 13-th request
Return Message: Policy check failed
Begin-End: 1592390435529-1592390435537
Time Cost: 8ms

The 14-th request
Return Message: Policy check failed
Begin-End: 1592390560562-1592390560571
Time Cost: 9ms

The 15-th request
Return Message: Policy check failed
Begin-End: 1592390685574-1592390685583
Time Cost: 9ms

The 16-th request
Return Message: Policy check failed
Begin-End: 1592390810520-1592390810546
Time Cost: 26ms

The 17-th request
Return Message: Policy check failed
Begin-End: 1592390935577-1592390935589
Time Cost: 12ms

The 18-th request
Return Message: Policy check failed
Begin-End: 1592391060554-1592391060583
Time Cost: 29ms

The 19-th request
Return Message: Policy check failed
Begin-End: 1592391185545-1592391185555
Time Cost: 10ms

The 20-th request
Return Message: Policy check failed
Begin-End: 1592391310559-1592391310569
Time Cost: 10ms

The 21-th request
Return Message: Policy check failed
Begin-End: 1592391435562-1592391435572
Time Cost: 10ms

The 22-th request
Return Message: Policy check failed
Begin-End: 1592391560572-1592391560583
Time Cost: 11ms

The 23-th request
Return Message: Policy check failed
Begin-End: 1592391685687-1592391685698
Time Cost: 11ms

The 24-th request
Return Message: Policy check failed
Begin-End: 1592391810849-1592391810864
Time Cost: 15ms

The 25-th request
Return Message: Policy check failed
Begin-End: 1592391936333-1592391936452
Time Cost: 119ms

The 26-th request
Return Message: Policy check failed
Begin-End: 1592392061114-1592392061126
Time Cost: 12ms

The 27-th request
Return Message: Policy check failed
Begin-End: 1592392185619-1592392185665
Time Cost: 46ms

The 28-th request
Return Message: Policy check failed
Begin-End: 1592392310578-1592392310586
Time Cost: 8ms

The 29-th request
Return Message: Policy check failed
Begin-End: 1592392435639-1592392435649
Time Cost: 10ms

The 30-th request
Return Message: Policy check failed
Begin-End: 1592392560536-1592392560548
Time Cost: 12ms
```

注意：设备发起访问控制时，应当首先获取目标设备绑定的 ACC 合约地址，该地址可以在 MC 中根据设备账户查询得到。

### 2.2 信誉系统加入对时间的影响

上一小节的时间是带有信誉系统测量得到的时间，本次我们将信誉系统从合约中删除，只剩下 MC 和 ACC，然后进行测试。修改后的两个合约代码位于仓库中。

未完待续...


