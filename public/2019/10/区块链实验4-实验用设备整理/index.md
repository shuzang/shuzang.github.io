# 区块链实验4-实验用设备整理


## 设备列表

开发板、MCU或其它套件、传感器/执行器的列表如下

**Demoboard**

| 名称                  | 实物图                                                       | 数量 |
| --------------------- | ------------------------------------------------------------ | ---- |
| Raspberry pi          | ![Raspberry pi](https://tse1-mm.cn.bing.net/th?id=OIP.FYIO5Tu0yFhQB6Za9NXFzQHaGH&w=226&h=181&c=7&o=5&pid=1.7) | 2    |
| BeagleBone Black(BBB) | ![BeagelBone Black](https://tse3-mm.cn.bing.net/th?id=OIP.Gwsoh9XH-CsXzxdBqjdnOQHaE7&w=282&h=187&c=7&o=5&pid=1.7) | 2    |

**MCU或其它**

| 名称                                                         | 实物图                                                       | 数量 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [LoRa/GPS Hat](https://www.dragino.com/products/lora/item/106-lora-gps-hat.html) | ![LoRa/GPS HAT](http://wiki.dragino.com/images/thumb/d/d6/Lora_GPS_HAT.png/300px-Lora_GPS_HAT.png) | 1    |
| [CC1350 LaunchPad](http://www.ti.com/tool/LAUNCHXL-CC1350)   | ![](http://www.ti.com/diagrams/med_launchxl-cc1350_launchxl-cc1350_dsc_2006.jpg) | 2    |
| [Grove XBee Carrier](http://wiki.seeedstudio.com/cn/Grove-XBee_Carrier/)和与之配套的[RFBee](http://wiki.seeedstudio.com/cn/RFbee_V1.1-Wireless_Arduino_compatible_node/) | ![Grove XBee Carrier](https://tse1-mm.cn.bing.net/th?id=OIP.AS_6GlgBhtmyfUeXRz8LOwHaFj&w=282&h=207&c=7&o=5&pid=1.7)![RFBee](https://tse2-mm.cn.bing.net/th?id=OIP.K7mJRbwY21_lhdwIzqyEyAHaFj&w=235&h=172&c=7&o=5&pid=1.7) | 2    |
| [UartSBee v5](http://wiki.seeedstudio.com/cn/UartSBee_v5/)   | ![UartSBee](https://tse4-mm.cn.bing.net/th?id=OIP.Pge4Qjh_NuOooLhfQKwNwgAAAA&w=236&h=192&c=7&o=5&pid=1.7) | 2    |

传感器/执行器

| 名称                                                         | 实物图                                                       | 数量 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [Grove-Loudness Sensor](http://wiki.seeedstudio.com/cn/Grove-Loudness_Sensor/) | ![](https://tse1-mm.cn.bing.net/th?id=OIP.ESsCkeUMKCENk1_9qbg09AHaGe&w=228&h=197&c=7&o=5&pid=1.7) | 1    |
| [Grove-Dust Sensor](http://wiki.seeedstudio.com/Grove-Dust_Sensor/) | ![](https://tse3-mm.cn.bing.net/th?id=OIP.RMDWwFhYOkp8HbWYMocrMAHaFj&w=279&h=209&c=7&o=5&pid=1.7) | 1    |
| [Grove-Chainable RGB LED](http://wiki.seeedstudio.com/cn/Grove-Chainable_RGB_LED/) | ![](https://tse4-mm.cn.bing.net/th?id=OIP.E6cI8zMJAuzuHgjSgp2nOgHaFj&w=251&h=186&c=7&o=5&pid=1.7) | 1    |

LoRa/GPS HAT，CC1350 LaunchPad都内置一个温度传感器，LoRa/GPS HAT除了支持LoRa通信，还可以收集并上传GPS数据

## 设备组合

基本拓扑：网关+终端设备

### 1. 有线直连

#### A. 树莓派/BBB + 传感器/执行器

将传感器/执行器直接接到树莓派或BBB开发板上，在开发板中使用Python相关库从传感器/执行器读取相关数据并进行处理。工作量较小

#### B. 树莓派 + Arduino + 传感器/执行器

Arduino连接传感器/执行器，并通过串口与树莓派通信，在树莓派中使用Python相关库处理Arduino传过来的数据并进行处理。工作量较小，但目前没有Arduino。

### 2. 无线通信

#### A. LoRa

Arduino连接传感器/执行器，同时连接LoRa模块作为LoRa节点，将收集自传感器的数据通过LoRa模块发送到网关。树莓派连接 LoRa/GPS Hat 作为 LoRa网关接受来自LoRa节点的数据，并提交到[The Things Network(TTN)](https://www.thethingsnetwork.org/) 网络，从TTN后台获取实时数据供区块链处理

目前没有Arduino，且缺少一个LoRa模块与LoRa/GPS HAT协作传输数据。

工作量较大。

#### B. Sub 1GHz

CC1350 LaunchPad作为终端设备从传感器获取数据，通过Sub 1GHz发送给另一台作为收集器的CC1350，收集器与BeagleBone Black串口通信，网关运行在BBB上。

CC1350 LaunchPad 编程处理传感器工作量比Arduino大，不熟悉；如何从运行在BBB上的本地网关获取暂时数据也不清楚

工作量较大
