# win10效率优化8-自动化


win10 一些重复操作的自动化，主要利用 Powershell 和windows任务计划程序，运行环境为 Windows 10 和 PowerShell 7.1，低版本 PowerShell 可能有中文路径无法识别问题。

<!--more-->

## 照片备份

手机拍摄的照片会在每月末保存到电脑中，为了进行备份，我希望照片能将照片进行加密压缩并替换Onedrive中的旧版本。

考虑使用 7zip 的命令行工具 7z进行加密压缩，在每月初备份一次，创建名为 Backup-photos.ps1 的文件，写入如下命令

```powershell
if (Test-Path D:/OneDrive/图片/照片.7z) {
    Remove-Item D:/OneDrive/图片/照片.7z
}
7z a "D:/OneDrive/图片/照片.7z" "D:/照片/*" -r -mhe=on -p3BB82427
```

脚本编写完成后，右键开始菜单，选择「计算机管理」，进入任务计划程序，执行如下步骤

1. 创建名为「备份照片」的任务，填入任务描述；
2. 新建触发器，选择「按预定计划」，勾选「每月」，然后选择所有月份的第一天；
3. 新建操作，选择「启动程序」，在「程序和脚本」栏中写入「pwsh.exe」，「添加参数」中写入「-nop -w hidden -file "D:\OneDrive\应用\Powershell\Backup-photos.ps1"」。`-w hidden` 会让脚本执行时终端只闪烁出现一次立刻消失。注意，如果提示无法找到文件，那么输入 pwsh.exe 在C盘的完整路径；
4. 取消勾选条件中的「只有计算机使用交流电源时才启动任务」，在设置中勾选「允许按需运行任务」和「如果过了计划时间，立即启动任务」，使错过设定的时间时任务也可以运行。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200808_windows计划任务设置.png)

随后的几种场景的 windows 计划任务设置与之相似，周期任务一类不再重复介绍，如果出现定时任务和事件任务会介绍其不同。

## 电影重命名

从电影天堂下载的电影总是带有 [电影天堂www.dytt89.com] 的前缀，希望能够将这个前缀删除掉。

因为目前没有找到办法检测指定文件夹的文件变化，所以依然采用定期重命名的方式，脚本如下

```powershell
$curPath = "D:\电影"
Get-ChildItem $curPath | Rename-Item -NewName {$_.Name -replace '\[电影天堂www.dytt89.com\]',''}
```

其中，`Get-ChildItem`  可以获取当前文件夹中所有文件，然后将它们通过管道传递给 `Rename-Item`。NewName 的值是一个脚本块，在将值提交给 NewName 参数之前运行。在脚本块中，`$ _` 自动变量代表每个文件对象，它们通过管道传递给命令。脚本块使用 `-replace` 运算符将每个文件的前缀替换为空。请注意，使用 -replace 运算符进行的匹配不区分大小写。

## windows聚焦壁纸保存

每天换一次的windows聚焦的锁屏壁纸非常好看，希望能定期将这些壁纸保存到 Onedrive 的壁纸文件夹，主要使用 Powershell 的 Copy-Item 和 Rename-Item 命令，脚本如下

```powershell
$temPath = "C:\Users\shuzang\AppData\Local\Packages\Microsoft.Windows.ContentDeliveryManager_cw5n1h2txyewy\LocalState\Assets\*"
Get-ChildItem $temPath | Rename-Item -NewName {$_.BaseName + ".jpg"}
Copy-Item -Path $temPath -Destination "D:\OneDrive\图片\壁纸\wallpaper"
Get-ChildItem $temPath | Rename-Item -NewName {$_.Name -replace ".jpg",""}
```

## 参考文献

[1] 少数派，[Windows 本地自动化工具，任务计划程序应用举例](https://sspai.com/post/66129)，Accessed: 2021-05-07.

[2] 7-Zip manual，[a (Add) command](https://7zip.bugaco.com/7zip/MANUAL/cmdline/commands/add.htm)，Accessed：2021-05-07.

[3] Microsoft Doc，[Rename-Item (Microsoft.PowerShell.Management)](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/rename-item?view=powershell-7)，Accessed：2021-05-07.



---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2020/efficient-use-of-win10-7-automation/  

