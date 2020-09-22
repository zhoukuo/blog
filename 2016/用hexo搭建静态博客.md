# 用HEXO搭建静态博客

zhoukuo@2016-07-31

一直纠结在哪里写博客，尝试过CSDN，开源中国，cnblogs，自己搭建过wordPress，但都不满意。
直到有一天，发现有人在github上搭建的个人博客，页面简洁、干净，于是到百度一通儿狂搜，一眼看中了hexo，这个逼格极高的程序猿写作方式，就它了！

hexo出自台湾大学生tommy351之手，是一个基于Node.js的静态博客程序，其编译上百篇文字只需要几秒。hexo生成的静态网页可以直接放到GitHub Pages，BAE，SAE等平台上。

* 如果你对默认配置满意，只需几个命令便可秒搭一个hexo。
* 如果你跟我一样喜欢折腾下，30分钟也足够个性化。
* 如果你过于喜欢折腾，可以折腾个把星期，尽情的玩。

hexo提供了非常简洁的用户接口，只需要3个命令就可以完成从编写到发布的全过程。

```bash
hexo n #写文章
hexo g #生成html
hexo d #部署
```

下面进入正题

## 环境准备

### 安装Nodejs
到[Nodejs](https://nodejs.org)官网下载相应平台的[最新版本](https://nodejs.org/en/download)，一路Next即可。

### 安装Git
到[Git](https://git-scm.com)官网下载相应平台的[最新版本](https://git-scm.com/download)，一路Next即可。

## 安装
Nodejs和Git都安装好之后，可执行如下命令安装hexo：
```bash
npm install hexo-cli -g
```
## 初始化
然后，执行init命令初始化hexo到你指定的目录：
```bash
hexo init <folder>
cd <folder>
npm install
```
好啦，至此，全部安装工作已经完成！

## 写文章
执行new命令，生成指定名称的文章至<folder>/source/_posts/name.md
```bash
hexo n "name" #新建文章
```
postName是md文件的名字，同时也出现在你文章的URL中，postName如果包含空格，必须用”将其包围，postName可以为中文。

接下来，你就可以用喜爱的编辑器尽情书写你的文章。关于markdown语法，可以参考我的文章[Markdown入门指南](https://zhoukuo.github.io/2016/08/04/Markdown入门指南/)。

## 生成静态页面
cd到你的init目录，执行generate命令，生成静态页面到你的初始化目录下 <folder>/public目录下。
```bash
hexo g
```
命令必须在init目录下执行，否则不成功，也不报错。

## 启动本地服务器
执行server命令，启动本地服务，进行文章预览。
```bash
hexo s
```
浏览器输入<http://localhost:4000>就可以看到效果了！

## 文章摘要
在需要显示摘要的地方添加如下代码即可：
```bash
以上是摘要
<!--more-->
以下是余下全文
```
## 标签和分类
new命令会生成一个新文件，打开文件默认会提供一些设置属性：
```bash
title: hexo你的博客
date: 2013-11-22 17:11:54
categories: default
tags: [hexo,blog]
```
categories对应的是分类，tags对应的是标签，多个需要用中括号括起来，多个标签或分类之间用逗号(,)分割。

## 主题安装
萝卜白菜各有所爱，玩博客换主题是必不可少的，hexo官方提供了主题列表：[Hexo Themes](https://hexo.io/themes/)。
通常这些主题都在github平台发布，上面提供Demo、安装文档等等说明，具体的安装步骤非常简单，和直接clone一个普通的代码库没什么区别，通常像这样：
```bash
$ git clone https://github.com/wuchong/jacman.git themes/jacman.git
```
注意要在init所在的folder执行。

然后修改folder下的_config.yml，修改theme字段为新主题的名字，然后hexo g;hexo d就可以了。

## Github托管
* 首先注册一个『GitHub』帐号，已有的默认请忽略
* 建立与你用户名对应的仓库，仓库名必须为『your_user_name.github.com』或『your_user_name.github.io』
* 添加SSH公钥到『Account settings -> SSH Keys -> Add SSH Key』
* Github托管配置

前两步忽略，直说第三步，如何添加SSH-Key。

生成密钥：
```bash
ssh-keygen -t rsa -C "<acount>"
```
输入文件路径：
```bash
H:\hexo\blog>ssh-keygen -t rsa -C "bu.ru@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (//.ssh/id_rsa): H:\git\myssh\ssh
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in H:\git\myssh\ssh.
Your public key has been saved in H:\git\myssh\ssh.pub.
The key fingerprint is:
b0:0c:2e:67:33:ab:c1:50:10:40:0a:ba:c1:80:59:22 bu.ru@qq.com
```
上述命令若成功，会在给定的目录下生成两个文件id)rsa和id_rsa_pub，最后一步：
* 用文本编辑器打开ssh.pub文件，拷贝其中的内容，将其添加到[Add SSH Key](http://github.com/settings/ssh)
最后可以验证一下：
```bash
ssh -T git@github.com
```

接下来说一下如何配置Github托管，安装 hexo-deployer-git，否则会报 ERROR Deployer not found: git 的错误。
```bash
npm install hexo-deployer-git --save
```
修改你的 _config.yml 配置文件如下：
```bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:flyoob/flyoob.github.io.git
  branch: master
```
注意这里选择的是 ssh 地址。
生成静态文件和部署：
```bash
hexo g
hexo d
```
最后出现如下提示就代表成功啦！
```
INFO  Deploy done: git
```

