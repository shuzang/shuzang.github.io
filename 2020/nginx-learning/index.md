# Nginx说明


本文学习 Nginx 的相关知识。

<!--more-->

## 1. 相关概念

- [**代理服务器**]([https://baike.baidu.com/item/%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8](https://baike.baidu.com/item/代理服务器))：用户浏览器与 Web 服务器之间的服务器，用于代替浏览器向目标服务器发出请求，主要作用是缓存服务器内容，但也可以用作流量控制、安全管理等。基本的工作过程是，用户在本地设定代理服务器地址后，所有的访问请求都会转交到代理服务器，如果代理服务器本地有这些内容，就会直接返回，如果没有，就向真正的目标服务器发出请求，获取到内容后再向用户返回。众所周知的 VPN 其实也是代理服务器的应用，我们使用服务商提供的或自己购买的服务器来访问其它网络，而不是使用自己的本地机器。
- [**反向代理服务器**]([https://baike.baidu.com/item/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86](https://baike.baidu.com/item/反向代理))：反向代理服务器和代理服务器所处的位置相同，都是用户浏览器和目标服务器中间，但它不是为用户服务的，而是作为目标服务器的辅助。其存在价值在于，将真正的服务器放在一个内网中，由防火墙保护，所有访问真正服务器的请求，都会交给反向代理服务器来处理，因为数据本身还是存放在真正的服务器上，对代理服务器的攻击并不会产生影响。实际上，对用户来说，反向代理服务器外在表现就是一台真正的 Web 服务器，和直接访问目标服务器没有太大区别。最后，当请求量比较大时，还可以设置多台反向代理服务器，来分担目标服务器的压力。
- [**Nginx**](https://baike.baidu.com/item/nginx)：反向代理服务器并不是说服务器本身有区别，而是其承担的功能有区别，这些功能由某些特定的软件来完成，Nginx 就是一种这样的软件，它可以作为 HTTP 服务器、反向代理服务器以及邮件代理服务器（IMAP/POP3/SMTP）。Nginx 由俄罗斯程序员 Igor Sysoev 开发，最初是作为邮件代理服务器使用的。Nginx 最大特点就是资源消耗低、稳定、支持高并发，因此得到了广泛的使用。

**Apache** 也是我们常提的 Web 服务器，它和 Nginx 的主要区别是：

1. Apache 一个连接对应一个进程，而 Nginx 可以多个连接（万级别）对应一个进程，因此 Nginx 更轻量，对并发的支持更好；
2. Apache 在处理动态请求方面更有优势，而 Nginx 处理静态请求性能更高；这里静态请求指的是返回的页面是固定的，不管谁访问都一样，动态请求指的是需要服务端根据请求做处理，填充完页面后再返回，返回的内容是根据用户或请求的不同而不同的；
3. Nginx 使用 C 编写，同样的 Web 服务，占用更少的内存和资源；
4. Nginx 配置简介，启动方便，运行稳定，几乎可以做到 7*24 不间断运行，连续运行数月也不需要重新启动。

总之，一般来说，需要性能的 web 服务，用 Nginx 。如果不需要性能只求稳定，更考虑 Apache 。更为通用的方案是，前端 Nginx 抗并发，后端 Apache 集群，配合起来会更好。

Tomcat 是另一个常听的名词，这是一个 Web 应用服务器，本质和 Apache 以及 Nginx 是不冲突的，可以这样理解，服务端的应用在 Tomcat 中保持运行，为 Apache 或 Nginx 提供服务，而 Apache 或 Nginx 代理用户的请求，它们本身并不对请求做处理，只是转发。Tomcat 一般只用作 Java 的应用服务器，当然，某种程度上也可以代理 Apache 或 Nginx 的功能，但显然性能不如这两者。

## 2. 安装

官方下载地址为 http://nginx.org/en/download.html，共提供了三个版本

- Mainline version：可以理解为开发版
- Stable version：稳定版
- Legacy version：曾发布的老版本的稳定版

Nginx 提供 windows 版本，下载解压后是 exe 格式，可以双击安装，不过，考虑到它本身是用作服务器的，所以只介绍 Linux 的安装使用。

### 2.1 源码安装

Nginx 可以从源码编译安装，教程参考 [Building nginx from sources](http://nginx.org/en/docs/configure.html)



### 2.2 二进制包安装

这里只介绍 Ubuntu，其它 Linux 系统的下载安装参考 [nginx:Linux packages](http://nginx.org/en/linux_packages.html)，

1. 安装依赖

   ```bash
   $ sudo apt install curl gnupg2 ca-certificates lsb-release
   ```

2. 为安装包建立 apt 仓库

   ```bash
   $ echo "deb http://nginx.org/packages/ubuntu 'lsb_release -cs' nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
   ```

   | 是管道符，将前面命令的输出作为后面命令的输入，tee 将内容同时输出的文件和当前终端，如果文件不存在，则创建，如果已存在，则覆盖。

3. 导入官方的验证签名，从而使系统可以自动的验证包的正确性

   ```bash
   $ curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
   ```

4. 验证

   ```bash
   $ sudo apt-key fingerprint ABF5BD827BD9BF62
   pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
         573B FD6B 3D8F BC64 1079  A6AB ABF5 BD82 7BD9 BF62
   uid   [ unknown] nginx signing key <signing-key@nginx.com>
   ```

5. 安装 Nginx

   ```bash
   $ sudo apt update
   $ sudo apt install nginx
   ```

6. 验证

   ```bash
   $ nginx -v
   nginx version: nginx/1.18.0
   $ lsb_release -a
   No LSB modules are available.
   Distributor ID:	Ubuntu
   Description:	Ubuntu Focal Fossa (development branch)
   Release:	20.04
   Codename:	focal
   ```

## 3. 使用

本节描述如何启动和停止 nginx 服务，重新加载配置，解释配置文件的结构，描述如何设置 nginx 使其服务于静态内容，以及如何配置其作为一个代理服务器。

nginx 有一个 master 进程和多个 worker 进程，master 进程的主要作用是读取和评估配置，以及维护 worker 进程。worker 进程对请求进行实际的处理。nginx使用基于事件的模型和依赖于操作系统的机制来有效地在 worker 进程之间分配请求。worker 进程的数量在配置文件中定义，可以固定，也可以自动调整为可用CPU内核的数量。

nginx 及其模块的工作方式在配置文件中定义。默认情况下，配置文件名为 nginx.conf，并放置在目录 `/usr/local/nginx/conf`，`/etc/nginx`或`/usr/local/etc/nginx `中。

### 3.1 启动和停止

直接执行 `nginx` 命令即可启动，此时由 root 用户启动了 master 进程，然后自行创建 nginx 用户创建 worker 进程

```bash
$ sudo nginx
```

启动后就可以使用 -s 参数发送信号到 master 进程进行控制

```bash
$ nginx -s signal
```

其中，signal 为参数，可选的有四种

- stop：快速停止
- quit：正常停止
- reload：重新加载配置文件
- reopen：重启打开日志文件

在重新加载配置文件或重新启动前，配置文件不会被应用。而一旦 master 进程接收到 reload 信号，就会检查新配置文件的语法有效性，并尝试应用其中提供的配置。如果成功，则主进程将启动新的工作进程并将消息发送到旧的工作进程，以要求它们关闭。否则，主进程将回滚更改并继续使用旧配置。旧的工作进程接收到关闭命令，停止接受新的连接并继续为当前请求提供服务，直到为所有此类请求提供服务。之后，旧的工作进程退出。

也可以借助 Unix工具（例如kill工具）将信号发送到 nginx 进程。在这种情况下，将信号直接发送给具有给定进程ID 的进程。默认情况下，nginx 主进程的进程ID写入目录 `/usr/local/nginx/logs` 或 `/var/run` 中的nginx.pid。例如，如果主进程ID为1628，要发送 QUIT 信号导致 nginx 正常关闭，请执行：

```bash
$ kill -s QUIT 1628
```

为了获取所有正在运行的nginx进程的列表，可以使用ps实用程序，例如，通过以下方式：

```bash
$ ps -ax | grep nginx
```

### 3.2 配置文件结构

nginx 由模块组成，这些模块受配置文件中指定的指令控制。指令分为简单指令和块指令。一个简单指令由名称和参数组成，名称和参数之间用空格分隔，并以分号（;）结尾。块指令同样由名称和参数组成，但它的参数就是一批简单指令，并用大括号包围。块指令内的内容称为上下文（比如事件，http，服务器和位置）。初始的配置文件内容如下，我们可以看到上述描述的基本结构

```conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

上下文外的指令（及块指令外的简单指令，比如上面第一行的 user nginx;）被视为位于主上下文中。事件 和 http指令位于主上下文中，服务器位于 http 中，位置位于服务器中。

`#` 后的内容视为注释

### 3.3 服务于静态内容

Web服务器的一项重要任务是分发文件（例如图像或静态HTML页面）。这里将实现一个示例，其中根据请求从不同的本地目录提供文件：/data/www（可能包含HTML文件）和 /data/images（包含图像）。这一任务需要编辑配置文件，在服务器块指令下的 http 块中使用两个 位置块。

1. 创建 /data/www 目录，将 index.html 文件放入其中，

2. 创建 /data/images 目录，放一些图片进去。

3. 打开配置文件，在 http 块中增加 server 块，结构如下

   ```conf
   http {
   	server {
   	}
   }
   ```

   通常，配置文件会包含多个服务器块，这些服务器块以服务器名和它们监听的端口号来区分。服务器处理的请求在下面的 location 子块中定义，如下

   ```conf
   location / {
       root /data/www;
   }
   ```

   接收到的请求会与 location 后的 / 进行匹配，如果匹配，则到块内部的位置寻找文件，继续举例，下面的例子中与 /images/ 路径匹配，然后到 root /data 中寻找文件

   ```conf
   location /images/ {
       root /data;
   }
   ```

   还会发现的是，如果 /images/ 请求传进来，实际上上面两个块都会匹配成功，在 nginx 中，如果有多个匹配成功的位置块，将会选择前缀最长的那个，也就是 /images/

   一个完整的结构如下

   ```conf
   server {
       location / {
           root /data/www;
       }
   
       location /images/ {
           root /data;
       }
   }
   ```

到现在为止，这就是一个可用的服务器配置了，它会监听标准的 80 端口，并且在本地计算机的 http:/localhost/ 上相应服务。以 /images/ 开头的 URL 请求，服务器将从 /data/images 目录查找文件并返回，例如，收到的请求为 http://localhost/images/example.png ，nginx 将返回 /data/images/example.png 文件，如果文件不存在，将返回 404 错误。

要应用上述配置文件，执行重新加载配置文件命令

```bash
$ nginx -s reload
```

如果不起作用，可以从 /usr/local/nginx/logs 或 /var/log/nginx 目录下的 access.log 和 error.log 文件中查找原因。

### 3.4 设置简单的代理服务器

nginx的一种常用用法是将其设置为代理服务器，这意味着服务器可以接收请求，将请求传递给代理服务器，从请求中检索响应并将它们发送给客户端。

我们将配置一个基本的代理服务器，该服务器为图像请求和本地目录中的文件提供服务，并将所有其他请求发送到代理服务器。在此示例中，两个服务器都将在单个nginx实例上定义。

首先，通过向nginx的配置文件中添加一个包含以下内容的服务器块来定义代理服务器：

```conf
server {
    listen 8080;
    root /data/up1;

    location / {
    }
}
```

这将是一个简单的服务器，它在端口8080上侦听（以前，由于使用了标准端口80，所以未指定listen指令），并将所有请求映射到本地文件系统上的/ data / up1目录。创建此目录，并将index.html文件放入其中。请注意，根指令位于服务器上下文中。当选择用于服务请求的位置块不包含自己的根指令时，将使用这种根指令。

接下来，使用上一部分中的服务器配置并对其进行修改以使其成为代理服务器配置。在第一个位置块中，将proxy_pass指令与参数中指定的代理服务器的协议，名称和端口放在一起（在本例中为http：// localhost：8080）：

```conf
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location /images/ {
        root /data;
    }
}
```

我们将修改第二个位置块，该位置块当前将带有/ images /前缀的请求映射到/ data / images目录下的文件，以使其与具有典型文件扩展名的图像的请求匹配。修改后的位置块如下所示：

```conf
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```

该参数是一个正则表达式，它匹配所有以.gif，.jpg或.png结尾的URI。正则表达式应以〜开头。相应的请求将映射到/ data / images目录。

当nginx选择一个位置块来服务请求时，它首先检查指定前缀的位置指令，记住最长的前缀位置，然后检查正则表达式。如果与正则表达式匹配，则nginx会选择此位置，否则，它将选择之前记住的位置。

代理服务器的最终配置如下所示：

```conf
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

该服务器将过滤以.gif，.jpg或.png结尾的请求，并将它们映射到/ data / images目录（通过将URI添加到root指令的参数中），并将所有其他请求传递给上述配置的代理服务器。

要应用新配置，请按照前面几节的说明将重新加载配置文件。还有许多[其他指令](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)可用于进一步配置代理连接。

更多的内容可以查看 [nginx documentation](http://nginx.org/en/docs/)，本文主体都翻译自该文档。
