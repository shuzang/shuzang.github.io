---
title: 基于Go语言开发在线论坛9-部署Go Web应用
author: xueyuanjun
date: 2020-06-07T20:50:00+08:00 
lastmod: 2020-06-07
tags: [Go实战]
categories: [Golang学习之路]
slug: Development of online forum based on golang 9-deploy go web application
---

部署 Go 应用相对简单，因为所有应用代码都被打包成一个二进制文件了（视图模板、静态资源和配置文件等非 Go 代码除外），并且不需要依赖其他库，不需要额外的运行时环境（比如 Java 需要再安装 JVM），也不需要部署额外的 HTTP 服务器。

<!--more-->

对于在线论坛项目，包含了静态资源文件（CSS、JavaScript、图片），所以我们将在 Go Web 应用之前前置一个 Nginx 服务器处理静态资源请求，然后通过反向代理处理动态资源请求（指向 Go 处理器方法的请求），对于那些不包含静态资源和视图模板的纯 API 项目，通常只需要打包一份二进制文件部署到服务器即可，更加便捷。

> 注：其实 Go 应用部署的最佳实践是基于 Docker，后续我们在部署专题中再介绍如何基于 Docker 将应用快速部署到远程云服务器。

## 1. 构建应用

首先，我们可以在本地项目根目录下通过如下命令将应用代码打包成二进制可执行文件：

```go
GOOS=linux GOARCH=amd64 go build
```

注意这里指定了 `GOOS` 和 `GOARCH` 选项进行交叉编译，因为我们是在 Win10 系统（`amd64`）中打包，并且目标二进制文件需要在 Linux 服务器（`linux`）执行。该命令执行成功后会在当前目录下生成和项目名称相同的二进制文件：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15871088999900.jpg)

然后我们可以将代码提交到 Github 或者其他代码仓库。

## 2. 部署应用

### 2.1 部署代码

再登录服务器到部署目录下拉取代码：

```bash
git clone https://github.com/nonfu/chitchat
```

初次拉取使用 `git clone`，后续在 `chitchat` 目录下运行 `git pull` 即可。

然后我们进入 `chitchat` 目录，配置 `config.json` 进行服务端数据库配置（正式项目不要将 `config.json` 提交到代码仓库，以免安全风险和后续拉取代码覆盖），确保 `logs` 目录对 Web 用户具有写权限（比如配置权限为 `777`，或者所属用户与 Web 用户组一致）。

> 注：当然我们这里部署代码的方式比较原始，对于多人协作的大型项目，可以借助持续集成工具（比如 Jenkins）进行自动化部署，并且由于项目比较简单，就不再演示单元测试、CI/CD 等其他 DevOps 工具的使用了。

### 2.2 数据库初始化

在服务端 MySqL 数据库中创建 `chitchat` 数据库，并初始化对应数据表。如果不了解如何安装和创建数据库，可以参考这篇教程：[将博客应用自动部署到线上服务器完整流程详解](https://xueyuanjun.com/post/9749.html#bkmrk-%E5%88%9B%E5%BB%BA%E7%BA%BF%E4%B8%8A%E6%95%B0%E6%8D%AE%E5%BA%93)。

### 2.3 访问应用

完成以上工作后，我们就可以在 `chitchat` 项目目录下运行 `chitchat` 二进制文件启动应用了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15871116372180.jpg)

然后我们在本地 `hosts` 文件中自定义一个测试域名与服务器 IP 的映射：

```
your-server-ip-address chitchat.test
```

将上述 `your-server-ip-address` 替换成自己的远程服务器 IP 地址，然后我们就可以在浏览器中通过 `http://chitchat.test:8080` 访问应用了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15871123187423.jpg)

## 3. 通过 Nginx 做反向代理

虽然上述方式可以正常运行，但是如果要高效处理静态资源文件并对其做缓存，可以借助 Nginx 作为反向代理服务器来完成，我们在 Nginx 虚拟主机配置目录 `/etc/nginx/sites-available` 中新增一个配置文件 `chitchat.conf`（以 Ubuntu 服务器为例）：

