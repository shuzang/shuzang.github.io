# hugo开发主题2-模板


Hugo 用到最后，终究还是免不了走上自己修改甚至开发主题的道路，本篇介绍 Hugo 的核心之一：模板。

<!--more-->

## 1. 基本介绍

模板指的是 HTML 的模板，是将 HTML 中的一些内容抽象为变量，在页面渲染时才进行替换的一种方法，如果学过 Go，会比较容易理解，因为 Hugo 的模板正式以 Go 的 `html/template` 和 `text/template` 两个库为基础的。

简而言之，Hugo 中提到模板就是提到 HTML，写模板就是写 HTML。

Hugo 主题中 HTML 的入口文件为 `_default/baseof.html`，这是所有页面渲染时的默认模板，除非你指定了在查找顺序上具有更高优先级的 `baseof.html` 文件。

*注：查找顺序将在第 2 节介绍。*

```html
<!-- layouts/_default/baseof.html -->

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>
        {{ block "title" . }}
            <!-- Blocks may include default content. -->
            {{ .Site.Title }}
    	{{ end }}
    </title>
  </head>
  <body>
    <!-- Code that all your templates share, like a header -->
    {{ block "main" . }}
      <!-- The part of the page that begins to differ between templates -->
    {{ end }}
    {{ block "footer" . }}
    <!-- More shared code, perhaps a footer but that can be overridden if need be in  -->
    {{ end }}
  </body>
</html>
```

上面的代码与普通 HTML 代码的区别就是以双大括号 `{{ }}` 包围的区域，这就是模板的写法。以 `{{ .Site.Title }}` 为例，在渲染时会用主题配置文件 `config.toml` 中定义的 `title` 字段替换，是不是就像被抽象而出的变量?

`{{ block "title" }}` 是一种更大的抽象范围：代码块，我们会在其它的 HTML 文件中定义这段代码块，然后在渲染时替换这一部分内容，比如，我们在 `layouts/_default/list.html` 中定义了 `title` 和 `main` 代码块如下

```html
{{ define "title" }}
  <!-- This will override the default value set in baseof.html; i.e., "{{.Site.Title}}" in the original example-->
  {{ .Title }} &ndash; {{ .Site.Title }}
{{ end }}

{{ define "main" }}
  <h1>{{ .Title }}</h1>
  {{ .Content }}
{{ end }}
```

渲染时会替换掉 `baseof.html` 文件中如下内容，注意，额外定义的这部分代码要以 `{{ block “title”}}` 开始，指明替换哪一段 block，所有的模板语法代码段都要以 `{{ end }}` 结束。

```html
{{ block "title" . }}
    <!-- Blocks may include default content. -->
    {{ .Site.Title }}
{{ end }}
```

实际的操作是 `list.html` 从 `baseof.html` 继承所有代码，然后用自己定义的 `title` 和 `main` 代码段替换相关内容，是在 `list.html` 中操作的，不是在 `baseof.html` 中操作的。

## 2. 页面查找顺序

Hugo 主题所有的 HTML 文件都位于主题目录下 `layouts` 目录中，我们所看到的博客的每个页面的渲染，都要首先从这里选择一个 HTML 文件作为模板，页面查找顺序指的是渲染页面时在 `layouts` 目录下的查找顺序。比如，Home Page 首先在 `layouts` 根目录下查找 `index.html` 文件，如果找不到，查找 `home.html` 文件或 `list.html` 文件，如果依然找不到，在 `layouts/_default` 目录下查找这三个文件。

与查找顺序相关的，除了上例提到的文件夹的不同，还和页面类型（Kind）、输出格式（Output Format）、语言（Language）等有关，我们主要关注的是**页面类型 Kind**。

页面类型有两种：single page 和 list page。前者用于单个页面渲染，后者用于在单个页面中渲染多个内容，举个例子，一个归档页面通常包含数十篇文章，这就属于 list page。其中，regular page 属于前者，section page、home page、taxonomy list page、taxonomy term page 属于后者。下面进行一一介绍。

*注：home page 是个例外，它虽然属于 list page，但有自己的专用模板。*

#### home page

网站首页通常和其它页面不同，因此有自己的专用模板，而且当网站只有一个页面时，home page 是唯一需要的文件。

