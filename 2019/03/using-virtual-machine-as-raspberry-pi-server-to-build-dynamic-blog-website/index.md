# 使用虚拟机搭建树莓派服务器，并建立动态博客网站


## 1. 前言

主要目标为利用虚拟机搭建树莓派服务器，完成wordpress的配置，建立动态博客网站。详细需求为：

- 在虚拟机上安装并访问raspbian系统
- 安装apche2，mariadb，php，phpmyadmin等软件
- 安装wordpress
- 登录wordpress后台，发送任一篇技术文章
- 使用本地计算机完成对虚拟机中wordpress的访问

## 2. 虚拟机及raspbian

| 名称     | 版本信息                                                     | 下载                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 虚拟机   | 产品：VMware Workstation 15 Pro <br>版本：15.0.0 build-10134415 |                                                              |
| raspbian | version: November 2018<br>Release date: 2018-11-26<br>Kernel version: 4.9 | 镜像：[2018-11-26-rpd-x86-stretch.iso](https://www.raspberrypi.org/downloads/raspberry-pi-desktop/) |

利用虚拟机安装操作系统过程中一些注意事项如下

- 客户机操作系统选择`Linux`，版本选择`Debian 9.x`
- 磁盘大小分配50G，存储为单个文件
- 自定义硬件更改内存到2G，处理器内核数改到4

开启虚拟机后，进入`install`，一些选项按如下选择

- keyboard：American English
- Partition disks：use entire disk
- Install the GRUB boot loader on a hard disk：/dev/sda

## 3. 安装各软件

### 3.1 安装apache2

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

### 3.2 安装mariadb

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

### 3.3 安装php

```bash
$ sudo apt-get install -y php php-mysql php-fpm
$ php -v
PHP 7.0.33-0+deb9u3 (cli) (built: Mar  8 2019 10:01:24) ( NTS )
```

有讲解决PHP无法解析（只显示代码）的问题，需要安装php7.0-mysql和libapache2-mod-php7.0，但是查看上面这条命令安装的软件包你会发现这两个已经装过了，所以不管它，后面也证实了没有出现这个问题。

### 3.4 安装phpmyadmin

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

## 4. 安装wordpress

```bash
$ cd /var/www/html
$ sudo wget https://cn.wordpress.org/latest-zh_CN.tar.gz
$ sudo tar -zxvf latest-zh_CN.tar.gz
$ sudo mv wordpress liuyang
$ sudo chmod 777 liuyang
```

## 5. 在phpmyadmin中新建数据库

浏览器地址栏输入`localhost/phpmyadmin`，出现界面，输入数据库的用户名密码

进入数据库管理页面，新建数据库，输入数据库名

空数据库就不用管了，不用新建表

## 6. 配置wordpress

浏览器地址栏输入`localhost/liuyang`，进入五分钟安装过程

填写用户名密码什么的

![wordpress建立](https://user-images.githubusercontent.com/26682846/54510972-63314a80-498a-11e9-8ca8-f79f635a7b9a.png)

填写博客信息

![wordpress建立](https://user-images.githubusercontent.com/26682846/54511003-7cd29200-498a-11e9-9d0c-54378a850572.png)

## 7. 启动wordpress，安装主题，发布博客

在第6节最后一步选择`登录`按钮，跳转到登录界面登录，登录后进入仪表盘，选择侧边栏`外观—>主题`

获取安装主题origami

```bash
$ cd /var/www/html/liuyang/wp-content/themes 
$ sudo wgethttps://github.com/syfxlin/origami/releases/download/v1.0.5/Origami-1.0.5.zip
$ sudo unzip Origami-1.0.5.zip
$ sudo rm -rf Origami-1.0.5.zip
```

回到主题页面刷新可以看到安装的主题，启用

点`文章—>写文章`，然后写一篇文章，发布。可以点右上角预览按钮预览，也可以点左上角主页按钮进入博客网站主页。博客页面如下：

![wordpress建立 (3)](https://user-images.githubusercontent.com/26682846/54511049-996eca00-498a-11e9-9fc1-7b81f6218c61.png)

在本地计算机访问虚拟机中wordpress时，CSS无法加载，需要在仪表盘`设置—>常规`更改`wordpress地址`和`站点地址`，把两个地址中的localhost都改成虚拟机ip

![wordpress建立 (4)](https://user-images.githubusercontent.com/26682846/54511071-ac819a00-498a-11e9-80a5-05bc66e9793f.png)

保存设置，然后在本地计算机使用该地址访问，即可成功显示网站和博客文章

![wordpress建立 (5)](https://user-images.githubusercontent.com/26682846/54511087-bc00e300-498a-11e9-9ff1-778efd2722e4.png)

## 参考文献

[1] cyzyjin. debian9 LAMP安装. 2018.12. https://www.jianshu.com/p/fd9f3743f094

[2] cyzyjin. LAMP wordpress安装 debian9. 2018.11. https://www.jianshu.com/p/07118ec2cbf2

[3] 霍莉雪特. 外网访问WordPress时无法加载样式表CSS. 2017.06. https://blog.csdn.net/qq_18995513/article/details/73012247

[4] NEUQ金课行动. 2019创客实战训练营-11树莓派搭建WORDPRESS网站. 2019.01. https://www.bilibili.com/video/av39657396

[5] NEUQ金课行动. 使用虚拟机安装X86版本的raspbian. 2019.03. https://www.bilibili.com/video/av45274204