```nginx
server {
    listen      80; 
    server_name chitchat.test www.chitchat.test;
    
    # 静态资源交由 Nginx 管理，并缓存一天
    location /static {
        root        /var/www/chitchat/public;
        expires     1d;
        add_header  Cache-Control public;
        access_log  off;
        try_files $uri @goweb;
    }
    
    location / {
        try_files /_not_exists_ @goweb;
    }
    
    # 动态请求默认通过 Go Web 服务器处理
    location @goweb {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Scheme $scheme;
        proxy_redirect off;
        proxy_pass http://127.0.0.1:8080;
    }
    
    error_log /var/log/nginx/chitchat_error.log;
    access_log /var/log/nginx/chitchat_access.log;
}
```

然后再启用该配置文件：

```bash
ln -s /etc/nginx/sites-available/chitchat /etc/nginx/sites-enabled/chitchat
```

重启 Nginx 服务：

```bash
service nginx restart
```

与此同时，我们可以把 `chitchat/config.json` 中的 `App` 配置项启动 IP 地址改为 `127.0.0.1`：

```bash
"App": {
    "Address": "127.0.0.1:8080",
    "Static": "public",
    "Log": "logs",
    "Locale": "locales",
    "Language": "zh"
  },
```

并再次重启这个 Go 应用，这样就只能通过 Nginx 访问应用，在浏览器中访问 `http://chitchat.test`：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15871168431728.jpg)而当你试图再通过 `http://chitchat.test:8080` 访问应用，则会报错：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15871169152394.jpg)

我们可以测试下注册登录功能以及创建新群组功能：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15871169671758.jpg)

## 4. 通过 Supervisor 维护应用守护进程

看起来一切都 OK 了，但是目前这种模式下，用户退出后 Go Web 应用进程会关闭，这显然是不行的，而且如果 Go Web 应用进程因为其他异常挂掉，也无法自动重启，每次需要我们登录到服务器进行启动操作，这很不方便，也影响在线应用的稳定性，为此，我们需要借助第三方进程监控工具帮我们实现 Go Web 应用进程以后台守护进程的方式运行。常见的进程监控工具有 Supervisor、Upstart、systemd 等，由于我的服务器之前部署过 Supervisor，所以我就借助它来管理 Go Web 应用进程。

> 注：对 Supervisor 安装配置不了解的同学，可以参考这篇教程 —— [队列系统解决方案：Horizon](https://xueyuanjun.com/post/21566#bkmrk-%E9%83%A8%E7%BD%B2-horizon)。

首先创建对应的 Supervisor 配置文件 `/etc/supervisor/conf.d/chitchat.conf`，这里需要设置进程启动目录及命令、进程意外挂掉后是否自动重启、以及日志文件路径等：

```
[program:chitchat]
process_name=%(program_name)s
directory=/var/www/chitchat
command=/var/www/chitchat/chitchat
autostart=true
autorestart=true
user=root
redirect_stderr=true
stdout_logfile=/var/www/chitchat/logs/chitchat.log
```

> 注意：我们需要进入 `chitchat` 所在目录执行启动命令，否则会找不到配置文件和其他资源路径，所以需要配置 `directory` 选项。

然后关闭之前通过手动运行 `chitchat` 启动的 Go Web 服务器，再运行如下指令通过 Supervisor 启动并维护 Go Web 应用进程：

```
supervisorctl reread
supervisorctl update
supervisorctl start chitchat
```

你可以通过 `ps -ef | grep chitchat` 查看进程是否启动成功：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15871330990937.jpg)

启动成功后，就可以在浏览器通过 `http://chitchat.test` 访问部署在远程服务器的在线论坛了：

![](https://qcdn.xueyuanjun.com/storage/uploads/images/gallery/2020-04/image-15871331767149.jpg)

并且无论是否退出远程服务器还是关闭 Go Web 应用进程，都不会影响在线论坛的访问，因为它是以守护进程的方式运行的，并且可以在关闭后自动重启。
