---
title: hugo搭建个人博客8-使用Github Action进行持续集成
date: 2020-02-05
tags: [博客写作]
categories: [爱编程爱技术的孩子]

slug: Continuous integration with Github Action
---

[GitHub Actions](https://github.com/features/actions) 是 GitHub 在2018年10月推出的一个[持续集成服务](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)，之前一直是试用阶段，去年(2019年)年末刚刚开放，据说比[Travis CI](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html) 更简单更好用，所以打算把持续集成工具切换到它。同时，之前博客部署使用了两个仓库，一个放源码，一个放生成的网页文件，目前来看可以统一成一个。本篇文章就打算做这两件事。

Github Actions入门可以阅读[官方文档](https://help.github.com/en/actions/automating-your-workflow-with-github-actions)或者阮一峰大神的[GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

## 1. 新建分支保存源码

统一使用`shuzang.github.io`这个仓库，`master`分支用于放置生成的博客网页文件，这是Github Pages的要求，新建分支`blog`存放博客源码和写的文章。

```bash
# 新建并切换到新分支
$ git checkout -b blog
$ git branch
* blog
  master
# 设置本地blog分支追踪远程blog分支
$ git branch --set-upstream blog origin/blog
# 查看分支跟踪关系
$ git branch -vv
* blog   c63526c [origin/blog] Update posts
  master aa725ba [origin/master: behind 1] Automated deployment: Wed Feb  5 09:25:59 UTC 2020 d8e4f52983c7d7b2128076df3b267bd27259d447
```

删除原来Travis CI用到的`.travis.yml`文件

```bash
$ rm .travis.yml
```

然后将本地blog分支的内容推送到远程，在网页端进入`shuzang.github.io`仓库的设置页面，将默认分支设置为blog分支。

![设置默认分支](https://user-images.githubusercontent.com/26682846/73832163-a1ee1e80-4842-11ea-9a22-fea818d84035.png)

## 2. 设置持续集成

生成公私钥用于持续集成

```bash
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f blog -N ""
# You will get 2 files in current file:
#   blog.pub (public key)
#   blog     (private key)
```

然后进入`shuzang.github.io`仓库设置页面，在`Deploy Keys`中添加公钥，在`Secrets`中添加私钥，私钥名设置为`ACTIONS_DEPLOY_KEY`

然后新建YAML配置文件，Github Action要求配置文件位于`.github/workflows`目录下，新建完成后目录结构如下

```bash
$ ls ./.github/workflows
main.yml
```

Github Action使用一种模块化的思路，即将很多持续集成的操作写成独立的脚本文件，放到代码仓库，让其它开发者是哦那个，因此进行持续集成时，可以直接引用别人写好的action，整个持续集成的过程，就是一个actions组合的过程。GitHub 做了一个[官方市场](https://github.com/marketplace?type=actions)，可以搜索到他人提交的 actions。另外，还有一个 [awesome actions](https://github.com/sdras/awesome-actions) 的仓库，也可以找到不少 action。

我们的基本思路如下

1. 整个流程在blog分支push时触发
2. 只有一个job，运行在ubuntu-18.04环境下
3. 使用[official action: checkout](https://github.com/actions/checkout)获取仓库源码，注意添加参数clone主题子模块
4. 使用[peaceiris/actions-hugo: GitHub Actions for Hugo](https://github.com/peaceiris/actions-hugo)部署hugo环境，注意使用`extentded`版本（主题要求）
5. 直接执行hugo命令
6. 使用[peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)将当前分支`public`目录下的内容部署到master分支，

完整的`main.yml`脚本内容如下

```yaml
name: hugo push to github pages

on:
  push:
    branches:
    - blog

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
    - uses: actions/checkout@v1
      with:
        submodules: true

    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2
      with:
        hugo-version: '0.59.1'
        extended: true

    - name: Build
      run: hugo --minify

    - name: Deploy
      uses: peaceiris/actions-gh-pages@v2
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        PUBLISH_BRANCH: master
        PUBLISH_DIR: ./public
```

保存上面的文件后，将本地仓库推送到远程，Github检测到`.github/workflow`目录和里面的`main.yml`文件，就会自动运行，在网页端可以查看运行日志，如果出现错误可以根据日志内容就行修改。

![Github Action 日志文件](https://user-images.githubusercontent.com/26682846/73834090-1080ab80-4846-11ea-965b-a5a69de687b7.png)

等到workflow运行结束，访问博客页面，就可以看到更新成功了。切换到master分支，也可以看到推送的网页文件，不过因为设置了默认分支为blog，以后打开网页端该仓库，以及在本地clone的时候，默认都是blog分支。