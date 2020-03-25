# hugo搭建个人博客1-基础建站


[Hugo](https://gohugo.io/)是由Go语言实现的静态网站生成器，可以快速建立一个静态网站，虽然多数情况下用来搭建个人博客，但实际上可做的事不止于此，还可以用作展示在线书籍、个人简历等。我换到Hugo的原因是文章数量变多以后，Hexo的生成速度变慢，难以忍受，同样的文章数量下，Hugo渲染文章几乎是即时的，不必等待，就很舒服。

本文为Hugo搭建个人博客系列的第一篇，基础建站，目的是先把博客网站搭建起来，并做一些基本配置，这些配置参考了`Mogeko的个人博客`[^1]

[^1]: [Mogeko的博客仓库](https://github.com/Mogeko/Blog)

## 1. 安装Hugo

安装使用繁琐也是一个我抛弃Hexo的原因，Hexo依赖于Node.js，运行需要电脑里安装node和npm，另外，开发者们提供了各种npm模块用来增强Hexo的功能，虽然使Hexo变得强大，但在维护、备份和恢复静态博客上带了诸多不便。Hugo则没有这方面的担忧，虽然使用Go开发，但安装时是独立的安装包，不需要额外安装任何东西，备份或者迁移都极为方便，甚至可以直接将博客源文件放在Onedrive中同步。

详细的安装说明参见官方文档[^2]，这里介绍win10和Ubuntu18.04的快速安装。

[^2]:[Hugo Quick Start](https://gohugo.io/getting-started/quick-start/)

### 1.1 Windows

win10下使用[chocolatey](https://chocolatey.org/)包管理工具快速安装Hugo

```bash
# 更新chocolatey到最新
> choco upgrade chocolatey

# 安装hugo-extended，因为将要使用的主题使用scss
> choco install hugo-extended -y

# 检查安装
> choco list --local
chocolatey 0.10.15
hugo-extended 0.58.3
2 packages installed.
```

### 1.2 Ubuntu

Ubuntu下使用apt-get会安装低版本Hugo，因此使用snap安装

```bash
$ snap install hugo --channel=extended
hugo (extended/stable) 0.58.3 from Hugo Authors installed
```

注意要安装extended版本，主要是因为很多主题都需要扩展版的功能，如果确认自己的主题不需要(阅读主题说明)，可以按照正常的版本。

## 2. 生成网站

执行下列命令将在本地生成博客网站项目文件夹，是之后所有操作执行的根目录(之后简称为项目根目录)，我的项目文件夹名为blog。

```bash
$ hugo new site blog
$ ls blog -l
total 1
drwxr-xr-x 1 lylw1 197609  0  9月 26 14:45 archetypes/
-rw-r--r-- 1 lylw1 197609 82  9月 26 14:45 config.toml
drwxr-xr-x 1 lylw1 197609  0  9月 26 14:45 content/
drwxr-xr-x 1 lylw1 197609  0  9月 26 14:45 data/
drwxr-xr-x 1 lylw1 197609  0  9月 26 14:45 layouts/
drwxr-xr-x 1 lylw1 197609  0  9月 26 14:45 static/
drwxr-xr-x 1 lylw1 197609  0  9月 26 14:45 themes/
```

浏览器打开github网站，创建和项目文件夹同名的仓库，该仓库用于存储博客项目所有文件。创建完成后，在本地电脑进入项目根目录，执行下列命令，将blog文件夹关联到远程仓库blog。

```bash
# 进入blog文件夹
$ cd blog
# 关联远程仓库
$ git init
$ git add .
$ git commit -m "Initial commit"
$ git remote add origin https://github.com/shuzang/blog.git
$ git push -u origin master
```

使用单独的仓库是为了备份博客网站源码和写作的文章，防止意外导致项目文件夹被破坏，到时候需要重新建立网站和进行配置，文章也可能丢失。

## 3. 安装主题

Hugo没有默认主题，需要自己从官方的主题列表[^3]下载安装。当然不是所有的主题都会在官方的主题列表中找到，因为需要主题作者自己提交，还有大量的主题单独存在于github或其它的版本控制仓库中，可以自己去寻找。总体来看，Hugo的主题数量和质量可能都不如Hexo，比如Next这种主题已经形成了庞大的开发社区，有大量的人在使用和维护，Hugo中就没有这种影响力很大的主题了，但还是有不少非常漂亮的主题。

[KeepIt](https://github.com/Fastbyte01/KeepIt)是其中一个较为简单优雅的主题，基于[LeaveIt](https://raw.githubusercontent.com/liuzc/LeaveIt/)主题二次开发而成，遵循「Less is more」的原则，是我最喜欢的主题。因为主题通常是单独的github仓库，因此将其作为博客项目的子模块进行管理。

[^3]: [Hugo themes](https://themes.gohugo.io/)

```bash
# 将主题项目作为子模块添加
$ git submodule add https://github.com/Fastbyte01/KeepIt.git themes/KeepIt
# 查看根目录增加的.gitmodules文件，该文件用于保存子模块信息
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   themes/KeepIt
# 查看子模块信息
$ git submodule
 87c33888f3fa86b8cc096bc3f6d7f2efe9ccba66 themes/KeepIt (v4-53-g87c3388)
```

复制主题提供的站点配置文件`config.toml`到Hugo根目录，覆盖Hugo本身的站点配置文件（第一次使用可以将exampleSite目录下的内容全部复制过来）

```bash
$ cp themes/KeepIt/exampleSite/. .
```

运行`hugo server`命令，在浏览器键入网址[http://localhost:1313](http://localhost:1313)预览主题效果（首页图片未加载是因为还没有放置头像文件）

![KeepIt theme preview](/images/hugo搭建个人博客1-基础建站/1Hpbad.png)

## 4. 网站配置

### 4.1 添加菜单栏页面

预览界面中可以看到菜单栏只有三个菜单，`Blog`、`Categories`和`About`，前两者分别是文章列表和分类页面，`About`则用来展示网站和自己的说明。

也可以自己建立新的菜单页面[^4] ，比如我新建`life`页面用作展示阅读的书籍、电影和游览的景点。建立页面文件的命令如下：

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

新建的life.md文件和原来的about.md文件都位于content目录下，编辑文件内容从而决定实际的页面显示内容。

```bash
$ ls content -l
total 2
-rw-r--r-- 1 lylw1 197609  658  9月 16 11:34 about.md
-rw-r--r-- 1 lylw1 197609 3343  9月 25 11:34 life.md
```

### 4.2 网站基本资料

编辑站点配置文件的以下内容

```toml
title = "shuzang's blog"  # 网站标题
[params]
    since = 2018                     # 站点建立时间
    author = "shuzang"               # 作者名
    subtitle = "世界钟爱热爱生活的人"       # 子标题
    home_mode = "" # post or other      # post模式会在主页面显示文章
    
    description = "shuzang的个人博客" # 网站描述
    keywords = "blog, Golang, Hugo, blockchain " # 网站关键词
```

### 4.3 头像

新建`static/images`文件夹，将头像文件存放在这里

```bash
$ mkdir static/images
# 复制头像文件到images文件夹,复制完成后目录结构如下
$ ls staitc -lR
static:
total 0
drwxr-xr-x 1 书葬 197610 0 11月 17 14:41 images/

static/images:
total 88
-rw-r--r-- 1 书葬 197610 87641 9月  28 18:21 avatar.png
```

修改站点配置文件`params`部分的`avatar`字段值

```toml
[params]
	avatar = "images/avatar.png" #comment it to use gravatar
```

也可以使用自己图床中的头像图片链接，或者如果有gravatar头像，可以在配置文件中寻找相关字段填写账号。

### 4.4 首页社交链接

修改站点配置文件中`params.social`部分的内容，需要的社交链接取消注释即可启用，比如我只启用了github和Email，github只需要填用户名，完整的URL在主题源码中自动生成。

```toml
[params.social]
    GitHub = "/shuzang"
    Email   = "xxxxx@qq.com"
```

### 4.5 网站Favicon

使用[favicon generator](https://www.google.com/search?q=favicon+generator) 生成配套的图标，放到/static目录下，可以设置网站在各平台的显示图标，放置后目录结构如下

```bash
$ ls static -l
total 278
-rw-r--r-- 1 书葬 197610 10366 4月  26  2019 android-chrome-192x192.png
-rw-r--r-- 1 书葬 197610 38060 4月  26  2019 android-chrome-512x512.png
-rw-r--r-- 1 书葬 197610  9529 4月  26  2019 apple-touch-icon.png
-rw-r--r-- 1 书葬 197610   246 4月  26  2019 browserconfig.xml
-rw-r--r-- 1 书葬 197610 87641 5月  19 22:50 cover.png
-rw-r--r-- 1 书葬 197610 15086 4月  26  2019 favicon.ico
-rw-r--r-- 1 书葬 197610   638 4月  26  2019 favicon-16x16.png
-rw-r--r-- 1 书葬 197610  1127 4月  26  2019 favicon-32x32.png
drwxr-xr-x 1 书葬 197610     0 11月 17 14:41 images/
-rw-r--r-- 1 书葬 197610 87641 5月  19 22:50 logo.png
-rw-r--r-- 1 书葬 197610  6431 4月  26  2019 mstile-150x150.png
-rw-r--r-- 1 书葬 197610   603 4月  26  2019 safari-pinned-tab.svg
-rw-r--r-- 1 书葬 197610   426 4月  26  2019 site.webmanifest
```

然后修改站点配置文件中以下内容

```toml
[author]
  name = "shuzang"

  [params.publisher]
    name = "shuzang"

    [params.publisher.logo]
      url = "logo.png"
      width = 127
      height = 40

  [params.logo]
    url = "logo.png"
    width = 127
    height = 40

  [params.image]
    url = "cover.png"
    width = 800
    height = 600
```

以上基本配置完成后，网站首页如下

![configuration finished](/images/hugo搭建个人博客1-基础建站/1HCWjK.png)

## 5. 文章发布

### 5.1 新建文章

在content目录下创建`posts`文件夹，写作的文章全部放到该目录下，在每篇文章开头添加元数据字段，可以是YAML或TOML格式，示例如下

```toml
title = "Getting Started with Hugo"
description = ""
type = ["posts","post"]
tags = [
    "go",
    "golang",
    "hugo",
    "development",
]
date = "2014-04-02"
categories = [
    "Development",
    "golang",
]
series = ["Hugo 101"]
[ author ]
  name = "Hugo Authors"
```

一般情况只需要选择`title`、`date`、`tags`等几个简单字段填充即可，需要注意的是字段的值即使只有一个，也要用中括号包围。

### 5.2 推送到远程

编辑站点配置文件，设置网站URL，设置HTML文件生存放的文件夹。之所以将默认生成的public文件夹更改为docs文件夹，是因为github pages[^6]使用只会从项目的docs目录读取文件

[^6]: [Github pages](https://pages.github.com/)

```toml
# 编辑博客源网址
baseURL = "https://shuzang.github.io/blog/"
# 添加publishDir字段，将网页生成的public目录更改为docs目录
publishDir = "docs"
```

使用`hugo`命令生成页面，然后推送项目到远程仓库。但要注意，以后每次生成页面前需要先删除原来的docs文件夹，从而清理旧文件

```bash
$ hugo
$ git add .
$ git commit -m "添加主题并生成博客网站"
$ git push origin master
```

推送完成后从浏览器打开blog仓库，在`setting—>GitHub Pages`勾选从`/docs`目录读取网站。关于网页如何托管在Github的详细说明可以参考[Host on Github](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

![github pages setting](/images/hugo搭建个人博客1-基础建站/1HChnO.png)

点击上图中顶部生成的URL即可查看自己的网站

注：题图来自网络。


