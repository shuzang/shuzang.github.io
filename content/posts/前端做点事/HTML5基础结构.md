---
title: HTML5基础结构
date: 2020-07-23T16:54:00+08:00
tags: [HTML/CSS]
categories: [前端做点事]
slug: HTML5 learning basic structure
typora-root-url: ..\..\..\static
---

HTML（HyperText Markup Language，超文本标记语言） 是前端三组件（HTML/CSS/JavaScript）的第一个，负责组织一个网页的结构。HTML5 是 HTML 的最新版本，这篇文章学习其基本结构。

<!--more-->

HTML 组织网页结构的意思是，它告诉浏览器当前显示的内容是一段文字、一个图片还是一个链接等，同时控制各部分之间的位置关系。

HTML 不是一个具备图灵完毕特性的编程语言，它最重要的概念有四个：**元素、标签、内容、属性**。下面是一个示例 HTML 语句，定义了一个段落

```html
<p>我的猫咪脾气爆:)</p>
```

这个最简单的 HTML 语句反映了元素、标签、内容之间的关系，如下图

![](https://mdn.mozillademos.org/files/16475/element.png)

标签的相关特性如下：

1. 标签由英文尖括号包围，一般成对出现，分别为开始标签和结束标签，结束标签多一个 `/` ；
2. 标签之间可以嵌套；
3. 标签不区分大小写，但一般以小写为准。

属性则是定义在标签中的语句，用于说明元素的额外信息。

![](https://mdn.mozillademos.org/files/16476/attribute.png)

属性的特性如下：

1. 包括属性名和属性值，两者以等号连接；
2. 属性名与标签名之间，连续的多个属性间，以空格分隔；
3. 属性值以引号包围，可以是单引号也可以是双引号，以个人习惯为准。

一个完整的 HTML 文档就是大量元素的一个组合，下面是一个最简单的 HTML 文档

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>制作我的第一个网页</title>
    </head>
    <body>
        <h1>Hello World</h1>
    </body>
</html>
```

里面包含了一个文档的基本组成

1. `!DOCKTYPE html`：声明文档类型，用来告诉浏览器这是一个 html 文档，很久以前这条语句很长，HTML5 标准中只需要这么短，该声明必须是 HTML 文档的第一行；
2. `<html></html>` ：整个页面需要用 `<html>` 标签对来包裹；
3. `<head></head>`：文档头部，包含所有你想包含在HTML页面中但不想在HTML页面中显示的内容。这些内容包括你想在搜索结果中出现的关键字和页面描述，CSS样式，字符集声明等等
4. `<body></body>`：包含了你访问页面时所有显示在页面上的内容，文本，图片，音频，游戏等等。

基本的页面结构就是这样，其它所有的内容都直接或嵌套的包含在 `<head>` 标签或 `<body>` 标签内。

