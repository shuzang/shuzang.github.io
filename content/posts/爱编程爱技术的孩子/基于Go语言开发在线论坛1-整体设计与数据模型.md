---
title: 基于Go语言开发在线论坛1-整体设计与数据模型
author: xueyuanjun
date: 2020-05-27T18:12:20+08:00 
lastmod: 2020-05-27
tags: [Go Web]
categories: [爱编程爱技术的孩子]
slug: Development of online forum based on golang 1-overall design and data model
typora-root-url: ..\..\..\static
---

学院君提供的教程，这里进行复现并深入理解，文章内容可能略有改动，会添加一些自己的理解。[原文地址](https://xueyuanjun.com/post/21519)

<!-- more -->

## 1. 项目介绍

要完成的是一个简单的在线论坛项目，主要仿照 [Google 网上论坛](https://groups.google.com/)进行开发。需要具备用户认证功能（注册、登录、退出等），认证后的用户才能创建新的群组、以及在群组中发表主题，访客用户访问论坛首页可以查看群组列表，进入指定群组页面可以查看对应的主题信息。

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-03/image-15850625575141.jpg)

## 2. 技术方案

采用典型的 MVC 架构，以群组详情页为例，假设对应的 URL 是 `http://chitchat.test/thread/read?id=123`，其中 `chitchat.test` 是请求域名，`thread/read` 是请求路由（查看群组），`?id=123` 是请求参数（群组ID），通过域名确定应用所在的服务器 IP，通过端口号（此处没有显式展示，一般默认是 80 端口）确定应用进程，应用接收到请求后，通过路由将请求分发到指定处理器方法（之前介绍的路由器，或者叫做多路复用器做的就是这个工作，路由器是整个应用请求分发的入口），通过请求参数对数据库进行查询，再将视图响应发送给请求用户，如果数据库查询没有结果，则返回 404 响应。这里，数据库承担的是 M 的角色（Model），视图响应承担的是 V 的角色（View），处理器方法承担的是 C 的角色（Controller）：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-03/image-15850624895084.jpg)

> 注：上图中 Client 代表客户端发起请求的用户，虚框内是部署在服务器已启动的在线论坛应用，Multiplexer 代表路由器（比如 `gorilla/mux` ），Handler 代码处理器/处理器方法，数据库操作位于处理器方法中，Templates 代表最终展示给用户的经过模板引擎编译过的视图模板。

其他页面和操作的请求/响应模型与此一致，不再重复介绍。

所以我们需要在本地按照这个 MVC 架构基于业务流程编写代码，最后将测试过的应用代码编译打包，部署到远程服务器（这样才能被普通用户访问），并启动该应用，等待客户端请求，这样就完成了整个应用开发流程。

## 3. 数据模型

根据我们之前拟定的需求，至少需要三个模型：

- 用户（User）
- 群组（Thread）
- 主题（Post）

另外，我们在本项目开发时，会把用户会话（Session）也存储到数据库，所以需要一个额外的会话模型，此外，为了简化应用，我们不会真的像 Google 网上论坛那样对用户做权限管理，整个应用只包含一种用户类型，并且具备所有操作权限：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-03/image-DraggedImage-1.png)

下一篇介绍数据表和模型类的创建，以及编写相应的数据库交互实现。