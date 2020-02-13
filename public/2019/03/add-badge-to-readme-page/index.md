# Git学习 README页面添加徽章


## 前言

我们在逛github的时候，经常能在README.md页面看到如下所示的徽章，通常展示了项目的相关信息，这种形式比单纯的文字描述更加吸引人，今天就来学一学如何在项目中插入这些徽章。

![编译进行中](https://img.shields.io/badge/build-passing-brightgreen.svg)![下载](https://img.shields.io/badge/downloads-120%2Fweek-green.svg)![协议](https://img.shields.io/badge/license-MIT-green.svg)![支持平台](https://img.shields.io/badge/platform-linux--64%20%7C%20win--32%20%7C%20osx--64%20%7C%20win--64-lightgrey.svg)

<!--more-->

## 徽章简介

GitHub 项目的 README.md 中可以添加徽章（Badge）对项目进行标记和说明，这些好看的小图标不仅简洁美观，而且还包含了清晰易读的信息。

这些徽标的本质仍然是图片，并没有脱离markdwon语法的限制。其基本原理是，徽标的官方网站[shields.io](https://shields.io/)提供了一批“标签小程序”，它们可以抓取一些github项目的动态数据并自动生成标签图片，比如抓取github上项目的最新release版本号生成release标签等。使用这种标签能够保证每次刷新网页都会重新抓取数据，并且更新标签上的文字，这样就实现了动态变化的徽章标签。

徽标图片的话一般由左半部分的名称和右半部分的值组成，徽章则主要由图片和对应的链接（链接可以不填）组成，如下所示，正是github上git项目的唯一徽标的格式，前半部分的方括号里的是图片，后半部分的圆括号里的是链接。

```bash
[![Build Status](https://dev.azure.com/git/git/_apis/build/status/git.git)](https://dev.azure.com/git/git/_build/latest?definitionId=11)
```

也可以点击[这里](https://github.com/git/git/blob/master/README.md)看一下Git项目中该徽章表现形式，当然，正式使用时可以在官网[shields.io](https://shields.io/)中可以预览徽标样式，然后选择自己喜欢的徽标添加到项目中。

**注**：徽章不是添加的越多越好，因为徽章的根本作用还是清晰易懂的说明项目相关信息，合理地选择适合项目地徽标做针对性地添加才是理性地做法。

## 徽标添加

我们进入[shields.io](<https://shields.io/category/platform-support>)的`Version`标签页，点开GitHub release这一条的链接，如下：

![Github release](https://shikieiki.github.io/image/20170301152451.png)

会看到如下界面

![徽章生成](https://user-images.githubusercontent.com/26682846/55003924-404f0800-5014-11e9-956e-cbfe33a3ea0b.png)

在which中选择`release`或`release-pre`，在user下填入用户名，在repo下填入项目名，以Tencent的tinker项目为例，徽章预览如图所示，点击下面的Copy Badge URL即可复制链接。然后放到README中使用即可。

**注**：style中可以选择徽章形式，链接复制有适用于markdown或其它文本等四种形式。

## 自定义徽章

如果想要生成的徽章字样和颜色shields没有，[shields.io](<https://shields.io/>)也支持自定义一个自己想要的徽章，从主页拉到`Your Badge`部分，如下图

![自定义徽章](https://user-images.githubusercontent.com/26682846/55003977-56f55f00-5014-11e9-8758-33c21b13fee1.png)

在上图中的框中填入相关信息，三条横线从前到后依次是`label`,`message`,`color`，color有选项可以选择，上图的预览样式如下：

![预览](<https://img.shields.io/badge/Hey!-shuzang-red.svg>)

## 常用徽标

### 1. 项目下载量

项目被下载地次数，各平台统计独立，详见 [shields.io](<https://shields.io/category/platform-support>) 的 `Downloads` 一栏，图标示例如下

![项目下载量](https://img.shields.io/badge/downloads-2M-brightgreen.svg)

### 2. 项目支持平台

详见 [shields.io](<https://shields.io/category/platform-support>) 的 `Platform & Version Support` 一栏，图标示例如下

![支持的平台](https://img.shields.io/badge/platform-linux--64%20%7C%20win--32%20%7C%20osx--64%20%7C%20win--64-lightgrey.svg)

### 3. 项目语言

即项目所用编程语言，通过自定义徽标实现，图标示例如下：

![编程语言](https://img.shields.io/badge/language-swift-orange.svg)

### 4. 开源协议类型

详见 [shields.io](<https://shields.io/category/platform-support>) 的 `License` 一栏，图标示例如下

![开源协议](https://img.shields.io/badge/license-MIT-green.svg)

还有其它很多，shields首页标签栏从`build`,`Downloads`到`Other`共提供了17类，还可以自定义标签。

<br>

## 参考文献

[1] EyreFree. GitHub项目徽章地添加与设置. https://juejin.im/post/5a32157c6fb9a0450b6667ac. 2017.12.

[2]  AnSwEr不是答案. Github徽章整理. https://blog.csdn.net/u011192270/article/details/51788886. 2016.06.

[3] ShikiEiki. 为你地Github README生成漂亮地徽章和进度条. 2017.03.

[4] Shields项目. https://github.com/badges/shields.
