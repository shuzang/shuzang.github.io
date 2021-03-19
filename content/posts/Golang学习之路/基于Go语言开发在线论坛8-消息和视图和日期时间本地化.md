---
title: 基于Go语言开发在线论坛8-消息、视图和日期时间本地化
author: xueyuanjun
date: 2020-06-07T20:30:00+08:00 
lastmod: 2020-06-07
tags: [Go实战]
categories: [Golang学习之路]
slug: Development of online forum based on golang 8-Message, view and date time localizationsingleton mode
---

由于之前所有页面和消息文本都是英文的，而我们开发的应用基本都是面向中文用户的，所以需要对项目进行本地化，因此本篇介绍如何在 Go Web 应用中进行国际化和本地化编程，由于项目比较简单，只介绍消息提示、视图模板和日期格式的本地化。

<!--more-->

## 1. 消息本地化

### 1.1 安装 go-i18n 扩展包

首先来看消息提示文本，消息提示文本通常包括表单验证消息、应用异常消息、接口响应消息等后端接口返回的消息字符串片段，关于这一块的本地化，可以借助 Go 官方自带的 `golang.org/x/text` 扩展包实现，这个扩展包扩展性好，但是上手起来有点复杂，因此我们使用的是一款更容易上手的第三方扩展包 —— [go-i18n](https://github.com/nicksnyder/go-i18n)。

在使用这个扩展包之前，先在项目根目录下运行如下命令下载相关的扩展包：

```bash
go get -u github.com/nicksnyder/go-i18n/v2/i18n
go get -u github.com/nicksnyder/go-i18n/v2/goi18n
```

下载完成后，我们可以运行 `goi18n -help` 确保 `goi18n` 命令可执行

### 1.2 通过 go-i18n 自动生成翻译文件

接下来，我们来编写消息文本模板用于生成翻译文件。在这个项目中，只有一个消息提示文本，那就是访问的群组不存在时返回的 `Cannot read thread`，因此，我们在项目根目录下创建 `messages.go`，并基于 `go-i18n` 提供的类型编写消息模板如下：

```go
package main

import "github.com/nicksnyder/go-i18n/v2/i18n"

var messages = []i18n.Message{
    {
        ID: "thread_not_found",
        Description: "Thread not exists in db",
        Other: "Cannot read thread",
    },
}
```

其中 `ID` 是消息文本的唯一标识，`Other` 则是对应的翻译字符串（默认是英文），然后基于 `goi18n` 命令自动生成翻译文件到 `locales` 目录（执行前先创建 `locales` 目录）：

```go
mkdir locales
goi18n extract -outdir=locales -format=json messages.go
```

这样，就会在 `locales` 目录下生成可以被 `go-i18n` 包识别并解析的 JSON 格式翻译文件 `active.en.json`：

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200607_Snipaste_2020-06-07_20-37-07.png)

### 1.3 编写中文版本翻译文件

然后，要进行本地化编程，可以在同级目录下创建并编辑 `active.zh.json` 用于存放消息文本的中文翻译：

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200607_Snipaste_2020-06-07_20-38-01.png)

### 1.4 本地化配置初始化

回到在在线论坛项目，打开配置文件 `config.json`，新增本地化目录和语言配置：

```go
{
  "App": {
    ...
    "Locale": "locales",
    "Language": "zh"
  },
  ...
}
```

然后在 `config/config.go` 中新增与之映射的结构体字段，以及对应的初始化设置：

```go
package config

import (
    "encoding/json"
    "github.com/nicksnyder/go-i18n/v2/i18n"
    "golang.org/x/text/language"
    "log"
    "os"
    "sync"
)

type App struct {
    ...
    Locale       string
    Language     string
}

...

type Configuration struct {
    App App
    Db  Database
    LocaleBundle *i18n.Bundle
}

var config *Configuration
var once sync.Once

// 通过单例模式初始化全局配置
func LoadConfig() *Configuration {
    once.Do(func() {
        file, err := os.Open("config.json")
        if err != nil {
            log.Fatalln("Cannot open config file", err)
        }
        decoder := json.NewDecoder(file)
        config = &Configuration{}
        err = decoder.Decode(config)
        if err != nil {
            log.Fatalln("Cannot get configuration from file", err)
        }
        // 本地化初始设置
        bundle := i18n.NewBundle(language.English)
        bundle.RegisterUnmarshalFunc("json", json.Unmarshal)
        bundle.MustLoadMessageFile(config.App.Locale + "/active.en.json")
        bundle.MustLoadMessageFile(config.App.Locale + "/active." + config.App.Language + ".json")
        config.LocaleBundle = bundle
    })
    return config
}
```

注意我们在 `App` 结构体中新增了一个 `*i18n.Bundle` 类型的 `LocaleBundle` 字段，用于存放全局本地化 Bundle 实例，并且在 `LoadConfig()` 方法中以单例模式初始化该实例。

### 1.5 在处理器方法中返回本地化消息

接下来，我们打开 `handlers/helper.go`，在 `init` 方法中初始化 Localizer 以便被所有处理器方法使用：

```go
package handlers

import (
    ...
    "github.com/nicksnyder/go-i18n/v2/i18n"
    . "github.com/xueyuanjun/chitchat/config"
)

var logger *log.Logger
var config *Configuration
var localizer *i18n.Localizer

func init()  {
    // 获取全局配置实例
    config = LoadConfig()
    // 获取本地化实例
    localizer = i18n.NewLocalizer(config.LocaleBundle, config.App.Language)
    ...
}

...
```

