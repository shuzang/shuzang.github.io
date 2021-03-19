# 树莓派使用网线直连电脑


树莓派刷好系统，要进行联网使用，手里没有显示屏和HDMI线，实验室没有路由器，电脑也没有无线网卡，唯一的网口也被占用。只能通过网线和USB网口转换器直连电脑并使用SSH登录。

#### 准备材料

1. 刷好系统的树莓派，已添加`ssh`文件启用ssh
2. 一根网线
3. 一台已联网的电脑
4. USB网口转换器（如果有多余的网口可以不需要）

#### 连接

使用USB网口转换器扩展一个网口出来，使用网线连接扩展的网口和树莓派。

#### 网络设置

打开网络和共享中心，可看到已联网的网络`以太网`和树莓派连接的网络`以太网3`

![网络与共享中心](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190904_3AXugP.png)

**设置网络共享**

点击`以太网3->属性->共享选项卡->双击“Internet协议版本4（TCP/IP)” –> 选择“使用下面的ip地址” `，填入后点击确认

![以太网3 IP设定](https://picped-1301226557.cos.ap-beijing.myqcloud.com/3AXUg0.png)

点击`以太网->属性->共享选项卡->勾选“允许其他网络用户通过此计算机的Internet连接来连接” –> 在家庭网络连接下面的下拉菜单中选择 “以太网3” `，点击确认（出现将ip设置为“192.168.137.1”的提示也点击确定，这里出现这个是因为事先设定了以太网3的IP）

![网络共享设置](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190904_3AXoUH.png)

**查询树莓派的IP**

在PowerShell或cmd中输入`arp -a`，寻找地址`192.168.137.1`下面的IP，第一个符合IP分配规则的地址就是树莓派的地址，如果查询不到，重新拔插树莓派的网线后再次查询即可。

![arp命令查询树莓派IP](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190904_3AjSaQ.png)

也可以使用`Advanced IP Scanner Portable`工具扫描查询，名称为`raspberrypi.mshome.net`的既是树莓派，见名知意。

![Advanced IP Scanner扫描树莓派IP](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190904_3AjFx0.png)

**使用ssh工具连接**

使用ssh工具（我用Putty），通过查询到的IP连接树莓派，默认用户名和密码是`pi`和`raspberry`
