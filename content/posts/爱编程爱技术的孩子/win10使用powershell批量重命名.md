---
title: win10使用powershell批量重命名
date: 2020-08-08
tags: [Win10]
categories: [爱编程爱技术的孩子]
slug: use powershell rename multiple files
typora-root-url: ..\..\..\static
---

win10 需要批量重命名的时候很多，每次打开专门的软件比较麻烦，于是查了下使用 powershell 如何达成这一目的。

<!--more-->

首先，win10 下无法在 git bash 中使用 rename 命令，不被支持。

其次，网上搜索得到的各种使用批处理文件的方法过于繁琐，每次新建批处理文件还不如使用批量重命名软件。

最后，powershell 提供了 [Rename-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/rename-item?view=powershell-7) 命令来完成重命名功能。

下面是官方提供的一个例子，将所有 `*.txt` 文件替换为 `*.log`

```powershell
Get-ChildItem *.txt

Directory: C:\temp\files

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/3/2019   7:47 AM           2918 Friday.TXT
-a----        10/3/2019   7:46 AM           2918 Monday.Txt
-a----        10/3/2019   7:47 AM           2918 Wednesday.txt

Get-ChildItem *.txt | Rename-Item -NewName { $_.Name -replace '.txt','.log' }
Get-ChildItem *.log

Directory: C:\temp\files

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/3/2019   7:47 AM           2918 Friday.log
-a----        10/3/2019   7:46 AM           2918 Monday.log
-a----        10/3/2019   7:47 AM           2918 Wednesday.log
```

其中，`Get-ChildItem`  可以获取当前文件夹中所有扩展名为 .txt 的文件，然后将它们通过管道传递给 `Rename-Item`。NewName 的值是一个脚本块，在将值提交给 NewName 参数之前运行。在脚本块中，`$ _` 自动变量代表每个文件对象，它们通过管道传递给命令。脚本块使用 `-replace` 运算符将每个文件的文件扩展名替换为 `.log`。请注意，使用 -replace 运算符进行的匹配不区分大小写。

注意，`Get-ChildItem` 和 `Rname-Item` 都有别名

```powershell
名称：Get-ChildItem
别名
    gci
    ls
    dir
    
名称：Rename-Item
别名
    rni
    ren
```

以最常用的电影名重命名为例，一个命令使用的例子为

```powershell
> ls | ren -NewName { $_.Name -replace '\[电影天堂www.dytt89.com\]',''}
```

最后，在当前文件夹打开 powershell 的操作为，按住 shift 右键鼠标，此时打开的右键菜单中会多出一个 `在此处打开powershell` 的选项。