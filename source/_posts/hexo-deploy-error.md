---
title: hexo-deploy-error
date: 2018-08-14 15:11:59
tags:
---
## 问题
1. 部署(hexo d)错误1
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
2. publickey不对
```
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
INFO  Copying files from extend dirs...
On branch master
nothing to commit, working tree clean
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
重新配置生成下 ssh key
```
1. cd ~/.ssh/ 使用ssh-keygen命令生成ssh密钥 ssh-keygen -t rsa  -C "540518665@qq.com" （enter后，第一个选择文件路径名称 example1 存储key，第二三选择密码空格就可以）
2. ssh-add ~/.ssh/example1
3. github上setting的SSH添加本地生成的example1 (cat ~/.ssh/example1.pub)
4. 测试 ssh git@github.com
```
3. 由于同步中未将Next主题提交。No layout: index.html
也是为了下次同步最新的Next主题，故才有多地同步是更新Next主题方案。
解决方案：直接获取/更新Next主题
``` 
    cd your-hexo-site
    git clone https://github.com/iissnan/hexo-theme-next themes/next
```
## 原理
```
为什么会出现这个错呢？
```
