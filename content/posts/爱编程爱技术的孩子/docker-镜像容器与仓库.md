---
title: Docker-镜像、容器与仓库
date: 2020-05-18T09:00:00+08:00
tags: [linux]
categories: [爱编程爱技术的孩子]
slug: Image Container and Repository 
---

Docker 的三个基本概念是镜像（Image）、容器（Container）和仓库（Repository），理解了这三个概念基本就理解了 Docker。

<!--more-->

## 1. 镜像

尽管都使用了镜像这个词，但 Docker 的镜像和虚拟机上安装的系统镜像并不完全相同。虚拟机安装的镜像是完整的操作系统，包括内核、根文件系统等部分，体积一般较大，比如 ubuntu20.04 LTS 就有2.5GB。而 Docker 的镜像不包括内核，可以看作一个定制的**最小化**的根文件系统，体积往往很小，本文写作时 docker ubuntu latest 仅有73.9MB。实际上，Docker 本身是基于宿主机的内核运行的。

### 1.1 Union FS

Docker 镜像保持的如此之小，仅依靠去除内核和删减根文件系统是不够的，更重要的原因是，Docker 使用了 Union FS 的技术，整体设计是一个分层存储的架构。联合文件系统（UnionFS）是一种轻量级的高性能分层文件系统，它支持将文件系统中的修改信息作为一次提交，并层层叠加，同时可以将不同目录挂载到同一个虚拟文件系统下，应用看到的是挂载的最终结果。简单来说，镜像不是一整个文件，而是由一组文件系统组成，更具体一点，由多层文件系统组成。镜像构建时，会一层层构建，前一层是后一层的基础，每一层构建完就不会再发生改变，后一层的任何改变只会发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。以 ubuntu 为例，下载镜像时我们可以很清楚的看到分了四层

```bash
$ docker pull ubuntu  
Using default tag: latest
latest: Pulling from library/ubuntu
d51af753c3d3: Pull complete  
fc878cd0a91c: Pull complete   
6154df8ff988: Pull complete   
fee5db0ff82f: Pull complete    
Digest: sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

使用 `docker history` 命令还可以查看镜像构建历史记录

```bash
$ docker history ubuntu
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
1d622ef86b13        3 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>           3 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>           3 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   811B
<missing>           3 weeks ago         /bin/sh -c [ -z "$(apt-get indextargets)" ]     1.01MB
<missing>           3 weeks ago         /bin/sh -c #(nop) ADD file:a58c8b447951f9e30…   72.8MB
```

分层存储的特征使得镜像的复用、定制变得更加容易，甚至可以用之前构建好的镜像作为基础层，然后一步步添加新层，以定制自己需要的内容，构建新的镜像。

### 1.2 镜像使用

Docker 运行容器前需要本地存在对应的镜像，如果本地不存在该镜像，Docker 会从镜像仓库下载该镜像。

#### 获取镜像

从 Docker 镜像仓库获取镜像的命令是 `docker pull`。其命令格式为：

```bash
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

选项部分包括三个参数，但很少用到，可以通过 `docker pull --help` 查看，这里介绍镜像名称的格式

- Docker 镜像仓库地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub（官方仓库地址）。

- 仓库名：仓库名是两段式名称，即 `<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。

仍以 ubuntu 为例

```bash
$ docker pull ubuntu  
Using default tag: latest
latest: Pulling from library/ubuntu
d51af753c3d3: Pull complete  
fc878cd0a91c: Pull complete   
6154df8ff988: Pull complete   
fee5db0ff82f: Pull complete    
Digest: sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub 获取镜像。镜像名称为 `ubuntu`，因此将会获取官方镜像 `library/ubuntu` 仓库中标签为 `latest` 的镜像。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 `sha256` 的摘要，以确保下载一致性。

