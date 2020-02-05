---
title: linux系统时间同步
date: 2019-12-20
tags: [linux]
categories: [爱编程爱技术的孩子]
---

虚拟机长时间不开机，系统时间和当前时间不同步，导致很多操作被拒绝，这里记录如何主动同步系统的时间和网络时间

## 正文

设置系统时区

```bash
$ timedatectl set-timezone Asia/Shanghai
```

安装ntpdate工具

```bash
$ sudo apt-get install ntpdate
```

同步时间

```bash
$ sudo ntpdate cn.pool.ntp.org
```

将系统时间写入硬件时间

```bash
$ sudo hwclock --systohc
```

ntpdate cn.pool.ntp.org是位于中国的公共NTP服务器