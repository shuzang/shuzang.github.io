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

## 5. 图片插入

博客维护所面临的一个重要问题就是图床，一开始使用Github作为图床，后来迁移到Github issue，再后来因为图片经常加载不出来，迁移到了路过图床，不过用别的图床总归不放心，图片不多的情况下，最终决定将所有图片放到本地，操作步骤如下

首先，对于Hugo来说，所有的静态文件都应该位于根目录的static目录下，比如，我在该目录下存放了 `example.img` 图片，那么在文章中就可以用以下方式引用图片

```
![](/example.img)
```

这是因为静态文件的查找规则就是以 static 为根目录，所以图片从这里寻找。为了保证清晰的文件结构，便于之后寻找图片，我们采用如下规则存放图片：一篇文章的所有图片放在以文章名命名的一个文件夹中，该文件夹放在`static/images`目录下。如本文文件名为「hugo搭建个人博客3-进阶设置」，则本文所有图片放在 `static/images/hugo搭建个人博客3-进阶设置`目录下，在文章中插入图片时，使用的路径为

```
![](/images/example.img)
```

其中，example.img为图片名。至此为止，网页中可以正常显示插入的图片，但是Typora中无法显示图片，因此我们需要设置Typora的图片根目录，方法是在文章开头的YAML头中添加如下字段

```yaml
typora-root-url: ..\..\..\static
```

路径根据自己的文章位置而定，也可以在工具栏选择「格式—>图像—>设置图片根目录」，产生的效果是相同的，也是在文章头部添加该字段。

最后，我们在Typora的偏好设置中，设置插入图片时自动复制到相应的目录下即可，这样所有的图片都可以和博客代码存放在一起。、

这种方法唯一的缺点是所有图片和博客项目源码的总大小不能超过1G，因为一个Github项目空间最大1G

## 6. 待完成 

- RSS
- 站内搜索
- CDN，可参考[为博客添加免费的 CDN (Cloudflare)](https://mogeko.me/2019/056/)
- SEO


