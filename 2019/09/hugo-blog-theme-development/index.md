# hugo搭建个人博客5-主题开发


本来想自己做主题开发的，但一看发现要实现的功能太多，自己又缺乏前端的开发经验，只能在原主题上做一些小调节了。关于主题开发的中文资料没有，这里参考[create a new theme](https://github.com/digitalcraftsman/hugo-steam-theme/blob/master/exampleSite/content/post/creating-a-new-theme.md)一文

#### 开发主题准备

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

#### 开发主题

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

进一步关于Go模板的语法参见[Introduction to Go Templates](https://github.com/digitalcraftsman/hugo-steam-theme/blob/master/exampleSite/content/post/goisforlovers.md)
