---
title: Git入门-基本概念, 基本原理, 安装使用
date: 2018-04-20T19:06:20+08:00 
lastmod: 2020-10-08
tags: [git]
categories: [爱编程爱技术的孩子]
slug: Git learning Getting started 
---

版本控制系统在代码开发过程中必不可缺，本系列学习 Git 的使用，版本托管服务则是 Github。以廖雪峰大神 [git教程](<https://www.liaoxuefeng.com/wiki/896043488029600>) 为主体，辅以使用过程中遇到的问题和心得。

<!--more-->

## 1. 版本控制系统

### 1.1 作用

版本控制系统用来解决两种情况

1. 文章不断进行修改，但又觉得之前的版本可能有用，于是产生大量重命名文件，代码编辑亦如是，而且更加不容易回溯。

   ```
   版本1.doc   
   版本2.doc   
   最终版本.doc   
   最终版本改.doc   
   真的是最终版本了.doc
   ```

2. 多人协作，每次都需要校对对方的改动，然后进行合并，会花费大量不必要的精力

版本控制的作用就是记录所有的修改历史，便于需要时回退，同时提供一个协作的平台。

### 1.1 历史

Git 属于分布式版本控制，在此之前主要是**集中式的版本控制**，基本原理是有一个服务器放所有的代码，要工作的时候下载回来，改动完上传回去，记录不同人的改动和合并都由服务器端完成，这种形式极度依赖于服务器性能和网速。

2005 年 Linux 内核代码版本控制使用的软件失效，Linux 创始人 Linus 花了一周开发了 Git 这个分布式版本控制系统，其后慢慢流行，现在已用于大多数的项目代码管理。

**分布式版本控制**和 torrent 差不多，每个人的电脑都是服务器，修改完直接把修改的部分推送给所有拥有这个项目的人。分布式版本控制也有服务器，但主要是负责做备份，并方便大家的交换用的，并不是必要。

### 1.2 Github

**Github**，发音为「 ɡɪthʌb 」，是为开源项目免费提供 Git 存储的网站，2008年上线。

注意，Git 和 Github 这两个单词在官方的网站上首字母都是大写，因此我们遵从这样的规则。

## 2. 安装使用
Git 是全平台的软件，命令行界面，连接到服务商（比如 Github）就能使用，目前 Github 已经推出了 windows 上的 GUI 版本，操作更加简便。

### 2.1 Linux安装

```bash
$ sudo apt-get install git
$ git version
```
安装完成后还要设置自己的名字和Email地址

```bash
$ git config --global  user.name   "your name"
$ git config --global  user.email  "email@example.com"
```

`--global`参数的意思是这个名字和邮箱应用于所有git仓库

### 2.2 windows安装
从[官网](https://git-scm.com/)下载`Download x.xx for Windows`按照，会自动配置环境变量。我一般使用chocolatey包管理工具，执行命令如下

```powershell
> choco install git
```

Windows下GUI版下载地址为：https://desktop.github.com/ ，嗯，真香，windows下我已经很久不用命令行的git了，一般都当作shell工具来使用，而不是执行git命令。

## 3. 仓库和文件

[仓库]^(repository)，本质是一个文件夹，存放归属同一个项目的所有文件，是版本控制中的一个重要概念，可分为本地仓库和云端仓库。本地仓库是自己的代码编辑工作区，云端仓库是代码的备份。

### 3.1 创建一个仓库

在本地任意路径创建`gittest`文件夹，路径自定义

```bash
$ mkdir gittest
```

进入创建的文件夹，使用 `git init` 命令把该目录变成git的仓库

```bash
$ cd gittest
$ git init
```

然后仓库就建好了，该目录下会多一个`.git`隐藏文件夹，用来跟踪管理仓库。使用`ls -a`命令可以看到该文件夹，默认隐藏说明它很重要，里面的文件改动的话很容易导致仓库损坏。`.git`文件夹下的内容包括

```bash
$ ls .git
config  description  HEAD  hooks/  info/  objects/  refs/
```

一些文件的作用在`暂存区和工作区`部分解释。

### 3.2 在仓库中添加文件

添加文件的类型有讲究 

- 对txt文档、代码文件等文件类型，版本控制系统可记录每次的改动

- 对图片、视频、PDF等文件类型，版本控制系统只能管理，不能记录改动，最多只能记录大小变化，Word文件实质是二进制格式，所以同样没法跟踪改动


首先新建一个txt文件并放在gittest目录下，编辑其文件内容

```bash
$ cd gittest
$ touch hellogit.txt
$ cat hellogit.txt
hello git
```

用`git add`命令将该文件添加到暂存区，暂存区的概念之后解释

```bash
$ git add  hellogit.txt
```


用git  commit命令将文件提交到仓库(这里指本地仓库)

```bash
$ git commit -m "first commit"
[master (root-commit) a91c248] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 hellogit.txt
```

双引号中为备注信息，是该次提交的说明，一般情况是描述此次修改的。commit添加的提交信息对于代码review具有重要的作用，要遵循一定的规范。

命令执行完后的提示包括以下内容： 

1. 提交到了主分支（master)，备注信息为「first commit」 
2. 改动了一个文件（hellogit.txt），加了1行内容(注意那个+号) 
3. 最后一行中100代表regular file，644代表文件权限 

`git add`命令可以有多个，分多次将文件添加到暂存区，之后执行`git commit`命令会一次性将之前add的所有文件一起提交，如

```bash
git add file1.txt
git add file2.txt file3.txt
git commit -m "add 3 files"
```

也可以使用如下命令一次性将所有修改添加到暂存区，然后执行`commit`命令

```bash
 $ git add .
```

### 3.3 对文件进行修改

版本控制的目的就是记录文件的改动，我们将hellogit.txt文件的内容进行第二次修改，用 git status 命令查看仓库当前状态

```bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   hellogit.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

返回的提示内容会包括

- 位于哪个分支
- 修改了哪个文件
- 「修改尚未加入提交」

如果我们想看到具体做了哪些修改，用`git diff`命令，diff就是diffenence，所以`git diff`的意思就是查看不同

```bash
$ git diff hellogit.txt
diff --git a/hellogit.txt b/hellogit.txt
index f09e9c3..a90a76d 100644
--- a/hellogit.txt
+++ b/hellogit.txt
@@ -1 +1,2 @@
-hello git
\ No newline at end of file
+hello git
+add a line
\ No newline at end of file
```

提示中说明了我们在末尾添加了一行，其内容为`add a line`。此时我们确认了做的修改是我们心里所想的，就可以再次提交了

```bash
$ git add hellogit.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   hellogit.txt

$ git commit -m "add a line"
[master ae3c27f] add a line
 1 file changed, 2 insertions(+), 1 deletion(-)

$ git status
On branch master
nothing to commit, working tree clean
```

提交过程中可以每一步都使用`git status`命令查看当前状态，最后commit提交完的状态提示为：

```
位于master分支
无文件要提交，干净的工作区
```

此时就放心了，每次修改完或者要离开工作岗位的时候提交一次，就像游戏存档，离开之前或者重要关卡前存档，失败的时候就能读档重来，这里修改出错也可以从上一次的commit重来

## 4. 远程仓库

本地仓库的代码编辑完成后，需要推送到远程仓库进行备份，我们使用Github提供的免费远程仓库。由于本地Git仓库和GitHub远程仓库之间的传输是通过SSH加密的，所以，需要先设置SSH密钥。

### 4.1 创建SSH Key

在用户主目录下，看看有没有`.ssh`(默认隐藏)目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开终端，创建SSH Key：

```bash
$ ssh-keygen -t rsa -C "youremail@example.com"
```

把邮件地址换成自己的邮件地址，然后一路回车，使用默认值即可，是否设置密码自己选择。

如果一切顺利的话，可以在用户主目录里找到`.ssh`目录，里面有私钥`id_rsa`和公钥`id_rsa.pub`两个文件。

### 4.2 登陆GitHub

点击右上角头像，打开`Settings——>SSH and GPG Keys——>New SSH Key`

填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容，然后点击`Add SSH key`就行了

GitHub需要SSH Key识别出你推送的提交确实是你推送的，而不是别人冒充的。当然，GitHub允许添加多个Key。假定有若干电脑，只要把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送了。

### 4.3 创建远程仓库

首先在Github创建一个**同名仓库**

![新建仓库](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180420_3A5tOA.png)
![新建仓库](https://user-images.githubusercontent.com/26682846/71865229-126b2800-313d-11ea-928e-ee03fc5ffdd1.png)

创建完成后跳转界面给我们提供了三个选择

- ...or create a new repository on the command line

   ```bash
  echo "# gittest" >> README.md
  git init
  git add README.md
  git commit -m "first commit"
  git remote add origin https://github.com/shuzang/gittest.git
  git push -u origin master
  ```

- ...or push an existing repository from the command line

   ```bash
  git remote add origin https://github.com/shuzang/gittest.git
  git push -u origin master
  ```

- ...or import code from another repository

    You can initialize this repository with code from a Subversion, Mercurial, or TFS project. 

第一个选项基本就是没有本地仓库的情况下我们要做的所有事情，不过现在我们已经有了本地仓库，所以选择第二个选项，将本地已存在的gittest仓库关联到Github。

按提示执行命令即可

```bash
$ git remote add origin https://github.com/shuzang/gittest.git
$ git push -u origin master
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (6/6), 448 bytes | 448.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0)
To https://github.com/shuzang/gittest.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

