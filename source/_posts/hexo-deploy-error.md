---
title: hexo-deploy-error
date: 2018-08-14 15:11:59
tags:
---

今天部署(hexo d)出现如下错误
```
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: EACCES: permission denied, unlink '/Users/weiqian/Desktop/hexo/.deploy_git/archives/index.html'
```
根据提示
```
rm -rf .deploy_git
hexo clean
```