最后在 `handlers/thread.go` 和 `handlers/post.go` 中调用 `errorMessage` 辅助函数的地方调用 Localizer 提供的方法对消息文本进行翻译并返回给用户：

```go
if err != nil {
    msg := localizer.MustLocalize(&i18n.LocalizeConfig{
        MessageID: "thread_not_found",
    })
    errorMessage(writer, request, msg)
} else {
    ...
}
```

### 1.6 测试消息本地化

重新启动应用，如果试图访问一个不存在的群组页面，就会返回如下中文提示信息：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15869249174748.jpg)说明我们的本地化翻译生效了，当然这里只是使用了 `go-i18n` 提供的最基本的功能，想要了解更多使用示例，可以参考如下链接：

*   [官方文档](https://github.com/nicksnyder/go-i18n)
*   [借助 go-i18n 更简单地实现全球化](https://zyfdegh.github.io/post/201809-translation-go-i18n/)

## 2. 视图本地化

所谓视图本地化指的是静态 HTML 视图模板的本地化，这里就不再适合使用消息文本翻译的方式实现了，最简单的方式就是为每个语言创建独立的视图模板进行本地化，然后在应用代码中通过读取全局配置、用户手动选择、客户端参数（比如 HTML 请求头中的 `Accept-Language` 字段）、或者域名信息来判断加载那种本地化视图模板，为了简化演示流程，这里我们使用全局配置的方式，也就是我们上面配置文件中设置的 `Language` 字段。

### 2.1 创建本地化视图模板

首先，我们在 `views` 目录下新增 `en` 和 `zh` 两个子目录，分别用于存放英文视图模板和中文视图模板，然后将原有视图文件移动到 `en` 目录下，并且在 `zh` 目录下创建每个视图模板的中文版本，以首页 `index.html` 为例，对应的中文版本如下：

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Snipaste_2020-06-07_20-40-32.png)

其他中文视图模板也是类似，将其中的英文文本统一翻译成中文即可。

### 2.2 通过配置加载本地化视图

打开 `handlers/helper.go`，在 `generateHTML` 方法中通过读取全局配置加载对应的本地化视图模板：

```go
func generateHTML(writer http.ResponseWriter, data interface{}, filenames ...string) {
    var files []string
    for _, file := range filenames {
        files = append(files, fmt.Sprintf("views/%s/%s.html", config.App.Language, file))
    }
    
    templates := template.Must(template.ParseFiles(files...))
    templates.ExecuteTemplate(writer, "layout", data)
}
```

非常简单，不再赘述。

> 注：同时移除 `parseTemplateFiles` 方法，并将调用该方法的地方调整为调用 `generateHTML` 以避免维护两个地方。

### 2.3 测试视图本地化

重启应用，访问首页，即可看到页面视图已经都是中文显示了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15869255064861.jpg)

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15869255266935.jpg)

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15869256552424.jpg)

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15869256718104.jpg)

## 3. 日期时间本地化

看起来都已经 OK 了，不过还有个小问题，那就是日期时间显示还是英文风格的，对应的实现代码在 `models/thread.go` 中：

```go
func (thread *Thread) CreatedAtDate() string {
    return thread.CreatedAt.Format("Jan 2, 2006 at 3:04pm")
}
```

我们当然可以直接修改这里来实现类似 `2006-01-02 15:04:05` 的日期时间格式（该时间节点是 Go 语言元年），不过，学院君这里换一种复杂一点的实现，以便顺手介绍下如何在 Go 视图模板中通过管道模式调用自定义函数。

### 3.1 将自定义函数应用到视图模板

打开 `handlers/helper.go`，新增一个格式化日期时间的函数 `formatDate`，然后在 `generateHTML` 方法中将这个函数通过 `template.FuncMap` 组装后再通过 `Funcs` 方法应用到视图模板中，这样，就可以在所有视图模板中通过 `fdate` 别名来调用 `formatDate` 函数了：

```go
// 生成 HTML 模板
func generateHTML(writer http.ResponseWriter, data interface{}, filenames ...string) {
    var files []string
    for _, file := range filenames {
        files = append(files, fmt.Sprintf("views/%s/%s.html", config.App.Language, file))
    }
    funcMap := template.FuncMap{"fdate": formatDate}
    t := template.New("layout").Funcs(funcMap)
    templates := template.Must(t.ParseFiles(files...))
    templates.ExecuteTemplate(writer, "layout", data)
}

...

// 日期格式化辅助函数
func formatDate(t time.Time) string {
    datetime := "2006-01-02 15:04:05"
    return t.Format(datetime)
}
```

### 3.2 调用自定义函数格式化本地日期时间

然后我们在所有视图文件中将群组创建日期渲染调整为如下方式，即通过管道连接符的方式将 `.CreatedAt` 变量作为参数传入 `fdate` 并输出返回值：

```go
{{ .CreatedAt | fdate }}
```

注意这里一定要使用 `.CreatedAt`，这个变量才是 `time.Time` 类型，而 `.CreatedAtDate` 是字符串类型。

再次重新启动应用，访问首页和群组详情页就可以看到格式化后的本地日期时间格式了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15869318235491.jpg)

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15869318394657.jpg)
