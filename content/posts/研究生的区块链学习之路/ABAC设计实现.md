---
title: ABAC设计实现
date: 2019-12-13
tags: [课题记录]
categories: [研究生的区块链学习之路]
draft: true
---

## 属性集

ABAC中有四种实体可以拥有属性，分别是主体(subject)、客体(object)、动作(action)和环境(environment)。在我们的方案中，所有的区块链账户背后都是一个设备（可能是IoT设备、网关或服务器等），每个设备都可能是发起访问控制的主体(subject)。设备拥有资源，比如数据、计算能力、存储能力等，每个资源是接收访问控制的客体(object)。动作(action)是可执行的操作，比如读、写、执行等。环境(environment)是一些独立于资源的需要满足的环境条件，比如两次访问控制请求的间隔，请求的频率等。

这四者都需要定义与存储其属性集，因此，第一个要考虑的事情就是属性集定义在哪里。不过在考虑这件事情前，先声明以下我们使用的合约结构。

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/6488907/8709863/8386853/zhang3-2847705-small.gif)

最终采用的合约结构依然是如上所述由RC、ACC和JC构成的三合约结构，RC对AC和JC进行管理，ACC进行访问控制，JC进行行为判决。其中，RC和JC是唯一的，而每个ACC都对应着一个设备。

在这种结构下，我们来考虑几种属性集分别应该定义在哪种合约中，又分别应该按什么样的格式来定义和管理。大致的想法有以下三种：

1. 属性集全部定义在RC中。由于RC的作用本来就是合约管理，似乎再添加属性集管理的功能也无可厚非，但这样带来的问题一是ACC进行访问控制时所有读取属性的操作都要跨合约进行，二是管理权问题，需要严格划分谁来定义和管理属性，三是类似于环境和操作属性等严格与设备相关，每个设备的这些属性都有可能不同，统一定义到RC中不好区分。
2. 属性集全部定义在ACC中，这样带来的问题是策略判决时访问主体(subject)的属性要先调用RC获取subject的合约地址，再访问subject关联的ACC获取它的属性，流程繁琐
3. 主体(subject，也即设备)的属性集定义在RC中，其它属性集定义在ACC中，独立维护主体的属性集的好处是调用属性不再需要两道流程，同时资源、操作、环境这些属性集由各自的管理者定义和管理，有助于根据具体的情况进行调整。



## 后续

待实现想法

1. 访问控制的论文再总结一次，归纳发展情况和不足，提出我们当前方案的改进
   - 设备可以阻止别人的未授权访问，当若设备本身已被控制，我们的方案无法解决，所以假设设备、网关是安全的，不考虑内部攻击，只考虑通过访问控制阻止未授权访问。访问控制的策略由资源所有者定义
   - 原方案存在的问题：过于分散。每个subject-resource对都要定义一个ACC，但不是所有的subject都会对每个资源进行访问，全部定义会有冗余，这是访问控制列表方法的缺点。何时布置ACC合约，如何授予权限都需要管理员管理，工作量极大。
2. 权限转移，不只是权限所有者，也可能是被授予权限的人  [Blockchain Based Access Control](https://link.springer.com/chapter/10.1007%2F978-3-319-59665-5_15)
3. 当前的行为检测是绝对的，但很多情况下根本无法判断某个行为是好的还是坏的，只能打分来进行量化，基于分数来判断行为是否出格

之后应该做哪些事

1. 利用相关工具对合约代码进行安全性检查
2. 在Remix中对相关功能进行测试
3. 利用Truffle部署，编写js脚本进行实际调用测试
4. 利用Ethscan等工具监测区块链获取相关数据

当前工作的关键词

Access Control, IIoT, ABAC, smart contract, right transfer, 





## 参考

论文

[1 ]  Figueroa S, Anorga J, Arrizabalaga S, et al. An Attribute-Based Access Control Model in RFID Systems Based on Blockchain Decentralized Applications for Healthcare Environments[J]. The first computers, 2019, 8(3).  [链接]( https://www.mdpi.com/2073-431X/8/3/57 )

[2]  李晓峰, 冯登国, 陈朝武, et al. 基于属性的访问控制模型[J]. 通信学报, 2008(04):95-103.  [链接]( http://www.cnki.com.cn/Article/CJFDTotal-TXXB200804018.htm )

[3]  基于属性的访问控制关键技术研究综述[J]. 计算机学报, 2017(7).  [链接](http://kns.cnki.net//KXReader/Detail?TIMESTAMP=637085548841398750&DBCODE=CJFD&TABLEName=CJFDLAST2017&FileName=JSJX201707013&RESULT=1&SIGN=Xd7ex2falR%2bt%2bAGP4Ai5jpxADxs%3d )

[4] Vincent C. [Attribute-Based Access Control](https://ieeexplore.ieee.org/document/7042715)

[5] NIST Guide to ABAC Definition and Considerations：[链接](https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.sp.800-162.pdf)

其它

[6] SCI 数据库查询页面：http://apps.webofknowledge.com/Search.do?product=UA&SID=7B86yaCsiTIZKDa1PZT&search_mode=GeneralSearch&prID=afa0ae1b-f225-42f0-b473-fc1243fa3746

[7] 知网查询页面：https://kns.cnki.net/kns/brief/default_result.aspx

[8] 访问控制模型简单介绍：https://blog.csdn.net/LngZd/article/details/100781310

[9] Golang实现的各种访问控制模型的库

- 官网：https://casbin.org/docs/zh-CN/abac
- github：https://github.com/casbin/casbin

要回顾的访问控制论文

- [x] Smart Contract-Based Access Control for the Internet of Things

- [x] Blockchain for IoT security and privacy: The case study of a smart home

- [ ] FairAccess: a new Blockchain‐based access control framework for the Internet of Things
- [x] [Blockchain Based Access Control](https://link.springer.com/chapter/10.1007%2F978-3-319-59665-5_15)



