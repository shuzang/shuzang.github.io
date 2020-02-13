# Hexo 搭建个人博客


[Hexo](https://hexo.io/zh-cn/index.html)是一款快速、简洁且高效的静态博客框架，使用Markdown渲染引擎解析文章，拥有着丰富的[主题社区](https://hexo.io/themes/)，可以帮助我们快速建立自己的博客，正在成为越来越多的技术人员制作自己博客的首选。

我选用了Hexo+github的博客部署模式，博客地址为：[https://shuzang.github.io](https://shuzang.github.io)

<!-- more -->

## 开始使用
Hexo使用Node.js编写，安装之前需要先安装两个工具

- Node.js，版本不低于8.6，建议10.0以上
- Git

我在win10环境下管理和部署博客，因此使用windows下的包管理工具[chocolate](https://chocolatey.org/)安装git和[nvs](https://github.com/jasongin/nvs/)，然后使用nvs安装和管理Node.js，默认会安装最新的软件版本，当然也可以自己指定版本号。

```powershell
> choco install git
> choco install nvs
> set-executionpolicy remotesigned
> nvs add lts
> nvs link lts
```

以上程序安装完成后，使用npm安装Hexo

```powershell
> npm install -g hexo-cli
```

此时即可执行以下命令建立博客

```bash
$ hexo init hexo_blog  # hexo_blog为博客项目文件夹
$ cd hexo_blog
```

当前hexo_blog文件夹的目录结构如下

```bash
.
├── node_modules       //依赖安装目录
├── scaffolds          //模板文件夹，新建的文章将会从此目录下的文件中继承格式
|   ├── draft.md         //草稿模板
|   ├── page.md          //页面模板
|   └── post.md          //文章模板
├── source             //资源文件夹，用于放置图片、数据、文章等资源
|   └── _posts           //文章目录
├── themes             //主题文件夹
|   └── landscape        //默认主题
├── .gitignore         //指定不纳入git版本控制的文件
├── _config.yml        //站点配置文件          
├── package.json       //Hexo软件版本信息和依赖的模块列表
└── package-lock.json
```

**_config.yml**是网站的[配置](https://hexo.io/zh-cn/docs/configuration) 信息，以[YAML](http://www.ruanyifeng.com/blog/2016/07/yaml.html?f=tt)语言编写，在此可以配置大部分的参数。

在根目录下执行如下命令启动 hexo 的内置 Web 服务器

```bash
$ hexo server
```

该命令将会调用 Markdown 引擎解析项目中的博客内容生成网页资源，并启动一个简易的 Web 服务器用于提供对内存中网页资源的访问，Web 服务器默认监听 4000 端口，用户可在浏览器中通过地址 `localhost:4000` 访问博客。

![Hexo默认主题首页](http://yearito-1256884783.image.myqcloud.com/hexo-get-started/20181102070503130.png)

其它的常用命令如下：

- `hexo new [layout] <title>`：新建文章或页面，`[layout]`表示模板，`<title>`表示标题，比如
  - `hexo new post 001`，表示新建了一个标题为`001`的文章
  - `hexo new page 001`，表示新建了一个标题为`001`的页面
- `hexo generate`：生成网站静态文件，生成后的网页会放在根目录下`public`文件夹里
- `hexo deploy`：部署网站，可以把生成的页面（public文件夹内容）部署到指定地方
- `hexo clean`：用来清空`public`文件夹

## 更换Chic主题

[Chic](https://github.com/Siricee/hexo-theme-Chic)主题是一款比较简洁的主题，虽然用的人不多，但基本功能都有，同时也避免出现大家博客页面都一样的尴尬。

在根目录下执行如下命令下载主题文件

```bash
$ git clone https://github.com/Siricee/hexo-theme-Chic.git themes/Chic
```

打开站点配置文件`_config.yml`将`theme`字段的值改为`Chic`

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: Chic
```

重新执行`hexo server`命令，打开浏览器进入http://localhost:4000/页面

![Chic主题首页](https://camo.githubusercontent.com/58dfbf6fc501c3cefa5d63bfe5c6baaace41a9b5/68747470733a2f2f692e6c6f6c692e6e65742f323031392f30362f31322f3564303036626432383961613332353033372e706e67)

### 1. 站点配置

主要编辑根目录和主题目录下的两个`_config.yml`文件进行站点的相关配置，注意每个字段的冒号与值之间需要间隔一个空格，修改完成后需要重新执行`hexo server`命令

#### 网站基本信息

根目录下的`_config.yml`文件

```yaml
# Site
title: shuzang's blog
subtitle: '世界钟爱热爱生活的人'
description: 'shuzang的个人博客'
keywords: 'blog, Golang, Hugo, blockchain' 
author: shuzang
language: en
timezone: ''
```

主题目录下的`_config.yml`文件

```yaml
# Header
navname: shuzang's Blog
nickname: ''  # nickname置空，因为navname中已经出现了我们的名字
description: 世界钟爱热爱生活的人
```

#### 头像

将自己的头像命名为`avatar.png`，放在主题目录下的`source/image`文件夹内，并在主题配置文件中设置如下

```yaml
avatar: /image/avatar.png
```

#### 网站 favicon

通过[favicon generator](https://www.google.com/search?q=favicon+generator) 网站生成favicon.ico作为网站favicon，放在主题目录下的`source`文件夹内，替换原来的图标

#### 社交链接

主题配置文件中，只开启github和RSS的链接，主题本身不支持RSS，之后我们按照其它插件开启，这里先显示图标

```yaml
links:
  Github: https://github.com/shuzang
  RSS: 
```

#### copyright

主题配置文件中，取消显示文章末尾copyright中的slogan

```yaml
post_copyright_slogan_enable: false
```

#### MathJax支持

主题本身支持数学公式，在主题配置文件中开启

```yaml
# plugin functions
## Mathjax: Math Formula Support
## https://www.mathjax.org
mathjax:
  enable: true
  import: demand # global or demand
```

demand和global为mathjax的加载方式，global会在所有页面都加载，但可能导致部分markdown语法被错误解析，demand则只需要在需要开启支持的文章Front-matter中声明即可

```markdown
---
title: MathJax Test
date: 2019-07-05 21:27:59
tags:
mathjax: true # 加入这个声明，这篇文章就会开启mathjax渲染
---
```

#### 中英文间自动添加空格

在主题配置文件的`# Writing`部分添加如下字段

```yaml
auto_spacing: true
```

执行`hexo g`命令重新生成页面

### 2. 添加菜单栏其它页面

主要指标签和分类页面，豆瓣电影和图书页面

#### Tag、Category、About页面

根目录下执行新建页面命令

```bash
$ hexo new page tag
$ hexo new page category
$ hexo new page about
```

进入`source/tag`目录，为`index.md`文件增加layout字段

```
// source\tag\index.md
---
title: Tag
layout: tag
---
```

category和about页面同理，layout字段值为category和about

#### 豆瓣电影和图书页面

为站点添加豆瓣电影和图书页面，使用模块hexo-douban完成，首先根目录下执行下列命令安装该模块

```bash
$ npm install hexo-douban --save
```

在站点配置文件中添加以下内容，不需要游戏页面，所以注释掉了

```yaml
douban:
  user:   # 个人豆瓣ID
  builtin: false
  book:
    title: '我的书单'
    quote: '唯有书籍慰藉心灵'
  movie:
    title: '我的电影'
    quote: '电影拯救孤独'
#  game:
#    title: 'This is my game title'
#    quote: 'This is my game quote'
  timeout: 10000
```

在主题配置文件中新增电影和书籍页面的入口，注意顺序，要放在About字段前，因为这决定了菜单栏的显示顺序

```yaml
# navigatior items
nav:
  Posts: /archives
  Categories: /category
  Tags: /tag
  Books: /book
  Movies: /movie  
  About: /about
```

然后在根目录下执行下列命令生成电影和图书页面

```bash
$ hexo douban
```

可选参数

- -b | --books: 只生成豆瓣读书页面
- -m | --movies: 只生成豆瓣电影页面

执行命令后，插件会根据用户提供的 ID 爬取豆瓣中的数据信息并在 `public` 目录下生成对应的页面，当服务器启动或部署后会将页面显示在对应的菜单路由下。

需要隔一定周期执行一次`hexo douban`命令，虽然在站点配置中设置了 `douban.builtin: true`会在每次部署时自动执行`hexo douban`命令，但这是没有必要的，因为该命令爬取页面的时间比较长，会大幅增加编译打包时间，而我们豆瓣的图书和电影数据变动则并不频繁。

同时，以后也无法使用`hexo d`作为`hexo deploy`的简化，因为`hexo douban`的简化也是它。

最后，实际运行时，这两个页面总要弹出about:blank#blocked的空页面，暂时没有找到原因

### 3. 其它细节改进

对整个网站的一些细节做调整

#### 页脚显示站点运行时间

首先在站点配置文件中新增`since`字段，设置其值为站点建立时间

```yaml
since: 2018
```

然后打开`themes/Chic/layout/_partial/footer.ejs`文件，修改其内容为

```html
<footer id="footer" class="footer">
    <div class="copyright">
        <span>© <%= config.since %> - <%= date(Date.now(), "YYYY") %>  <%= config.author %> | Powered by <a href="https://hexo.io" target="_blank">Hexo</a> & <a href="https://github.com/Siricee/hexo-theme-Chic" target="_blank">Chic</a></span>
    </div>
</footer>
```

#### 修改文章建立时间格式

打开`themes/Chic/layout/_partial/post.ejs`文件，修改以下内容

```html
<span class="post-time">
       Date: <a href="#"><%- date(page.date, theme.date_format)  %>&nbsp;&nbsp;<%- time(page.date, theme.time_format)%></a>
</span>
```

删除其中关于具体每天几点几分几秒的显示

```html
<span class="post-time">
       Date: <a href="#"><%- date(page.date, theme.date_format)  %>&nbsp;&nbsp;</a>
</span>
```

#### 开启RSS

在站点根目录下安装`hexo-generator-feed`模块

```bash
npm install hexo-generator-feed
```

在站点配置文件中添加以下内容启用插件，注意放到`#Extensions`字段后面

```yaml
# Extensions
plugins:
  hexo-generator-feed
#Feed Atom
feed:
  type: atom
  path: atom.xml
  limit: 20
```

在主题配置文件的社交链接部分更改RSS的值，我们之前已经启用了图标

```yaml
links:
  RSS: atom.xml
```

执行`hexo g`命令生成atom.xml文件，然后就启用了

#### 支持mermaid

安装模块`hexo-filter-mermaid-diagrams`

```bash
$ npm install hexo-filter-mermaid-diagrams
```

在站点配置文件中添加如下内容

```yaml
# mermaid chart 
mermaid: ## mermaid url https://github.com/knsv/mermaid 
  enable: true  # default true 
  version: "7.1.2" # default v7.1.2 
  options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js 
    #startOnload: true  // default true 

```

在`themes/Chic/layout/_partial/footer.ejs`文件中添加如下内容

```html
<% if (theme.mermaid.enable) { %>
  <script src='https://unpkg.com/mermaid@<%= theme.mermaid.version %>/dist/mermaid.min.js'></script>
  <script>
    if (window.mermaid) {
      mermaid.initialize({theme: 'forest'});
    }
  </script>
<% } %>
```

使用和普通代码相同，只是要声明为mermaid格式

## 部署博客

我选择了部署到github，主要使用了github pages功能

1. 在github网站创建一个新的公开仓库**hexo_blog**  

2. 在仓库页面选择 `Settings——>Options——GitHub Pages——>Source`，在下拉列表中选择**master branch**，选择**Save**，生成一个GitHub pages网址，这就是我们的网站域名了

3. 安装部署插件

    ```bash
   $ npm install hexo-deployer-git --save
   $ npm install hexo-server --save
   ```

4. 如果这台电脑第一次安装使用git，还需要配置git，但不属于本文范畴

5. 配置`_config.yml`文件以下几部分

    ```yaml
   # URL
   ## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
   url: https://github.com/shuzang/hexo_blog
   root: /hexo_blog/
   
   # Deployment
   ## Docs: https://hexo.io/docs/deployment.html
   deploy:
       type: git
       repo: git@github.com:shuzang/hexo_blog.git
       branch: master
   ```

这时候就已经万事俱备了，只要使用Hexo部署命令，就可以成功发布到Github

```bash
$ hexo clean && hexo deploy -g
```

## 项目恢复

hexo+github模式的个人博客部署好了之后，hexo_blog文件夹中存在大量的文件，不仅仅是各种配置，还有我们所写的文章。在操作过程中，可能面对两种情况：

- 系统崩溃，所有文件丢失
- 要换电脑了

所以我们应未雨绸缪，经多方查找，终于找到了一种比较简单而有效的方法，即使用Travis CI做持续集成，项目文件放在`hexo`分支下，渲染后的静态文件在`master`分支下，只需要更新`hexo`分支中的文章就可以自动更新，进行配置也只需要克隆`hexo`分支然后进行改动，不需要每次都重新`hexo init`并配置

## 主题收集

- [Next主题](https://github.com/theme-next/hexo-theme-next)
- [Chic主题](https://github.com/Siricee/hexo-theme-Chic)

## 参考

[1] [segmentfault-hexo原理浅析](https://segmentfault.com/a/1190000008784436)

[2] [简书-Hexo博客配置RSS插件](https://www.jianshu.com/p/2aaac7a19736)

[3] [NPM-hexo-filter-mermaid-diagrams](https://www.npmjs.com/package/hexo-filter-mermaid-diagrams)

[4] [Yearito-Hexo搭建个人博客系列](http://yearito.cn/tags/Hexo/)

[5] [hexo和next主题相关设置](https://www.jianshu.com/p/3a01cc514ce7?utm_source=oschina-app)

[6] [打造个性超赞博客Hexo+NexT+GithubPages的超深度优化](https://reuixiy.github.io/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html)

[7] [hexo的next主题个性化教程:打造炫酷网站](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html)

[8] [关于HEXO搭建个人博客的点点滴滴](https://juejin.im/post/5a6ee00ef265da3e4b770ac1)
