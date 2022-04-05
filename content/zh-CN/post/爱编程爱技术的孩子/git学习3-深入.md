---
title: Git深入-子模块, 徽章, 开源协议
date: 2019-09-30
lastmod: 2020-10-07
tags: [git]
categories: [爱编程爱技术的孩子]
slug: Git learning deep into 
---

学习一些更高级或更有趣的 Git 用法，包括子模块、徽章、开源协议选择等。

<!--more-->

## 1. 开源协议

代码的开源协议比较多，这里附一张阮一峰大神的图片说明如何选择。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190930_free_software_licenses.png)

## 2. 子模块

当我们在一个 Git 项目上工作时，有时候需要在其中使用另外一个 Git 项目。这个情况可以在 Git 中使用子模块 `submodule` 来进行管理。`submodule` 允许我们将一个 Git 仓库当作另外一个 Git 仓库的子目录。这允许我们克隆另外一个仓库到自己的项目中并且保持我们的提交相对独立。

Hugo 的博客源码文件中，主题的源码就是作为子模块进行添加和管理。本文中以本地项目 blog 为例，将远程主题项目 LoveIt 作为子模块进行管理。

### 2.1 添加子模块

将远程项目 `LoveIt` 克隆到本地 `blog/themes` 文件夹，`blog` 需要本身就是一个 Git 项目

```bash
$ git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

添加子模块后运行`git status`, 可以注意增加了一个文件 `.gitmodules`,

```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   themes/LoveIt
```

这个文件用来保存子模块的信息。

```bash
[submodule "themes/LoveIt"]
	path = themes/LoveIt
	url = https://github.com/dillonzq/LoveIt.git
```

### 2.2 查看子模块

```bash
$ git submodule
 bf7c4b5173c3baba02b87a410ce04909c1b86cf6 themes/LoveIt (v0.1.4)
```

### 2.3 更新子模块

作为子模块的主题经常需要追随远程更新，这也是最常见的一种情况，即只使用子项目并不时获取更新，而不在子项目中做任何更改。

最容易的方式是添加参数 `--remote`，Git 会进入子模块然后抓取并更新

```bash
$ git submodule update --remote
```

该命令假设我们想要更新子模块仓库的 `master` 分支，如果想要设置为其它的分支，可以在 `.gitmodules` 中进行设置

```bash
$ git config -f .gitmodules submodule.themes/LoveIt.branch develop
$ git submodule update --remote
Submodule path 'themes/LoveIt': checked out 'b6ce753ae7892839899962b879e2cc5808c60732'
```

使用 `git diff` 可以查看到变动，`.gitmodules`多了一行，子模块的分支号有了变动

```bash
$ git diff
diff --git a/.gitmodules b/.gitmodules
index a6f32d0..5376abf 100644
--- a/.gitmodules
+++ b/.gitmodules
@@ -1,3 +1,4 @@
 [submodule "themes/LoveIt"]
        path = themes/LoveIt
        url = https://github.com/dillonzq/LoveIt.git
+       branch = develop
diff --git a/themes/LoveIt b/themes/LoveIt
index bf7c4b5..b6ce753 160000
--- a/themes/LoveIt
+++ b/themes/LoveIt
@@ -1 +1 @@
-Subproject commit bf7c4b5173c3baba02b87a410ce04909c1b86cf6
+Subproject commit b6ce753ae7892839899962b879e2cc5808c60732
```

当运行 `git submodule update --remote` 时，Git 默认会尝试更新 **所有** 子模块， 所以如果有很多子模块的话，你可以传递想要更新的子模块的名字。比如

```bash
$ git submodule update --remote themes/LoveIt
```

### 2.4 克隆包含子模块的项目

如果像下面这样直接克隆包含子模块的项目，虽然有子模块目录，但是是空的

```bash
 $ git clone https://github.com/shuzang/blog.git
 $ cd blog
 $ ls themes/LoveIt
 $