返回的结果告诉我们分支master设置为跟踪来自origin的远程分支master，代码已推送到远程仓库。此时查看远程仓库如下

![gittest仓库](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180420_3A5DfS.png)

点击左侧中间的`2 commits`可以查看提交的具体情况

![提交信息](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180420_3A52mn.png)

继续点击每个具体的提交可以查看代码或文件的改动

![add a line](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180420_3A5opF.png)

查看别人的代码时，可以从这里查看一次次提交的改动和commit信息，有助于帮助我们学习别人的代码和编写思路

以后编辑和提交代码，主体就三个命令

```bash
$ git add .
$ git commit -m "commit message"
$ git push origin master
```

其它的git命令都是辅助，之后会慢慢介绍

### 4.4 从远程仓库克隆到本地

换到新电脑或者不是自己从头新建仓库，而是使用别人的仓库进行合作开发，使用`git clone`命令，

```bash
$ git clone 仓库地址
```

如果是下载某个指定分支的内容，命令为

```bash
$ git clone -b 分支名 仓库地址
```

## 5. 历史版本

修改的次数会逐渐变多，随着时间的流逝，我们完全不会记得每次修改了哪些内容，当然，git的出现就是为了帮我们解决这个问题，所以理所当然的会有查看历史版本的能力，以及回退到某个版本的能力，云端查看历史版本如上节所述，本地命令行如何查看在本节详细解释，以及介绍如果丢弃错误的提交。

