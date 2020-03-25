---
title: hugo搭建个人博客4-部署优化
date: 2019-09-29
lastmod: 2020-02-12
tags: [Hugo]
categories: [爱编程爱技术的孩子]

slug: Hugo build blog Deployment optimization
featured_image: https://s2.ax1x.com/2020/02/12/1HVXJe.jpg
typora-root-url: ..\..\..\static
---

## 1. 使用CI持续集成

为了存放博客网站源码和博文，一直使用的都是blog仓库，但这同时导致了博客的域名只能是`shuzang.github.io/blog`，不符合习惯，因为大家习惯性的都会输入`username.github.io`来寻找每个人的个人博客。

通过Travis CI做博客的持续集成，可以每次自动渲染生成新的博客网页并推送到`shuzang.github.io`仓库，从而实现调整域名的目的。但之前尝试了多次持续集成都没有成功，还把博客搞坏了，最终只能重建网站。这一次发现了之前存在的问题，终于成功了。

### 1.1 一些说明

有很多人使用一个仓库`username.github.io`来完成整个流程，将源码存放在`blog`分支中，然后开启持续集成将网页文件推送到`master`分支。但我用的是另一种方式，建立两个仓库，`blog`存放源码，`username.github.io`展示网站，这种方式用的人少，但有以下优点：

- 结构清晰。两个仓库各司其职（其实主要是使用一个仓库出错，解决不了）
- 源码存放在master分支下，clone和提交更容易

下面就是开启持续集成的整个流程了

### 1.2 申请Token

