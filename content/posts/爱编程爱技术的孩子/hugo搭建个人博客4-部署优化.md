---
title: hugo搭建个人博客4-部署优化
date: 2019-09-29
lastmod: 2019-11-17
tags: [博客写作]
categories: [爱编程爱技术的孩子]

slug: Hugo build blog Deployment optimization
featured_image: https://user-images.githubusercontent.com/26682846/71819350-a0f09280-30c6-11ea-8166-a7068290223f.jpg
---

## 1. 持续集成

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

![申请Token](https://user-images.githubusercontent.com/26682846/65833196-f9620800-e2ff-11e9-923c-5ab7726e1b4d.png)

之后跳转的页面会显示Token的值，一定要记下来，因为离开这个页面之后这个值就再也无法查看。我因为已经做过一次了，这里就只查看一下。

![blog token](https://user-images.githubusercontent.com/26682846/65833191-f8c97180-e2ff-11e9-83a5-d21174067cff.png)

### 1.3 设置Travis CI

 [Travis CI](https://travis-ci.org/account/repositories)是一个持续集成的工具，使用GitHub账号登陆，然后开启`blog`仓库，选择`setting`。

![开启blog仓库持续集成](https://user-images.githubusercontent.com/26682846/65833195-f9620800-e2ff-11e9-8009-b9e901addc64.png)

在设置页面填写**Environment Variables**。

- **Name** 填写： `GITHUB_TOKEN`
- **Value** 填写：刚刚在 GitHub 申请到的 Token 的值

![填写环境变量](https://user-images.githubusercontent.com/26682846/65833197-f9fa9e80-e2ff-11e9-80eb-84318c6447a8.png)

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

![持续集成通过](https://user-images.githubusercontent.com/26682846/65833193-f8c97180-e2ff-11e9-8cb0-d3659fd2b0c7.png)

然后访问[博客首页](https://shuzang.github.io)，不出意外就可以看到新的改动了。如果部署失败，在网页端的日志记录中找到失败原因，然后修改代码重新提交即可，新的提交通过后，原先失败的提交将会被解决。

## 2. 添加独立域名

给部署在 GitHub Pages 上的博客添加独立域名，是早就想做的事，以前是因为没什么需求，.me域名比较贵，国内购买域名备案又比较麻烦，所以一直没做。这次选择了.top域名，买了三年才67，可以了，三年以后开始工作再考虑换.me域名，毕竟这个冷门的字段应该没人要。而且发现部署在github pages不需要备案，因为服务器不在国内。

### 2.1 选购域名

可能很多人已经买好了域名了，你可以[跳过这部分](#GitHub 上的设置)

要想给博客绑定独立域名，首先你得拥有一个域名。得去域名注册网站购买域名，国内的域名商推荐[万网](https://wanwang.aliyun.com/)，国外的推荐 [GoDaddy](https://www.godaddy.com/)，因为他们分别是中国最大和世界最大的域名注册服务商。不了解的领域当然挑知名有信誉的总是没错的。

要是买`.me`域名就真的只能在GoDaddy买了，首年24，续费不知道，但是还需要购买隐私保护，这样就变成了每年60，要是工作后还负担的起，现在买这个还是算了。`.io`首年就320，更负担不起了。国内的话，`.com`，`.net`这些就不想了，基本都已经被注册了，于是选择了`.top`，一口气买了三年，一共67，而且在国内购买的好处是本身对WHOIS中的个人信息提供隐私保护。阿里云之前对域名提供的隐私保护服务已经停用了，主要是因为

> 为落实[ICANN临时规范](https://www.icann.org/news/announcement-2018-05-22-zh)要求，自2018年5月25日起阿里云WHOIS查询公开信息中将不再显示域名注册人、管理联系人和技术联系人的个人数据，包括姓名、邮箱、电话、街道地址等。

这样域名的注册信息已得到默认保护，不再需要额外开通隐私保护。

具体选购过程大致是：**搜索域名 -> 将喜欢的域名加入购物车 ->选择域名注册模板(没有的话要创建)-> 付款 -> 购买成功**，实际使用还需要进行邮箱验证和实名认证，否则域名无法启用。

最为繁琐的域名备案过程可以跳过，因为博客挂载在github pages，而只有服务器在国内的网站才需要备案。

### 2.2 GitHub 上的设置

购买好域名后，首先到 Github 上，你部署博客的那个 Git 仓库的设置里，在 `Custom domain` 这里填上你购买的域名

![github域名设置](https://user-images.githubusercontent.com/26682846/65857582-b56e1200-e396-11e9-8449-52e5d2378683.png)

或者在创建一个名为 `CNAME` 的文件放在根目录，其中的内容**只**写上你的域名，像这样

```bash
example.com
```

如果使用了 Travis CI 这类持续集成服务来部署博客的话推荐使用第二种方式进行设置。

如果 `Custom domain` 下方有 `Enforce HTTPS` 这个选项的话一并勾选上

![](https://mogeko.github.io/blog-images/r/048/gh_setting_HTTPS.png)

Github 跟 Let’s Encrypt 有合作，如果勾选了这个选项，Let’s Encrypt 就会给你的博客签发一张 SSL 证书，免费的。

### 2.3 DNS 上的设置

终于到了最关键的一步了。

**现在要做的是让域名指向正确的 IP 地址，GitHub 为此提供了四条 IP，使用 [A 记录](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F#%E8%AE%B0%E5%BD%95%E7%B1%BB%E5%9E%8B) 指向这四条 IP 地址就可以了**

这四条 IP 分别是 (来自 [GitHub 的官方文档](https://help.github.com/en/articles/setting-up-an-apex-domain))：

> **185.199.108.153  185.199.109.153  185.199.110.153  185.199.111.153**

你需要到你购买域名的域名商的域名管理页面进行设置，虽然不同的域名商域名管理页面不同，不过原理都是相同的。

这里以 万网 为例，在域名控制台点击域名，选择左侧的**域名解析 -> 解析设置**

![域名解析记录](https://user-images.githubusercontent.com/26682846/65857585-b606a880-e396-11e9-817f-45aa26b479f9.png)

选择**添加记录**

![域名解析](https://user-images.githubusercontent.com/26682846/65857584-b606a880-e396-11e9-842c-1bf4233f133e.png)

> **类型 (Type)：A**  
> **主机 (Host)：@**  
> **指向 (Points to)：185.199.108.153**  
> **TTL：保持默认** @ 表示顶级域名，也就是你注册的域名本身  

以相同的方式配置剩下的三条 IP 地址

> **类型 (Type)：A**  
> **主机 (Host)：@**  
> **指向 (Points to)：185.199.109.153**  
> **TTL：保持默认**  

> **类型 (Type)：A**  
> **主机 (Host)：@**  
> **指向 (Points to)：185.199.110.153**  
> **TTL：保持默认**  

> **类型 (Type)：A**  
> **主机 (Host)：@**  
> **指向 (Points to)：185.199.111.153**  
> **TTL：保持默认**  

等几分钟 (刷新 DNS 缓存)，然后在浏览器中输入你的域名，回车；不出意外的话你应该可以看到你的博客了。

### 2.4 设置二级域名

除了通过顶级域名进行访问外你还可以设置二级域名，例如 `www.shuzang.top` `

只需要在**添加记录**时调整参数即可。

![二级域名解析](https://user-images.githubusercontent.com/26682846/65857583-b56e1200-e396-11e9-81da-544fd33784d9.png)

不过这次添加的类型 (Type) 不是 **A 记录**而是 **CNAME**

> **类型 (Type)：CNAME**  
> **主机 (Host)：www**  
> **指向 (Points to)：shuzang.top  
> **TTL：保持默认**  

此时，你不仅可以通过 `example.com` 访问你的博客，还可以通过 `www.example.com` 访问到你的博客。

### 2.5 其他玩法

除了将域名绑定给博客外博客，还可以用域名干一些别的事。

比如，**使用 A 记录将 `mail.shuzang.me` 这个二级域名指向 `207.46.149.80` 就可以 “搭建” 一个 [临时邮箱](http://mail.shuzang.top/)服务** (感谢 [萌咖 | MoeClub.org](https://moeclub.org/) 提供的服务器)

如果你还有一台拥有**公网 IP** 的服务器，可玩性就更高了！

如果有能力，你甚至可以拥有自己的[搜索引擎](https://nutch.apache.org/)

## 参考

[1] [Mogeko的博客](https://github.com/Mogeko/Blog)

[2] [为博客添加独立域名](https://mogeko.me/2019/048/)

