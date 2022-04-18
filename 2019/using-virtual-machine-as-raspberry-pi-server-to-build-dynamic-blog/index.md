# 利用树莓派作为服务器建立动态博客


## 1. 前言

该工作为课程作业，主要目标为利用树莓派作为服务器，完成wordpress的配置，建立动态博客网站。详细的作业要求如下：

1. 制作raspbian系统的镜像并成功启动树莓派
2. 安装apche2，mariadb，php，phpmyadmin等软件
3. 安装wordpress
4. 登录wordpress后台，发送任一篇技术文章
5. 使用本地计算机完成对博客网站的访问

## 2. 树莓派启动

在官网下载[raspbian](https://www.raspberrypi.org/downloads/raspbian/)系统，利用「Win32DiskImager」软件将下载好的镜像写入准备好的SD卡。写入完成后，在boot目录下新建 `wpa_supplicant.conf` 文件，复制下面的内容到该文件并修改WIFI名和密码，保存该文件，这一步是为了在树莓派启动时令其自动连接到电脑。

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

然后同样在boot目录下新建名为`ssh`的文件，要注意小写且没有扩展名，从而开启SSH连接。

将配置好的SD卡从电脑卸载并以正确的方式插入树莓派，通电启动，在路由器后台查看新加入的名称中包含`raspberry`的设备，记录其ip地址，然后使用Putty通过该地址登录树莓派。

## 3. 软件安装

### apache2

```bash
$ sudo apt-get install -y apache2
$ apachectl -v
Server version: Apache/2.4.25 (Debian)
Server built:   2018-11-03T18:46:19
```

浏览器地址栏输入localhost，显示`It works!`界面

查看apache2状态

```bash
$ service apache2 status
● apache2.service - The Apache HTTP Server
   Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: 
   Active: active (running) since Thu 2019-03-14 09:48:04 CST; 2min 31s ago
 Main PID: 2788 (apache2)
    Tasks: 55 (limit: 4915)
   CGroup: /system.slice/apache2.service
           ├─2788 /usr/sbin/apache2 -k start
           ├─2869 /usr/sbin/apache2 -k start
           └─2870 /usr/sbin/apache2 -k start

3月 14 09:47:48 raspberry systemd[1]: Starting The Apache HTTP Server...
3月 14 09:48:04 raspberry apachectl[2709]: AH00558: apache2: Could not reliably d
3月 14 09:48:04 raspberry systemd[1]: Started The Apache HTTP Server.
lines 1-13/13 (END)
```

### mariadb

以下两条命令没什么区别，安装的软件包数量和名称都一样

```bash
$ sudo apt-get install -y mariadb-server        # 或下面这条，执行任一句即可
$ sudo apt-get install -y mysql-server mysql-client
$ mysql -V
mysql  Ver 15.1 Distrib 10.1.37-MariaDB, for debian-linux-gnu (i686) using readline 5.2
```

进入数据库设置密码，新建用户，授予权限

```bash
$ sudo mysql -u root -p          #按回车键输入密码
> create user 'shuzang'@'localhost' identified by '2427';      #按回车键
> grant all privileges on *.* to 'shuzang'@'localhost';      #给权限
> flush privileges;                     # 刷新权限
> show grants for 'shuzang'@'localhost';   #查看用户权限
> exit;
```

### php

```bash
$ sudo apt-get install -y php php-mysql php-fpm
$ php -v
PHP 7.0.33-0+deb9u3 (cli) (built: Mar  8 2019 10:01:24) ( NTS )
```

有讲解决PHP无法解析（只显示代码）的问题，需要安装php7.0-mysql和libapache2-mod-php7.0，但是查看上面这条命令安装的软件包你会发现这两个已经装过了，所以不管它，后面也证实了没有出现这个问题。

### phpmyadmin

```bash
$ sudo apt-get install -y phpmyadmin
```

安装时会出现一些选项，选择如下：

- Web server：apache2
- Configure database for phpmyadmin with dbconfig-common?：yes
- mysql application password for phpmyadmin：xxx   ('xxx'是自己输的密码)
- 再次输入密码确认

授予执行权限和开启rewrite模块

```bash
$ sudo chmod 777 /var/www/html
$ sudo a2enmod rewrite
```

把phpmyadmin软连接到/var/www/html

```bash
$ sudo ln -s /usr/share/phpmyadmin /var/www/html
```

重启apache2

```bash
$ sudo systemctl restart apache2
```

### wordpress

```bash
$ cd /var/www/html
$ sudo wget https://cn.wordpress.org/latest-zh_CN.tar.gz
$ sudo tar -zxvf latest-zh_CN.tar.gz
$ sudo mv wordpress liuyang
$ sudo chmod 777 liuyang
```

## 4. 建立动态博客

### 新建数据库

浏览器地址栏输入`localhost/phpmyadmin`，出现界面，输入数据库的用户名密码

进入数据库管理页面，新建数据库，输入数据库名

空数据库就不用管了，不用新建表

### 配置wordpress

浏览器地址栏输入`localhost/liuyang`，进入五分钟安装过程。

![填写用户名密码](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190313_3AOSOS.png)

填写博客信息

![填写博客信息](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190313_3AOkYn.png)

在最后一步选择「登录」按钮，跳转到登录界面登录

### 主题安装

登录后进入仪表盘，选择侧边栏「外观—>主题」，获取安装主题origami(可以自由选择其它主题)

```bash
$ cd /var/www/html/liuyang/wp-content/themes 
$ sudo wgethttps://github.com/syfxlin/origami/releases/download/v1.0.5/Origami-1.0.5.zip
$ sudo unzip Origami-1.0.5.zip
$ sudo rm -rf Origami-1.0.5.zip
```

回到主题页面刷新可以看到安装的主题，启用

### 发布文章

点「文章—>写文章」，然后写一篇文章，发布。可以点右上角预览按钮预览，也可以点左上角主页按钮进入博客网站主页。博客页面如下：

![博客页面预览](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190313_3AOKw4.png)

在本地计算机访问虚拟机中wordpress时，CSS无法加载，需要在仪表盘`设置—>常规`更改`wordpress地址`和`站点地址`，把两个地址中的localhost都改成虚拟机ip

![ip设置](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190313_3AO0kd.png)

保存设置，然后在本地计算机使用该地址访问，即可成功显示网站和博客文章

![最终显示效果](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190313_3AOcX8.png)

## 参考资料

[1] 简书-cyzyjin. [debian9](https://www.jianshu.com/p/fd9f3743f094) LAMP安装. 2018.12. 

[3] CSDN-霍莉雪特. [外网访问WordPress时无法加载样式表CSS](https://blog.csdn.net/qq_18995513/article/details/73012247). 2017.06.

[4] NEUQ金课行动. [2019创客实战训练营-11树莓派搭建WORDPRESS网站](https://www.bilibili.com/video/av39657396). 2019.01. 

[5] NEUQ金课行动. [使用虚拟机安装X86版本的raspbian](https://www.bilibili.com/video/av45274204). 2019.03. 