### 5.1 查看历史版本

使用 `git log` 命令可以查看历史改动

```bash
$ git log
commit ae3c27f99068e6eb301fd7d28960b97ec56b9733 (HEAD -> master, origin/master)
Author: shuzang <844327696@qq.com>
Date:   Tue Jan 7 10:52:14 2020 +0800

    add a line

commit a91c24851ecd74e933bd638fd30773050bd7c692
Author: shuzang <844327696@qq.com>
Date:   Tue Jan 7 10:42:01 2020 +0800

    first commit
```

提示内容是按从近到远的顺序把所有改动列举一遍，包括用户名、邮箱、修改日期，当时添加的message信息，不想看名称、邮箱和日期可以使用参数简化显示，如下：

```bash
$ git log --pretty=oneline
ae3c27f99068e6eb301fd7d28960b97ec56b9733 (HEAD -> master, origin/master) add a line
a91c24851ecd74e933bd638fd30773050bd7c692 first commit
```

也可以用以下命令查看参数使用方式

```bash
$ git log --help
```

提示内容中每次修改最前面的一串长字符是版本号（commit  id），用十六进制表示，用来代表我们每次修改的版本

### 5.2 回退

首先，在git中，用`HEAD`表示当前版本，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然历史版本数多的时候写`^`写不过来，所以写成`HEAD~number`，如往上100个版本就是`HEAD-100`  
回退命令为`git reset`，比如，回退到上一个版本

```bash
$ git  reset --hard HEAD^
HEAD is now at a91c248 first commit
```