*如果从 Docker Hub 下载镜像非常缓慢，可以参照* [*镜像加速器*](https://yeasy.gitbook.io/docker_practice/install/mirror) *配置加速器。*

#### 列出镜像

要想列出已经下载下来的镜像，可以使用 `docker image ls` 命令。

```bash
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              1d622ef86b13        3 weeks ago         73.9MB
mysql               8.0                 0c27e8e5fcfa        3 weeks ago         546MB
mysql               latest              0c27e8e5fcfa        3 weeks ago         546MB
hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
```

列表包含了 `仓库名`、`标签`、`镜像 ID`、`创建时间` 以及 `所占用的空间`。j镜像ID 是镜像的唯一标识，一个镜像可能对应多个标签，比如上面例子中的 `mysql:8.0` 和 `mysql:latest`。

不加任何参数的情况下，`docker image ls` 会列出所有顶层镜像，但是有时候我们只希望列出部分镜像。`docker image ls` 有好几个参数可以帮助做到这个事情。

```bash
$ docker image ls ubuntu #根据仓库名列出镜像
$ docker image ls ubuntu:18.04 #列出指定仓库名和标签的镜像
$ docker image ls -f since=mongo:3.2 #使用过滤器列出指定条件的镜像
```

#### 删除本地镜像

如果要删除本地的镜像，可以使用 `docker image rm` 命令，其格式为：

```bash
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```

其中，`<镜像>` 可以是 `镜像短 ID`、`镜像长 ID`、`镜像名` 或者 `镜像摘要`。

```bash
$ docker image rm mysql:8.0
Untagged: mysql:8.0
$ docker image rm ubuntu
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Deleted: sha256:1d622ef86b138c7e96d4f797bf5e4baca3249f030c575b9337638594f2b63f01
Deleted: sha256:279e836b58d9996b5715e82a97b024563f2b175e86a53176846684f0717661c3
Deleted: sha256:39865913f677c50ea236b68d81560d8fefe491661ce6e668fd331b4b680b1d47
Deleted: sha256:cac81188485e011e56459f1d9fc9936625a1b62cacdb4fcd3526e5f32e280387
Deleted: sha256:7789f1a3d4e9258fbe5469a8d657deb6aba168d86967063e9b80ac3e1154333f
```

可以注意到删除行为分两类，一类是 `Untagged`，另一类是 `Deleted`。前面介绍过，镜像的唯一标识是 ID 和摘要，而一个镜像可以有多个标签，因此当我们使用上面命令删除镜像的时候，实际上是在要求删除某个标签的镜像。所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的 `Untagged` 的信息。因为一个镜像可以对应多个标签，因此当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像，如果是这种情况，那么 `Delete` 行为就不会发生。所以并非所有的 `docker image rm` 都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。

镜像的删除也是分层进行的，如果某一层正被其他镜像使用，则不会被删除。另外，如果有基于当前镜像的容器正在进行，该镜像也不会被删除。

#### 虚悬镜像

有时候镜像列表中会出现没有仓库名，也没有标签的特殊镜像

```bash
<none>               <none>              00285df0df87        5 days ago          342 MB
```

这个镜像原本应当是有镜像名和标签的，但随着官方镜像维护，发布了新版本后，重新执行 `docker pull` 时，镜像名和标签被转移到了新下载的镜像上，旧的镜像上的名称则被取消，从而成为 `<none>`。除了 `docker pull` 可能导致这种情况，`docker build` 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 `` 的镜像。这类无标签镜像也被称为 **虚悬镜像(dangling image)** 。一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除。

```bash
$ docker image prune
```

## 2. 容器

镜像与容器的关系，大致可以比对面向对象中的类与实例。具体来说，容器是独立运行的一个或一组应用，以及它们的运行态环境。

### 2.1 启动

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`stopped`）的容器重新启动。

因为 Docker 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。

#### 新建并启动

所需要的命令主要为 `docker run`。

例如，下面的命令输出一个 “Hello World”，之后终止容器。

```bash
$ docker run ubuntu /bin/echo 'hello world'
# windows下使用git命令行需要输入 //bin/echo
hello world
```

这跟在本地直接执行 `/bin/echo 'hello world'` 几乎感觉不出任何区别。

下面的命令则启动一个 bash 终端，允许用户进行交互。

```bash
$ docker run -t -i ubuntu:18.04 /bin/bash
root@af8bae53bdd3:/#
```

其中，`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。这两个是最常使用的选项

在交互模式下，用户可以通过所创建的终端来输入命令，例如

```bash
root@af8bae53bdd3:/# pwd
/
root@af8bae53bdd3:/# ls
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

#### 启动已终止容器

已启动的容器在执行完毕后不会被自动删除，利用 `docker container start` 命令，可以将一个已经终止的容器重新启动运行。

容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 `ps` 或 `top` 来查看进程信息。

```bash
root@ba267838cc1b:/# ps
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   11 ?        00:00:00 ps
```

可见，容器中仅运行了指定的 bash 应用。这种特点使得 Docker 对资源的利用率极高，是货真价实的轻量级虚拟化。

### 2.2 守护态运行

更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。比如使用 `-d` 参数执行上面的命令

```bash
$ docker run -d ubuntu /bin/echo 'hello'
08e90e1961f8b12434931d5f0a64fa5f4615c9613d4b942310c991de17b9bc40
```

此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面，输出结果可以用 `docker logs` 查看

```bash
$ docker container logs 08e
hello
```

注：容器是否会长久运行，是和 `docker run` 指定的命令有关，和 `-d` 参数无关，容器只有在执行完命令后才会关闭，一种极端情况是，执行的命令是一个无限循环，这时容器永远不会关闭。

### 2.3 终止

我们已经知道，当 Docker 容器中指定的应用终结时，容器也自动终止。不过，也可以使用 `docker container stop` 来终止一个运行中的容器。

对于前面提到的启动了内置终端的容器，用户通过 `exit` 命令或 `Ctrl+d` 来退出终端时，所创建的容器立刻终止。

```bash
$ docker run -t -i ubuntu:18.04 /bin/bash
root@af8bae53bdd3:/#
# 输入 exit 或 使用 Ctrl+d 快捷键退出
```

使用 `docker container ls` 仅能看到运行状态的容器，要查看终止状态的容器需要使用 `docker container ls -a`

```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                         PORTS                               NAMES
08e90e1961f8        ubuntu              "//bin/echo hello"       9 minutes ago       Exited (0) 9 minutes ago                                           lucid_napier
20cd094abd6c        ubuntu              "//bin/bash"             16 minutes ago      Exited (0) 10 minutes ago                                          awesome_fermat
b57c3358e522        ubuntu              "//bin/echo 'Hello w…"   18 minutes ago      Exited (0) 18 minutes ago                                          gifted_murdock
e47304d78c1a        ubuntu:latest       "C:/Program Files/Gi…"   23 minutes ago      Created                                                            serene_shamir
81284613ca04        hello-world         "/hello"                 About an hour ago   Exited (0) About an hour ago                                       nostalgic_meitner
cd6182d13e68        mysql               "docker-entrypoint.s…"   3 weeks ago         Exited (255) 3 weeks ago       0.0.0.0:3306->3306/tcp, 33060/tcp   chitchat
```

处于终止状态的容器，可以通过 `docker container start` 命令来重新启动。

此外，`docker container restart` 命令会将一个运行态的容器终止，然后再重新启动它。

### 2.4 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。某些时候可能需要进入容器内进行操作，可以使用 `docker attach` 命令或 `docker exec` 命令，推荐使用 后者。

#### attach 命令

```bash
$ docker run -it -d ubuntu
e132b98549ad94ba0809a73e86bd63a3fd5f86067735beb83b6b74a14e51772f

$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e132b98549ad        ubuntu              "/bin/bash"         5 seconds ago       Up 4 seconds                            intelligent_galois

$ docker attach e13
root@e132b98549ad:/# 
```

如果从这个 stdin 中 exit，会导致容器的停止

#### exec 命令

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```bash
$ docker run -it -d ubuntu
8486e83970e0c8d2594dff9277df369a867dc662feb1ac3b28c84dd305c8f078

$ docker exec -i 848 bash
ls
bin
boot
dev
...

$ docker exec -it 848 bash
root@69d137adef7a:/#
```

如果从这个 stdin 中 exit，不会导致容器的停止，这就是为什么推荐使用 `docker exec`

### 2.5 导出与导入

如果要导出本地某个容器，可以使用 `docker export` 命令。

```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
8486e83970e0        ubuntu              "/bin/bash"         5 minutes ago       Up 5 minutes                                   hardcore_haslett
e132b98549ad        ubuntu              "/bin/bash"         7 minutes ago       Exited (0) 5 minutes ago                       intelligent_galois

$ docker export 848 > ubuntu.tar
```

这样将导出容器快照到本地文件，导出的目录为执行该命令的当前目录。

可以使用 `docker import` 从容器快照文件中再导入为镜像，例如

```bash
$ cat ubuntu.tar | docker import - test/ubuntu:v1.0
sha256:8b3f836f35916cf3aa2199b06e84e917a6421438a58ed4fc1924cd6d07b85d45

$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test/ubuntu         v1.0                8b3f836f3591        6 seconds ago       73.9MB
ubuntu              latest              1d622ef86b13        3 weeks ago         73.9MB
ubuntu              18.04               c3c304cb4f22        3 weeks ago         64.2MB
mysql               latest              0c27e8e5fcfa        3 weeks ago         546MB
hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
```

此外，也可以通过指定 URL 或者某个目录来导入，例如

```bash
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```

*注：用户既可以使用* *`docker load`* *来导入镜像存储文件到本地镜像库，也可以使用* *`docker import`* *来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。*

### 2.6 删除

可以使用 `docker container rm` 来删除一个处于终止状态的容器。

```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
8486e83970e0        ubuntu              "/bin/bash"         11 minutes ago      Up 11 minutes                                   hardcore_haslett
e132b98549ad        ubuntu              "/bin/bash"         13 minutes ago      Exited (0) 11 minutes ago                       intelligent_galois

$ docker container rm e13
e13

$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8486e83970e0        ubuntu              "/bin/bash"         11 minutes ago      Up 11 minutes                           hardcore_haslett
```

如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器。

```bash
$ docker container rm -f 848
848

$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

如果终止状态的容器太多，一个个删除很麻烦，可以使用下面的命令清理所有处于终止状态的容器

```bash
$ docker container prune
```

## 3. 仓库

仓库（`Repository`）是集中存放镜像的地方。

一个容易混淆的概念是注册服务器（`Registry`）。实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。例如对于仓库地址 `docker.io/ubuntu` 来说，`docker.io` 是注册服务器地址，`ubuntu` 是仓库名。

目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)，其中已经包括了数量超过 [3,480,000](https://hub.docker.com/search/?type=image) 的镜像。大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。

通过 `docker search` 命令来查找官方仓库中的镜像，通过 `docker pull` 命令来将它下载到本地。

也可以构建自己的私有仓库，但这里不做介绍。

参考：[前言 - Docker —— 从入门到实践 (gitbook.io)](https://yeasy.gitbook.io/docker_practice/)

