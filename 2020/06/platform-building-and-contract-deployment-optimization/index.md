# 区块链实验8-平台搭建及合约部署优化


前一篇中已经在 Remix 中验证了合约的正确性，本篇记录 Quorum 区块链平台新的搭建方式及合约在该平台中的部署测试。

<!--more-->

## 1. 平台搭建

先前使用的区块链搭建方式是在虚拟机中起一个多节点的区块链网络，节点间以端口号区分，然后在树莓派中安装 Quorum 客户端，加入已建立的网络，最后连接到 Truffle 编写脚本进行部署测试。这种方式具有以下问题：

1. 搭建繁琐易出错。从 genesis.json 文件开始，手动编辑配置文件、创建节点、组建网络实在是一件耗费时间而且错误率较高的事情，不仅步骤非常复杂，且一旦出错就要重新开始，浪费了大量无意义的精力；
2. 无法可视化区块链状态。私链网络、区块、交易、智能合约等都需要实时的监控，一个区块链浏览器是非常必要的工具，但现有已知的开源 Quorum 区块链浏览器都无法直接连接宿主机的 Quorum 私链网络，而是建立在 Quorum 节点的 Docker 集群上。

综上，为了更好的进行实验和得到结果，此次搭建实验平台使用 Docker 集群建立 Quroum 网络。另外，使用开源工具 [quorum-maker](https://github.com/synechron-finlabs/quorum-maker) 来简化网络搭建的流程。

但使用 Docker 也有如下的缺点：

1. 由于对 Docker 网络结构理解的还不够深入，加上 Docker 集群实际上仍然是在虚拟机中启动，如何将作为测试节点的树莓派连接到虚拟机中的 Docker 集群，是一件需要研究的事情；
2. Docker 集群所在的虚拟机，是我的老旧笔记本电脑，是 2014 年的配置，运行现有的实验压力比较大，得到的结果可能差距较大；另外，之前实验用的树莓派现在还放在学校实验室的抽屉，由于疫情原因无法拿到手，因此现在只能在虚拟机中直接建立某个区块链账户用来表示树莓派设备。

下面介绍 Quorum 私链网络搭建的详细过程。

最开始的思路是遵照官方教程建立 [7-nodes示例网络](https://github.com/jpmorganchase/quorum-examples)，然后使用 [Epirus-free](https://github.com/blk-io/epirus-free) 作为区块链浏览器查看状态，但启动的 Epirus 迟迟无法加载出当前区块链的状态。这次失败的建立过程简单记录如下

```bash
# Get Quorum 7 node example
$ git clone https://github.com/jpmorganchase/quorum-examples
$ cd quorum-examples
$ docker-compose up
# Set up Epirus-free
$ git clone https://github.com/blk-io/epirus-free.git
$ cd epirus-free
# start epirus
$ NODE_ENDPOINT=http://quorum-examples_node1_1:8545 docker-compose -f docker-compose.yml -f epirus-extensions/docker-compose-quorum.yml up
# Now we are connecting to node 1, and we could navigate to localhost see the loading page
```

因此最终决定利用 [quorum-maker](https://github.com/synechron-finlabs/quorum-maker) 建立基于 Docker 的 Quorum 网络，由于该工具暂时不支持 IBFT 共识算法，我们只能使用 Raft 共识。quorum-maker 提供了 Web UI  作为区块链浏览器，可查询关于区块、交易和合约的相关信息，满足我们的基本需求，同时，我们也可以使用简单的命令快速建立一个多节点的 Quorum 网络。

### 1.1 环境准备

鉴于上述分析，我们先准备实验环境，之所以不在 Win10 下启动 Docker 集群是因为 Win10 Home Edition 不支持 Docker。另外，实验中设计的命令操作和脚本执行都比较多，Linux 环境比较合适，因此使用了虚拟机。

1. 安装 VMware Workstation 15 Pro，输入批量许可激活，建立 Ubuntu20.04 系统的虚拟机，分配内存 4G（有条件应为8G，这里是因为电脑配置比较低，一共只有8G，再多发生内存交换的概率比较大）、硬盘100GB。

2. 进入 Ubuntu 20.04，更新系统，设置语言，安装git、golang、docker 和 docker-compose

   ```bash
   # 安装git,version 2.25.1
   $ sudo apt install -y git
   # 安装golang,
   $ sudo apt install -y golang
   # 安装 docker CE
   $ curl -fsSL get.docker.com -o get-docker.sh
   $ sudo sh get-docker.sh --mirror Aliyun
   # 启动 docker CE
   $ sudo systemctl enable docker
   $ sudo systemctl start docker
   # 建立 docker 用户组并将当前用户加入 docker 组，这样就不需要 root 权限了
   $ sudo groupadd docker
   $ sudo usermod -aG docker $USER
   # 测试安装
   $ docker run hello-world
   # 安装 docker-compose
   $ sudo curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
   $ sudo chmod +x /usr/local/bin/docker-compose
   ```

3. 下载 Quorum-Maker 的项目代码

    ```bash
   $ git clone https://github.com/synechron-finlabs/quorum-maker 
   ```

### 1.2 建立测试网络

进入 Quorum-Maker 项目目录，执行脚本

```bash
$ cd quorum-maker
$ ./setup.sh dev -p TestNetwork -n 5 -e
```

一个名为 `TestNetwork` 的目录会被创建，其中包含一个 `docker-compose.yml` 文件和 5 个节点目录，进入该目录并启动测试网络

```bash
$ cd TestNetwork
$ docker-compose up
```

一系列命令完成后，将会从命令行看到各节点启动，默认子网段为 `10.50.0.0/16`，网关为 `10.50.0.1`，节点管理器的端口为 `22004`，前面运行脚本时加入的 `-e` 参数就是为了把此端口暴露出来。

现在，在浏览器输入网址 `http://10.50.0.2:22004` 即可连接到区块链浏览器，并查看到 UI，连接其它4个节点只需要分别更改对应的 IP即可。

![screenshot9](/images/区块链实验8-平台搭建及合约部署优化/screenshot9.png)

### 1.3 使用Truffle部署与测试合约

安装 Node，要求Node.js版本高于v8.9.4

```bash
$ sudo apt-get install npm
$ sudo npm install npm@latest -g
$ sudo npm install n -g
$ sudo n lts
```

安装 Truffle

```bash
$ sudo npm install -g truffle
```

创建空项目并初始化，注意 `truffle init` 命令需要连接外网

```bash
$ mkdir ACS
$ cd ACS
$ truffle init
```

## 2. 实验设计

这部分我们需要明确两件事，一是实验测试的场景，二是需要测量的值。

### 2.1 场景

我们计划提取已有的该方向论文所使用的场景，并作一定的分析

#### Smart home

```
A. Dorri, S. S. Kanhere, and R. Jurdak, 
“Blockchain in internet of things: Challenges and Solutions,” arXiv:1608.05187 [cs], 
Aug. 2016, Accessed: Mar. 17, 2020. [Online]. Available: http://arxiv.org/abs/1608.05187.
```

Dorri 的论文是该领域起始的几篇论文之一，也是引用最多的论文之一，他的应用场景是 Smart home，其中涉及的访问控制交易如下

首先是**存储数据**。每个设备都有数据存储的需求，以一个智能恒温器为例，假设 Alice 在云中有一个账户并设置了上传数据的权限，当恒温器有存储数据的请求时，需要首先将数据发送到本地矿工，矿工根据已定义的策略对恒温器的权限进行验证，验证通过后将数据、数据哈希一起发送到云存储，云存储检查是否有剩余空间并匹配数据哈希，在存储完成后，将数据 Hash 和区块号返回，相关的交易收集到区块链中。

![存储数据](/images/区块链实验8-平台搭建及合约部署优化/存储数据.png)

然后是**访问数据**。服务提供者可能需要周期性的访问存储的数据或某个设备的全部数据，请求交易经过 Smart home 的矿工验证权限后，矿工从存储中获取数据，最后用请求者的公钥加密数据后将数据发给请求者。

![访问数据](/images/区块链实验8-平台搭建及合约部署优化/访问数据.png)

最后是**监控数据**。某些时候，智能家庭的所有者可能需要实时地访问家里某些设备的信息，比如恒温器当前配置。矿工收到用户请求后，验证用户权限然后返回设备的实时数据。

![监控数据](/images/区块链实验8-平台搭建及合约部署优化/监控数据.png)

以上三个关于数据操作的示例是访问控制的核心场景之一，具有较大的实验价值，但实际的测试还需要添加存储系统比如 IPFS，然后编写一个存储合约与现有系统关联，由于涉及的东西比较多，可以作为之后的工作。

```
P. Wang, Y. Yue, W. Sun, and J. Liu, 
“An Attribute-Based Distributed Access Control for Blockchain-enabled IoT,” 
in 2019 International Conference on WiMob, Barcelona, Spain, Oct. 2019, pp. 1–6, 
doi: 10.1109/WiMOB.2019.8923232.
```

这篇论文使用的场景也是 Smart home，运用了 ABAC模型，场景结构图如下

![](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8913409/8923119/8923232/wang1-p6-wang-small.gif)



该场景中有三种实体：IoT设备、网关和服务器。诸如智能手机、电脑、智能电视等设备通过 Wi-Fi 或有线连接直接访问互联网，摄像头、温湿度传感器等其它类型的设备则通过蓝牙、ZigBee等技术先连到网关，由网关访问互联网。服务器负责存储 IoT 设备的数据以及提供相应的服务，比如从传感器收集环境信息或发送操作命令给执行器。这些实体间有大量的数据交换和访问请求。

作者最终实验选用的实例是关于电视开关权限的，假设父母拥有对客厅电视的完全控制权，他们可以随时发起访问请求打开或关闭电视，而孩子只有在特定时间（环境属性）内才有权限打开电视。

Smart home 场景在基于区块链的物联网访问控制中具有适用性的原因是，随着物联网时代的到来，Smart home 不再是一个单一的信任域，大部分的智能家居设备数据都会传回生产商的服务器，然后由生产商提供各种各样的服务。我们以摄像头为典型用例，如今，很多家庭都选择在家里安装摄像头以提供防盗或其它功能，但是由于家用设备存储能力的不足，或者自身缺乏足够专业的能力进行远程访问，一般都是用设备厂商或第三方服务商提供的软件服务来存储摄录的内容及进行远程访问，监控的内容不可避免地需要上传到他们的服务器，近年来，第三方导致的监控内容泄露的情况频繁发生，用户缺乏对这些数据的控制能力的根本原因，另外，摄像头权限被非法获取也会造成隐私泄露及其它严重的安全问题。基于区块链对摄像头进行访问控制，可以令用户拥有对监控内容的完全控制能力，从而保证用户的隐私与安全。

我们进行测试时，MC 和 RC都可以由生产商/服务商提供，ACC则由 Smart home 属主自身部署，然后定义合理的数据存储、读取、摄像头控制等权限。由于存储系统暂未添加，我们可以暂时以摄像头控制为例，为家庭成员设置摄像头的控制权限，而阻止服务商和未知用户的控制，仅为服务商提供必要的权限用来为我们提供服务。

Smart home 是生活类、单信任域场景中的典型用例之一，可以有效证明方案的适用性。

#### Healthcare

```
Figueroa, Añorga, and Arrizabalaga, 
An Attribute-Based Access Control Model in RFID Systems Based on Blockchain Decentralized Applications for Healthcare Environments,
Computers, vol. 8, no. 3, p. 57, Jul. 2019, doi: 10.3390/computers8030057.
```

这篇论文提到的是一个典型的医疗领域的资产流动场景，考虑这样一件事：医院拥有大量的资产，比如外科手术器械，这些器械会在消毒部门、手术室、实验室等区域周期性的流动，器械位于错误的位置可能危及患者生命，缺乏详细的资产记录也可能造成资产损失。

下图用于详细说明整个流程。源房间（如灭菌室）将一些资产（如外科手术器械）运送到目的房间（如0号手术室、1号手术室）。由于 $Asset_1$ 已被分配到目的房间1（例如，1号手术室），假如由于人为错误试图访问目的房间0（例如，0号手术室），其访问将被拒绝。简而言之，这一系统的目的是建立医疗资产访问控制系统，防止由于人为错误或外部安全威胁导致资产进入错误区域（如房间）。

![图2 Healthcare system](/images/区块链实验8-平台搭建及合约部署优化/医疗系统.png)

医疗也是区块链和访问控制的重点场景之一，拥有大量的研究和实例。这篇论文提到的基于区块链医疗资产访问控制系统，实用价值在于可以保持一个不可篡改的资产记录，阻止未经授权的访问及命令，

#### Smart Factory

```
J. Wan, J. Li, M. Imran, D. Li, and Fazal-e-Amin, 
“A Blockchain-Based Solution for Enhancing Security and Privacy in Smart Factory,” 
IEEE Trans. Ind. Inf., vol. 15, no. 6, pp. 3652–3660, Jun. 2019, doi: 10.1109/TII.2019.2894573.
```

这篇论文的大背景是 IIoT 中的 Smart Factory，但具体的实例是温度采集。论文描述了温度采集过程中涉及的数据交互过程。一个设备节点申请存储权限的流程如下图所示，温度计发出的数据存储请求由微型计算机代为处理，设备通过一个唯一ID标识，管理中心验证存储权限后，微型计算机将数据加入缓冲池，待数据量满足一定规模或者到达某个周期时间，加密后的温度数据就上传到数据库，数据哈希和相关交易记录存储到区块链中。

![The intranet flowchart](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/9424/8736410/8621042/imran4-2894573-large.gif)

这是一个借用区块链的自动化系统，与传统方案相比的优点是，保持了不可变的存储记录，阻止了未经授权的存储请求，可以有效抵御非法授权的攻击。

#### Log-House Construction

```
T. Kumar, A. Braeken, V. Ramani, I. Ahmad, E. Harjula, and M. Ylianttila, 
“SEC-BlockEdge: Security Threats in Blockchain-Edge based Industrial IoT Networks,” 
presented at the 2019 11th International Workshop on RNDM, Nicosia, Cyprus, Oct. 2019, p. 7.
```

这篇论文提出了一个木屋建造场景作为典型的 IIoT 用例，整个工业过程主要由下图所示的几个关键流程组成

![木屋建造场景](https://ieeexplore.ieee.org/mediastore_new/IEEE/content/media/8944089/8949080/8949107/Paper06-fig-1-source-large.gif)

在收割阶段，传感器/执行器和其它低功耗设备将感知、收集和处理与原材料和收割过程相关的信息，这些信息将用于其它阶段。运输、制造和存储阶段的日志将会被其它阶段利用。分发阶段需要保证由正确的车辆通过正确的路线准时送达。整个过程涉及传感器/执行器、边缘设备/服务器、服务/网络提供者、其它第三方等多种角色，需要保证每个角色都只能访问需要访问的资源以及提供需要提供的必要信息。作者并没有给出一个具体的实验实例。

#### 电子门锁与自动售货机

```
V. A. Siris, D. Dimopoulos, N. Fotiou, S. Voulgaris, and G. C. Polyzos, 
“Trusted D2D-Based IoT Resource Access Using Smart Contracts,” 
in 2019 IEEE 20th International Symposium on WoWMoM, Washington, DC, USA, Jun. 2019, pp. 1–9, 
Accessed: Apr. 03, 2020. [Online]. Available: https://ieeexplore.ieee.org/document/8793041/.
```

这篇论文介绍了两个用例。

第一个用例涉及的是旅馆或公寓的短租房间的电子门锁（门锁作为一种 IoT 资源）。如果一个人想预定旅馆或公寓的房间几个晚上，可以从智能手机（客户端）向处理门锁授权请求的授权服务器（AS）发送请求，AS建立一个智能合约来接收预付款，一旦验证客户已支付，AS将向客户提供必要的凭据（数字钥匙或者称作访问令牌），在确定的时间内，客户可以通过利用手机通过蓝牙等通信手段发起请求，验证权限后可以打开门锁。

第二个用例是从自动售货机购买物品。假设自动售货机具有持续的网络连接，但并不与区块链直接交互，一个人通过手环或手机从自动售货机购买物品。这种情况下，客户端（手环）付款后获得支付凭据，然后通过D2D通信向自动售货机验证，售货机验证支付后出货。

#### 总结

我们选定 Smart home 中的摄像头用例，木屋制造所代表的供应链场景，以这两个场景为例，证明我们使用区块链完成物联网访问控制的普适性。

### 2.2 测量值

不论是访问控制系统，还是信誉系统，都需要一些测量值来证明我们的结论，本部分讨论实验应该获得哪些测量结果。

#### 测量什么

对于**访问控制系统**，主要目的是证明我们方案的有效性，因此应当模拟上一小节所示的各种场景，完成各种不同情况的访问控制。由于我们使用的主要工具是智能合约，应当获得并记录合约部署的 Gas 消耗和时间消耗，尽管 Quorum 私链中 Gas 的消耗并没有意义，但作为一种常用测量值和人们通常关注的方面，还是要简单的给出结果。最后，进行一次访问控制的时间消耗是访问控制系统最重要的一个参数，我们可以确定 ABAC 模型相比于传统的 ACL 是更加物联网友好的，但对于同样的 ABAC 设计实现，究竟哪种设计实施访问控制比较快，还有待验证，因此应当复现其它的一些方案，并进行对比。

关于**信誉系统**，首先要确定以什么样的模型进行输入，从而真实的反应实际系统中恶意行为发生的情况，测量值应包括信誉值随时间的变动曲线，各种参数的最优取值，以及阻塞时间的变动趋势。

#### 如何获取

合约部署消耗很容易可以得到，可以通过 Remix 或 Truffle 部署的命令返回得到。

访问控制时间作调用 AccessControl() 合约函数的交易执行时间，可以在测试脚本中编写代码计算发起请求和得到返回值之间的时间，但要注意这一结果受实验环境影响很大。

输入模型我们之后阅读论文来查找，信誉值、阻塞时间都在智能合约中通过 Event 发出，只要即时监控即可。

关于惩罚函数的值在确定相关权重值后也可以得到，

## 3. 进行实验

部署合约

事实上，实际的测试过程接下来就可以执行测试了，Subject 首先根据设备的区块链地址从 MC 中获取设备的 ACC 合约地址，或者直接从网络获取设备管理者公布的 设备 ACC 合约地址，然后发起调用 accessControl() 函数，传入参数为要访问的资源和要执行的操作。返回的结果为 true 或 false，同时，调用者地址、结果、结果说明、访问时间等会通过事件发送出去，可以在 Js 中通过调用事件查看。
