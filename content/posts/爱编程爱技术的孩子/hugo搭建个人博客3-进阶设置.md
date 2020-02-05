---
title: hugo搭建个人博客3-进阶设置
date: 2019-07-12
lastmod: 2020-01-07
tags: [博客写作]
categories: [爱编程爱技术的孩子]

slug: Hugo blog advanced setup
featured_image: https://user-images.githubusercontent.com/26682846/71819349-a057fc00-30c6-11ea-8634-e43a4d103b19.jpg
---

hugo虽然生成速度快，管理起来也简单，但是在主题方面真是一言难尽。当初因为emoji的支持问题从Even换到LeaveIt，又换到KeepIt，但用到现在还是发现很多需求无法满足，这里总结如下，慢慢去改进

- ~~中文URL显示问题~~
- RSS无法正确加载文章的问题
- 添加文章目录栏
- 代码高亮格式调整
- 表格宽度自动调整使铺面页面
- 支持基于 [mermaid](https://github.com/knsv/mermaid) 的图表和流程图生成功能
- 支持对文章的搜索
- ~~支持文章更新日期显示~~
- ~~支持使用数学公式~~
- ~~支持内嵌Bilibili视频~~
- ~~支持基于 [APlayer](https://github.com/MoePlayer/APlayer) 和 [MetingJS](https://github.com/metowolf/MetingJS) 的内嵌音乐播放器~~
- CDN缓存优化，加快网站在国内的加载速度
  - 参考[为博客添加免费的 CDN (Cloudflare)](https://mogeko.me/2019/056/)
- 提交各大搜索引擎收录
- ~~持续集成推送到shuzang.github.io仓库~~
- ~~评论系统集成（powered by gitalk）~~

## 1. 代码高亮

KeepIt主题本身对代码高亮的渲染是在本地执行的，使用的是[google code prettify](https://github.com/google/code-prettify)，高亮风格少的可怜，何况作者还没提供对高亮风格的选择。因此打算删除这部分内容，通过JavaScript直接在浏览器里进行渲染，使用`Highlight.js`库。

为了最大程度的保持原主题文件的代码，我们首先将主题文件夹下`layouts/partials/js.html`文件复制到Hugo项目文件夹，位于同样的目录。

删除`js.html`中以下语句

```html
{{ $prettify := resources.Get "/js/prettify.min.js" }}
```

然后删除`js.html`文件中所有`$prettify`字段。

接下来复制主题文件夹`layouts/partials/head.html`文件到项目文件夹，并添加如下内容

```html
<link href="https://cdn.bootcss.com/highlight.js/9.15.10/styles/atom-one-light.min.css" rel="stylesheet">
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/highlight.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/go.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

以上代码使用了[bootCDN](https://www.bootcdn.cn/highlight.js/)对`Highlight.js`的支持，并额外添加了对go语言代码高亮的支持。

最后需要删除主题文件夹中的某些内容，包括

1. `assets/js/prettify.min.js`文件

2. `assets/css/main.scss`的以下内容

    ```scss
   @import "_common/_prettyprint/default.scss"
   ```

此时代码就可以以浅色的atom格式高亮了，不过可惜的是不显示代码行数，就只能等以后再说了。

还要注意的是以后每次更新主题都要删除主题文件夹中以上两个内容。

## 2. 文章特性图片

实际上这是本身就支持的一个功能，只是很少用。方法是在原本文章的元数据部分新添`feature_image`和`description`字段，前者粘贴图片URL，后者书写图片描述。一个添加特性图片后的文章元数据字段内容如下：

```markdown
title: 在博客中内嵌bilibili视频
date: 2019-11-12
tags: [博客写作]
categories: [爱编程爱技术的孩子]
featured_image: https://i.vimeocdn.com/video/771515958.jpg?mw=1920&mh=1080&q=70
description: 悲剧傻缺视频集锦的某次截图
```

## 3. 内嵌视频

内嵌视频可以直接使用html代码，各视频网站分享链接中一般会提供该代码，如以下代码内嵌一个bilibili最近的八尾妖姬的悲剧傻缺视频集锦。

```html
<iframe src="//player.bilibili.com/player.html?aid=75064361" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="width: 100%;height: 600px;" > </iframe>
```

效果如下，添加bilibili的视频只需要把`aid`后面的数字改成视频网址最后的数字即可。

<iframe src="//player.bilibili.com/player.html?aid=75064361" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="width: 100%;height: 600px;" > </iframe>
实际上也可以使用短代码来完成这一功能，无需书写复杂的HTML代码，之后介绍。要注意的是，引入视频会导致网页加载速度大幅下滑，如非必要，最好不要引入。

## 4. 内嵌音乐播放器

使用 [APlayer](https://github.com/MoePlayer/APlayer) 和 [MetingJS](https://github.com/metowolf/MetingJS) 可以在博客中内嵌音乐播放器

首先在head.html中添加如下内容

```html
<!-- require APlayer -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.js"></script>
<!-- require MetingJS -->
<script src="https://cdn.jsdelivr.net/npm/meting@2/dist/Meting.min.js"></script>
```

然后在想添加播放器的博文中使用如下语句

```html
<meting-js auto="https://y.qq.com/n/yqq/song/000xRG4E4diqXD.html"></meting-js>
```

效果如下

<meting-js auto="https://y.qq.com/n/yqq/song/000xRG4E4diqXD.html"></meting-js>

需要在浏览器从各音乐播放平台获取播放链接，当然，像视频一样使用iframe也可以。

## 5. 使用Shortcode简化书写

Hugo写文章的主要格式是Markdown，但是很多高级的语法默认的渲染引擎是不支持的，需要使用纯HTML代码来编写，这和Markdown的本意是违背的。因此Hugo提供了[Shortcodes](https://gohugo.io/templates/shortcode-templates/)来提供对这些语法的支持。

首先，Shortcodes的模板文件应该放在`layouts/shortcodes`文件夹下，每个模板作为一个子文件存在。以下我们添加对内嵌视频、Mermaid流程图的短代码支持，分别命名为`audio.html`，`video.html`，`mermaid.html`

短代码的调用方法为将代码放在两个大括号和一个尖括号的包围中，代码本身应与两侧尖括号有一个空格分隔

### 5.1 音频

我们之前已经使用 [APlayer](https://github.com/MoePlayer/APlayer) 和 [MetingJS](https://github.com/metowolf/MetingJS) 在博客中内嵌音乐播放器，这次更改为短代码主要为了简化书写。

编辑audio.html文件如下

```html
<meting-js auto={{ .Get 0 }}></meting-js>
```

使用如下引号中的代码填充短代码调用内容，参数填充音频网址，即从网易云或QQ音乐网站对应歌曲复制的url

```diff
audio  "https://y.qq.com/n/yqq/song/000xRG4E4diqXD.html"
```

### 5.2 视频

主要针对bilibili视频（因为后续可能添加对其它网站视频的支持，故文件名为video.html），video.html文件编辑如下

```html
<iframe src="//player.bilibili.com/player.html?aid={{ .Get 0 }}" 
        scrolling="no" border="0" frameborder="no" framespacing="0" 
        allowfullscreen="true" 
        style="width: 100%;height: 600px;" > 
</iframe>
```

使用如下代码填充短代码调用内容，参数填充视频代码，即URL最后的一串数字

```markdown
video 75064361
```

另外，Hugo本身就提供了一些Shortcodes（[Build-in](https://gohugo.io/content-management/shortcodes/)），例如

### 5.3 图片

markdown图片语法的扩展，一个填充的参数例子如下

```markdown
figure src="https://cn.bing.com/th?id=OIP.Z0rMYDbFTqFcFNh1bEQSDQHaEo&pid=Api&rs=1" title="Earth"
```

使用一些预置的参数对图片进行管理，常用参数包括

- src：要显示的图片的URL
- title：图片名
- caption：图片说明
- height：图片高度
- width：图片宽度

以上例子效果如下

{{< figure src="https://cn.bing.com/th?id=OIP.Z0rMYDbFTqFcFNh1bEQSDQHaEo&pid=Api&rs=1" title="Earth">}}

### 5.4 gist

指的是Github gists，如果一个gist的url如下

```diff
https://gist.github.com/spf13/7896402
```

那么可以使用下面的填充参数

```markdown
gist spf13 7896402
```

如果gist包括多个文件但我们只想用一个，可以传入文件名作为第三个参数

```markdown
gist spf13 7896402 "img.html"
```

## 6. 数学公式

总结论文的时候出现数学公式简直是再寻常不过的事了，但是Hugo本身不提供对这种语法的支持，KeepIt主题也没有提供这个功能，因此需要自己添加。

主要方法是在已经生成好的HTML页面中使用JavaScript来渲染LaTex形式的数学公式，因此选择了这方面最流行的库[MathJax](https://www.mathjax.org/)。

与代码相同，数学公式也有行内(inline)和区块(block)两种，下面展示了这两种写法

```latex
When $a \ne 0$, there are two solutions to `\(ax^2 + bx + c = 0\)` and they are:

$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}$$
```

Hugo官方其实给了一个[解决办法](https://gohugo.io/content-management/formats/#enable-mathjax)，分为三步：

1. 引入`<script>`标签

    ```js
   <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
   </script>
   ```

    需要确保这段代码嵌入网站所有页面中，因此把它添加到`layouts/partials/head.html`文件中。

2. 行内公式和特殊字符转义

    第一步执行完后行内公式依然无法显示，如下划线`_`等特殊字符转义也有问题。因此需要继续添加如下代码，仍然是`layouts/partials/head.html`文件，两段代码前者做了一些配置，后者自动给`className`加`has-jax`后缀。

   ```js
   <script type="text/x-mathjax-config">
   MathJax.Hub.Config({
     tex2jax: {
       inlineMath: [['$','$'], ['\\(','\\)']],
       displayMath: [['$$','$$'], ['\[','\]']],
       processEscapes: true,
       processEnvironments: true,
       skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
       TeX: { equationNumbers: { autoNumber: "AMS" },
            extensions: ["AMSmath.js", "AMSsymbols.js"] }
     }
   });
   </script>
   
   <script type="text/x-mathjax-config">
     MathJax.Hub.Queue(function() {
       // Fix <code> tags after MathJax finishes running. This is a
       // hack to overcome a shortcoming of Markdown. Discussion at
       // https://github.com/mojombo/jekyll/issues/199
       var all = MathJax.Hub.getAllJax(), i;
       for(i = 0; i < all.length; i += 1) {
           all[i].SourceElement().parentNode.className += ' has-jax';
       }
   });
   </script>
   ```

3. CSS调整

    最后还需要在CSS中对这种特殊的MathJax进行样式处理，否则行内公式的显示会很奇怪。

   ```css
   code.has-jax {
       font: inherit;
       font-size: 100%;
       background: inherit;
       border: inherit;
       color: #515151;
   }
   ```

然后以上方法步骤繁琐最后还不一定起作用，因此采用把所有修改写成一个`layouts/partials/mathjax.html`文件的方法，把官方提到的三处修改合并成一个partial，然后把MathJax的CDN修改到国内。

```html
<script type="text/javascript"
        async
        src="https://cdn.bootcss.com/mathjax/2.7.3/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\\(','\\)']],
    displayMath: [['$$','$$'], ['\[','\]']],
    processEscapes: true,
    processEnvironments: true,
    skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
    TeX: { equationNumbers: { autoNumber: "AMS" },
         extensions: ["AMSmath.js", "AMSsymbols.js"] }
  }
});

