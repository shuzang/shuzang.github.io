---
title: hugo搭建个人博客7-主题迁移
date: 2019-12-29
lastmod: 2020-02-12
tags: [Hugo]
categories: [爱编程爱技术的孩子]

slug: migrate to LoveIt themes
featured_image: https://s2.ax1x.com/2020/02/12/1H3q8x.jpg
---

[KeepIt]( https://github.com/Fastbyte01/KeepIt )主题的作者在issue中明确回答了不会添加目录功能，但最近一段时间我越来越喜欢写长文，或者在梳理结构的时候把原先几篇作为材料重新整理成一篇长文，目录功能是在是必不可少。除此之外，RSS、代码高亮格式、对mermaid的支持、文章更新日期等大量遗留的问题也令人不堪其扰，这些都是很常用的功能但都没有很好的解决。

有人在KeepIt的基础上重构设计了[LoveIt]( https://github.com/dillonzq/LoveIt )主题，我关心的大部分问题都得到了解决，包括

- 添加已用和自动展开的目录功能
- 支持**Katex**的数学公式
- 支持**mermaid**的图表和流程图
- 增加了对文献引用语法的支持
- 增加了对专业名词注音的语法支持
- 增加了对文章更新日期的显示

新主题还存在一些问题，包括

- 字体样式和大小，边距，颜色等没有原主题看着舒服
- 标题栏固定，虽然便于随时切换页面，但是缩小了内容空间
- Font Awesome图标的使用，导致网页加载速度明显慢于原来的主题
- 暗黑/明亮模式的切换图标没有原来的看着舒服

但这些问题不影响基本内容的展示和日常使用，加载速度的问题之后想办法用CDN加速解决，样式问题可以慢慢调整，所以最后决定迁移到LoveIt主题。

## 1. 主题安装

考虑到各种不确定性，没有完全抛弃原主题，而是采用两种主题并存的方式。

在博客根目录执行如下命令，将新主题作为子模块添加

```bash
$ git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

重命名原主题的配置文件`config.toml`为`config.toml.KeepIt`

```bash
$ cp config.toml config.toml.KeepIt
```

在根目录下的`layouts`文件夹中新建`KeepIt_config`文件夹，并将原来用于覆盖样式的一些文件移动到该目录下

```bash
$ mkdir layouts/KeepIt_config
$ mv layouts/_default layouts/partials layouts/shortcodes -t layouts/KeepIt_config
```

然后复制主题文件夹下的配置文件和CSS样式文件到博客根目录

```bash
$ cp themes/LoveIt/exampleSite/config.toml .
$ cp themes/LoveIt/exampleSite/static/css static
```

最后删除博文中自定义的插入视频和音频文件的shortcode语法

## 2. 配置文件调整

对根目录下`config.toml`做编辑，适应自己的使用

```toml
baseURL = "https://shuzang.github.io"   # 更改URL
title = "Shuzang's Blog"                # 设置网站标题
theme = "LoveIt"                        # 设置主题为LoveIt
hasCJKLanguage = true                   # 启用对中文统计的支持，不启用字数统计会出问题
paginate = 10                           # 每页的文章数量设置为10，便于根据页数快速计算总文章数
publishDir = "docs"                     # 适应.travis.yml文件中的配置

# 作者名设置
[author]                                            
  name = "shuzang"
  
# 菜单栏新添life页面并调整顺序
[menu]                                               
  [[menu.main]]
    identifier = "posts"
    name = "Posts"
    url = "posts"
    weight = 1

  [[menu.main]]
    identifier = "categories"
    name = "Categories"
    url = "categories"
    weight = 2  

  [[menu.main]]
    identifier = "tags"
    name = "Tags"
    url = "tags"
    weight = 3

  [[menu.main]]
    identifier = "life"
    name = "Life"
    url = "life"
    weight = 4

  [[menu.main]]
    identifier = "about"
    name = "About"
    url = "about"
    weight = 5
    
 # 基本参数设置，包括网站描述字段、关键词、主页文章显示模式、子标题
 [params]
  description = "shuzang's personal blog"                      # site description
  keywords = ["blog", "Golang", "Hugo", "blockchain", "go", "life"]     # site keywords
                                              
  home_mode = "other"                                # [post, other] home mode
  subtitle = "The world loves people who love life"  # subtitle shown in home page
  
  #头像、主页社交链接
  [params.gravatar]
    email = "lylw1996@qq.com"

  [params.social]                                    # Social Info in home page
    GitHub = "shuzang"
    Email = "lylw1996@qq.com"
    Steam = "76561198343669932"
    Skype = "live:844327696"
    
  # gitalk设置涉及clientId和clientSecret，忽略
```

## 3. 新的语法

每篇文章开头的配置字段有了新的调整，一个完整的参考案例如下

```yaml
title: "Markdown 基本语法预览"
date: 2019-08-25T16:22:42+08:00
lastmod: 2019-12-29T16:22:42+08:00
draft: false

tags: ['Hugo', '主题']
categories: ['预览']

featured_image: 'https://strangesounds.org/wp-content/uploads/2016/08/milky-way-bolivia-Salar-de-Uyuni.jpg'
comment: true
toc: true
math: true
```

`lastmod`字段可设置文章的更新日期，会显示在每篇文章的最后

`draft`字段用于设置文章是草稿还是正式发布，以前就有但没用过，现在发现很有用

`featured_imgae`字段设置特性图片，原先博文中设置该字段时没有加引号，新主题下无法显示，要做调整

`comment`控制评论是否开启

`toc`控制目录是否开启

`math`控制是否支持对数学公式的显示

新的主题还对Markdown语法的显示格式做了调整，比较突出的是

### 3.1 语法高亮

`ls` 命令

```go
// You can edit this code!
// Click here and start typing.
package main

import "fmt"

func main() {
  fmt.Println("Hello, 世界", "Hello, 世界", "Hello, 世界", "Hello, 世界", "Hello, 世界", "Hello, 世界", "Hello, 世界", "Hello, 世界")
}
```

- This is a list that contains multiple code blocks.

  - Here is an indented block

    ```Code
    This will still be parsed
    as a normal indented code block.
    ```

  - Here is a fenced code block:

    ```Code
    This will still be parsed
    as a fenced code block.
    ```


### 3.2 表格

| Tables        |      Are      |  Cool |
| :------------ | :-----------: | ----: |
| col 3 is      | right-aligned | $1600 |
| col 2 is      |   centered    |   $12 |
| zebra stripes |   are neat    |    $1 |

### 3.3 Font Awesome

:(fab fa-weixin fa-2x):

### 3.4 脚注

This is a footnote[^footnode]

[^footnode]: This is a footnote

Test all features in development[^link test].

[^link test]: https://www.google.com/

### 3.5 专有名词

当前的研究方向是[区块链]^(Blockchain)

### 3.6 kbd

<kbd>ctrl</kbd>+<kbd>alt</kbd>+<kbd>del</kbd>

### 3.7 图片

![Gif](https://static.dillonzq.com/images/20190817130904-U6cPUk.jpg "鼠标放在图片上不动可以显示该内容")

### 3.8 混合列表

- item 1
  * [x] item A
  * [ ] item B
    more text
    1. item a
    2. itemb
    3. item c
  * [x] item C
- item 2
- item 3

Really Mixed Lists

- item 1

  * [x] item A

  - item B
    more text

    1. item a

    + itemb
    + [ ] item c

  1. item C

1. item 2

- [x] item 3

## 4. shotcode

LoveIt提供了9种shortcode的语法，下面一一介绍

### 4.1 Admonition

{{% admonition quote %}}
biu biu biu.
{{% /admonition %}}

{{% admonition quote "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition example "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition bug "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition danger "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition failure "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition warning "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition question "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition success "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition tip "I'm title!" false %}}
biu biu biu.
{{% /admonition %}}

{{% admonition info "I'm title!" true %}}
**biu biu biu.**
{{% /admonition %}}

{{% admonition note "I'm title!" false %}}
**biu biu biu.**
{{% /admonition %}}

{{% admonition type="abstract" title="Test Admonition" %}}
This is a Admonition.
{{% /admonition %}}

### 4.2 Align

分三种：align-center, align-left, align-right，作者只是建立了文件，还没填充内容，因此该语法现在还不能用

### 4.3 bilibili



### 4.4 countdown



### 4.5 流程图

{{< mermaid >}}
sequenceDiagram
    participant Alice
    participant Bob
    Alice->John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
{{< /mermaid >}}

### 4.6 ECharts

新版本中调整为开发状态，不再支持

### 4.7 Float right

同样还只是空文件，语法不可用

### 4.8 music

和视频类似，以`music`关键词声明，传入三个参数：`server`,`type`和`id`

`server`定义平台，有 `netease`, `tencent`, `kugou`, `xiami`, `baidu` 五个可选项，`type`是类型，有 `song`, `playlist`, `album`, `search`, `artist` 五个可选项，id是 song id / playlist id / album id / search keyword 

### 4.9 Typeit

{{< typeit group="test" tag="h3" >}}
Here is a story about love...
{{< /typeit >}}

{{< typeit group="test" code="Go" >}}

package main 
{{< /typeit >}}