home page 可以读取 `config.toml` 中定义的网站全局变量，也可以读取 `content/_index.md` 中的元数据（通常以 yaml/toml 格式定义在开头，称为 front matter）。

一些重要的文件查找顺序如下，其中 Home page 是主页面，Base template for home page 是主页面继承的模板，RSS home 是主页面的 RSS 文件。

| Example                     | OutputFormat | Suffix | Template Lookup Order                                        |
| :-------------------------- | :----------- | :----- | :----------------------------------------------------------- |
| Home page                   | HTML         | html   | layouts/index.html<br> layouts/home.html<br> layouts/list.html<br> layouts/\_default/index.html<br> layouts/\_default/home.html<br> layouts/\_default/list.html |
| Base template for home page | HTML         | html   | layouts/index-baseof.html<br> layouts/home-baseof.html<br> layouts/list-baseof.html<br> layouts/baseof.html<br> layouts/\_default/index-baseof.html<br> layouts/\_default/home-baseof.html<br> layouts/\_default/list-baseof.html<br> layouts/\_default/baseof.html |
| RSS home                    | RSS          | xml    | layouts/index.xml<br> layouts/home.xml<br> layouts/list.xml<br> layouts/\_default/index.xml<br> layouts/\_default/home.xml<br> layouts/\_default/list.xml |

我们重点介绍一下 Base template for home page，在 Home page 中，定义的是一段代码块，正如我们在第一节中所述，它需要先继承一个 HTML 文件的结构，然后替换其中使用 `{{ block "name" }}` 定义的部分，这里所继承的文件就是 Base template for home page。下面是一个例子

```html
<!-- layouts/index.html -->
{{ define "main" }}
    <main aria-role="main">
      <header class="homepage-header">
        <h1>{{.Title}}</h1>
        {{ with .Params.subtitle }}
        <span class="subtitle">{{.}}</span>
        {{ end }}
      </header>
      <div class="homepage-content">
        <!-- Note that the content for index.html, as a sort of list page, will pull from content/_index.md -->
        {{.Content}}
      </div>
      <div>
        {{ range first 10 .Site.RegularPages }}
            {{ .Render "summary"}}
        {{ end }}
      </div>
    </main>
{{ end }}
```

#### section page

section 是一组页面的集合，默认的，`content` 目录下每个一级目录都是一个 section，更深层次的目录，如果想成为 section，需要建立 _index.md 文件。一个例子如下

```bash
content
└── blog        <-- Section, because first-level dir under content/
    ├── funny-cats
    │   ├── mypost.md
    │   └── kittens         <-- Section, because contains _index.md
    │       └── _index.md
    └── tech                <-- Section, because contains _index.md
        └── _index.md
```

section page 就是用来渲染 section 的模板，可以访问 section 中的所有内容和 front matter，其查找顺序如下

| Example                          | OutputFormat | Suffix | Template Lookup Order                                        |
| :------------------------------- | :----------- | :----- | :----------------------------------------------------------- |
| Section list for "posts" section | HTML         | html   | layouts/posts/posts.html<br> layouts/posts/section.html<br> layouts/posts/list.html<br> layouts/section/posts.html<br> layouts/section/section.html<br> layouts/section/list.html<br> layouts/\_default/posts.html<br> layouts/\_default/section.html<br> layouts/\_default/list.html |

下面是一个 section page 的例子

```html
<!-- layouts/_default/section.html -->
{{ define "main" }}
  <main>
      {{ .Content }}
          <ul class="contents">
          {{ range .Paginator.Pages }}
              <li>{{.Title}}
                  <div>
                    {{ partial "summary.html" . }}
                  </div>
              </li>
          {{ end }}
          </ul>
      {{ partial "pagination.html" . }}
  </main>
{{ end }}
```

`.Site.GetPage` 函数可以用来获取一组给定类型的页面，假设 content 目录结构如下

```bash
.
└── content
    ├── blog
    │   ├── _index.md # "title: My Hugo Blog" in the front matter
    │   ├── post-1.md
    │   ├── post-2.md
    │   └── post-3.md
    └── events #Note there is no _index.md file in "events"
        ├── event-1.md
        └── event-2.md
```

输出 blog section 的 title 可以使用如下语句

```html
<h1>{{ with .Site.GetPage "section" "blog" }}{{ .Title }}{{ end }}</h1>
<!-- 输出如下 -->
<h1>My Hugo Blog</h1>
```