MathJax.Hub.Queue(function() {
    // Fix <code> tags after MathJax finishes running. This is a
    // hack to overcome a shortcoming of Markdown. Discussion at
    // https://github.com/mojombo/jekyll/issues/199
    var all = MathJax.Hub.getAllJax(), i;
    for(i = 0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
    }
});
</script>

<style>
code.has-jax {
    font: inherit;
    font-size: 100%;
    background: inherit;
    border: inherit;
    color: #515151;
}
</style>
```

最后把以上partial模板添加到`head.html`模板中

```html
{{ partial "mathjax.html" . }}
```

但直接修改主题中的文件并不合适，因为原作者可能会进行一定的更新（对我这样的半吊子很依赖作者对主题的更新），好在Hugo对文件的寻找是有优先级的，会首先在项目文件中寻找相应的渲染模板，然后才去主题文件夹寻找，因此将`layouts/partials/mathjax.html`和`layouts/partials/head.html`这两个修改后的文件复制到Hugo项目的相同位置，此时就可以直接反馈在页面上了。此时在hugo项目根目录查看目录结构如下：

```bash
$ ls
archetypes/  content/  layouts/    static/
config.toml  data/     resources/  themes/

$ find layouts -type f | xargs ls -l
-rw-r--r-- 1 lylw1 197609 2679 11月 12 15:51 layouts/partials/head.html
-rw-r--r-- 1 lylw1 197609 1056 11月 12 15:51 layouts/partials/mathjax.html