```

查看子模块时前面有一个`-`，也说明子模块文件还未检入（空文件夹）。

```bash
$ git submodule
-87c33888f3fa86b8cc096bc3f6d7f2efe9ccba66 themes/KeepIt
```

此时需要运行两个命令，`git submodule init` 用来初始化子模块

```bash
$ git submodule init
Submodule 'themes/KeepIt' (https://github.com/Fastbyte01/KeepIt.git) registered for path 'themes/KeepIt'
```

 `git submodule update` 用来抓取数据

```bash
$ git submodule update
Cloning into 'C:/Users/lylw1/Desktop/blog/themes/KeepIt'...
Submodule path 'themes/KeepIt': checked out '87c33888f3fa86b8cc096bc3f6d7f2efe9ccba66'
```

不过有一个更简单的方法，就是给 `git clone` 命令传递 `--recurse-submodules` 参数，这样就会自动初始化并更新仓库中每一个子模块，包括可能存在的嵌套子模块

```bash
 $ git clone https://github.com/shuzang/blog.git --recurse-submodules
```

### 2.5 修改子模块

在子模块中修改文件后，直接提交到远程项目分支。

```bash
$ git add .
$ git commit -m "modify submodule"
$ git push origin master
```

### 2.6 删除子模块

之前从网上找到的办法是，手动删除相关的文件，以删除`KeepIt`文件夹为例

1.  删除子模块文件夹
    
    ```bash
    $ git rm --cached themes/KeepIt
	$ rm -rf themes/KeepIt
    ```
    
1.  删除`.gitmodules`文件中相关子模块信息
    
    ```bash
    [submodule "themes/KeepIt"]
	path = themes/KeepIt
    	url = https://github.com/Fastbyte01/KeepIt.git
    ```
    
1.  删除`.git/config`中的相关子模块信息
    
    ```bash
    [submodule "themes/KeepIt"]
	url = https://github.com/Fastbyte01/KeepIt.git
    ```
    
1.  删除`.git`文件夹中的相关子模块文件
    
    ```bash
    $ rm -rf .git/modules/themes/KeepIt
    ```
    

不过实际上，这是一种比较野的做法，不建议使用。根据官方文档的说明，应该使用 `git submodule deinit` 命令卸载一个子模块。这个命令如果添加上参数 `--force`，则子模块工作区内即使有本地的修改，也会被移除。

```bash
$ git submodule deinit themes/LoveIt
$ git rm themes/LoveIt
```

执行 `deinit` 的效果，是自动在 `.git/config` 中删除了如下信息

```
[submodule "themes/LoveIt"]
	url = https://github.com/dillonzq/LoveIt.git
```

执行 `rm` 的效果，是移除了 `themes/LoveIt` 文件夹，并删除了 `.gitmodules` 中的相关内容

此时关于子模块的信息基本已经删除完毕，只剩 `.git/modules` 下还有一些残留，但执行 `commit` 后残留会被删除

```bash
$ git commit -m "delete submodule themes/LoveIt"
```

## 3. 徽章

逛github的时候，经常能在 README.md 页面看到如下所示的徽章，通常展示了项目的相关信息，这种形式比单纯的文字描述更加吸引人，今天就来学一学如何在项目中插入这些徽章。

![编译进行中](https://img.shields.io/badge/build-passing-brightgreen.svg)![下载](https://img.shields.io/badge/downloads-120%2Fweek-green.svg)![协议](https://img.shields.io/badge/license-MIT-green.svg)![支持平台](https://img.shields.io/badge/platform-linux--64%20%7C%20win--32%20%7C%20osx--64%20%7C%20win--64-lightgrey.svg)

### 3.1 徽章简介

GitHub 项目的 README.md 中可以添加徽章（Badge）对项目进行标记和说明，这些好看的小图标不仅简洁美观，而且还包含了清晰易读的信息。

这些徽标的本质仍然是图片，并没有脱离markdwon语法的限制。其基本原理是，徽标的官方网站[shields.io](https://shields.io/)提供了一批“标签小程序”，它们可以抓取一些github项目的动态数据并自动生成标签图片，比如抓取github上项目的最新release版本号生成release标签等。使用这种标签能够保证每次刷新网页都会重新抓取数据，并且更新标签上的文字，这样就实现了动态变化的徽章标签。

徽标图片的话一般由左半部分的名称和右半部分的值组成，徽章则主要由图片和对应的链接（链接可以不填）组成，如下所示，正是github上git项目的唯一徽标的格式，前半部分的方括号里的是图片，后半部分的圆括号里的是链接。

```bash
[![Build Status](https://dev.azure.com/git/git/_apis/build/status/git.git)](https://dev.azure.com/git/git/_build/latest?definitionId=11)
```

也可以点击[这里](https://github.com/git/git/blob/master/README.md)看一下Git项目中该徽章表现形式，当然，正式使用时可以在官网[shields.io](https://shields.io/)中可以预览徽标样式，然后选择自己喜欢的徽标添加到项目中。

**注**：徽章不是添加的越多越好，因为徽章的根本作用还是清晰易懂的说明项目相关信息，合理地选择适合项目地徽标做针对性地添加才是理性地做法。

### 3.2 徽章添加

我们进入[shields.io](<https://shields.io/category/platform-support>)的`Version`标签页，点开GitHub release这一条的链接，如下：

![Github release](https://shikieiki.github.io/image/20170301152451.png)

会看到如下界面

![徽章生成](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190930_3AoAKJ.png)

在which中选择`release`或`release-pre`，在user下填入用户名，在repo下填入项目名，以Tencent的tinker项目为例，徽章预览如图所示，点击下面的Copy Badge URL即可复制链接。然后放到README中使用即可。

**注**：style中可以选择徽章形式，链接复制有适用于markdown或其它文本等四种形式。

### 3.3 自定义徽章

如果想要生成的徽章字样和颜色shields没有，[shields.io](<https://shields.io/>)也支持自定义一个自己想要的徽章，从主页拉到`Your Badge`部分，如下图

![自定义徽章](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190930_3Aon56.png)

在上图中的框中填入相关信息，三条横线从前到后依次是`label`,`message`,`color`，color有选项可以选择，上图的预览样式如下：

![预览](<https://img.shields.io/badge/Hey!-shuzang-red.svg>)

### 3.4 常用徽章

#### 项目下载量

项目被下载地次数，各平台统计独立，详见 [shields.io](<https://shields.io/category/platform-support>) 的 `Downloads` 一栏，图标示例如下

![项目下载量](https://img.shields.io/badge/downloads-2M-brightgreen.svg)

#### 项目支持平台

详见 [shields.io](<https://shields.io/category/platform-support>) 的 `Platform & Version Support` 一栏，图标示例如下

![支持的平台](https://img.shields.io/badge/platform-linux--64%20%7C%20win--32%20%7C%20osx--64%20%7C%20win--64-lightgrey.svg)

#### 项目语言

即项目所用编程语言，通过自定义徽标实现，图标示例如下：

![编程语言](https://img.shields.io/badge/language-swift-orange.svg)

#### 开源协议类型

详见 [shields.io](<https://shields.io/category/platform-support>) 的 `License` 一栏，图标示例如下

![开源协议](https://img.shields.io/badge/license-MIT-green.svg)

还有其它很多，shields首页标签栏从`build`,`Downloads`到`Other`共提供了17类，还可以自定义标签。

## 参考文献

[1] EyreFree. GitHub项目徽章地添加与设置. https://juejin.im/post/5a32157c6fb9a0450b6667ac. 2017.12.

[2]  AnSwEr不是答案. Github徽章整理. https://blog.csdn.net/u011192270/article/details/51788886. 2016.06.

[3] ShikiEiki. 为你地Github README生成漂亮地徽章和进度条. 2017.03.

[4] Shields项目. https://github.com/badges/shields.

[5] 知乎-孤单彼岸. [Git中submodule的使用](https://zhuanlan.zhihu.com/p/87053283?utm_source=cn.ticktick.task&utm_medium=social&utm_oi=1153472097597743104). 2019-10-17

[6] Git文档. [Git工具-子模块](https://git-scm.com/book/zh/v2/Git-工具-子模块). 2nd Edition.

