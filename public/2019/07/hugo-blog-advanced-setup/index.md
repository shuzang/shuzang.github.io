# hugo搭建个人博客3-进阶设置


本篇文章介绍如何为Hugo开启更多的功能。

## 1. 字数统计

LeaveIt本身在博文末尾提供了字数统计功能，使用的是`.WordCount`模板变量，但并不统计中文([详情](https://github.com/nodejh/hugo-theme-cactus-plus/issues/18))，所以显示的字数总是很小。可以通过在`config.toml`中添加如下语句启用。

```toml
hasCJKLanguage = true
```

## 2. 开启评论

使用gitalk，首先在GitHub 上创建一个 [Github Application](https://github.com/settings/applications/new)，记录 `Client ID` 和 `Client Secret`

然后编辑`config.toml`以下字段

```toml
[params.gitalk]
owner = "shuzang"       # 你的GitHub ID       
repo = "blog"        # 博客网址    
clientId = ""    # 刚刚记录的client ID      
clientSecret = "" # 刚刚记录的client secret
labels = "gitalk"
```

提交到仓库后就能看到博文下开启了评论区

## 3. 打赏

有些主题支持加入打赏图片，但KeepIt不支持，不过这里还是记录一下。利用[第九工厂](<http://www.9thws.com/#>)的免费模板制作好看的打赏二维码。

## 4. 英文URL

文章分享和访问时，中文URL是最麻烦的一件事情，为了解决这个问题，我阅读了一遍hugo文档中的[URL Management](https://gohugo.io/content-management/urls/)，最终决定使用如下办法。

首先修改站点配置文件`config.toml`中的`Permalinks`字段如下

```toml
[Permalinks]
 posts = "/:year/:month/:slug/"
```

初始状态该字段值为`posts = "/:year/:month/:filename/"`，使用文件名作为URL，但是文件名作为管理和查找的手段，使用英文较为不便，而文章标题为了便于阅读，也不方便命名为英文，只能使用额外的`slug`字段。

之后在每篇文章头部的配置信息中添加`slug`字段即可，一个例子如下

```toml
title: Golang语法基础100问
date: 2019-12-23
slug: "100 question of Golang basic syntax"
tags: [Golang]
categories: [爱编程爱技术的孩子]
```

这篇文章最后的URL将会是` http://shuzang.github.io/2019/12/100-question-of-golang-basic-syntax/ `

## 5. 待完成 

- RSS
- 站内搜索
- CDN，可参考[为博客添加免费的 CDN (Cloudflare)](https://mogeko.me/2019/056/)
- SEO


