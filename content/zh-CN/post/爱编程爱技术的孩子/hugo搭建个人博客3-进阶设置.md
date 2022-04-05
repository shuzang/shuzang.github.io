---
title: hugo搭建个人博客3-进阶设置
date: 2019-07-12
lastmod: 2020-04-05
tags: [Hugo]
categories: [爱编程爱技术的孩子]

autoCollapseToc: false
slug: Hugo blog-advanced setup
---

本文是 Hugo 使用记录的第二篇，介绍如何为Hugo开启更多的功能。

## 1. 添加菜单栏页面

我们可以自己建立新的菜单页面[^4] ，比如我新建`life`页面用作展示阅读的书籍、电影和游览的景点。建立页面文件的命令如下：

[^4]: [菜单栏添加页面](https://gohugo.io/templates/menu-templates/)

```bash
$ hugo new life.md
```

然后编辑项目根目录下的站点配置文件`config.toml`，添加页面说明。其中`weight`字段的值决定了该页面在菜单栏的顺序[^5]，其值越大，顺序越靠后。按照习惯，`About`页面放在最后，所以其`weight`字段值设置为5。

[^5]: [菜单栏标题排序](https://discourse.gohugo.io/t/ordering-menu-items/1841)

```toml
# 添加Tags和Life菜单，并修改About菜单的weight字段值为5（为了调整菜单栏显示顺序）
  [[menu.main]]
    name = "Tags"
    url = "/tags/"
    weight = 3

  [[menu.main]]
    name = "Life"
    url = "/life/"
    weight = 4
```

新建的life.md文件位于content目录下，编辑文件内容从而决定实际的页面显示内容。

```bash
$ ls content -l
total 2
-rw-r--r-- 1 lylw1 197609  658  9月 16 11:34 about.md
-rw-r--r-- 1 lylw1 197609 3343  9月 25 11:34 life.md
```

## 2. 字数统计

中文([详情](https://github.com/nodejh/hugo-theme-cactus-plus/issues/18))统计的支持可以通过在`config.toml`中添加如下语句启用

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

现在更换使用Valine评论系统，步骤暂未总结

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

```

路径根据自己的文章位置而定，也可以在工具栏选择「格式—>图像—>设置图片根目录」，产生的效果是相同的，也是在文章头部添加该字段。

最后，我们在Typora的偏好设置中，设置插入图片时自动复制到相应的目录下即可，这样所有的图片都可以和博客代码存放在一起。、

这种方法唯一的缺点是所有图片和博客项目源码的总大小不能超过1G，因为一个Github项目空间最大1G

## 6. 自定义样式

通过定义自定义 `.scss` 样式文件, **LoveIt** 主题支持可配置的样式.

包含自定义 `.scss` 样式文件的目录相对于 **你的项目根目录** 的路径为 `config/css`.

在 `config/css/_override.scss` 中, 你可以覆盖 `themes/LoveIt/assets/css/_variables.scss` 中的变量以自定义样式.

这是一个例子:

```scss
@import url('https://fonts.googleapis.com/css?family=Fira+Mono:400,700&display=swap&subset=latin-ext');
$code-font-family: Fira Mono, Source Code Pro, Menlo, Consolas, Monaco, monospace;
```

在 `config/css/_custom.scss` 中, 你可以添加一些 CSS 样式代码以自定义样式.

## 8. 添加独立域名

给部署在 GitHub Pages 上的博客添加独立域名，是早就想做的事，以前是因为没什么需求，.me域名比较贵，国内购买域名备案又比较麻烦，所以一直没做。这次选择了.top域名，买了三年才67，可以了，三年以后开始工作再考虑换.me域名，毕竟这个冷门的字段应该没人要。而且发现部署在github pages不需要备案，因为服务器不在国内。

### 3.1 选购域名

可能很多人已经买好了域名了，你可以[跳过这部分](#GitHub 上的设置)

要想给博客绑定独立域名，首先你得拥有一个域名。得去域名注册网站购买域名，国内的域名商推荐[万网](https://wanwang.aliyun.com/)，国外的推荐 [GoDaddy](https://www.godaddy.com/)，因为他们分别是中国最大和世界最大的域名注册服务商。不了解的领域当然挑知名有信誉的总是没错的。

要是买`.me`域名就真的只能在GoDaddy买了，首年24，续费不知道，但是还需要购买隐私保护，这样就变成了每年60，要是工作后还负担的起，现在买这个还是算了。`.io`首年就320，更负担不起了。国内的话，`.com`，`.net`这些就不想了，基本都已经被注册了，于是选择了`.top`，一口气买了三年，一共67，而且在国内购买的好处是本身对WHOIS中的个人信息提供隐私保护。阿里云之前对域名提供的隐私保护服务已经停用了，主要是因为

> 为落实[ICANN临时规范](https://www.icann.org/news/announcement-2018-05-22-zh)要求，自2018年5月25日起阿里云WHOIS查询公开信息中将不再显示域名注册人、管理联系人和技术联系人的个人数据，包括姓名、邮箱、电话、街道地址等。

这样域名的注册信息已得到默认保护，不再需要额外开通隐私保护。

具体选购过程大致是：**搜索域名 -> 将喜欢的域名加入购物车 ->选择域名注册模板(没有的话要创建)-> 付款 -> 购买成功**，实际使用还需要进行邮箱验证和实名认证，否则域名无法启用。

最为繁琐的域名备案过程可以跳过，因为博客挂载在github pages，而只有服务器在国内的网站才需要备案。

### 3.2 DNS解析

使用CNAME别名映射域名，比设置A记录更方便，最重要的是A记录无法开启https。参数设置如下图所示。

![CNAME设置](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190712_1Heu1H.png)

访问网址时可能会加www前缀，因此可以设置一个二级域名解析，方法相同。

### 3.3 GitHub 上的设置

到 Github `shuzang.github.io`仓库设置里，在 `Custom domain` 这里填写`shuzang.top`域名并保存。

![github域名设置](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190712_1He3HP.png)

 `Custom domain` 下方 `Enforce HTTPS` 这个选项一并勾选，Github 跟 Let’s Encrypt 有合作，如果勾选了这个选项，Let’s Encrypt 就会给你的博客签发一张 SSL 证书，免费的。

![启用HTTPS](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20190712_1HeJN8.png)



发现上面这种方法每次提交后都需要重新填写`custom domin`字段，因此采用另一种方式，创建一个名为 `CNAME` 的文件放在`content`目录，其中的内容**只**写上你的域名，像这样

```bash
shuzang.top
```

使用hugo命令生成文件会将CNAME文件直接复制到public目录，并通过持续集成将其推送到master分支根目录。

等几分钟 (刷新 DNS 缓存)，然后在浏览器中输入`shuzang.top`，回车，不出意外看到了自己的博客。

### 3.4 其他玩法

除了将域名绑定给博客外博客，还可以用域名干一些别的事。

比如，**使用 A 记录将 `mail.shuzang.me` 这个二级域名指向 `207.46.149.80` 就可以 “搭建” 一个 [临时邮箱](http://mail.shuzang.top/)服务** (感谢 [萌咖 | MoeClub.org](https://moeclub.org/) 提供的服务器)

如果你还有一台拥有**公网 IP** 的服务器，可玩性就更高了！

如果有能力，你甚至可以拥有自己的搜索引擎

## 9. 代码高亮

KeepIt主题本身对代码高亮的渲染是在本地执行的，使用的是[google code prettify](https://github.com/google/code-prettify)，高亮风格较少而且作者没提供配置选项，通过使用`Highlight.js`库可以自定义代码高亮格式[^3]。

[^3]: [Hugo中添加代码高亮支持](https://note.qidong.name/2017/06/24/hugo-highlight)

为了最大程度的保持原主题文件的代码，我们首先将主题文件夹下`layouts/partials/js.html`文件复制到Hugo项目文件夹，令其位于同样的目录，然后删除文件中的以下语句

```html
{{ $prettify := resources.Get "/js/prettify.min.js" }}
```

之后删除`js.html`文件中所有`$prettify`字段。

复制主题文件夹`layouts/partials/head.html`文件到项目文件夹，并添加如下内容

```html
<link href="https://cdn.bootcss.com/highlight.js/9.15.10/styles/atom-one-light.min.css" rel="stylesheet">
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/highlight.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/go.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

以上代码使用了[bootCDN](https://www.bootcdn.cn/highlight.js/)对`Highlight.js`进行加速，并额外添加了对go语言代码高亮的支持。

最后需要删除主题文件夹中的某些内容，包括

1. `assets/js/prettify.min.js`文件

2. `assets/css/main.scss`的以下内容

   ```scss
   @import "_common/_prettyprint/default.scss"
   ```

此时代码就可以以浅色的atom格式高亮了，不过可惜的是不显示代码行数，就只能等以后再说了。

还要注意的是以后每次更新主题都要删除主题文件夹中以上两个内容。

## 10. 视频

可以使用html代码直接在文章中嵌入视频，因为各视频网站一般会在分享链接中提供iframe格式的代码，以B站为例，复制如下代码放在文章中相应的位置即可

```html
<iframe src="//player.bilibili.com/player.html?aid=75064361" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="width: 100%;height: 600px;" > </iframe>
```

注意添加`style`字段调整高度与宽度，否则可能显示效果不会很好，上面的代码效果如下

<iframe src="//player.bilibili.com/player.html?aid=75064361" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="width: 100%;height: 600px;" > </iframe>

也可以使用短代码来完成，Hugo提供了[Shortcodes](https://gohugo.io/templates/shortcode-templates/)功能可以帮助我们定制一些语法，仍以B站视频为例，在项目根目录建立`/layouts/shortcodes`文件夹，在该文件夹内新建`bilibili.html`文件，文件内容编辑如下

```html
<iframe src="//player.bilibili.com/player.html?aid={{ .Get 0 }}" 
        scrolling="no" border="0" frameborder="no" framespacing="0" 
        allowfullscreen="true" 
        style="width: 100%;height: 600px;" > 
</iframe>
```

以后在文章中添加视频就可以使用如下格式的代码，其中的参数为视频代码，可以在视频URL中找到

```bash
video 75064361
```

使用iframe即使在本地的Typora编辑器也可以查看效果，但是代码较多，而是用shortcodes可以精简代码，但只有开启预览或将文章推送到云端才能看到，各有利弊，自己权衡。

最后还需要提一点，引入视频会导致网页加载速度大幅下滑，如非必要，最好不要引入。

## 11. 音频

在文章中嵌入音频也是一个需求，尤其是歌曲，使用频率比较高，我们在这里注意使用使用 [APlayer](https://github.com/MoePlayer/APlayer) 和 [MetingJS](https://github.com/metowolf/MetingJS) 完成这一功能。

在`layouts`目录下的head.html文件中添加如下内容

```html
<!-- require APlayer -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.js"></script>
<!-- require MetingJS -->
<script src="https://cdn.jsdelivr.net/npm/meting@2/dist/Meting.min.js"></script>
```

然后在文章中可以通过如下语句添加音频

```html
<meting-js auto="https://y.qq.com/n/yqq/song/000xRG4E4diqXD.html"></meting-js>
```

播放链接可以通过浏览器从各音乐平台获取，当然，和视频一样使用iframe格式的代码嵌入也是可以的

最后，也可以使用shortcodes缩短音频代码，在`layouts/shortcodes`目录下新建audio.html文件，编辑内容如下

```html
<meting-js auto={{ .Get 0 }}></meting-js>
```

在文章中就可以像下面这样使用，参数为音频网址

```bash
audio  "https://y.qq.com/n/yqq/song/000xRG4E4diqXD.html"
```

## 12. 数学公式

文章中使用数学公式也是一种需求，但是Hugo本身不提供对这种语法的支持，KeepIt主题也没有提供这个功能，因此需要自己添加[^4]。

[^4]: [在Hugo中使用MathJax](https://note.qidong.name/2018/03/hugo-mathjax/)

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

## 待完成 

- RSS
- 站内搜索
- CDN，可参考[为博客添加免费的 CDN (Cloudflare)](https://mogeko.me/2019/056/)
- SEO

