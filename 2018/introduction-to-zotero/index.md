# 科研基础2-文献管理工具 Zotero 学习指南


在文献管理方面的需求包括：

1. 文献可以方便地导入工具中并提取准确丰富地文献信息
2. 工作和学习时，可以快速在工具中找到自己想要的文献
3. 在写作时，可以方便的导出工具中的文献

由于 Zotero 没有内置的 PDF 阅读工具，因此忽略阅读层面的需求，除此之外，最大的缺点是没有移动端的应用程序。下面分别就几个主要的方面探索使用技巧。

注：本文主要参考 [少数派-文献管理神器 Zotero 学习路径指南](https://sspai.com/post/56724)

## 1. 文献导入

Zotero 支持多种导入方式，包括：

- 通过 Zotero Connector 浏览器插件导入
- 输入文献对应的 ISBN、DOI、PMID 或 arXiv ID 来导入
- 复制 BibTex 信息从剪贴板导入
- 从文件（BibTeX, RIS, Zotero RDF等）导入
- 将下载好的PDF文件直接拖入软件，然后自动提取文件中的元数据
- 自行添加

经过实践，**通过输入  DOI 或 arXiv ID 导入的文献信息最为丰富**，因此有条件的情况下应采用这种方式。

### 1.1 通过标识符添加

单击 Zotero 窗格中间列顶部的「通过标识符添加条目」按钮，输入标识符后按 Enter。如果要一次输入多个标识符，在输入第一个标识符后按 Shift + Enter 可以进入多行模式，随后输入其余标识符（每行一个），并通过 Enter 换行。输入完成后，再按 Shift + Enter 即可一次导入所有文献，示例如下图，是否带有「DOI:」或「arXiv:」前缀不影响导入效果，下图中第一条和第三条是 DOI，中间一条是arXiv ID。

![通过标识符导入文献](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8Mv34O.png)

### 1.2 通过 feed 进行订阅

文献需要及时跟踪，通常是通过邮件订阅或者RSS，Zotero 本身提供了一种简单的订阅方法，相比于前两种，这种方式更受我喜爱，也是现在主要使用的方式。以 IEEE Internet of Things Journal 期刊为例，在 [期刊主页](https://ieeexplore.ieee.org/xpl/mostRecentIssue.jsp?punumber=6488907) 获取订阅链接（和RSS订阅的链接是一样的），然后在 Zotero 中点击左上角的「新建文献库」按钮，选择「新建订阅」，然后选择「来自URL」

![新建订阅](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8Mvmv9.png)

在弹出的「订阅设置」对话框中填入刚刚复制的链接（URL），稍等一会儿就会自动获取到标题，最后在「高级选项」中设置更新频率等信息。

![订阅设置](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvBUf.png)

保存之后稍等一会儿就可以看到订阅的情况，之后会按照我们设置的时间每24小时更新一次，也可以在右键菜单中主动更新。订阅的论文并不在我们的文献库中，需要手动添加。

![订阅成功](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvD58.png)

## 2. 文献和笔记管理

文献管理的目的在于需要时快速而准确的找到所需论文，笔记则提供记录思维的工具。

### 2.1 分类与标签

分类指的是多个层级的文件夹，这里我的命名规则是不同时期的研究方向，虽然总的划分是区块链，但在不同时期，可能关心不同的领域，比如 access control 或 anomaly detection。有些论文可能从属于不同的分类，可以在一个分类中添加后拖动到另一个分类中，这里的拖动产生的是复制操作，不是剪切，而且，两者指向同一个 论文条目和 PDF 文件，不会产生存储冗余。因此，在删除一篇论文时，可以选择「从分类中移除条目」或「删除条目」，前者只会从当前分类移除论文的链接，如果其它分类中也有该文献，不会受到影响，如果其它分类中没有该文献，文献被移动到「未分类条目」。

![多个层级的文件夹](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8Mv0VP.png)

除了分类外，还可以为每个文献添加若干标签。为了避免使用维度的重合，标签体系中无必要情况不应再按照研究的领域进行命名，若有必要，先考虑使用「子分类」的方式。我对标签的使用是在应用维度，首先，新加入的文献放入它所属分类中，然后添加「未读」标签，这是因为很多时候添加文献都是批量添加的，然后才慢慢看，等到看完后，删除「未读」标签，添加重要性标签。使用「P1、P2 和 P3」对文献的重要性进行分级，P1 最重要，P3 最不重要，重要性的划分依据这篇论文的热度、引用数和自己感觉，有些论文是领域内公认权威的，引用数爆表，比如每个领域的起始文献，有些论文是自己看完感觉有很多想法可以实践，或者感觉思路很好有借鉴意义，可以添加「P1」标签。最后一个是写作时的分类，将文献划分为「中期」「毕业」「小论文」等几个使用场景，然后添加这些标签。

每个标签可以设置对应的颜色，这个色块会自动的显示在每个条目的开头，方便查看，Zotero 中只提供了9中颜色待选，不过已经足够了，通常我们使用的不超过3个。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvlE6.png)

### 2.2 内容寻找

基本的查找思路依然是分类和标签，对于最后得到的结果，可以按照文献相关信息进行排序，比如标题、创建者或者文献类型等。最后，在此基础上，Zotero 还支持对结果进行二次排序。

![指定二次排序](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvuuR.png)

另外一种查找方法是快速搜索，搜索内容包括标题、创建者等文献信息和标签，在 PDF 文档已建立的情况下，选择「所有内容」甚至可以搜索到文档的文本。

![快速搜索](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8Mvtvd.png)

另外，Zotero 的高级搜索还提供和 Web of Science 等文献数据库类似的功能，可以通过文献信息和各种逻辑运算符来控制筛选条件。单击主界面上方的放大镜图标或者在工具栏的「编辑」菜单中可以打开高级搜索窗口。

![高级搜索](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvaDI.png)

最后，不同的文献之前、文献和笔记之间都可以进行关联，我们可以将一些相关的文献关联到一起。

### 2.3 笔记

每一篇文献支持插入多个笔记，默认的笔记是富文本格式，但由于 Zotero 的插件和 Chrome 或 Firefox 等浏览器的插件格式相同，只需要将浏览器的 [markdown-here](https://github.com/adam-p/markdown-here) 插件重新打包然后导入 Zotero[^1]，即可支持 Markdown 语法。

[^1]:[Zotero导入Markdown here插件](https://www.cnblogs.com/Jay-CFD/p/10968876.html)

![markdown 转换](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvWbq.png)

## 3. 文献导出

在书写论文的过程中插入参考文献是最重要的需求之一，也是文献管理软件存在的意义之一，很多人都是因为这个需求才开始使用文献管理软件。

### 3.1 平时使用

指的是在自己的博客或者笔记中插入引用，因为不需要遵守严格的引用需求，可以使用直接拖拽的方式，单篇或多篇文献都可以，非常方便。

![多篇拖拽引用效果](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8Mvdbt-1585143274529.gif)

### 3.2 在 Microsoft Word 中编辑论文

Zotero 提供相关的加载项供使用，通常在软件安装时就会提示自动安装，也可以自己到「首选项—>引用—>文字处理软件」中进行安装。

![安装 Word 加载项](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8Mv62Q.png)

安装好后打开 Word 可以看到该插件

![Word 加载项](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvRrn.png)

各按钮功能介绍如下

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8Mv2Ks.png)

在正式插入引文之间，需要设置引文的格式，通过管理样式可以轻松找到海量的引文格式。

![引文格式设置](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvKD1.png)

在需要添加引文的位置点击「Add/Edit Citation」按钮，调出引文搜索

![调出引文搜索](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvGCD.png)

在出现的搜索框中输入一个标题或作者等搜索特定的参考文献

![选择文献](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvegJ.png)

一旦选中，点击气泡或按下「Ctrl + ↓」添加页码、前缀或后缀

![添加页码、前缀或后缀](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvJ8e.png)

可以一次选择多篇文献，最后按「Enter」即可将文献引用添加到文档中，不过如下图所示，此时添加的只是下标

![加入引用](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvUKA.png)

添加新行并移动光标到新行末尾，点击「Add/Edit Bibliography」将参考书目加入文档。

![最终添加结果](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvMHx.png)

添加新的引文或文献编辑完成后可以点击刷新更新所有引文和参考书目

### 3.3 在 Latex 中编辑论文

此时的核心是导出 BibTex 文件，由于原始的导出功能不够完善，导出的字段也无法自定义，因此需要安装 [Better BibTex](https://github.com/retorquere/zotero-better-bibtex)插件。

导出时，选择所有需要的文献，在右键菜单中选择「导出条目」，然后选择「Better BibTex」格式，点击OK即可导出 bib 文件。

## 4. 其它事项

主要指同步和协作、插件系统和Zotero的其它高级功能。

### 4.1 备份与协作

Zotero 本身只提供 300M 免费的存储空间，这些空间存储 PDF 文档完全不够，很多文章包括官方都推荐使用 WebDAV 服务，但国内支持的好像只有「坚果云」。因为我的主力软件是 OneDrive，所以这里介绍 Zotero 如何与 OneDrive 配合来存储PDF文件。

![Zotero的数据备份](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8Mvcvj.png)



Zotero 的数据和文件的同步是分离的，数据指的是文献的分类、标签和各种元数据信息，文件指的是PDF和笔记等各种附件，这就给了我们机会[^2]。如上图所示，在「同步」设置页面，对数据同步的选项全部勾选，而文件同步的选项全部取消勾选。

[^2]:[利用ONEDRIVE同步ZOTERO数据文件夹的方法](https://www.junjienotes.com/tips/利用onedrive同步zotero数据文件夹的方法/)

然后切换到「高级—>文件和文件夹」选项卡，点击打开数据文件夹，获取数据文件的存储路径。

![打开数据文件夹](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvsPS.png)

所有的附件（包括PDF和笔记）都存储在数据文件夹的`storage`目录下。我们可以采用软链接[^3]的方式来管理这些文件，建立软链接后，原数据文件夹下的`storage`文件夹成为一个快捷方式，指向OneDrive存储空间的实际文件，因此不会占用大量存储空间。在命令行中使用如下命令建立原始路径和新路径的软链接，记得执行命令前删除原 storage 文件夹，否则会报错。

[^3]:[windows中的软链接硬链接等](https://www.cnblogs.com/Akkuman/p/9688311.html)

```bash
mklink /j C:\Users\lylw1\Zotero\storage F:\OneDrive\课题\论文库
```

- `/j`参数以绝对路径的方式创建软链接，将所有源文件夹文件移动到新文件夹后，清空源文件夹，这样可以保证旧的附近不再占用存储空间
- `C:\Users\lylw1\Zotero\storage`是所有文件的原始路径
- `F:\OneDrive\课题\论文库`是所有文件的新路径，位于 OndDrive 存储库中，新的文件夹名不必和原来的`storage`相同，可以任意命名。

如果想迁移到新的位置，按如下步骤执行

1. 删除 C:\Users\lylw1\Zotero\storage 中的源文件夹
2. 使用上述命令建立指向新位置的软链接
3. 将 F:\OneDrive\课题\论文库 中的文件复制到新的目标文件夹中

如果想要对软链接的目标文件夹更名，可以采用和迁移相同的方式，需要注意的是，一定要注意目标文件夹内数据的完整性，如果被破坏，那么链接将不再起作用。最后，如果嫌弃这种方式比较麻烦，可以选择使用插件 [ZotFile](http://zotfile.com/)。

### 4.2 插件系统

插件系统是 Zotero 的一大优势，前面已经提到了几款插件，更多插件可以通过 [官方插件网站](https://www.zotero.org/support/plugins) 进行查看，或者自行在网上搜索安装。一些推荐的插件如下

- [Better BibTex](https://github.com/retorquere/zotero-better-bibtex)：Make Zotero useful for us LaTeX holdots.
- [Markdown Here](https://github.com/adam-p/markdown-here)：用 Markdown 书写笔记
- [Zotero DOI Manager](https://github.com/bwiernik/zotero-shortdoi)：Zotero plugin for auto-fetching and validating DOI and shortDOIs
- [papermachines](https://github.com/papermachines/papermachines)：文献可视化
- [Zotero Scihub](https://github.com/ethanwillis/zotero-scihub)：自动从 sci-hub 下载文献的 PDF
- [ZotFile](http://zotfile.com/)：管理 PDF 文件

### 4.3 时间轴

Zotero 有个时间轴功能，可以将所有文献反映在一条时间轴上，时间可以是文献的发表日期、添加日期和修改日期，时间轴的跨度单位可以是日、月、年、十年、百年等，不过这个功能不常用，目前没有想到这个功能的意义。

![时间轴功能](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20181111_8MvYgH.png)

### 4.4 小技巧

- 当选择了一个项目时，可以通过按住 Option(Ctrl) 键突出显示包含此项目的所有集合，也就是知道这篇文献所在的分组
- 在集合列表或项目列表中的键盘上按 +（加号）可以展开所有节点，按 -（减号）则可以折叠
- 要查看所选库或集合中的项目数可以单击然后使用 Command-A(Ctrl-A) 全选，计数将显示在右侧
- 若使用快速复制功能，在将项目拖放到文本文档时按住 Shift 键能实现插入引文而不是完整引用
- 可以单击详细信息中的 DOI 和 URL 字段标签直接打开链接

## 5. 最后

主流的文献管理工具还有

- [Endnote](https://endnote.com/) ：老牌且知名的文献管理工具，付费价格相对较贵但多数高校和科研机构会统一购买，移动端（iPad）使用体验优秀。
- [Mendeley](https://www.mendeley.com/) ：2013 年以开源软件身份被 Elsevier 高价收购，背靠大树。

不过很多人推荐[Papers](https://www.papersapp.com/)，除了基本的文献和笔记管理外，还有界面美观和内置PDF阅读器两个优点，说实话很心动，但是要付费订阅，如果走科研道路就算了，可惜并没有打算读博。


