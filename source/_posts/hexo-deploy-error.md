---
title: hexo-deploy-error
date: 2018-08-14 15:11:59
tags:
---

1. 部署(hexo d)错误

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
1⃣️  cd ~/.ssh/ 使用ssh-keygen命令生成ssh密钥 ssh-keygen -t rsa  -C "540518665@qq.com" （enter后，第一个选择文件路径名称 example1 存储key，第二三选择密码空格就可以）
2⃣️  ssh-add ~/.ssh/example1
3⃣️  github上setting的SSH添加本地生成的example1 (cat ~/.ssh/example1.pub)
4⃣️  测试 ssh git@github.com
```

3. 由于同步中未将Next主题提交，页面白屏。No layout: index.html

``` 
// 也是为了下次同步最新的Next主题，故才有多地同步是更新Next主题方案。
// 解决方案：直接获取/更新Next主题
cd your-hexo-site
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
4. 又出现了git@github.com: Permission denied (publickey)，莫非这里的publickey的会过期，过一段就要添加一下？ssh-add ~/.ssh/id_rsa_hexo
``` 
➜  hexo git:(master) ✗ hexo d
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
INFO  Copying files from extend dirs...
[master 7a089f0] Site updated: 2018-09-06 15:50:05
 2 files changed, 74 insertions(+), 4 deletions(-)
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (/Users/weiqian/Desktop/shopee/hexo/node_modules/hexo-util/lib/spawn.js:37:17)
    at ChildProcess.emit (events.js:182:13)
    at maybeClose (internal/child_process.js:947:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:236:5)
➜  hexo git:(master) ✗ ssh-add ~/.ssh/id_rsa_hexo
Identity added: /Users/weiqian/.ssh/id_rsa_hexo (/Users/weiqian/.ssh/id_rsa_hexo)
➜  hexo git:(master) ✗ ssh git@github.com
PTY allocation request failed on channel 0
Hi weiqian93! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
➜  hexo git:(master) ✗ hexo d
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
INFO  Copying files from extend dirs...
On branch master
nothing to commit, working tree clean
To github.com:weiqian93/weiqian93.github.io.git
   b07fcdf..7a089f0  HEAD -> master
Branch 'master' set up to track remote branch 'master' from 'git@github.com:weiqian93/weiqian93.github.io.git'.
INFO  Deploy done: git 
``` 

## 原理

```
为什么会出现这个错呢？
```