```

## 7. 字数统计

LeaveIt本身在博文末尾提供了字数统计功能，使用的是`.WordCount`模板变量，但并不统计中文([详情](https://github.com/nodejh/hugo-theme-cactus-plus/issues/18))，所以显示的字数总是很小。可以通过在`config.toml`中添加如下语句启用。

```toml
hasCJKLanguage = true
```

## 8. 开启gittalk评论

在GitHub 上创建一个 [Github Application](https://github.com/settings/applications/new)，记录 `Client ID` 和 `Client Secret`

编辑`config.toml`以下字段

```toml
[params.gitalk]
owner = "shuzang"       # 你的GitHub ID       
repo = "blog"        # 博客网址    
clientId = ""    # 刚刚记录的client ID      
clientSecret = "" # 刚刚记录的client secret
labels = "gitalk"
```

提交到仓库后就能看到博文下开启了评论区

## 9. 打赏

有些主题支持加入打赏图片，但KeepIt不支持，不过这里还是记录一下。利用[第九工厂](<http://www.9thws.com/#>)的免费模板制作好看的打赏二维码。

## 10. URL

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

## 参考

[1] [Hugo中添加代码高亮支持](https://note.qidong.name/2017/06/24/hugo-highlight/)

[2] [Hugo 静态博客中迄今为止最佳数学公式支持-利用 MathJax & MMark](https://butui.me/post/yet-best-math-formula-support-for-hugo-with-mathjax/)

[3] [在Hugo中使用MathJax](https://note.qidong.name/2018/03/hugo-mathjax/)

