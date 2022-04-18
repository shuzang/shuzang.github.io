# win10效率优化7-终端和winget


Windows terminal 的安装配置和 winget 的基本使用，主要原因是 Powershell 不好看。

## 1. Windows terminal

界面友好的终端应用，凡是命令行程序都可以用它启动，可以将 win10 下的 cmd、Powershell 和 git bash 都集中在一起，当然，最重要的是好看。

基本上有什么问题都可以从 [官方教程](https://docs.microsoft.com/zh-cn/windows/terminal/) 里查到，这里只是介绍我的配置过程。

### 1.1 安装

我选择从 [Microsoft Store](https://aka.ms/terminal) 安装 Windows terminal。初始界面如下

![](https://docs.microsoft.com/zh-cn/windows/terminal/images/first-run.png)

### 1.2 快捷键启动

右键菜单已经有 git bash 了，不需要把它添加进去。

快捷键启动我没有采用网上流传的教程，而是把 Windows terminal 固定在了任务栏第一个，然后使用  `Win+1` 键打开。

### 1.3 添加 git bash

默认添加的应用包括 Powershell、cmd、Azure Cloud Shell 和我之前安装的 WSL（Ubuntu 20.04）。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20201007_Snipaste_2020-10-07_18-50-39.png)

这里将 git bash 添加进去，并设置为启动时默认使用的应用（之前启动 Windows termianl 默认打开 Powershell），并设置起始目录。最终效果如上图

1. 点击标题栏的下箭头，点击「设置」，可以在上图中看到；

2. 在打开的配置文件中 `list` 部分添加如下内容。其中，`guid` 是唯一标识符，注意不要和已有的四个应用相同即可；`name` 是应用名；`commandline` 设置 git bash 路径，根据自己的安装位置设置；`icon` 设置上图中显示的缩略图，自己从 [网上](https://gitforwindows.org/img/gwindows_logo.png) 下载放到合适的位置并设置；`startingDirectory` 设置起始目录，我这里设置了我最常打开的目录。 

   ```json
   {
       "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b9}",
       "name": "Git-bash",
       "commandline": "C:\\Program Files\\Git\\bin\\bash.exe",
       "icon": "C:\\Program Files\\Git\\gwindows_logo.png",
       "startingDirectory": "F:\\OneDrive\\博客"
   }
   ```

3. 修改全局字段（在配置文件的最前面几行中找）中的 `defaultProfile` 为 `Git-bash`，这里可以使用上面的 `name` 或 `guid` 字段

   ```json
   "defaultProfile": "Git-bash",
   ```

### 1.4 配色及背景图

大部分人用 Windows terminal 就是因为它可定制，所以出现了各种各样好看的配色和主题。

![我的终端最终形态](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20201007_Snipaste_2020-10-07_19-07-45.png)

我从 [主题网站](https://windowsterminalthemes.dev/) 选择了 Builtin Tango Light 主题，配置代码如下，需要将这段代码复制到配置文件的 `schemes` 字段中。

```json
{
    "name": "Builtin Tango Light",
    "black": "#000000",
    "red": "#cc0000",
    "green": "#4e9a06",
    "yellow": "#c4a000",
    "blue": "#3465a4",
    "purple": "#75507b",
    "cyan": "#06989a",
    "white": "#d3d7cf",
    "brightBlack": "#555753",
    "brightRed": "#ef2929",
    "brightGreen": "#8ae234",
    "brightYellow": "#fce94f",
    "brightBlue": "#729fcf",
    "brightPurple": "#ad7fa8",
    "brightCyan": "#34e2e2",
    "brightWhite": "#eeeeec",
    "background": "#ffffff",
    "foreground": "#000000"
}
```

然后在 `defaults` 字段中添加

```json
"colorScheme": "Builtin Tango Light"
```

为了不打扰正式内容的显示，我决定将背景图放在右下角，同时，由于终端主体颜色是白色，背景图除了主体的人或物，其它颜色也应该是白色，正好我有一张谏山黄泉的图是这样的。

由于要放在右下角，还要在图片左边和上边扩展大量的白色区域。我选择的方法是打开 Microsoft Whiteboard 应用，将背景颜色设置为白色，然后将图片放在右下角，导出此时的白板。

将导出的图片放在合适的位置，在配置文件的 `defaults` 字段中添加如下内容（和主体设置在一起），第一行设置背景图路径，第二行设置不透明度。此时打开终端发现图片显示可能有问题，无法正好在右下角显示整个人物，这时候通过裁剪背景图上方和左边的空白，不断调整和预览，可以获得想要的效果，最终效果就是本小节开头的图片。

```json
"backgroundImage": "F:\\OneDrive\\图片\\收藏\\谏山黄泉-背景图.png",
"backgroundImageOpacity": 0.8
```

注1：不少人喜欢下面这种标签式的效果，叫做 Powerline，但我没感觉，所以没添加，想使用可以参考 [Windows 终端的 PowerShell 主题中的 Powerline](https://docs.microsoft.com/zh-cn/windows/terminal/custom-terminal-gallery/powerline-in-powershell)

![](https://docs.microsoft.com/zh-cn/windows/terminal/images/powerline-powershell.png)

注2：官方提供了几种不错的主题，包括 [毛玻璃效果](https://docs.microsoft.com/zh-cn/windows/terminal/custom-terminal-gallery/frosted-glass-theme) 和 [Raspberry Ubuntu](https://docs.microsoft.com/zh-cn/windows/terminal/custom-terminal-gallery/raspberry-ubuntu) 我都很喜欢，效果如下

![毛玻璃](https://docs.microsoft.com/zh-cn/windows/terminal/images/frosted-glass-theme.png)

![Raspberry Ubuntu](https://docs.microsoft.com/zh-cn/windows/terminal/images/raspberry-ubuntu.png)

## 2. winget

win10 预览版用户直接就可以使用，是系统自带的，非预览版用户从 [github仓库](https://github.com/microsoft/winget-cli) 自行下载安装。使用说明可以参考 [官方文档](https://docs.microsoft.com/zh-cn/windows/package-manager/winget/)，安装后可以直接在终端使用

![winget使用](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20201007_Snipaste_2020-10-07_19-20-51.png)






