---
title: 基于Go语言开发在线论坛7-通过单例模式获取全局配置
author: xueyuanjun
date: 2020-06-07T16:25:00+08:00 
lastmod: 2020-06-07
tags: [Go实战]
categories: [Golang学习之路]
slug: Development of online forum based on golang 7-Get global configuration through singleton mode
---

本文介绍如何将敏感信息或可变信息通过配置文件进行配置，然后在应用中读取这些配置文件来获取配置信息。

<!--more-->

将敏感信息通过配置文件读取是为了避免随着代码提交到公开库造成敏感信息的泄露，给线上环境带来安全隐患，这些敏感信息包括数据库连接信息、第三方 SDK （比如微信、支付宝、Github）的密钥等。

将可变信息通过配置文件读取是为了避免硬编码，将经常变动的信息通过配置文件配置可以极大的提高代码的可维护性，这些可变信息通常包括应用服务器监听的地址和端口、目录路径设置、当前运行环境、超时时间等。

使用公开库时，如 github，配置文件需要写入 .gitignore 文件从而避免提交到线上。

## 1. 定义全局配置文件

接下来，我们为在线论坛项目设置配置文件 `config.json`，将一些敏感信息和可变信息提交到 JSON 配置文件中

```json
{
  "App": {
    "Address": "0.0.0.0:8080",
    "Static": "public",
    "Log": "logs"
  },
  "Db": {
    "Driver": "mysql",
    "Address": "localhost:3306",
    "Database": "chitchat",
    "User": "root",
    "Password": "root"
  }
}
```

应用相关的可变信息配置到 `app` 配置项，数据库相关的敏感信息配置到 `Db` 配置项

## 2. 通过单例模式初始化全局配置

在根目录下创建 `config` 目录，然后在该目录下新增 `config.go` 用来存放配置初始化代码

```go
package config

import (
    "encoding/json"
    "log"
    "os"
    "sync"
)

type App struct {
    Address      string
    Static       string
    Log          string
}

type Database struct {
    Driver      string
    Address        string
    Database    string
    User        string
    Password    string
}

type Configuration struct {
    App App
    Db  Database
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
    })
    return config
}
```

定义 `Configuration` 结构体以便和全局配置文件 `config.json` 字段进行映射，注意这里的首字母都需要大写

定义一个 `LoadConfig` 方法以**单例模式**返回全局配置实例的指针，这里使用单例的原因是因为应用代码中可能多处都要获取配置值，重复加载配置文件进行 JSON 解码存在性能损耗（当然，定义 `init` 方法本身就可以支持全局运行一次，这里主要演示下单例模式如何实现）。在 Go 语言中，我们可以借助并发编程中的 `sync.Once` 类型来实现单例模式，保证并发安全，在 `once.Do` 中定义的匿名函数全局只会执行一次

## 3. 项目代码重构

最后，我们将项目代码中相应位置的硬编码调整为通过上面方法返回的全局配置实例获取配置值

### 3.1 Web 服务器启动参数

首先需要在 `main.go` 的入口位置初始化全局配置

```go
package main

import (
    . "github.com/xueyuanjun/chitchat/config"
    . "github.com/xueyuanjun/chitchat/routes"
    "log"
    "net/http"
)

func main()  {
    startWebServer()
}

// 通过指定端口启动 Web 服务器
func startWebServer()  {
    // 在入口位置初始化全局配置
    config := LoadConfig()
    r := NewRouter() // 通过 router.go 中定义的路由器来分发请求

    // 处理静态资源文件
    assets := http.FileServer(http.Dir(config.App.Static))
    r.PathPrefix("/static/").Handler(http.StripPrefix("/static/", assets))

    http.Handle("/", r)

    log.Println("Starting HTTP service at " + config.App.Address)
    err := http.ListenAndServe(config.App.Address, nil)

    if err != nil {
        log.Println("An error occured starting HTTP listener at " + config.App.Address)
        log.Println("Error: " + err.Error())
    }
}
```

我们在 `startWebServer` 方法的入口位置初始化全局配置，并且全局配置实例只在这里进行一次初始化，后续不会再执行加载配置文件和 JSON 解码操作，而是直接返回对应的 `config` 实例：

```go
config := LoadConfig()
```

然后将 Web 服务器的启动参数和静态资源目录都调整为通过配置值获取，这样我们后续只需要更改配置文件即可对其进行调整，而不需要修改任何代码，降低了代码维护成本。

### 3.2 数据库连接配置

接下来，打开 `models/db.go`，将数据库连接信息调整为通过配置文件读取：

```go
package models

import (
    "crypto/rand"
    "crypto/sha1"
    "database/sql"
    "fmt"
    _ "github.com/go-sql-driver/mysql"
    . "github.com/xueyuanjun/chitchat/config"
    "log"
)

var Db *sql.DB

func init() {
    var err error
    config := LoadConfig() // 加载全局配置实例
    driver := config.Db.Driver
    source := fmt.Sprintf("%s:%s@(%s)/%s?charset=utf8&parseTime=true", config.Db.User, config.Db.Password,
        config.Db.Address, config.Db.Database)
    Db, err = sql.Open(driver, source)
    if err != nil {
        log.Fatal(err)
    }
    return
}
```

虽然，在这里页调用了 `LoadConfig()`，但是由于是单例模式，所以会直接返回 `config` 实例，不会再进行初始化操作，然后我们获取配置值填充对应的 `sql.Open` 连接配置。

## 4. 整体测试

至此，我们已经完成了通过配置文件读取应用配置的代码重构，我们可以为项目编写[单元测试](https://xueyuanjun.com/post/21494)，也可以直接通过在浏览器访问这个在线论坛项目验证重构后应用是否可以正常运行，重新启动 Web 服务器，输出如下：

```bash
$ go run main.go
2020/06/07 16:54:55 Starting HTTP server at 0.0.0.0:8080
```

表示启动服务器时读取配置信息正常，然后访问应用首页：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15863962655932.jpg)

成功，对用户来说，没有任何感知后台的变动。