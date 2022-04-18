---
title: U盘只读状态清除
date: 2021-08-05
tags: [Win10]
categories: [爱编程爱技术的孩子]
slug: clear USB falsh disk read-only status
---

U盘变只读是经常出现的情况，以前都是使用写保护开关，一旦没有开关就毫无办法，今天又遇到这种情况，解决问题后这里总结一下解决办法。

<!--more-->

## 1. 检查写保护开关

某些情况下，U盘 或 SD 卡侧面或底部有一个可以拨动的开关，即写保护开关，如果发现 U盘变只读，先尝试拨动这里解锁。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20210805_U盘解除只读.jpg)

## 2. Diskpart 命令

虽然傲梅和 DiskGenius 等软件都有解除只读状态的功能，但我从来没有成功过，所以直接介绍使用 Diskpart 解除只读状态的方法。

```powershell
# 以管理员权限打开 powershell
# 进入 diskpart 执行状态
> diskpart
# 首先获取计算机中所有磁盘列表
> lisk disk 
# 根据磁盘大写识别哪个是U盘，查看首列的磁盘编号，选中
> select disk 1
# 清除只读状态
> attr disk clear readonly
```

有时候U盘无法显示在资源管理器中，但磁盘管理中可以看到，这时候一般是由于其处于脱机状态，同样使用 diskpart 命令可以更改状态为联机，前三条命令相同，最后一条命令为

```powershell
> online disk
```

