---
title: Git提高-分支管理, 多人协作, 标签管理, gitignore, commit信息
date: 2018-04-22T10:21:16+08:00
lastmod: 2020-10-07
tags: [git]
categories: [爱编程爱技术的孩子]
slug: Git learning improvement 
---

上一篇我们学习了Git的基本知识，包括仓库创建，提交、修改、推送、回退等等操作，本篇学习一些高级的功能。转自 [廖雪峰的官方网站-git教程](<https://www.liaoxuefeng.com/wiki/896043488029600>)。

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

## 5. commit 信息编写

简单来讲，commit 就是在本地仓库中编写的文件的**快照**。与印象中不同的是，Git 不仅存储不同版本文件之间的差异，还存储了所有文件的完整版本。对于两个 commit 之间没有被修改的文件，Git 只存储指向前一个完全相同的文件的链接。

下面的图片展示了 git 如何随着时间存储数据，其中每个「Version」都是一个 commit：

![checking over time](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20180422_3AIT4f.png)

### 5.1 Commit message

Git 每次提交代码，都要写 Commit message（提交说明），否则就不允许提交。

 ```bash
$ git commit -m "hello world"
 ```

上面代码的`-m`参数，就是用来指定 commit mesage 的。

如果一行不够，可以只执行`git commit`，就会跳出文本编辑器，让你写多行。

 ```bash
$ git commit
 ```

commit信息可以随意编写，但一般来说，应该清晰明了，说明本次提交的目的。 一个清晰的、规范化的Commit message，有如下好处

1. 提供更多的历史信息，方便快速浏览

2. 过滤某些commit（比如文档改动），便于快速查找信息

3. 可以直接从commit生成Change log，Change Log 是发布新版本时，用来说明与上一个版本差异的文档 

   ![Change Log](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016010603.png)

而且，查看commit信息的多数是团队成员或其它对该项目感兴趣的人，规范清晰的commit信息对它们有如下意义

- 加快和简化代码审查（code reviews）
- 帮助理解一个更改
- 解释不能只由代码描述的「为什么」
- 帮助未来的维护人员弄清楚为什么以及如何产生的更改，从而使故障排查和调试更容易

### 5.2 写法规范

目前，社区有多种 Commit message 的写法规范。但[Angular 规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0)是使用最广的写法，比较合理和系统化，并且有配套的工具，因此学习和使用这种规范。

Commit message分为三部分： Header，Body 和 Footer。 

```bash
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

其中，Header 是必需的，Body 和 Footer 可以省略。

不管是哪一个部分，任何一行都不得超过72个字符（或100个字符）。这是为了避免自动换行影响美观。

#### Header

Header部分只有一行，包括三个字段：`type`（必需）、`scope`（可选）和`subject`（必需）。 

**type**

 `type`用于说明 commit 的类别，只允许使用下面7个标识。 

```bash
feat：新功能（feature）
fix：修补bug
docs：文档（documentation）
style： 格式（不影响代码运行的变动）
refactor：重构（即不是新增功能，也不是修改bug的代码变动）
test：增加测试
chore：构建过程或辅助工具的变动
```

如果`type`为`feat`和`fix`，则该 commit 将肯定出现在 Change log 之中。其他情况（`docs`、`chore`、`style`、`refactor`、`test`）由你决定，要不要放入 Change log，建议是不要。 

**scope**

`scope`用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。 

**subject**

`subject`是 commit 目的的简短描述，不超过50个字符。 

- 以动词开头，使用第一人称现在时，比如`change`，而不是`changed`或`changes`
- 第一个字母小写
- 结尾不加句号（`.`）

一些例子如下

```
docs(error/$rootScope/inprog): add missing "$timeout"
fix(loader): use `false` as default value for `transclude` in compone
feat($compile): Allow ES6 classes as controllers with `binToControll`
```

#### Body

Body 部分是对本次 commit 的详细描述，可以分成多行。下面是一个范例。 

```
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```

有两个注意点。

- 使用第一人称现在时，比如使用`change`而不是`changed`或`changes`。
- 应该说明代码变动的动机，以及与以前行为的对比。

#### Footer

Footer 部分只用于两种情况。 

**不兼容变动**

如果当前代码与上一个版本不兼容，则 Footer 部分以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法。 

```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

**关闭Issue**

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

 ```
Closes #234
 ```

 也可以一次关闭多个 issue 。 

```
Closes #123, #245, #992
```

#### Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以`revert:`开头，后面跟着被撤销 Commit 的 Header。 

```
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成`This reverts commit <hash>.`，其中的`hash`是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的`Reverts`小标题下面。

### 5.3 生成Change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成，如下例

![Change Log](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016010603.png)

 生成的文档包括以下三个部分。 

- New features
- Bug fixes
- Breaking changes

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，还可以添加其他内容。 一般使用相关工具自动生成Change log。

### 5.4 相关命令

- `rebase -i`可以用来压缩提交（squash commits）、 编写信息、 重写/删除/重新编排 commit 等。

- `fixup`可用来清理 commit，而不需要复杂的 rebase。[这篇文章](http://fle.github.io/git-tip-keep-your-branch-clean-with-fixup-and-autosquash.html)提供了很好的示例，说明了如何以及何时进行此操作。

- `cherry-pick`在你 commit 到了错误的分支而不需要重新编码时非常有用。

  ```bash
  $ git cherry-pick 790ab21
  [master 094d820] Fix English grammar in Contributing
   Date: Sun Feb 25 23:14:23 2018 -0300
   1 file changed, 1 insertion(+), 1 deletion(-)
  ```

### 5.5 其它注意

遵循以上格式的同时，还有一些建议。

1. commit message解释当前 commit 所解决的问题，重点描述产生此更改的原因，而不是手段，因为代码会解释一切。应当解释清楚是否存在副作用以及其他不直观的影响。

2. 只看注释便可明白而无需查看变更内容

   ```bash
   # Good
   Add `use` method to Credit model 
   为 Credit 模块添加 `use` 方法
   # Bad
   Add `use` method 
   添加 `use` 方法
   ```

   ```bash
   # Good
   Increase left padding between textbox and layout frame 
   在 textbox 和 layout frame 之间添加向左对齐
   # Bad
   Adjust css 
   就改了下 css
   ```

3. 使用信息本身来解释“原因”、“目的”、“手段”和其他的细节

   ```bash
   # Good
   Fix method name of InventoryBackend child classes
   
   Classes derived from InventoryBackend were not
   respecting the base class interface.
   
   It worked because the cart was calling the backend implementation
   incorrectly.
   ```

   ```bash
   # Good
   Serialize and deserialize credits to json in Cart
   
   Convert the Credit instances to dict for two main reasons:
   
     - Pickle relies on file path for classes and we do not want to break up
       everything if a refactor is needed
     - Dict and built-in types are pickleable by default
   ```

   ```bash
   # Good
   Add `use` method to Credit
   
   Change from namedtuple to class because we need to
   setup a new attribute (in_use_amount) with a new value
   ```

4. 保持语言的一致性

   对于项目所有者而言：选择一种语言并使用该语言编写所有的 commit 信息。理想情况下，它应与代码注释、默认翻译区域（用于本地化项目）等相匹配。

   对于贡献者而言：使用与现有 commit 历史相同的语言编写 commit 信息。

   ```bash
   # Good
   ababab Add `use` method to Credit model
   efefef Use InventoryBackendPool to retrieve inventory backend
   bebebe Fix method name of InventoryBackend child classes
   ```

    ```bash
   # Good (Portuguese example)
   ababab Adiciona o método `use` ao model Credit
   efefef Usa o InventoryBackendPool para recuperar o backend de estoque
   bebebe Corrige nome de método na classe InventoryBackend
    ```

    ```bash
   # Bad (mixes English and Portuguese)
   ababab Usa o InventoryBackendPool para recuperar o backend de estoque
   efefef Add `use` method to Credit model
   cdcdcd Agora vai
    ```

## 参考及扩展阅读
- [git cheat sheet](https://github.com/shuzang/books/blob/master/git-cheatsheet.pdf) 

- [git官网](https://git-scm.com/) 

- [git中文指南](https://git-scm.com/book/zh/v2)

- [廖雪峰的官方网站-git教程](<https://www.liaoxuefeng.com/wiki/896043488029600>)

- [如何编写 Git Commit Message](https://chris.beams.io/posts/git-commit/)
- [Pro Git Book - Commit 指南](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project#_commit_guidelines)
- [关于 Git Commit Messages 的说明](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
- [合并与变基](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)
- [Pro Git Book - 改写历史](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)
- [Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
- [AngularJS Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0)
- [commit-messages-guide_中文版](https://github.com/RomuloOliveira/commit-messages-guide/blob/master/README_zh-CN.md)
- [Understanding Semantic Commit Messages Using Git and Angular](https://nitayneeman.com/posts/understanding-semantic-commit-messages-using-git-and-angular/)