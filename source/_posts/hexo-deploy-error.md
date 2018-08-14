---
title: hexo-deploy-error
date: 2018-08-14 15:11:59
tags:
---
## 问题
今天部署(hexo d)出现如下错误
```
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: EACCES: permission denied, unlink '/Users/weiqian/Desktop/hexo/.deploy_git/archives/index.html'
```
刚开始以为是权限问题，git代码仓库可以正常提交，但是deploy d的时候总是不对，重新生成了[ssh密钥](https://weiqian93.github.io/2018/07/17/Git-ssh-key/)，依旧没有用，随后清理.deploy_git及.public目录。

```
rm -rf .deploy_git 
hexo clean 
hexo d
```
OK啦，可以正常工作了呢

## 原理
```
为什么会出现这个错呢？
```
