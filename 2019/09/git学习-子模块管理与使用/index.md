# Git学习 子模块管理与使用




当我们在一个 Git 项目上工作时，有时候需要在其中使用另外一个 Git 项目。这个情况可以在 Git 中使用子模块`submodule`来进行管理。`submodule`允许我们将一个 Git 仓库当作另外一个 Git 仓库的子目录。这允许我们克隆另外一个仓库到自己的项目中并且保持我们的提交相对独立。

hugo的博客源码文件中，主题的源码就是作为子模块进行添加和管理。本文中以本地项目blog为例，将远程主题项目KeepIt作为子模块进行管理。

### 1. 添加子模块

将远程项目`KeepIt`克隆到本地`themes`文件夹。

```bash
$ git submodule add https://github.com/Fastbyte01/KeepIt.git themes/KeepIt
```

添加子模块后运行`git status`, 可以看到目录有增加 1 个文件`.gitmodules`, 这个文件用来保存子模块的信息。

```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   themes/KeepIt
```

### 2. 查看子模块

```bash
$ git submodule
 87c33888f3fa86b8cc096bc3f6d7f2efe9ccba66 themes/KeepIt (v4-53-g87c3388)
```

### 3. 更新子模块

作为子模块的主题经常需要追随远程更新，可以使用如下方法

*   更新项目内子模块到最新版本
    
    ```bash
    $ git submodule update
    ```
    
*   更新子模块为远程项目的最新版本
    
    ```bash
    $ git submodule update --remote
    ```
    

### 4. 克隆包含子模块的项目

克隆包含子模块的项目有二种方法：一种是先克隆父项目，再更新子模块；另一种是直接递归克隆整个项目。

#### 克隆父项目，再更新子模块

1.  克隆父项目
    
    ```bash
     $ git clone https://github.com/shuzang/blog.git
 $ cd blog
    ```
    
1.  查看子模块
    
    ```bash
    $ git submodule
    -87c33888f3fa86b8cc096bc3f6d7f2efe9ccba66 themes/KeepIt
    ```
    
    子模块前面有一个`-`，说明子模块文件还未检入（空文件夹）。

3. 初始化子模块

   ```bash
   $ git submodule init
   Submodule 'themes/KeepIt' (https://github.com/Fastbyte01/KeepIt.git) registered for path 'themes/KeepIt'
   ```

    初始化模块只需在克隆父项目后运行一次。

4. 更新子模块

   ```bash
   $ git submodule update
   Cloning into 'C:/Users/lylw1/Desktop/blog/themes/KeepIt'...
   Submodule path 'themes/KeepIt': checked out '87c33888f3fa86b8cc096bc3f6d7f2efe9ccba66'
   ```

#### 递归克隆整个项目

```bash
 $ git clone https://github.com/shuzang/blog.git --recursive
```

递归克隆整个项目，子模块已经同时更新了，一步到位。

### 5. 修改子模块

在子模块中修改文件后，直接提交到远程项目分支。

```bash
$ git add .
$ git commit -m "modify submodule"
$ git push origin master
```

### 6. 删除子模块

**删除子模块比较麻烦，需要手动删除相关的文件，否则在添加子模块时有可能出现错误**  
同样以删除`KeepIt`文件夹为例

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
    
    


