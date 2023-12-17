# 树莓派常用设置


记录树莓派使用过程中经常使用的一些设置。

## 1. WiFi连接

这里指的是初次启动直连WiFi，主要是因为手里没有屏幕，因为使用的小米随身WiFi，所以可能有些废话，但为了保存资料，就写这里了。

### 1.1 随身WiFi设置

按随身WiFi附带的说明会下载安装一个网络管理软件[miwifi](http://www.miwifi.com/miwifi_download.html)，驱动会默认安装。但使用这一软件，接入设备的ip无法ping通，只能选择卸载该软件单独安装驱动。驱动名为`Xiaomi 802.11n USB Wireless Adapter`，可以用必应或谷歌直接搜索[下载](https://www.driverscape.com/download/xiaomi-802.11n-usb-wireless-adapter)即可。驱动安装完成后，安装[猎豹免费wifi](http://wifi.liebao.cn/)，用作wifi管理软件，查看到的设备ip可以使用。

*注：使用win10自带的热点开启工具会出现不少问题，即使成功开启了，设备也连接不上。唯一能成功启用的方式还是在安装猎豹免费wifi后进行启用，然后关闭猎豹，不过这样就没什么意义了。*

小米，百度，360的随身WiFi使用的都是mt7601u。其它关于小米随身WiFi的一些参数，查询如下：

![小米随身WiFi的参数](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190904_3AL0zV.png)

### 1.2 树莓派设置

将刷好 Raspbian 系统的 SD 卡用电脑读取。在 boot 分区，也就是树莓派的 `/boot` 目录下新建 wpa_supplicant.conf 文件，按照下面的参考格式填入内容并保存 wpa_supplicant.conf 文件。

```bash
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=CN

network={
    ssid="WiFi名"
    psk="密码"
    key_mgmt=WPA-PSK
    priority=1
}
```

priority：连接优先级，数字越大优先级越高（不可以是负数）  
scan_ssid：连接隐藏WiFi时需要指定该值为1  

key_mgmt：加密方式，WPA和WPA2都填WPA-PSK，小米随身WiFi使用这种，其他的还有WEP等。

### 1.3 开启SSH连接

在boot分区新建名为`ssh`的文件，要注意小写且没有扩展名。

### 1.4 树莓派启动并访问

将配置好的SD卡卸载并插入树莓派，通电启动。不久即可以在WiFi管理软件中看到，设备名会是`raspberrypi`，极好辨认，同时能看到分配的ip。通过该ip使用SSH登录即可。

*注：可以使用`arp -a`命令或IP扫描工具扫描，都不影响，只是从wifi管理软件看更快。*

## 2. 数据源更新

基于众所周知的原因，需要将源更换为国内源，这里选择清华。

登录树莓派，使用管理员权限编辑/etc/apt/sources.list文件

```bash
sudo nano /etc/apt/sources.list
```

注释掉原来的源，将源更新为：

```bash
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
```

最后持续键入CTRL+O  -> ENTER-> CTRL+X退出

使用管理员权限编辑/etc/apt/sources.list.d/raspi.list文件

```bash
sudo nano /etc/apt/sources.list.d/raspi.list
```

将源更新为：

```bash
deb http://mirror.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
deb-src http://mirror.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
```

更新源文件列表，更新软件

```bash
$ sudo apt-get update
$ sudo apt-get upgrade
```

## 3. 远程访问

树莓派安装xrdp

```bash
$ sudo apt-get install xrdp
```

win10使用Cortana搜索远程桌面连接，计算机名填写树莓派ip，用户名填写pi(如果未更改)，点击连接即可，后续设置不必调整，最后可进入树莓派桌面。是在没有买专门的屏幕的情况下一种可视化的方式。

## 4. 修改pi账户密码，开启root账户

使用`password pi`可修改pi账户密码。

由于树莓派使用的Linux基于debian，root账户默认没有密码，同时没有开启。若要启用，在pi账户下执行命令

```bash
$ sudo passwd root
```

执行此命令后会提示输入两遍root密码，输入密码后执行

```bash
$ sudo passwd --unlock root
```

即可解锁root账户

## 5. 利用ftp进行文件传输

安装vsftpd

```bash
$ sudo apt-get install vsftpd
```

启用ftp服务

```bash
$ sudo service vsftpd start
```

编辑配置文件

```bash
$ sudo nano /etc/vsftpd.conf
```

启用对树莓派进行写操作，不然只能从树莓派往PC传文件，没法往树莓派传

```bash
# Uncomment this to enable any form of FTP write command.
write_enable=YES
```

保存退出，重启vsftpd

```bash
$ sudo service vsftpd restart
```

最后在PC下使用FileZilla通过ssh登录即可进行文件互传。

## 6. 设置静态ip

涉及网络相关的项目，设置静态ip是必要的事。需要修改`/etc/dhcpcd.conf`文件

```bash
$ sudo nano /etc/dhcpcd.conf
```

在末尾添加如下内容

```bash
interface wlan0

static ip_address=192.168.191.2/24
static routers=192.168.191.1
static domain_name_servers=192.168.191.1
```

字段解释：

`interface`：eth0代表有线，wlan0代表无线，多网卡事先用`ifconfig`命令查看确认

`static ip_address`：静态ip地址，要确认在网段范围内

`static routers`：网关地址

`static domain_name_servers`：域名服务器地址，多个地址使用空格分隔。这里填了网关地址。

保存退出，`reboot`命令重启，重启后即可使用静态ip登录。

## 7. 系统备份

好不容易配置好的树莓派系统，更新了源，设置了静态ip，装好了环境或需要的软件，结果一个出错需要重新来，简直崩溃，所以备份一个系统是必要的事。

备份系统很简单，将配置好的SD卡插入电脑，使用将系统写入SD卡的win32diskimager软件，新建以`.img`为后缀的文件，在路径栏选择该文件，选择读取等待进度条完成即可。需要注意的是，这样备份的镜像文件大小是SD卡的容量大小，所以，如果备份了一个32G大的SD卡镜像，之后无法写入16G的新SD卡。

![备份树莓派系统](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190904_3AL6Z4.png)

---

> 作者:   
> URL: https://shuzang.github.io/2019/%E6%A0%91%E8%8E%93%E6%B4%BE%E5%B8%B8%E7%94%A8%E8%AE%BE%E7%BD%AE/  

