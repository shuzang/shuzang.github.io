---
title: hugo开发主题1-内容管理
date: 2020-11-11
tags: [Hugo]
categories: [爱编程爱技术的孩子]
slug: Hugo develop themes 1-content management
---

Hugo 用到最后，终究还是免不了走上自己修改甚至开发主题的道路，本篇首先介绍 Hugo 的内容如何管理。

<!--more-->

## 1. 目录结构

Hugo 建立的项目根目录初始有如下几个子目录

```bash
$ ls
archetypes/  config.toml  content/  data/  layouts/  static/  themes/
```

在使用过程中，我们书写的所有文章都应当放在 `content` 子目录下，其组织方式要参照博客网站的结构，因为 Hugo 会假设你在 `content` 目录中存放文件的结构就是你的网站结构，下面是一个例子：

```bash
# 假设网站 baseURL 为 https://example.com
.
└── content
	└── 1-logo.png
	└── _index.md
    └── about
    |   └── index.md  // <- https://example.com/about/
    ├── posts
    |   ├── firstpost.md   // <- https://example.com/posts/firstpost/
    |   ├── happy
    |   |   └── ness.md  // <- https://example.com/posts/happy/ness/
    |   └── secondpost.md  // <- https://example.com/posts/secondpost/
    └── quote
        ├── first.md       // <- https://example.com/quote/first/
        └── second.md      // <- https://example.com/quote/second/
```

其中，content 根目录、about 目录、posts 目录和 quote 目录都是一个单独的页面，里面可以包含要显示的 markdown 文档，也可以包含图片等资源文件，这样一个子目录叫做一个 `Page Bundles`。

*注：页面资源和图片如何在 HTML 中访问和处理可以参考第 2 节和第 3 节。*

索引页面，即 `_index.md` 在 Hugo 中扮演一个特殊的角色，它可以定义大量网站 HTML 文件可访问的元数据，一般通过 `.Site.GetPage` 函数来获取。





