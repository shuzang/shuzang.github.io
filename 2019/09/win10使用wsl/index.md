# win10使用WSL


因为只是想熟悉一下Linux基本操作，每次都打开虚拟机未免有些麻烦，于是决定使用win10上的WSL(Windows Subsystem for Linux, windows下的Linux子系统)。

### 启用WSL

以管理员身份打开PowerShell并运行

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

 或者在`启用或关闭windows功能`直接勾选`适用于Windows的Linux子系统`。

![选择WSL](/images/win10-使用WSL/64679858-28c6e700-d4af-11e9-9ade-ccd5b9a87395.png)

出现提示时重新启动计算机

### 选择Linux发行版本

要自己选一个Linux分发版，比较方便的是从win10商店安装，我选了用的顺手的Ubuntu18.04 LTS

![Ubuntu 18 04 for WSL](/images/win10-使用WSL/64679855-28c6e700-d4af-11e9-8509-f95adbf0f0ea.png)

### 初始化

安装完成后将Ubuntu WSL 固定到开始屏幕，方便启动。单击运行，将会打开一个控制台窗口，首次启动会使用几分钟来安装，之后Ubuntu WSL将解压缩并存在win10系统中，随时可以使用，之后启动将是即时的。

安装完成后，系统会提示创建新的用户账户和密码，这将是登录的默认用户，意味着下次打开WSL将不需要输入用户名密码而自动登录，而且，默认情况下它是sudo组的成员。更多说明参见[用户账户和权限](https://docs.microsoft.com/zh-cn/windows/wsl/user-support)

![安装好的WSL](/images/win10-使用WSL/64679857-28c6e700-d4af-11e9-8ad0-89566df73c22.png)

### 安装并使用WSL2

基于[WSL2](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-index)更好的性能，我们决定将Ubuntu18.04作为WSL2运行。( 每个 Linux 发行版都可以作为 WSL 1 或 WSL 2 发行版运行, 并可随时进行切换)

1. 启用`虚拟机平台`可选组件

    以管理员身份打开PowerShell并运行下列命令，或者或者在`启用或关闭windows功能`中直接勾选

   ```bash
   Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
   ```

    启用更改后重启计算机。
   
2. 使用命令行设置由WSL2支持

    在PowerShell中运行下列命令，我直接将WSL2设置成了默认体系结构

   ```powershell
   wsl --set-default-version 2
   ```



命令运行不成功，发现WSL2需由18917或更高版本，现在我只是18362，不想启用预览版，算了，以后再说。

(09.12)看起来升到18917也不行，开启`虚拟机平台`和VMware本身有冲突，两者只能选一个，只能把这个功能关掉了。

### 与win10互传文件

安装完成后，wsl的Ubuntu系统home目录与win10的以下目录对应

```bash
C:\Users\lylw1\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs\home\shuzang
```

路径中的用户名更换为自己的用户名即可，测试一下，将win10系统桌面的`智能合约审计.md`文件复制到该目录下，在wsl中查看

```bash
$ cd ~
$ ls
test  win10  智能合约审计.md
```

但如此复杂的路径访问多有不便，可以利用软链接功能将win10下某个指定文件夹链接到系统路径，如下，在`C:\Users\lylw1`目录下新建`wsl`目录，使用如下命令将该目录与wsl home目录下的win10文件夹建立链接

```bash
$ ln -s /mnt/c/Users/lylw1/wsl ~/win10 
```

复制一些文件到win10的`C:\Users\lylw1\wsl`目录，在wsl home目录下直接进入win10文件夹，可看到这些文件

```bash
$ cd ~/win10
$ pwd
/home/shuzang/win10
$ ls
NewACC.sol  NewJC.sol  NewRC.sol
```

当需要调整文件夹时，可以删除软链接并重新建立新的链接，删除当前软链接的命令为

```bash
$ sudo rm -rf ~/win10
```

### 备份及还原系统

在 powershell 或 其它 terminal 下备份和还原，注意要停止 WSL 再操作，备份系统的命令为

```bash
$ wsl --export Ubuntu c:\temp\Ubuntu.tar
```

Ubuntu 为要导出的 WSL 分发版名称

还原子系统的命令为

```bash
$ wsl --import Ubuntu c:\WSL c:\temp\Ubuntu.tar
```

c:\WSL为安装位置

#### 参考

[1] [适用于 Linux 的 Windows 子系统文档](https://docs.microsoft.com/zh-cn/windows/wsl/about)
