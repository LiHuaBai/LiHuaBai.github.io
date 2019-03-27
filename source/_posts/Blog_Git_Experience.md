---
title: 重开博客采坑记录
date: 2019-03-19 15:20:13
categories:
- 采坑
- Git
---

再次利用hexo搭建个人博客的过程中，依旧踩了很多坑，主要之前Git用的不熟练，现在顺便也补一下Git的配置和使用方法。

## Git主机秘钥配置
使用Git来进行分布式管理，首先要配置用户名，邮箱，以及主机秘钥
``` bash
git config --global user.name "yourname"
git config --global user.email "youremail"
```
邮箱与用户名与github一致
`ssh-keygen -t rsa -C "youremail"`
会让设置密码什么的，一般可以直接回车不设置，完成后在~.ssh/文件夹里，会有公钥id_rsa.pub和私钥id_rsa，公钥在github在github的账号settings的SSH keys处配置。

## 两个github账号引起的错误
想着本来那个账号注册的比较随便，用户名有点怪怪的，于是打算换一个，结果换了一个配置好秘钥后居然不能push到仓库也不能用hexo deploy，原因是第一次用git提交到远程仓库时，需要输入github的账号密码，系统自动记住了，换账号后，需要主要删除之前账号的信息，在控制面板->用户账户->凭据管理器（管理windows凭据）处，删除git凭据

## hexo博客搭建各类坑
原先开启标签和分类后，切换主题后再切回来会出错，导致主题样式异常，需要重新开启标签和分类

所用主题开启标签和分类后，还需要去改标签和分类的index.md的内容，才能正确显示

评论系统，开启gitment
原先是这样的
``` bash
gitment: false
# gitment:
#   owner: LiHuaBai
#   repo: LiHuaBai.github.io
#   client_id:
#   client_secret:
```
正确开启写法
``` bash
# gitment: false
gitment:
  owner: LiHuaBai
  repo: LiHuaBai.github.io
  client_id:申请的id
  client_secret:申请的secret
```
在[此处](https://github.com/settings/applications/new)申请OAuthid
其中的Homepage URL和Authorization callback URL可以都写成https://username.github.io
申请后的OAuth Apps在[这里](https://github.com/settings/developers)查看

然而配置好依旧不能用，查找后发现是原开发者的服务器已经关闭了，找了一个别人搭好的服务器，在主题的layout文件夹里面找到gitment的部分，修改如下
``` bash
<link rel="stylesheet" href="https://jjeejj.github.io/css/gitment.css">
<script src="https://jjeejj.github.io/js/gitment.js"></script>
```
可正常使用评论功能

## github存储hexo本地文件
为了之后换电脑啥的博客不在丢失，觉得最好还是把本地文件存在github上，这样也容易找回，一般来说可以用同一个仓库，新建一个分支，先把仓库中所存储的博客的静态html文件clone下来
`git clone git@github.com:LiHuaBai/LiHuaBai.github.io.git`
只留下git文件，其余删除，把hexo文件夹内部的以下文件复制过来
1. scaffords
2. source
3. themes
4. .gitignore
5. \_conig.yml
6. package.json
然后将这些文件保存在仓库的另一个分支下
```
git checkout -b hexo_local_file
git add --all
git commit -m "save local file"
git push --set-upstream origin hexo
```
## 利用github重新恢复博客
将hexo文件从仓库中clone下来，重新安装依赖库后，就可以重新启动了
```
git clone -b hexo_local_file git@github.com:LiHuaBai/LiHuaBai.github.io.git
cnpm install
hexo g
hexo s
```