在GitHub 上申请一个新的 [personal access token](https://github.com/settings/tokens/new)。

`Token description`随便填，只要之后查看的时候知道是博客的就行。勾选所有`repo`列表项目，其它项目不要选。点击`Generate token`生成Token。

![申请Token](/images/hugo搭建个人博客4-部署优化/1HVOiD.png)

之后跳转的页面会显示Token的值，一定要记下来，因为离开这个页面之后这个值就再也无法查看。我因为已经做过一次了，这里就只查看一下。

![blog token](/images/hugo搭建个人博客4-部署优化/1HVjRH.png)

### 1.3 设置Travis CI

 [Travis CI](https://travis-ci.org/account/repositories)是一个持续集成的工具，使用GitHub账号登陆，然后开启`blog`仓库，选择`setting`。

![开启blog仓库持续集成](/images/hugo搭建个人博客4-部署优化/1HVbdK.png)

在设置页面填写**Environment Variables**。

- **Name** 填写： `GITHUB_TOKEN`
- **Value** 填写：刚刚在 GitHub 申请到的 Token 的值

![填写环境变量](/images/hugo搭建个人博客4-部署优化/1HVhRJ.png)

填写完成后点击`Add`添加

### 1.4 编写 .travis.yml

在博客根目录下创建并编辑`.travis.yml`文件，该文件的作用是告诉Travis CI如何部署博客

```bash
$ ls -al blog
total 30
drwxr-xr-x 1 书葬 197610    0 9月  29 20:28 ./
drwxr-xr-x 1 书葬 197610    0 9月  28 18:20 ../
drwxr-xr-x 1 书葬 197610    0 9月  29 20:28 .git/
-rw-r--r-- 1 书葬 197610  101 9月  28 18:21 .gitmodules
-rw-r--r-- 1 书葬 197610 1810 9月  29 20:10 .travis.yml
drwxr-xr-x 1 书葬 197610    0 9月  28 18:21 archetypes/
-rw-r--r-- 1 书葬 197610 4946 9月  29 18:27 config.toml
drwxr-xr-x 1 书葬 197610    0 9月  28 18:24 content/
drwxr-xr-x 1 书葬 197610    0 9月  29 18:56 docs/
-rw-r--r-- 1 书葬 197610  456 9月  29 20:28 README.md
drwxr-xr-x 1 书葬 197610    0 9月  28 18:21 resources/
drwxr-xr-x 1 书葬 197610    0 9月  28 18:21 static/
drwxr-xr-x 1 书葬 197610    0 9月  28 18:21 themes/
```

文件内容如下

```yaml
language: go

go:
  - "1.12"  # 指定Golang 1.12

dist: bionic  # Ubuntu 18.04

env:
 global:
   # Github Pages
   - GH_REF: github.com/shuzang/shuzang.github.io

# Specify which branches to build using a safelist
# 分支白名单限制：只有 master 分支的提交才会触发构建
# branches:
#   only:
#     - master

before_install:
  # 安装依赖
  #  - wget -q -O libstdc++6.deb http://security.ubuntu.com/ubuntu/pool/main/g/gcc-5/libstdc++6_5.4.0-6ubuntu1~16.04.10_amd64.deb
  #  - sudo dpkg --force-all -i libstdc++6.deb
  # 删除docs文件夹
  - rm -rf ./docs
  # 安装 hugo （version: v0.58.0）
  - wget -q -O hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.58.3/hugo_extended_0.58.3_Linux-64bit.deb
  - sudo dpkg -i hugo.deb

install:
# 运行hugo命令
  - hugo

script:
  - cd ./docs
  - git init
  - git config user.name "shuzang"
  - git config user.email "lylw1996@qq.com"
  - git add .
  - git commit -m "Update Blog By TravisCI With Build $TRAVIS_BUILD_NUMBER"
  # Github Pages
  - git push --force --quiet "https://$GITHUB_TOKEN@${GH_REF}" master:master

# before_deploy:
#   # 防止其与 mogeko.github.io 冲突
#   - rm -f CNAME

# # 备份
# deploy:
#   provider: pages # 重要，指定这是一份github pages的部署配置
#   skip-cleanup: true # 重要，不能省略
#   local-dir: public # 静态站点文件所在目录
#   # target-branch: master # 要将静态站点文件发布到哪个分支
#   github-token: $GITHUB_TOKEN # 重要，$GITHUB_TOKEN是变量，需要在GitHub上申请、再到配置到Travis
#   # fqdn: blog.yuantops.com # 如果是自定义域名，此处要填
#   keep-history: true # 是否保持target-branch分支的提交记录
#   on:
#     branch: master # 博客源码的分支
```

大部分都有注释说明，这里要注意的两点是

1. 在运行`hugo`命令前要先删除`docs`文件夹，是为了清除历史网页文件，这是Hugo官方建议的
2. 注释掉了常规github pages进行持续集成时使用的`deploy`部分，因为我们只需要单纯的把网页文件提交到另一个仓库。只有当普通的项目启用github pages时需要这部分，这里的博客部署不需要，开启`deploy`部分反而会出现错误，目前不知道原因。

### 1.5 使用说明

大部分工作都通过Travis CI自动进行了，以后提交只需要在改动之后直接`git push`到远程仓库，将会触发自动构建，只需要一两分钟，就可以在[Travis CI](https://travis-ci.org/) 上查看部署情况。

<font color="green">绿色</font> 代表部署成功  <font color="yellow">黄色</font> 代表正在部署  <font color="red">红色</font> 代表部署失败  <font color="gray">灰色</font> 代表部署被取消

![持续集成通过](/images/hugo搭建个人博客4-部署优化/1HVos1.png)

然后访问[博客首页](https://shuzang.github.io)，不出意外就可以看到新的改动了。如果部署失败，在网页端的日志记录中找到失败原因，然后修改代码重新提交即可，新的提交通过后，原先失败的提交将会被解决。

## 2 使用Github Action持续集成

[GitHub Actions](https://github.com/features/actions) 是 GitHub 在2018年10月推出的一个[持续集成服务](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)，之前一直是试用阶段，去年(2019年)年末刚刚开放，据说比[Travis CI](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html) 更简单更好用，所以打算把持续集成工具切换到它。同时，之前博客部署使用了两个仓库，一个放源码，一个放生成的网页文件，目前来看可以统一成一个。本篇文章就打算做这两件事。

Github Actions入门可以阅读[官方文档](https://help.github.com/en/actions/automating-your-workflow-with-github-actions)或者阮一峰大神的[GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

### 2.1 新建分支保存源码

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

![设置默认分支](/images/hugo搭建个人博客4-部署优化/1HePXR.png)

### 2.2 设置持续集成

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

![Github Action 日志文件](/images/hugo搭建个人博客4-部署优化/1HeehD.png)

等到workflow运行结束，访问博客页面，就可以看到更新成功了。切换到master分支，也可以看到推送的网页文件，不过因为设置了默认分支为blog，以后打开网页端该仓库，以及在本地clone的时候，默认都是blog分支。

## 3. 添加独立域名

给部署在 GitHub Pages 上的博客添加独立域名，是早就想做的事，以前是因为没什么需求，.me域名比较贵，国内购买域名备案又比较麻烦，所以一直没做。这次选择了.top域名，买了三年才67，可以了，三年以后开始工作再考虑换.me域名，毕竟这个冷门的字段应该没人要。而且发现部署在github pages不需要备案，因为服务器不在国内。

### 3.1 选购域名

可能很多人已经买好了域名了，你可以[跳过这部分](#GitHub 上的设置)

要想给博客绑定独立域名，首先你得拥有一个域名。得去域名注册网站购买域名，国内的域名商推荐[万网](https://wanwang.aliyun.com/)，国外的推荐 [GoDaddy](https://www.godaddy.com/)，因为他们分别是中国最大和世界最大的域名注册服务商。不了解的领域当然挑知名有信誉的总是没错的。

要是买`.me`域名就真的只能在GoDaddy买了，首年24，续费不知道，但是还需要购买隐私保护，这样就变成了每年60，要是工作后还负担的起，现在买这个还是算了。`.io`首年就320，更负担不起了。国内的话，`.com`，`.net`这些就不想了，基本都已经被注册了，于是选择了`.top`，一口气买了三年，一共67，而且在国内购买的好处是本身对WHOIS中的个人信息提供隐私保护。阿里云之前对域名提供的隐私保护服务已经停用了，主要是因为

> 为落实[ICANN临时规范](https://www.icann.org/news/announcement-2018-05-22-zh)要求，自2018年5月25日起阿里云WHOIS查询公开信息中将不再显示域名注册人、管理联系人和技术联系人的个人数据，包括姓名、邮箱、电话、街道地址等。

这样域名的注册信息已得到默认保护，不再需要额外开通隐私保护。

具体选购过程大致是：**搜索域名 -> 将喜欢的域名加入购物车 ->选择域名注册模板(没有的话要创建)-> 付款 -> 购买成功**，实际使用还需要进行邮箱验证和实名认证，否则域名无法启用。

最为繁琐的域名备案过程可以跳过，因为博客挂载在github pages，而只有服务器在国内的网站才需要备案。

### 3.2 DNS解析

使用CNAME别名映射域名，比设置A记录更方便，最重要的是A记录无法开启https。参数设置如下图所示。

![CNAME设置](/images/hugo搭建个人博客4-部署优化/1Heu1H.png)

访问网址时可能会加www前缀，因此可以设置一个二级域名解析，方法相同。

### 3.3 GitHub 上的设置

到 Github `shuzang.github.io`仓库设置里，在 `Custom domain` 这里填写`shuzang.top`域名并保存。

![github域名设置](/images/hugo搭建个人博客4-部署优化/1He3HP.png)

 `Custom domain` 下方 `Enforce HTTPS` 这个选项一并勾选，Github 跟 Let’s Encrypt 有合作，如果勾选了这个选项，Let’s Encrypt 就会给你的博客签发一张 SSL 证书，免费的。

![启用HTTPS](/images/hugo搭建个人博客4-部署优化/1HeJN8.png)



发现上面这种方法每次提交后都需要重新填写`custom domin`字段，因此采用另一种方式，创建一个名为 `CNAME` 的文件放在`content`目录，其中的内容**只**写上你的域名，像这样

```bash
shuzang.top
```

使用hugo命令生成文件会将CNAME文件直接复制到public目录，并通过持续集成将其推送到master分支根目录。

等几分钟 (刷新 DNS 缓存)，然后在浏览器中输入`shuzang.top`，回车，不出意外看到了自己的博客。

### 3.4 其他玩法

除了将域名绑定给博客外博客，还可以用域名干一些别的事。

比如，**使用 A 记录将 `mail.shuzang.me` 这个二级域名指向 `207.46.149.80` 就可以 “搭建” 一个 [临时邮箱](http://mail.shuzang.top/)服务** (感谢 [萌咖 | MoeClub.org](https://moeclub.org/) 提供的服务器)

如果你还有一台拥有**公网 IP** 的服务器，可玩性就更高了！

如果有能力，你甚至可以拥有自己的[搜索引擎](https://nutch.apache.org/)

## 参考

[1] [Mogeko的博客](https://github.com/Mogeko/Blog)

[2] [为博客添加独立域名](https://mogeko.me/2019/048/)

