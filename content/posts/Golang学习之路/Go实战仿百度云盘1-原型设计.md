---
title: Go实战仿百度云盘1-原型设计
date: 2020-08-10T19:28:00+08:00 
lastmod: 2020-08-10
tags: [仿百度云盘]
categories: [Golang学习之路]
slug: Go implement distributed cloud storage system 1-Prototype design
typora-root-url: ..\..\..\static
---

慕课网买了一个 Golang 的课程，实现一个分布式云存储系统，涉及到了大部分的知识点，开一个系列做一下学习记录。本文是第一篇，原型设计。

<!--more-->

## 1. 原型说明

整个项目是一个递进式的实现过程，一开始，先实现一个简单的原型系统，架构如下

![服务架构说明](/images/Go实战仿百度云盘1-原型设计/服务架构说明.png)

根据该架构图，先设计接口，列表如下

| 接口描述               | 接口URL            |
| ---------------------- | ------------------ |
| 文件上传接口           | POST /file/upload  |
| 文件查询接口           | GET /file/query    |
| 文件下载接口           | GET /file/download |
| 文件删除接口           | POST /file/delete  |
| 文件修改（重命名）接口 | POST /file/update  |

接口设计对于前后端的协同比较重要，因此对于接口通常也有一定的规范，最常用的是 RESTful 架构，我主要看了阮一峰大神的两篇文章。文章中介绍 URL 的作用是标识实体，不应该含有动词，增删查改的行为应该通过 GET、POST 这些方法来表达，上面的接口设计中，upload、query、download、update都有名词的词性，因此没有问题，但删除应当使用 deletion，另外，删除接口没有使用 DELETE 而是使用了 POST 方法，修改没有使用 PUT 也使用了 POST。

我们暂时还是按照课程中定义的这些来，但是要明白确实有不合理之处。

[1] 阮一峰，[理解 RESTful 架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)，2011.09.12

[2] 阮一峰，[RESTful 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)，2014.05.22

## 2. 环境准备

虚拟机安装 Ubuntu20.04，所有的代码在 Ubuntu20.04 中编辑运行。



