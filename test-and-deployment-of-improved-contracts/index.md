# 区块链实验8-改进合约的测试与部署


共RC、ACC、JC三个合约，含义分别为Register Contract、Access Control Contract、Judge Contract

## Gas消耗

三个合约部署的Gas消耗统计如下

| 合约名 | transaction cost | execution cost |
| ------ | ---------------- | -------------- |
| RC     | 3285811 gas      | 2457443 gas    |
| ACC    | 5380922 gas      | 4047334 gas    |
| JC     | 1375161 gas      | 1002445 gas    |

消耗的代币数量 = gas × gasprice，gasprice的货币单位决定代币的货币单位。

在Quorum网络中，这些Gas消耗没有实际意义，因为gasprice = 0，合约部署前会判断用户是否拥有足够的gas，但不会真的扣除。

## 合约结构

RC实现的功能大致分为两类，第一部分对合约进行管理，第二部分对属性进行管理

ACC实现的功能分为三类，第一部分对设备自身拥有的资源属性进行管理，第二部分对访问控制策略进行管理，第三部分是访问控制函数

JC实现两个函数，第一个是恶意行为判决，第二个用来查询恶意行为

## 部分测试日志

RC账户地址：0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c
RC合约地址：0xdc04977a2078c8ffdf086d618d1f961b6c546222
transaction cost: 1794101

JC输入参数：base=2, interval=3
JC账户地址：0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c
JC合约地址：0x8c1ed7e19abaa9f23c476da86dc1577f1ef401f5
transaction cost: 1076285

ACC输入参数：RC和AC的合约地址
ACC账户地址：0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C
ACC合约地址：0x6431fd0c29d024c5b04c7dab157fccd329e62e55
transaction cost: 4344993

合约管理：
1. JC合约注册
    输入参数：Judger, JC, 0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c, 0x8c1ed7e19abaa9f23c476da86dc1577f1ef401f5
    transaction cost: 112762
2. 获取JC合约地址
    输入参数：Judger
    transaction cost: 25115(Cost only applies when called by a contract)
3. ACC合约注册
    输入参数：Sensor11, ACC, 0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C, 0x6431fd0c29d024c5b04c7dab157fccd329e62e55
    transaction cost: 112954
4. 更新Sensor11的合约地址
    输入参数：Sensor11, scAddress, 0x8c1ed7e19abaa9f23c476da86dc1577f1ef401f5
    transaction cost: 25243(Cost only applies when called by a contract)
5. 删除Sensor11的ACC合约
    输入参数：Sensor11
    transaction cost: 24438
6. 重新注册ACC合约，和第3步相同

设备属性管理：
1. 设备注册
    输入参数：0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C，0x4B0897b0513fdC7C541B6d9D7E929C4e5364D2dB, sensor, IoT device
     transaction cost: 93254
2. 属性查看
    输入参数：0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C, sensor
3. 添加新属性
   输入参数：0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C，currentState, active
4. 查询新属性，方法同2
