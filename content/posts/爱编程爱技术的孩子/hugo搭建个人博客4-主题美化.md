---
title: hugo搭建个人博客4-主题美化
date: 2019-09-30
lastmod: 2020-02-12
tags: [Hugo]
categories: [爱编程爱技术的孩子]

slug: Hugo blog theme beautify
featuredImage: https://cdn.wispx.cn/wallpaper/2018/07/28/f45cfdcb1f79bc01d95c22adc63a2854.jpg!wallpaper.detail
---

## 1. 基础知识

通过一个简单的主题开发流程，理解需要的基本知识，为自己进行主题修改和美化打基础，这里参考[create a new theme](https://github.com/digitalcraftsman/hugo-steam-theme/blob/master/exampleSite/content/post/creating-a-new-theme.md)一文。

### 开发准备

Ubuntu安装hugo

```bash
$ snap install hugo --channel=extended
hugo (extended/stable) 0.58.3 from Hugo Authors installed
```

该命令将安装`extended`Sass/SCSS版本(apt-get 安装的hugo版本较低)

建立博客文件

```bash
$ hugo new site newTheme
$ ls -l
total 28
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 15:52 archetypes
-rw-r--r-- 1 shuzang shuzang   82 Sep 25 15:52 config.toml
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 15:52 content
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 15:52 data
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 15:52 layouts
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 15:52 static
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 15:52 themes
```

创建主题

```bash
$ hugo new theme LeaveIt
$ find themes -type f | xargs ls -l
-rw-r--r-- 1 shuzang shuzang    8 Sep 25 16:09 themes/LeaveIt/archetypes/default.md
-rw-r--r-- 1 shuzang shuzang    0 Sep 25 16:09 themes/LeaveIt/layouts/404.html
-rw-r--r-- 1 shuzang shuzang    0 Sep 25 16:09 themes/LeaveIt/layouts/_default/list.html
-rw-r--r-- 1 shuzang shuzang    0 Sep 25 16:09 themes/LeaveIt/layouts/_default/single.html
-rw-r--r-- 1 shuzang shuzang    0 Sep 25 16:09 themes/LeaveIt/layouts/index.html
-rw-r--r-- 1 shuzang shuzang    0 Sep 25 16:09 themes/LeaveIt/layouts/partials/footer.html
-rw-r--r-- 1 shuzang shuzang    0 Sep 25 16:09 themes/LeaveIt/layouts/partials/header.html
-rw-r--r-- 1 shuzang shuzang 1081 Sep 25 16:09 themes/LeaveIt/LICENSE.md
-rw-r--r-- 1 shuzang shuzang  436 Sep 25 16:09 themes/LeaveIt/theme.toml
```

`find themes -type f`将themes目录及其子目录的所有一般文件列出

`xargs ls -l`在`find`命令基础上列出这些文件的基本信息

LeaveIt 包含 templates (以 .html为后缀的文件), license 文件, 主题描述 (theme.toml 文件), 和一个空的archetype.

license文件默认填充MIT协议

根据引号中的描述编辑theme.toml文件，声明主题基本信息，参照[hugoThemes](https://github.com/gohugoio/hugoThemes#themetoml )的说明

```bash
$ nano themes/LeaveIt/theme.toml
name = "Theme Name"
license = "MIT"
licenselink = "Link to theme's license"
description = "Theme description"
homepage = "Website of your theme"
tags = ["blog", "company"]
features = ["some", "awesome", "features"]
min_version = "0.57.0"

[author]
    name = "Your name"
    homepage = "Your website"

# If porting an existing theme
[original]
    author = "Name of original author"
    homepage = "His/Her website"
    repo = "Link to source code of original theme"
```

编辑config.toml文件，在文件名添加theme字段，启用主题

```bash
$ nano config.toml
baseURL = "http://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "LeaveIt"
```

重新生成网站，public文件夹和themes/LeaveIt都将生成js和css文件夹用于控制格式

```bash
$ hugo --verbose
$ ls -l public
total 24
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 15:56 categories
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 16:09 css
-rw-r--r-- 1 shuzang shuzang  468 Sep 25 16:43 index.xml
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 16:09 js
-rw-r--r-- 1 shuzang shuzang  437 Sep 25 16:43 sitemap.xml
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 15:56 tags
$ find themes/LeaveIt -type d | xargs ls -ld
drwxr-xr-x 5 shuzang shuzang 4096 Sep 25 16:32 themes/LeaveIt
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 16:09 themes/LeaveIt/archetypes
drwxr-xr-x 4 shuzang shuzang 4096 Sep 25 16:09 themes/LeaveIt/layouts
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 16:09 themes/LeaveIt/layouts/_default
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 16:09 themes/LeaveIt/layouts/partials
drwxr-xr-x 4 shuzang shuzang 4096 Sep 25 16:09 themes/LeaveIt/static
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 16:09 themes/LeaveIt/static/css
drwxr-xr-x 2 shuzang shuzang 4096 Sep 25 16:09 themes/LeaveIt/static/js
```

### 开发流程

删除public文件夹，执行hugo server命令并添加--watch参数，打开浏览器查看网站，修改主题文件，浏览器页面将会实时更改

建议：不要直接在主题上开发，使用git，在开发分支开发，确认功能无误后合并到主分支

```bash
# 删除public文件夹
$ rm -rf public
# 以watch模式实时查看改动
$ hugo server --watch --verbose
```

更新Home Page 模板，位于layout/目录下：

- index.html
- _default/list.html
- _default/single.html

现在，模板文件是空的，添加以下内容到模板文件

```bash
$ nano themes/LeaveIt/layouts/index.html
<html> 
<body> 
  <p>hugo says hello!</p> 
</body> 
</html>
```

浏览器将显示p标签中的语句

hugo中有三种类型的模板，home页面模板刚刚已修改，single类型的模板用于生成单个内容的文件，list类型的模板用于生成多个内容的文件，即single.html和list.html

还存在partial，content views和terms三种类型的模板，这里不做详细介绍。

更新index.html使之能显示主页

```html
<!DOCTYPE html>
<html>
<body>
  {{ range first 10 .Data.Pages }}
    <h1>{{ .Title }}</h1>
  {{ end }}
</body>
</html>
```

hugo使用Go模板引擎，引擎将扫描模板文件中”{{“和”}}“中间的命令内容并执行，上面的index.html文件中包含三个命令

1. range
2. .Title
3. end

range是迭代器，用于遍历前十个page，hugo创建的每个html文件都属于page，所以遍历pages将查看创建的每个文件

.Title打印`title`变量的值，hugo从markdown文件前面的格式说明中获取该值

end结束range开启的迭代

hugo会在创建html文件前将所有内容加载到变量中，然后模板对其进行处理

因此，更新后的index.html文件将显示first.md和second.md两篇博文的标题

到此开发一个主题的所有知识就全部介绍完毕了，剩下的就是熟悉Go模板的命令及使用它。但这里还有一些内容可以令我们更容易地创建模板

修改single.html文件使之展示博文内容

```html
<!DOCTYPE html>
<html>
<head>
  <title>{{ .Title }}</title>
</head>
<body>
  <h1>{{ .Title }}</h1>
  {{ .Content }}
</body>
</html>
```

修改index.html为博文标题标题添加链接

```html
<!DOCTYPE html>
<html>
<body>
  {{ range first 10 .Data.Pages }}
    <h1><a href="{{ .Permalink }}">{{ .Title }}</a></h1>
  {{ end }}
</body>
</html>
```

在content目录创建about.md文件，作为主页面顶级菜单

```markdown
+++
title = "about"
description = "about this site"
date = "2014-09-27"
slug = "about time"
+++

## about us

i'm speechless
```

浏览器主页面将显示about菜单，单击即可跳转查看其内容，但它和博文标题放在一起，因此修改index.html文件

```html
<!DOCTYPE html>
<html>
<body>
  <h1>posts</h1>
  {{ range first 10 .Data.Pages }}
    {{ if eq .Type "post"}}
      <h2><a href="{{ .Permalink }}">{{ .Title }}</a></h2>
    {{ end }}
  {{ end }}

  <h1>pages</h1>
  {{ range .Data.Pages }}
    {{ if eq .Type "page" }}
      <h2><a href="{{ .Permalink }}">{{ .Title }}</a></h2>
    {{ end }}
  {{ end }}
</body>
</html>
```

主页面将分两部分显示：博文和页面。并链接到不同的页面。

改变配置文件中的permalinks，设置页面名

```toml
[permalinks]
	page = "/:title/"
	about = "/:filename/"
```

如果模板文件存在重叠，可以把重叠部分以共享模板的形式放在themes/LeaveIt/layouts/partials/目录。通常，模板文件引用都会有一个指定的路径，但是partials没有，hugo会沿着TODO定义的搜索路径去使用，因此可以编辑这里的内容来覆盖主题的表达。

```bash
$ vi themes/LeaveIt/layouts/partials/header.html
<!DOCTYPE html>
<html>
<head>
	<title>{{ .Title }}</title>
</head>
<body>

$ vi themes/LeaveIt/layouts/partials/footer.html
</body>
</html>
```

partials和普通模板调用的不同是没有路径，比如

```html
{{ template "theme/partials/header.html" . }}
vs
{{ partial "header.html" . }}
```

更新index.html文件内容

```html
{{ partial "header.html" . }}

  <h1>posts</h1>
  {{ range first 10 .Data.Pages }}
    {{ if eq .Type "post"}}
      <h2><a href="{{ .Permalink }}">{{ .Title }}</a></h2>
    {{ end }}
  {{ end }}

  <h1>pages</h1>
  {{ range .Data.Pages }}
    {{ if or (eq .Type "page") (eq .Type "about") }}
      <h2><a href="{{ .Permalink }}">{{ .Type }} - {{ .Title }} - {{ .RelPermalink }}</a></h2>
    {{ end }}
  {{ end }}

{{ partial "footer.html" . }}
```

更新single.html文件

```html
{{ partial "header.html" . }}

  <h1>{{ .Title }}</h1>
  {{ .Content }}

{{ partial "footer.html" . }}
```

此时浏览器中主页面about菜单将额外展示类型和标题

更新single.html文件，添加对博文日期的渲染

```html
{{ partial "header.html" . }}

  <h1>{{ .Title }}</h1>
  <h2>{{ .Date.Format "Mon, Jan 2, 2006" }}</h2>
  {{ .Content }}

{{ partial "footer.html" . }}
```

此时浏览器博文页面将展示创建日期

有多种方式使日期只展示在博文页面中，一种是在index.html中添加if条件语间，另一种是为博文创建单独的模板。当主题比较复杂时，创建针对博文的单独模板

```bash
$ mkdir themes/LeaveIt/layouts/post
$ nano themes/LeaveIt/layouts/_default/single.html
{{ partial "header.html" . }}

  <h1>{{ .Title }}</h1>
  {{ .Content }}

{{ partial "footer.html" . }}
$ nano themes/LeaveIt/layouts/post/single.html
{{ partial "header.html" . }}

  <h1>{{ .Title }}</h1>
  <h2>{{ .Date.Format "Mon, Jan 2, 2006" }}</h2>
  {{ .Content }}

{{ partial "footer.html" . }}
```

移除了默认的single模板中对日期的指定，而在post的single模板添加

至此对主题的基本结构有了了解，接下来应该进一步学习Hugo中Go模板的语法，参见[Introduction to Go Templates](https://github.com/digitalcraftsman/hugo-steam-theme/blob/master/exampleSite/content/post/goisforlovers.md)

注意，以下的调整都是对原主题样式的调整，需要修改主题源码，因此最好clone主题仓库进行开发。

## 2. 调整引用样式

主题原有的引用样式如下

> 引用样式示例

修改`assets/css/_common/_core/base.scss`文件如下调整引用样式

```diff
  }

  blockquote {
-    font-size: 1rem;
-    display: block;
-    border-width: 1px 0;
-    border-style: solid;
-    border-color: $light-border-color;
-    padding: 1.5em 1.2em 0.5em 1.2em;
-    margin: 0 0 2em 0;
-    position: relative;
-
-    &:before {
-      content: '\201C';
-      position: absolute;
-      top: 0em;
-      left: 50%;
-      transform: translate(-50%, -50%);
-      width: 3rem;
-      height: 2rem;
-      font: 6em/1.08em 'PT Sans', sans-serif;
-      color: $light-post-link-color;
-      text-align: center;
-
-       .dark-theme &{
-         color: $dark-post-link-color;
-       }
-    }
-    &:after {
-      content: "#blockquote" attr(cite);
-      display: block;
-      text-align: right;
-      font-size: 0.875em;
-      color: $light-post-link-color;
-
-       .dark-theme &{
-         color: $dark-post-link-color;
-       }
-    }
+    font: 14px/22px normal helvetica, sans-serif;
+    margin-top: 10px;
+    margin-bottom: 10px;
+    margin-left: 2%;
+    margin-right: 0%;
+    padding-left: 15px;
+    padding-top: 10px;
+    padding-right: 10px;
+    padding-bottom: 10px;
+    border-left: 3px solid #ccc;
+    background-color:#f1f1f1;  

    .dark-theme & {
-      border-color: $dark-border-color;
+      background-color:#252529;
    }
  } 
```

## 3. 添加阅读进度条

首先在`layouts/partials/header.html`文件中如下所示插入代码

```diff
<nav class="navbar">
+    {{ if (and .IsPage (not .Params.notsb)) }}
+        <div class="top-scroll-bar"></div>
+    {{ end }}
    <div class="container">
        <div class="navbar-header header-logo">
        	<a href="{{ .Site.BaseURL }}">{{ .Site.Title }}</a>
@@ -13,7 +16,10 @@
    </div>
</nav>
<nav class="navbar-mobile" id="nav-mobile" style="display: none">
+    {{ if (and .IsPage (not .Params.notsb)) }}
+        <div class="top-scroll-bar"></div>
+    {{ end }}
    <div class="container">
        <div class="navbar-header">
            <div>  <a href="javascript:void(0);" class="theme-switch"><i class="iconfont icon-sun"></i></a>&nbsp;<a href="{{.Site.BaseURL}}">{{ .Site.Title }}</a></div>
            <div class="menu-toggle">

```

然后在`assets/css/_cusstom.scss`文件中添加进度条样式

```scss
// 顶部阅读进度条
.top-scroll-bar {
    position: fixed;
    top: 0;
    left: 0;
    z-index: 9999;
    display: none;
    width: 0;
    height: 3px;
    background: #ef3982;
  } 
```

之后新建一个 js 脚本文件 `assets/js/_custom.js` ，来控制进度条

```js
// 顶部阅读进度条
$(document).ready(function () {
    $(window).scroll(function(){
      $(".top-scroll-bar").attr("style", "width: " + ($(this).scrollTop() / ($(document).height() - $(this).height()) * 100) + "%; display: block;");
    });
  }); 
```

最后， js 脚本引入到博客中，使其生效。

在 `/layouts/partials/js.html` 文件中添加以下内容，然后将 `$custom` 加入到变量 `$vendorscript` 中

```html
{{ $custom := resources.Get "/js/_custom.js" }}
```

重新编译后预览博客就可以看到阅读进度条了。

## 4. 添加侧边栏目录

原本LeaveIt主题不支持文章目录导航，可以按如下方法添加，但样式不是很合适

首先在`/assets/css/_custom.scss`添加侧边栏目录(toc)样式

```scss
// 添加toc栏
  .post-toc {
    position: absolute;
    width: 200px;
    margin-left: 800px;
    padding: 10px;
    word-wrap: break-word;
    box-sizing: border-box;

    .post-toc-title {
        margin: 0;
        font-weight: 400;
        text-transform: uppercase;
    }

    .post-toc-content {
        &.always-active ul {
            display: block;
        }

        >nav>ul {
            margin: 10px 0;
        }

        ul {
            padding-left: 0;
            list-style: none;

            ul {
            padding-left: 15px;
            display: none;
            }

            .has-active > ul {
                display: block;
            }
        }
    }

    a:hover {
        color: #c05b4d;
        -webkit-transform: scale(1.1);
        -ms-transform: scale(1.1);
        transform: scale(1.1);
    }

    a {
        display: block;
        line-height: 30px;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
        -webkit-transition-duration: .2s;
        transition-duration: .2s;
        -webkit-transition-property: -webkit-transform;
        transition-property: -webkit-transform;
        transition-property: transform;
        transition-property: transform,-webkit-transform;
        -webkit-transition-timing-function: ease-out;
        transition-timing-function: ease-out;
    }
}
@media only screen and (max-width: 1224px) {
    .post-toc {
        display: none;
    }
} 
```

然后在 `layouts/partials/` 下新建 `toc.html` 文件，内容如下

```html
<div class="post-toc" id="post-toc">
  <h2 class="post-toc-title">{{ T "toc" }}</h2>
  {{ $globalAutoCollapseToc := .Site.Params.autoCollapseToc | default false }}
  <div class="post-toc-content{{ if not (or .Params.autoCollapseToc (and $globalAutoCollapseToc (ne .Params.autoCollapseToc false))) }} always-active{{ end }}">
    {{.TableOfContents}}
  </div>
</div>

<script type="text/javascript">
  window.onload = function () {
    var fix = $('.post-toc');
    var end = $('.post-comment');
    var fixTop = fix.offset().top, fixHeight = fix.height();
    var endTop, miss;
    var offsetTop = fix[0].offsetTop;
    $(window).scroll(function () {
      var docTop = Math.max(document.body.scrollTop, document.documentElement.scrollTop);
      if (end.length > 0) {
        endTop = end.offset().top;
        miss = endTop - docTop - fixHeight;
      }
      if (fixTop < docTop) {
        fix.css({ 'position': 'fixed' });
        if ((end.length > 0) && (endTop < (docTop + fixHeight))) {
          fix.css({ top: miss });
        } else {
          fix.css({ top: 0 });
        }
      } else {
        fix.css({ 'position': 'absolute' });
        fix.css({ top: offsetTop });
      }
    })
  }
</script>  
```

在文章页的模板 `layouts/_default/single.html` 中`</header>`标签后引入toc模板

```html
 {{ if ( .Site.Params.toc | default true ) }}
     {{ partial "toc.html" . }}
 {{ end }}
```

添加后重新使用hugo生成静态页面，只要文章有多级标题，就能在侧边栏看到导航目录

最后在站点配置文件`config.toml`中添加如下配置

```toml
toc = true                # 是否开启目录
autoCollapseToc = true   # Auto expand and collapse toc
```

## 5. 图片添加阴影

在 `assets/css/style.scss`中添加以下代码

```scss
// 图片阴影
.post-content {
	img {
        box-shadow: 0 0 15px 5px rgba(0, 0, 0, .28);
        max-width: 1080px;
	}
}
```

