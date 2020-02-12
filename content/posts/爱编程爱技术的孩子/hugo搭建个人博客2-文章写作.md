---
title: hugo搭建个人博客2-文章写作
date: 2019-05-15 
lastmod: 2020-01-06
tags: [博客写作]
categories: [爱编程爱技术的孩子]

slug: Hugo blog article write
featured_image: https://s2.ax1x.com/2020/02/12/1HSMXF.jpg
---

本文介绍关于文章的一些问题，包括分类管理、排版技巧、特殊语法等。

Hugo支持的文章格式为`.md`，即用markdown语言编辑的文章。所有的文章都放在`content/posts`目录下，支持级联目录，即在`posts`目录下按分类建立多个子文件夹放置文章，比如本博客的文章按分类放在四个子文件夹下。

```bash
$ ls posts
爱编程爱技术的孩子/  我所热爱的生活啊/
平日里的白日梦/      研究生的区块链学习之路/
```

## 1. Front-Matter

每篇文章需要使用[Front-Matter](https://gohugo.io/content-management/front-matter/)指定一些参数，一个例子如下

```yaml
---
title: hugo搭建个人博客2-写作技巧  
date: 2019-05-15 
lastmod: 2020-01-06
tags: [博客写作]
categories: [爱编程爱技术的孩子]

slug: Hugo builds a personal blog 2
featured_image: https://strangesounds.org/wp-content/uploads/2016/08/milky-way-bolivia-Salar-de-Uyuni.jpg
toc: true
---
```

Front-Matter就是`---`包围的内容，当然，`---`是YAML格式的写法，也可以使用`+++`或`{ }`，分别是TOML和JSON格式的写法。

Hugo支持的Front-Matter格式的变量有20多个，几个主要的变量如下

| 变量        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| title       | 文章名                                                       |
| date        | 文章创建日期                                                 |
| description | 文章内容描述                                                 |
| draft       | 草稿，设置为true，这篇文章将不会被渲染到网站，一些需要长时间编辑的文章可以使用该变量 |
| keywords    | 文章关键词                                                   |
| lastmod     | 文章最后一次修改的时间                                       |
| linkTitle   | 用于创建文章链接，设置该值后，Hugo将在使用该值而不是title变量值作为文章链接，同时，文章的排序也会根据该值进行 |
| slug        | 该值将出现在文章URL尾部，替换基于文件名的URL                 |
| url         | 设置一个相对于网站根目录的url                                |
| taxonomies  | 用户定义的一个分类字段，像KeepIt主题中的tags, categories, featured_image都属于这种 |

## 2. 标签与分类

标签和分类字段是归档文章最常用的两种形式。为了更好的使用，我们需要弄清楚它们的区别。以生活与健身的韦恩图为例[^1]，饮食与睡眠同时属于这两个分类，当出现这类型文章时，就可以添加标签来管理。

[^1]:[活用标签，提高笔记搜索效率]( https://sspai.com/post/57467 )

![标签与分类](https://cdn.sspai.com/2019/11/17/07062af1ba37669fa8b7e91484e1d3da.jpg?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)



博客文章的存储方式决定了它无法拥有太多的分类，因此，树形逐级检索的方法不适用这种情况，我们应当在将博客分为几个大类的同时，使用标签来管理和检索文章，当同样也要注意不应滥用标签，否则会带来视觉上的混乱和管理的不便。

在每篇文章的元数据(Front-Matter字段)中添加`categories`字段作为文章分类，添加`tags`字段作为文章标签，比如本文：

```yaml
tags: [博客写作]
categories: [爱编程爱技术的孩子]
```

标签可以有多个，以逗号分隔，不过即使只有一个标签，外面的中括号也不能省略。

## 3. 排版风格规范

本节内容参考少数派写作排版指南[^2]

[^2]: [少数派写作排版指南](https://sspai.com/post/37815)

### 3.1 空格

行文时在中文与英文、中文与数字、英文与数字之间增加空格。例如：

- **推荐**：苹果公司在 2015 年 9 月 9 日发布了 iPhone 6s。
- **不推荐**：苹果公司在2015年9月9日发布了iPhone 6s。

 一段文字中有超链接的部分，在超链接的前后使用空格。例如：

- **推荐**：你可以前往 [苹果官网](https://www.apple.com/cn) 了解详情。
- **不推荐**：你可以前往[苹果官网](https://www.apple.com/cn)了解详情。

英文前后接全角标点符号或者表示单位的角标符号时，不需要加空格。例如：

- **正确**：新款屏幕的可视角度为 125°，相比旧款提升了 25%。
- **错误**：新款屏幕的可视角度为 125 °，相比旧款提升了 25 %。

对于有特殊用法的专有名词，如 4K、1080p、iOS 10 等，英文和数字之间是否空格以官方标准为准。

每段文字的开头不需要空两格。

### 3.2 标点符号

引号使用直角引号「」，而不是弯引号  “”。对于微软拼音输入法可以键入`uubd`四个字母开启标点符号的选择。

省略号使用 …… 的标准用法，正确输入方法是 shift + 6。不要使用三个句号。。。，也不要使用三个英文句点 ...。

不要重复使用标点符号，尤其是在表达强烈情感的时候。例如：

- **推荐**：这个提议真棒！我喜欢。
- **不推荐**：这个提议真棒！！！我喜欢~~~~

同时，抒发情感的方法有很多，不建议在文中大量使用感叹号等表达强烈情感的符号，也不建议使用波浪号。

### 3.3 中文和西文符号

一般情况下，一个中文句子中出现了英文部分，仍然使用中文标点，即全角符号。例如：

- **正确**：我常用的电子设备是 Kindle、iPad Pro、iPhone。
- **错误**：我常用的电子设备是 Kindle, iPad Pro, iPhone.

如果引用一段完整的英文句子，或是出现在专有名词中的标点，则不需要更改标点符号。例如：

- **正确**：乔布斯说「Stay hungry, Stay foolish.」
- **错误**：乔布斯说「Stay hungry，Stay foolish。」
- **正确**：我最喜欢的手机游戏是 Lifeline: Silent Night。
- **错误**：我最喜欢的手机游戏是 Lifeline：Silent Night。

### 3.4 专有名词

所有品牌名称以官方网站写法为准，应用名称遵循 Google Play 或 App Store 的命名。若无官方中文名称，可使用网络上被大家熟知的通用译名，否则请直接使用外文名称，不要自行翻译。

请正确书写常用英文名词的大小写和空格。例如：

- **正确**：iOS 10、macOS、App Store、Android
- **错误**：iOS10、MacOS、AppStore、android

若应用名称过长可在文中自定缩写，但需用括号注明缩写名称，并声明「下同」。

一般情况下，品牌和应用名称不需要使用直角引号「」括起来。

在称呼 app 时，请使用「应用」「应用程序」或「app」，而非「软件」「程序」。

### 3.5 样式工具

虽然可以使用加粗、斜体、删除线、引用等样式工具，这些工具的使用也可以增加文章的可读性，但是过度使用则会造成排版混乱，因此建议正确、克制地使用样式工具。

例如，需要着重显示的部分请使用「加粗」功能，不要使用「斜体」，更不要使用「加粗 + 斜体」的组合。

### 3.6 注明引用来源

文中有使用外站或外部内容的，务必在引用最后部分注明来源。

文中有使用外站图片，必须在文末标明。来源如果来自外站必须添加链接。例如，

- 本文部分图片来自 [The Verge](http://www.theverge.com/)
- 题图来自电影《春娇与志明》截图

若文章为全文翻译，必须在文中注明原作者及原文地址，并添加原文链接。

### 3.7    写作建议

理清文章结构，动笔之前可以先列一下写作大纲。

用主动语态，不要用被动语态。一般情况下，主动语态比被动语态更有力。

使用具体、明确、展示细节的词汇，能激发想象，使读者自己代入情境。「把硬币放进口袋里，他咧开嘴笑了」，远远强过「他满意地拿走了辛苦挣来的奖赏」。

减少形容词的使用，少用 「的」。

文中涉及到参数规格、数据的部分，要保证严谨性。

文章完成之后通读一遍，记住，不要让读者猜测你在讲什么。

## 4. 特殊语法

除Markdown基本语法外，灵活运用Hugo提供的一些功能，可以增加文章的灵活性。

### 4.1 题图

也就是文章开头的图片，在有些主题中也会用于缩略图显示。添加方法是在元数据字段使用`feature_image`和`description`字段，前者粘贴图片URL，后者书写图片描述。一个添加特性图片后的文章元数据字段内容如下：

```yaml
title: 在博客中内嵌bilibili视频
date: 2019-11-12
tags: [博客写作]
categories: [爱编程爱技术的孩子]
featured_image: https://i.vimeocdn.com/video/771515958.jpg?mw=1920&mh=1080&q=70
description: 悲剧傻缺视频集锦的某次截图
```

### 4.2 代码高亮

KeepIt主题本身对代码高亮的渲染是在本地执行的，使用的是[google code prettify](https://github.com/google/code-prettify)，高亮风格较少而且作者没提供配置选项，通过使用`Highlight.js`库可以自定义代码高亮格式[^3]。

[^3]:[Hugo中添加代码高亮支持](https://note.qidong.name/2017/06/24/hugo-highlight)

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

### 4.3 视频

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

### 4.4 音频

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

### 4.5 数学公式

文章中使用数学公式也是一种需求，但是Hugo本身不提供对这种语法的支持，KeepIt主题也没有提供这个功能，因此需要自己添加[^4]。

[^4]:[在Hugo中使用MathJax](https://note.qidong.name/2018/03/hugo-mathjax/)

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


## 附录

### 写作工具

+ 使用Chrome下的插件诸如`stackedit`与`markdown-here`等，不用担心平台受限 
+ 在线如CSDN、简书、知乎、Github等都支持Markdown写作，现在越来越多的网站都已经开始支持Markdown格式的文章
+ 客户端软件目前使用Typora，其它如VS code、有道云笔记、为知笔记等都支持markdown写作      

### Shortcodes说明

Hugo写文章的主要格式是Markdown，但是很多高级的语法默认的渲染引擎是不支持的，需要使用纯HTML代码来编写，这和Markdown的本意是违背的。因此Hugo提供了[Shortcodes](https://gohugo.io/templates/shortcode-templates/)来提供对这些语法的支持。

短代码的调用方法为将代码放在两个大括号和一个尖括号的包围中，代码本身应与两侧尖括号有一个空格分隔

另外，Hugo本身就提供了一些Shortcodes（[Build-in](https://gohugo.io/content-management/shortcodes/)），比如github gist，如果一个gist的url如下

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



