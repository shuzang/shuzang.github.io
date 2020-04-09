# Git学习 子模块管理与使用


当我们在一个 Git 项目上工作时，有时候需要在其中使用另外一个 Git 项目。这个情况可以在 Git 中使用子模块 `submodule` 来进行管理。`submodule` 允许我们将一个 Git 仓库当作另外一个 Git 仓库的子目录。这允许我们克隆另外一个仓库到自己的项目中并且保持我们的提交相对独立。

Hugo 的博客源码文件中，主题的源码就是作为子模块进行添加和管理。本文中以本地项目 blog 为例，将远程主题项目 LoveIt 作为子模块进行管理。

## 1. 添加子模块

将远程项目 `LoveIt` 克隆到本地 `blog/themes` 文件夹，`blog` 需要本身就是一个 Git 项目

```bash
$ git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

添加子模块后运行`git status`, 可以注意增加了一个文件 `.gitmodules`,

```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   themes/LoveIt
```

这个文件用来保存子模块的信息。

```bash
[submodule "themes/LoveIt"]
	path = themes/LoveIt
	url = https://github.com/dillonzq/LoveIt.git
```

## 2. 查看子模块

```bash
$ git submodule
 bf7c4b5173c3baba02b87a410ce04909c1b86cf6 themes/LoveIt (v0.1.4)
```

## 3. 更新子模块

作为子模块的主题经常需要追随远程更新，这也是最常见的一种情况，即只使用子项目并不时获取更新，而不在子项目中做任何更改。

最容易的方式是添加参数 `--remote`，Git 会进入子模块然后抓取并更新

```bash
$ git submodule update --remote
```

该命令假设我们想要更新子模块仓库的 `master` 分支，如果想要设置为其它的分支，可以在 `.gitmodules` 中进行设置

```bash
$ git config -f .gitmodules submodule.themes/LoveIt.branch develop
$ git submodule update --remote
Submodule path 'themes/LoveIt': checked out 'b6ce753ae7892839899962b879e2cc5808c60732'
```

使用 `git diff` 可以查看到变动，`.gitmodules`多了一行，子模块的分支号有了变动

```bash
$ git diff
diff --git a/.gitmodules b/.gitmodules
index a6f32d0..5376abf 100644
--- a/.gitmodules
+++ b/.gitmodules
@@ -1,3 +1,4 @@
 [submodule "themes/LoveIt"]
        path = themes/LoveIt
        url = https://github.com/dillonzq/LoveIt.git
+       branch = develop
diff --git a/themes/LoveIt b/themes/LoveIt
index bf7c4b5..b6ce753 160000
--- a/themes/LoveIt
+++ b/themes/LoveIt
@@ -1 +1 @@
-Subproject commit bf7c4b5173c3baba02b87a410ce04909c1b86cf6
+Subproject commit b6ce753ae7892839899962b879e2cc5808c60732
```

当运行 `git submodule update --remote` 时，Git 默认会尝试更新 **所有** 子模块， 所以如果有很多子模块的话，你可以传递想要更新的子模块的名字。比如

```bash
$ git submodule update --remote themes/LoveIt
```



## 4. 克隆包含子模块的项目

如果像下面这样直接克隆包含子模块的项目，虽然有子模块目录，但是是空的

```bash
 $ git clone https://github.com/shuzang/blog.git
 $ cd blog
 $ ls themes/LoveIt
 $
```

查看子模块时前面有一个`-`，也说明子模块文件还未检入（空文件夹）。

```bash
$ git submodule
-87c33888f3fa86b8cc096bc3f6d7f2efe9ccba66 themes/KeepIt
```

此时需要运行两个命令，`git submodule init` 用来初始化子模块

```bash
$ git submodule init
Submodule 'themes/KeepIt' (https://github.com/Fastbyte01/KeepIt.git) registered for path 'themes/KeepIt'
```

 `git submodule update` 用来抓取数据

```bash
$ git submodule update
Cloning into 'C:/Users/lylw1/Desktop/blog/themes/KeepIt'...
Submodule path 'themes/KeepIt': checked out '87c33888f3fa86b8cc096bc3f6d7f2efe9ccba66'
```

不过有一个更简单的方法，就是给 `git clone` 命令传递 `--recurse-submodules` 参数，这样就会自动初始化并更新仓库中每一个子模块，包括可能存在的嵌套子模块

```bash
 $ git clone https://github.com/shuzang/blog.git --recurse-submodules
```

## 5. 修改子模块

在子模块中修改文件后，直接提交到远程项目分支。

```bash
$ git add .
$ git commit -m "modify submodule"
$ git push origin master
```

## 6. 删除子模块

之前从网上找到的办法是，手动删除相关的文件，以删除`KeepIt`文件夹为例

1.  删除子模块文件夹
    
    ```bash
    $ git rm --cached themes/KeepIt
	$ rm -rf themes/KeepIt
    ```
    
1.  删除`.gitmodules`文件中相关子模块信息
    
    ```bash
    [submodule "themes/KeepIt"]
	path = themes/KeepIt
    	url = https://github.com/Fastbyte01/KeepIt.git
    ```
    
1.  删除`.git/config`中的相关子模块信息
    
    ```bash
    [submodule "themes/KeepIt"]
	url = https://github.com/Fastbyte01/KeepIt.git
    ```
    
1.  删除`.git`文件夹中的相关子模块文件
    
    ```bash
    $ rm -rf .git/modules/themes/KeepIt
    ```
    

不过实际上，这是一种比较野的做法，不建议使用。根据官方文档的说明，应该使用 `git submodule deinit` 命令卸载一个子模块。这个命令如果添加上参数 `--force`，则子模块工作区内即使有本地的修改，也会被移除。

```bash
$ git submodule deinit themes/LoveIt
$ git rm themes/LoveIt
```

执行 `deinit` 的效果，是自动在 `.git/config` 中删除了如下信息

```
[submodule "themes/LoveIt"]
	url = https://github.com/dillonzq/LoveIt.git
```

执行 `rm` 的效果，是移除了 `themes/LoveIt` 文件夹，并删除了 `.gitmodules` 中的相关内容

此时关于子模块的信息基本已经删除完毕，只剩 `.git/modules` 下还有一些残留，但执行 `commit` 后残留会被删除

```bash
$ git commit -m "delete submodule themes/LoveIt"
```

## 参考文献

[1] 知乎-孤单彼岸. [Git中submodule的使用](https://zhuanlan.zhihu.com/p/87053283?utm_source=cn.ticktick.task&utm_medium=social&utm_oi=1153472097597743104). 2019-10-17

[2] Git文档. [Git工具-子模块](https://git-scm.com/book/zh/v2/Git-工具-子模块). 2nd Edition.


