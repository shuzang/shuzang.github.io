---
title: Docker-Win10 Home安装Docker
date: 2020-04-23
tags: [linux]
categories: [爱编程爱技术的孩子]
slug: Install Docker Desktop on Windows 10 Home
---

Docker Desktop 是 Windows 安装 Docker 的推荐安装方式，然而系统需求是 Pro, Enterprise 或 Education 版本，Home 版不支持，因为没有 Hyper-V。

这篇文章记录网上找到的 Win10 Home 版安装 Docker Desktop 的方式，主要思路是自行安装 Hyper-V 及 相关服务，并修改注册表欺骗 Docker 的安装检测。

## 1. 安装 Hyper-V

新建 txt 文档，复制以下内容到文档中。

```txt
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
```

将文档格式后缀由 `.txt` 更改为 `.cmd`，然后以管理员方式执行。该脚本会自动安装 Hyper-V 服务，安装完成后按提示键入 `Y`，电脑自动重启。注意，这里键入 `Y` 后直接就重启了，所以有其它未保存任务及时保存。

重启完成后进入搜索「启用或关闭 Windows 功能」并打开，查看 「Hyper-V」选项是否被选中，如果没有，勾选并重启电脑。

再次新建 txt 文档并填充如下内容

```txt
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*containers*.mum >containers.txt
for /f %%i in ('findstr /i . containers.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del containers.txt
Dism /online /enable-feature /featurename:Containers -All /LimitAccess /ALL
pause
```

更改后缀为 `.cmd` 并以管理员方式执行，安装完毕后键入 `Y` 重启电脑。至此第一步完成

## 2. 伪装成专业版绕过安装检测

Docker Desktop 安装时会检测系统版本，因此我们修改注册表

```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion

EditionID = Professional
```

然后遵照[官方文档](https://docs.docker.com/docker-for-windows/install/)下载安装 Docker即可。

注：重启后修改的注册表项会自动还原，但不影响 Docker 运行。

## 3. 测试

查看 Docker 版本

```bash
$ docker --version
Docker version 19.03.8, build afacb8b
```

运行 `hello-world` 镜像测试安装是否成功

```bash
$ docker run hello-world                                                                                                
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

出现 `Hello from Docker!` 说明安装成功。

参考：[Orientation and setup | Docker Documentation](https://docs.docker.com/get-started/)

