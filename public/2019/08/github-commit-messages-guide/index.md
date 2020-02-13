# Git学习 commit信息编写指南


## 1. 什么是“commit”？

简单来讲，commit 就是在本地仓库中编写的文件的**快照**。与印象中不同的是，Git 不仅存储不同版本文件之间的差异，还存储了所有文件的完整版本。对于两个 commit 之间没有被修改的文件，Git 只存储指向前一个完全相同的文件的链接。

下面的图片展示了 git 如何随着时间存储数据，其中每个「Version」都是一个 commit：

![checking over time](https://user-images.githubusercontent.com/26682846/71801318-9538a800-3095-11ea-8163-15dc0cf1ac19.png)

## 2. Commit message

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

## 3. 写法规范

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

### 3.1 Header

Header部分只有一行，包括三个字段：`type`（必需）、`scope`（可选）和`subject`（必需）。 

#### type

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

#### scope

`scope`用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。 

#### subject

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

### 3.2 Body

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

### 3.3 Footer

Footer 部分只用于两种情况。 

#### 不兼容变动

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

#### 关闭Issue

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

 ```
Closes #234
 ```

 也可以一次关闭多个 issue 。 

```
Closes #123, #245, #992
```

### 3.4 Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以`revert:`开头，后面跟着被撤销 Commit 的 Header。 

```
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成`This reverts commit <hash>.`，其中的`hash`是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的`Reverts`小标题下面。

## 4. 生成Change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成，如下例

![Change Log](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016010603.png)

 生成的文档包括以下三个部分。 

- New features
- Bug fixes
- Breaking changes

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，还可以添加其他内容。 一般使用相关工具自动生成Change log。

## 5. 其它注意

遵循以上格式的同时，还有一些建议。

1. commit message解释当前 commit 所解决的问题，重点描述产生此更改的原因，而不是手段，因为代码会解释一切。应当解释清楚是否存在副作用以及其他不直观的影响。

2.  只看注释便可明白而无需查看变更内容

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

3.  使用信息本身来解释“原因”、“目的”、“手段”和其他的细节

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

## 6. 相关 git 命令

- `rebase -i`可以用来压缩提交（squash commits）、 编写信息、 重写/删除/重新编排 commit 等。

- `fixup`可用来清理 commit，而不需要复杂的 rebase。[这篇文章](http://fle.github.io/git-tip-keep-your-branch-clean-with-fixup-and-autosquash.html)提供了很好的示例，说明了如何以及何时进行此操作。

- `cherry-pick`在你 commit 到了错误的分支而不需要重新编码时非常有用。

  ```bash
  $ git cherry-pick 790ab21
  [master 094d820] Fix English grammar in Contributing
   Date: Sun Feb 25 23:14:23 2018 -0300
   1 file changed, 1 insertion(+), 1 deletion(-)
  ```


## 参考及扩展阅读

- [如何编写 Git Commit Message](https://chris.beams.io/posts/git-commit/)
- [Pro Git Book - Commit 指南](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project#_commit_guidelines)
- [关于 Git Commit Messages 的说明](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
- [合并与变基](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)
- [Pro Git Book - 改写历史](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)
- [Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
- [AngularJS Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0)
- [commit-messages-guide_中文版](https://github.com/RomuloOliveira/commit-messages-guide/blob/master/README_zh-CN.md)
- [Understanding Semantic Commit Messages Using Git and Angular](https://nitayneeman.com/posts/understanding-semantic-commit-messages-using-git-and-angular/)
