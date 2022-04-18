---
title: Docker-使用 Docker 安装运行 mysql
date: 2020-04-23
tags: [linux]
categories: [爱编程爱技术的孩子]
slug: Using docker installation to run MySQL 
---

Docker Hub 中的 mysql 镜像 地址为 https://hub.docker.com/_/mysql，安装运行过程如下。

## 1. 拉取镜像

```bash
$ docker pull mysql
# 查看
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest              0c27e8e5fcfa        6 hours ago         546MB
hello-world         latest              bf756fb1ae65        3 months ago        13.3kB
```

## 2. 运行 mysql 容器

```bash
$ docker run -p 3306:3306 --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

上面各命令的含义为

```
run                 运行一个docker容器
--name           	后面这个是生成的容器的名字some-mysql
-p 3306:3306  		表示这个容器中使用3306（第二个）映射到本机的端口号也为3306（第一个） 
-e MYSQL_ROOT_PASSWORD=123456  初始化root用户的密码
-d                   表示使用守护进程运行，即服务挂在后台
```

`tag` 是 MySQL 版本，比如可以填写 `5.7`，如果没有设置版本，Dcoker 会自动在本地检测有没有最新的，如果没有会自动去 Docker Hub 下载。该字段的选项如下

- 8.0.19，8.0，8，latest
- 5.7.29，5.7，5
- 5.6.47，5.6

```bash
# 查看当前容器运行状态
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
94e4725784bb        mysql:8.0           "docker-entrypoint.s…"   5 seconds ago       Up 4 seconds        3306/tcp, 33060/tcp   chitchat
```

如果暂时关闭了容器界面，隔了一天后回来，使用`docker ps`命令可能看不到任何东西，这时候添加`-a`参数



## 3. 进入 mysql 容器

执行 `docker exec` 命令进入 mysql 容器，该命令会在容器中启动一个命令行

```bash
$ docker exec -it chitchat bash
```

日志文件可通过如下命令查看

```bash
$ docker logs chitchat
```

之后操作为正常的 mysql 命令操作，如

```bash
root@94e4725784bb:/# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## 4. 测试连接

第一种是使用 Navicat for MySQL 测试，Navicat 安装在宿主机的 win10系统中

![测试连接](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200423_%E6%B5%8B%E8%AF%95%E8%BF%9E%E6%8E%A5.png)

点击「测试连接」查看连接是否成功，成功后点确定可以进入以图形化方式查看数据库

因为我们使用 Golang，这里写一段代码测试Golang连接是否成功

```go
package tes

import (
	"database/sql"
	"fmt"

	_ "github.com/go-sql-driver/mysql"
)

func query() {
	//"用户名:密码@[连接方式](主机名:端口号)/数据库名"
	db, _ := sql.Open("mysql", "root:****@(127.0.0.1:3306)/chitchat") // 设置连接数据库的参数
	defer db.Close()                                                  //关闭数据库
	err := db.Ping()                                                  //连接数据库
	if err == nil {
		fmt.Println("数据库连接成功")
		return
	}
}

```

如果连接成功，会输出「数据库连接成功」字样，此时同时也说明了我们在 Docker 中部署 MySQL，然后从宿主机连接的方式是可行的，宿主机中不需要再安装 MySQL client 等。

## 5. 退出与重连

输入`exit`退出 docker 内的命令行，再次输入 `exit` 退出 docker，此时退出容器但没有关闭

```bash
$ docker exec -it chitchat bash                                                                                         root@cd6182d13e68:/# exit
exit

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
cd6182d13e68        mysql               "docker-entrypoint.s…"   11 hours ago        Up 8 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp   chitchat
```

进入已启动的容器可以使用

```bash
$ docker attach chitchat
```

不过重启电脑后运行 `docker ps`发现没了，此时执行 `docker start`命令可以再次启动

```bash
$ docker start chitchat
```

本文参考：[使用Docker安装、运行mysql - 简书 (jianshu.com)](https://www.jianshu.com/p/d9b6bbc7fd77)

不使用docker直接在win10中安装mysql参考[win10安装Mysql教程](https://www.cnblogs.com/xiaokang01/p/12092160.html)

