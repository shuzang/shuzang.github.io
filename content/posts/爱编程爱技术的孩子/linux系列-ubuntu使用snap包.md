---
title: ubuntu使用snap包
author: Bboysoul
date: 2019-05-20T04:50:00+08:00
tags: [linux]
categories: [爱编程爱技术的孩子]
---

Ubuntu 16.04引入了snap包管理，它是一种全新的软件包安装管理方式，它类似一个容器拥有一个应用程序所有的文件和库，各个应用程序之间完全独立。所以使用snap包的好处就是它解决了应用程序之间的依赖问题，使应用程序之间更容易管理。但是由此带来的问题就是它占用更多的磁盘空间。snap软件包一般安装在/snap目录下。

本文主要是安装hugo时发现可以使用snap安装最新版，寻找相关snap说明如下。

> [snap官网地址](<https://snapcraft.io/>)
>
> 以下内容转载自[Bboysoul的博客：ubuntu使用snap包]([https://www.bboysoul.com/2017/11/15/ubuntu%20%E4%BD%BF%E7%94%A8snap%E5%8C%85/](https://www.bboysoul.com/2017/11/15/ubuntu 使用snap包/))



### 一些常用命令

其实使用 snap 包很简单，下面我来介绍一下一些常用的命令

- `sudo snap list`
  列出已经安装的 snap 包
- `sudo snap find <text to search>`
  搜索要安装的 snap 包
- `sudo snap install <snap name>`
  安装一个 snap 包
- `sudo snap refresh <snap name>`
  更新一个 snap 包，如果你后面不加包的名字的话那就是更新所有的 snap 包
- `sudo snap revert <snap name>`
  把一个包还原到以前安装的版本
- `sudo snap remove <snap name>`
  删除一个 snap 包

### 简单的使用

下面我就安装一个编辑器来演示怎么安装删除一个软件包
首先我想安装 hello-world
那么先找一下有没有 hello-world

<pre>➜  bin sudo snap search hello-world
Name                Version  Developer  Notes  Summary
hello-world         6.3      canonical  -      The 'hello-world' of snaps
hello-world-om26er  0.2      om26er     -      A great snap
hello-lhc           1.0      cprov      -      Hello world application for LHC

</pre>

没错有，那么我们就来安装

<pre>➜  ~ snap install hello-world
hello-world 6.3 from 'canonical' installed

</pre>

下载可能会很慢，所以最好挂代理
接着我们看一下有没有安装好

<pre>➜  bin snap list
Name         Version    Rev   Developer  Notes
core         16-2.28.5  3247  canonical  core
hello-world  6.3        27    canonical  -

</pre>

最后我们删除它
➜ ~ snap remove hello-world
hello-world removed

### snap 包的地址

如果你不知道可以下载什么 snap 包，你可以在下面的地址下载 snap 包来安装 [https://uappexplorer.com/snaps](https://uappexplorer.com/snaps)

### 一个报错

因为安装 douban-fm 的时候时间太长了，我就终止了一下这个操作，之后再次安装这个包的时候就报了一个错误
`error: cannot install "douban-fm": snap "core" has changes in progress`

解决方式很简单
首先查看一下正在进行的 change

<pre>➜  / snap changes
ID   Status  Spawn                 Ready                 Summary
2    Done    2017-11-15T02:33:51Z  2017-11-15T02:33:51Z  Refresh all snaps: no updates
3    Error   2017-11-15T03:20:07Z  2017-11-15T03:20:23Z  Install "douban-fm" snap
4    Done    2017-11-15T03:20:07Z  2017-11-15T03:20:10Z  Initialize device
5    Error   2017-11-15T03:20:38Z  2017-11-15T03:34:21Z  Install "douban-fm" snap
6    Doing   2017-11-15T03:34:27Z  -                     Install "douban-fm" snap

</pre>

没错 install douban-fm 还在 doing 中，所以要终止这个进程

<pre>➜  / sudo snap abort 6
➜  / snap changes
ID   Status  Spawn                 Ready                 Summary
2    Done    2017-11-15T02:33:51Z  2017-11-15T02:33:51Z  Refresh all snaps: no updates
3    Error   2017-11-15T03:20:07Z  2017-11-15T03:20:23Z  Install "douban-fm" snap
4    Done    2017-11-15T03:20:07Z  2017-11-15T03:20:10Z  Initialize device
5    Error   2017-11-15T03:20:38Z  2017-11-15T03:34:21Z  Install "douban-fm" snap
6    Error   2017-11-15T03:34:27Z  2017-11-15T03:40:51Z  Install "douban-fm" snap

</pre>

之后再次执行安装就好了