然后我们就看到真的变回去了，如果记下了版本号，我们还可以再跳回最后一个修改版本，也就是说，我们突然不想回退了，或者退多了

```bash
$ git reset  --hard ae3c
HEAD is now at ae3c27f add a line
```

`ae3c`为其版本号，但版本号没必要写全，因为它的作用是为了区分各版本，所以只要找到第一个不相同的那位字母，写到那里就行了

版本切换的速度非常快，主要是因为`HEAD`只是一个指向当前版本的指针，跳转版本的时候也仅仅移动指针 

在不记得版本号的情况下，git还提供了`git reflog`命令记录每次`HEAD`指针移动

```bash
$ git reflog
ae3c27f (HEAD -> master, origin/master) HEAD@{0}: reset: moving to ae3c
a91c248 HEAD@{1}: reset: moving to HEAD^
ae3c27f (HEAD -> master, origin/master) HEAD@{2}: commit: add a line
a91c248 HEAD@{3}: commit (initial): first commit
```

该命令会显示每次修改版本的版本号

## 6. 三大工作区域

使用 Git 最好理解它的三大工作区域：工作目录、暂存区和版本仓库。

**工作目录**就是代码编辑的主体目录，存放整个项目的源码和相关文件。

**暂存区**也叫做索引，用来跟踪工作目录的所有文件。事实上，工作目录中的每一个文件要么处于已跟踪状态，要么处于未跟踪状态，执行这个跟踪动作的，就是索引。索引所对应的静态文件是工作目录下 `.git` 隐藏文件夹中的 `index` 文件，其中记录每个已跟踪文件的文件名、文件时间戳、文件长度等元数据信息。需要注意的是，索引并不保持文件的具体内容，只记录文件的改动，文件的具体内容保持在版本仓库中。`git add` 命令可以将改变提交给暂存区。

**版本仓库**就是本地电脑具体保存所有代码的地方。具体位置是 `.git/objects` 文件夹，暂存区的文件经过 `git commit` 命令可以提交到本地版本仓库。本地版本仓库需要和远程的云端仓库（也就是存在 Github 服务器里的仓库文件）同步，如果有修改，需要推送到远程仓库，如果远程有别人的提交，需要拉取和本地代码合并。

版本库里有暂存区，还有自动创建的第一个分支master，以及指向master的一个指针叫HEAD
 ![工作区与版本库图解](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180420_3AIQns.png)
`git add`实际上是把文件修改添加到暂存区；
`git commit`实际上是把暂存区的所有内容提交到当前分支

在推送到远程仓库之前，一切修改都有挽救的机会

### 6.1 丢弃修改

**场景1**：当改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，使用命令

```bash
# 撤销单个文件修改
$ git checkout -- file
# 撤销从上次提交后所有修改
$ git checkout -- *
```

**场景2**：当不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令

```bash
$ git reset HEAD file
$ git checkout -- file
```

**场景3**：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考上一节进行版本回退

最后，如果代码已经提交到了远程仓库，那么只能进行修复然后进行新的提交

### 6.2 删除文件

一般情况，是直接在文件夹中用rm命令删除，但这样工作区和版本库就不一致了

```bash
$ rm hellogit.txt
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    hellogit.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git add .
$ git commit -m "remove hellogit.txt"
```

可以用命令恢复

```bash
$ git checkout -- hellogit.txt
```

正常从版本库中删除该文件，用`git rm`命令，并且`git commit`

```bash
$ git rm hellogit.txt
rm 'hellogit.txt'
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    hellogit.txt

$ git commit -m "remove hellogit.txt"
```

两者的区别是直接删除需要额外执行`git add`命令，而`git rm`删除后可以直接`commit`，当然，其实没有多少影响，具体用什么方法，自己选择就好。

## 参考与扩展

[1] [廖雪峰的官方网站-git教程](<https://www.liaoxuefeng.com/wiki/896043488029600>)

[2] [猴子都能懂的git入门](https://backlog.com/git-tutorial/cn/intro/intro1_1.html)

[3] [Github Learning Lab](https://lab.github.com/)

[4] [Hellogithub](https://hellogithub.com/)