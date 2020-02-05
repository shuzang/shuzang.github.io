---
title: Git学习 提高
date: 2018-04-22T10:21:16+08:00
lastmod: 2020-01-07
tags: [git]
categories: [爱编程爱技术的孩子]
slug: Git learning improvement
---

上一篇我们学习了Git的基本知识，包括仓库创建，提交、修改、推送、回退等等操作，本篇学习一些高级的功能。

## 1. 分支管理

每个仓库都可能有几条不同的分支，比如master分支，比如用于开发的dev分支。分支操作的实质是对指针的操作

### 1.1 分支管理策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

- `master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面工作；

- 工作都在`dev`分支上，也就是说，`dev`分支是不稳定的，到大版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布大版本；

- 团队中每个人都在`dev`分支上干活，但每个人都有自己的分支，时不时地往`dev`分支上合并就可以了

### 1.2 创建分支

创建dev分支，然后切换到dev分支：

```bash
$ git checkout -b dev
Switched to a new branch 'dev'
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```bash
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
```
用`git branch`命令查看当前分支，该命令会列出所有分支，当前分支前面会标一个`*`号

```bash
$ git branch
* dev
  master
```

然后就可以在`dev`分支上进行修改提交，`commit`操作完成后对分支的操作也就完成，可以切换回`master`分支

```bash
$ git checkout master
```

切换回`master`分支后看不到刚才的修改，因为修改在`dev`分支上，必须先进行合并

### 1.3 合并分支

将`dev`分支的修改合并到`master`分支

```bash
$ git merge dev
```


`git merge`命令用于合并指定分支到当前分支。合并后，就能在`master`分支查看到刚刚在`dev`分支做的修改

合并完成后`dev`分支就没用了，可以删除它

```bash
$ git branch -d dev
```


强行删除分支使用参数`-D`

```bash
$ git branch -D dev
```
删除后查看分支，会发现只剩下`master`

```bash
$ git branch
```

### 1.4 解决冲突

有时候，不同的分支对同一处做了修改，此时合并会产生冲突，因为系统不知道该采用哪种修改。此时若强行合并，是无法通过的，系统会提示必须手动解决冲突后再提交。可以用`git status`命令查看冲突文件是哪个，手动解决冲突后重新提交即可成功，然后可以用带参数的`git log`命令查看合并情况

```bash
$ git log --graph --pretty=oneline --abbrev-commit
```
合并完成后删除分支

### 1.5 Bug分支

软件开发中，出现bug是正常的事情，有了bug就需要修复。在git中，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除

**背景**：当你需要修复某个bug时，当前在`dev`分支上进行的工作还没有提交，而且因为工作只进行到一半，完全没法提交，但bug却必须马上修复

**解决办法**：`git stash`命令，可以把当前工作现场保存，等以后恢复现场工作后继续工作

```bash
$ git stash
$ git status
```
使用`git status`查看工作区确认工作区是干净的，然后按如下步骤进行

1. 切换到bug所在分支
2. 创建bug修复分支
3. 修复
4. 提交
5. 切换回bug所在分支
6. 合并分支
7. 删除bug修复分支
8. 切换回工作的dev分支

查看修复bug前保存的工作现场

```bash
$ git stash list
```
恢复工作现场

```bash
$ git stash pop
```

## 2. 多人协作

当我们从远程仓库克隆时，实际上Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin`
要查看远程库的信息

```bash
$ git remote
```
显示远程库更详细的信息

```bash
$ git remote -v
```
使用`-v`参数会显示可抓取和推送的`orign`的地址，但若没有推送权限，则看不到push地址

### 2.1 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程仓库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

```bash
$ git push origin master
```
如果要推送其他分支，比如dev，就改成：

```bash
$ git push origin dev
```
但是，并不是一定要把本地分支往远程推送

- master分支是主分支，因此要时刻与远程同步；
- dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
- bug分支只用于在本地修复bug，没必要推到远程

在git中，分支完全可以在本地自己使用，是否推送，视心情而定

### 2.2 抓取分支

在本地创建和远程分支对应的分支，本地和远程分支的名称最好一致

```bash
git checkout -b branch-name origin/branch-name
```
建立本地分支和远程分支的关联

```bash
git branch --set-upstream branch-name origin/branch-name
```
从远程抓取分支

```bash
git pull
```

## 3. 标签管理

因为版本号太复杂，所以git提供了标签功能 

命令`git tag <name>`用于新建一个标签，默认为当前分支，也可以指定一个commit id； 

命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息； 

命令`git tag -s <tagname> -m "blablabla..."`可以用PGP签名标签； 

命令`git tag`可以查看所有标签 

命令`git push origin <tagname>`可以推送一个本地标签； 

命令`git push origin --tags`可以推送全部未推送过的本地标签； 

命令`git tag -d <tagname>`可以删除一个本地标签； 

命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签

## 4. 忽略特殊文件

有时候，git工作目录里的某些文件并不想同步到远程的github仓库，此时可以在git工作区根目录下创建`.gitignore`文件，然后把要忽略的文件名填进去，git会自动忽略这些文件

**注**：`.gitignore`文件本身要上传到版本库进行管理

## 参考资料
[1] [git cheat sheet](https://github.com/shuzang/books/blob/master/git-cheatsheet.pdf) 

[2] [git官网](https://git-scm.com/) 

[3] [git中文指南](https://git-scm.com/book/zh/v2)

[4] [廖雪峰的官方网站-git教程](<https://www.liaoxuefeng.com/wiki/896043488029600>)