如果是 event section，因为没有 _index.md 文件，会输出文件名

```html
<h1>{{ with .Site.GetPage "section" "events" }}{{ .Title }}{{ end }}</h1>
<!-- 输出如下 -->
<h1>Events</h1>
```

#### taxonomy page

包括 taxonomy list pages 和 taxonomy terms pages，用于用户自定义的内容分组，比如分类和标签页面，展示了文章间的一种逻辑关系。下面解释几个术语

- Taxonomy：对内容进行分类的字段
- Term：分类的键
- Value：分类的值，就是内容

举个例子，`演员` 是 Taxonomy，`克里斯·埃文斯` 是  Term，`《美国队长》` 是 Value，如果按 Taxonomy 组织页面，就是 taxonomy list pages，如果按 Term 组织页面，就是 taxonomy terms pages，如果是显示单个内容，那么使用 single page。页面的查找顺序如下

| Example                     | OutputFormat | Suffix | Template Lookup Order                                        |
| :-------------------------- | :----------- | :----- | :----------------------------------------------------------- |
| Taxonomy list in categories | HTML         | html   | layouts/categories/terms.html<br> layouts/categories/taxonomy.html<br> layouts/categories/list.html<br> layouts/category/terms.html<br> layouts/category/taxonomy.html<br> layouts/category/list.html<br> layouts/taxonomy/terms.html<br> layouts/taxonomy/taxonomy.html<br> layouts/taxonomy/list.html<br> layouts/\_default/terms.html<br> layouts/\_default/taxonomy.html<br> layouts/\_default/list.html |
| Taxonomy term in categories | HTML         | html   | layouts/categories/term.html<br> layouts/categories/category.html<br> layouts/categories/taxonomy.html<br> layouts/categories/list.html<br> layouts/term/term.html<br> layouts/term/category.html<br> layouts/term/taxonomy.html<br> layouts/term/list.html<br> layouts/taxonomy/term.html<br> layouts/taxonomy/category.html<br> layouts/taxonomy/taxonomy.html<br> layouts/taxonomy/list.html<br> layouts/category/term.html<br> layouts/category/category.html<br> layouts/category/taxonomy.html<br> layouts/category/list.html<br> layouts/\_default/term.html<br> layouts/\_default/category.html<br> layouts/\_default/taxonomy.html<br> layouts/\_default/list.html |

#### single page

single page 是 Hugo 最主要的一个模板，用来渲染每一篇 Markdown 文章。页面的查找顺序如下

| Example                                          | OutputFormat | Suffix | Template Lookup Order                                        |
| :----------------------------------------------- | :----------- | :----- | :----------------------------------------------------------- |
| Single page in "posts" section                   | HTML         | html   | layouts/posts/single.html<br>layouts/_default/single.html    |
| Base template for single page in "posts" section | HTML         | html   | layouts/posts/single-baseof.html<br>layouts/posts/baseof.html<br>layouts/\_default/single-baseof.html<br>layouts/\_default/baseof.html |

一个 single page 的示例如下

```html
<!-- layouts/posts/single.html -->
{{ define "main" }}
<section id="main">
  <h1 id="title">{{ .Title }}</h1>
  <div>
        <article id="content">
           {{ .Content }}
        </article>
  </div>
</section>
<aside id="meta">
    <div>
    <section>
      <h4 id="date"> {{ .Date.Format "Mon Jan 2, 2006" }} </h4>
      <h5 id="wordcount"> {{ .WordCount }} Words </h5>
    </section>
    {{ with .Params.topics }}
    <ul id="topics">
      {{ range . }}
        <li><a href="{{ "topics" | absURL}}{{ . | urlize }}">{{ . }}</a> </li>
      {{ end }}
    </ul>
    {{ end }}
    {{ with .Params.tags }}
    <ul id="tags">
      {{ range . }}
        <li> <a href="{{ "tags" | absURL }}{{ . | urlize }}">{{ . }}</a> </li>
      {{ end }}
    </ul>
    {{ end }}
    </div>
    <div>
        {{ with .PrevInSection }}
          <a class="previous" href="{{.Permalink}}"> {{.Title}}</a>
        {{ end }}
        {{ with .NextInSection }}
          <a class="next" href="{{.Permalink}}"> {{.Title}}</a>
        {{ end }}
    </div>
</aside>
{{ end }}
```


