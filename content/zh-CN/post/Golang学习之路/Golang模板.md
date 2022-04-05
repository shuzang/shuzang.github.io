---
title: Golang模板
date: 2020-05-29T17:08:00+08:00
lastmod: 2020-05-29
tags: [Go实战]
categories: [Golang学习之路]
slug: Golang template 
---

模板就是在写动态页面时不变的部分，服务端程序渲染可变部分生成动态网页，Go 语言提供了 html/template 包来支持模板渲染。

<!--more-->

## 1. 基本语法

模板的基本语法来自于 text/template 包，与大多数语言一样，用 `{{` 和 `}}` 来做标识，`{{ }}` 里可以是表达式，也可以是变量

### 1.1 变量

模板中的变量通过{{.}} 来访问。Golang渲染template的时候，可以在模板文件中读取变量内的值并渲染到模板里。有两个常用的传入类型。一是struct，在模板内可以读取该struct的内容。二是map[string]interface{}，在模板内可以使用key来进行渲染。

以结构体为例，假设初始化了一个结构体 User

```go
type User struct {
    UserId int
    Username string
    Age uint
    Sex string
}
user := User{1, "Steven", 35, "男"}
```

模板获取数据的方式如下

```html
{{.}}{{.Username}}{{.UserId}}{{.Age}}{{.Sex}}
```

那么渲染后的模板内容如下

```go
{1 Steven 35 男}Steven135男
```

也可以在模板中定义变量，初始化后就可以进行调用，如下所示

```html
{{$MyUserName := "StevenWang"}}
{{$MyUserName}}
```

### 1.2 逻辑判断

Golang 模板支持 if 条件判断，当前支持最简单的 bool 类型和 string 类型，定义方式如下

```html
{{if .confition}}
{{end}}
```

当.condition是bool类型时，值为true表示执行。当.condition是string类型时，值非空表示执行。加入 else 时，形式如下

```html
{{if .confition}}
{{else}}
{{end}}
```

此模板也支持if…else if嵌套，定义方式如下所示

```html
{{if .confition}}
{{else if .confition}}
{{end}}
```

Golang的模板提供了一些内置的模板函数来执行逻辑判断，下面列举目前常用的一些内置模板函数

| 函数语法                                      | 函数作用    |
| --------------------------------------------- | ----------- |
| {{if not .condition}}<br>{{end}}              | not 非      |
| {{if and .condition1 .condition2}}<br>{{end}} | and 与      |
| {{if or .condition1 .condition2}}<br>{{end}}  | or 或       |
| {{if eq .var1 .var2}}<br>{{end}}              | eq 等于     |
| {{if ne .var1 .var2}}<br>{{end}}              | ne 不等于   |
| {{if lt .var1 .var2}}<br>{{end}}              | lt 小于     |
| {{if le .var1 .var2}}<br/>{{end}}             | le 小于等于 |
| {{if gt .var1 .var2}}<br/>{{end}}             | gt 大于     |
| {{if ge .var1 .var2}}<br/>{{end}}             | ge 大于等于 |

假设在 Go 程序中定义了一个map，如下

```go
locals := make(map[string]interface{})
locals["username1"] = "Steven"
locals["username2"] = "Daniel"
```

在模板文件中进行逻辑判断如下

```html
{{if eq .username .user}}
	OK:账号名称一致
{{else if ne .username .user}}
	Err:账号名称不一致
{{end}}
```

逻辑判断也可以使用 with 作为关键词

```html
{{with .condition}}
{{end}}
```

或者

```html
{{with .condition}}
{{else}}
{{end}}
```

### 1.3 循环遍历

Golang 的模板支持 range 循环来遍历 map、slice中的内容，语法如下

```html
{{range $index, $value := .slice}}
{{end}}
```

在这个range循环内，遍历数据通过`$index`和 `$value`来实现。还有一种遍历方式，语法格式如下所示

```html
{{range .slice}}
{{end}}
```

这种方式无法访问到`$index`和`$key`的值，需要通过{{.}}来访问对应的$value。这种情况下，在循环体内，外部变量需要使用{{$.}}来访问。

模板文件的示例代码如下

```html
{{range $index, $value := .filelist}}
  <figure>
      <a href="/html/upload/{{$value}}"><img src="/html/upload/{{$value}}"/></a>
      <figcaption>
      {{$value}} <br/>
          <a href="/delete?id={{$value}}">[删除]</a> {{$.username}}上传
      </figcaption>
  </figure>
{{end}}
```

循环也有 else 语句，如果作为循环条件的数组、切片、映射或通道长度为0，就执行 else 后的语句。

```html
{{range .slice}}
{{else}}
{{end}}
```

### 1.4 模板嵌套

在编写模板的时候，经常需要将公用的模板进行整合，比如每一个页面都有导航栏和页脚，通常的做法是将其编写为一个单独的模块，让所有的页面进行导入，这样就不用重复编写了。

任何网页都有一个主模板，然后可以在主模板内嵌入子模板来实现模块共享。当模板想要引入子模板时，通常使用如下语句

```html
{{template "navbar"}}
```

这样就会载入名为 navbar 的子模版，同时，我们需要定义 navbar 子模版的实现

```html
{{define "navbar"}}
{{end}}
```

在定义中间的内容最终会替代源文件中的 {{template "navbar"}} 这一条语句

如果想要子模板获得父模板的变量，使用如下方法

```html
{{template "navbar" .}}
```

## 2. 模板函数

上面的语法一般情况都是定义在前端文件中的，要对模板进行处理，html/template 还提供了一系列函数。

一个简单的例子如下

```go
import "text/template"
...
t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
err = t.ExecuteTemplate(out, "T", "<script>alert('you have been pwned')</script>")
```

涉及到的几个函数分别是 New，Parse 和 ExecuteTemplate，解释如下

```go
// 创建一个名为name的模板
func New(name string) *Template  
// Parse方法将字符串text解析为模板
func (t *Template) Parse(src string) (*Template, error)
// 将解析好的模板应用到data上，使用名为name的t关联的模板产生输出。
func (t *Template) ExecuteTemplate(wr io.Writer, name string, data interface{}) error
```

所以 New 和 Parse 创建一个模板并将字符串解析到该模板，但实际使用时模板文件通常是单独的，这时一般使用 Must 和 ParseFiles 函数。

ParseFiles方法解析filenames指定的文件里的模板定义并将解析结果与t关联。如果发生错误，会停止解析并返回nil，否则返回(t, nil)。至少要提供一个文件。

```go
func (t *Template) ParseFiles(filenames ...string) (*Template, error)
```

可以看到 ParseFiles 返回 (*Template, error)，Must 函数正好用来封装这种返回形式的函数

```go
func Must(t *Template, err error) *Template
```

它会在 err 非 nil 时 panic，一般用于变量初始化

```go
var t = template.Must(template.New("name").Parse("html"))
```

另外，ExecuteTemplate 将输出指定到名为 name 的模板上，但还有一种简单形式如下

```go
func (t *Template) Execute(wr io.Writer, data interface{}) error
```

Execute方法将解析好的模板应用到data上，并将输出写入wr，唯一的区别就是没有绑定模板。