这是 shuzang 的个人博客仓库，使用 [Hugo](https://gohugo.io/) 生成，主题使用 [LoveIt](https://github.com/dillonzq/LoveIt) （做了一定修改），源码和文章放在 `blog` 分支下，生成的页面利用 Github Action 推送到 master 分支，通过 Github Pages 展示页面。

你可以通过以下步骤快速使用该仓库构建自己的博客

1. Fork 这个仓库；

2. 将 Fork 后的仓库改名为 name.github.io，其中 name 为你的 Github 账户名；

3. 在仓库的 Settings 中开启 Github Pages, 在 source 中将 branch 选择为 master；

4. git clone 改名后的仓库到本地，修改 config.toml 配置文件第一行的 baseURL，替换自己的账户名，配置文件中的其它字段可以根据自己情况调整；

5. 进入 content/posts 目录，删除原有文章，添加自己的文章，注意，每篇文章都需要一些固定格式的头部元数据，下面是一个例子；

   ```toml
   title: 文章题目
   date: 2019-02-17
   tags: [科研记录, 杂谈]
   categories: [研究生的区块链学习之路]
   ```

6. 以正常的 git commit 方式提交修改，几分钟后即可利用第三步中修改后的 baseURL 在浏览器中查看自己的博客。

我的博客地址为：[https://shuzang.github.io/](https://shuzang.github.io/)，你可以从此链接查看博客最终的效果。

