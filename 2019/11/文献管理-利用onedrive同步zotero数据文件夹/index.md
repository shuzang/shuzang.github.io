# 文献管理-利用Onedrive同步Zotero数据文件夹


文献管理总算起了个头，最终选择了开源软件Zotero，在尝试使用一段时间后打算确定下来，不过这里还有一个小问题需要解决，那就是同步的问题。Zotero提供的免费同步空间不够存放论文的PDF文件，打算使用onedrive来存储。方法是把Zotero的storage文件夹剪切到自己的OneDrive文件夹下，在它们之间建立软连接。

首先在Zotero工具栏选择`编辑—>首选项—>高级—>文件和文件夹`，点击`打开数据文件夹`。

将其中的`storage`文件夹剪切到onedrive文件夹下，可以重命名。然后在管理员权限的cmd下使用下列命令建立软链接。

```bash
mklink /j C:\Users\lylw1\Zotero\storage F:\OneDrive\课题\论文库
```

之后在`首选项`中选择`同步`，取消文件同步即可。

**参考**

1. [利用ONEDRIVE同步ZOTERO数据文件夹的方法](https://www.junjienotes.com/tips/%E5%88%A9%E7%94%A8onedrive%E5%90%8C%E6%AD%A5zotero%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6%E5%A4%B9%E7%9A%84%E6%96%B9%E6%B3%95/)
2. [windows中的软链接硬链接等](https://www.cnblogs.com/Akkuman/p/9688311.html)
