# Work-in-Progress Integrating Low-Power IoT devices to


Özyılmaz, Kazım Rıfat, and Arda Yurdakul. "Work-in-Progress: Integrating low-power IoT devices to a blockchain-based infrastructure." 2017 International Conference on Embedded Software (EMSOFT). IEEE, 2017.

说明：关于物联网和区块链结合，节点资源有限的一种解决方案

## Introduction

引言部分引用了几篇物联网发展现状的论文，做背景分析可能用到

这篇文章主要是为基于LPWAN的物联网部署到区块链基础设施做一个概念验证，也就是证明这事可以做。还提出了各种终端设备的集成到区块链的方法。他们在网关上的软件解决方案利用区块链功能来（a）促进分散的物联网平台，（b）标准化终端设备和物联网基础设施之间数据传输的方式，（c）连接任何类型的物联网终端设备到基于区块链的物联网平台

把LoRa网关接入以太坊区块链，实现一个基于事件的通信机制

## Blockchain and IoT Integration

把物联网终端设备和网关集成到基于区块链的物联网平台上可以通过以下方法实现：

- 网关作为区块链的全节点，终端设备和网关通信
- 网关作为区块链的轻节点，终端设备和网关通信
- 终端设备作为常规传感器，电池供电的终端设备不足以集成区块链客户端，由物联网网关把数据推送到区块链
- 终端设备作为服务器可信的客户端：使用BCCAPI这样接口的简单形式的客户端可以集成到电池供电的物联网终端设备中
- 如果终端设备不是由电池供电并且始终开启，终端设备可以作为区块链的轻节点

## Proof of Concept

场景描述：LPWAN技术可以跟踪覆盖数公里范围的人员或设备。这篇文章中，电池供电的物联网终端设备将位置数据发送到LoRa网关。然后，LoRa网关使用智能合约将此数据流通过官方Go-lang的以太坊客户端Geth路由到私有的以太坊区块链。使用连接到Dragino LoRa / GPS Hat 的Raspberry Pi 2构建LoRa终端设备，使用Raspberry Pi 3构建LoRa网关。为了实现这样的双向LoRaWAN-Ethereum代理，网关应运行LoRa协议软件以与终端设备通信，并运行以太坊客户端以将数据路由到区块链网络。LoRa协议软件用于将数据包转发到应用程序服务器。此外，使用初始生成块创建私有以太坊网络，其具有简单的挖掘设置，以实现更快的响应时间，即更短的挖掘时间。

要使网关和区块链交互，需要部署智能合约，部署后智能合约的地址和程序二进制接口用于与其交互。使用一个smart proxy从LoRa包转发器捕获数据，然后通过JSON-RPC接口将其提供给Geth并调用智能合约

基于事件的通信主要是指智能合约的程序结构

