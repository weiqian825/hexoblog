---
title: 'git daily operation '
date: 2018-07-30 15:41:28
tags:
---

一、新建代码库
``` 
git init  #在当前目录新建一个git仓库
git clone url  #下载一个历史和整个代码
```
二、在一个master分支下工作
```
git pull  #拉取代码
git status #查看本地仓库修改状态
git add . #暂存文件
git commit #提交文件
git pull  #拉取最新代码
git push  #提交代码
git log #查看日志
```
三、创建一个本地分支feature
```
git checkout -b test master  #新建分支并切换到新建分支
git branch  #本地分支
git branch -d #删除分支(删除分支之前先checkout分支)
git checkout #切换分支
git branch -a #查看所有分支
git branch -r #查看远程分支
```
四、合并代码到release分支

方案一: 合并master到test，然后在把test合并到master这个时候比较绕进行了2次操作，以防止意外。
```
git pull origin master #假设在当前在test分支，git pull合并代码，如果有冲突，解决重突。解决完冲突后直接正常add. commit pull push 同样的代码在执行一次。checkout master去拉取test分支。
git reset --hard HEAD #撤销merge 
```
方案二：直接从master拉取指定的test分支

```
git pull origin test 
```


五、解决冲突

```
当出现冲突时，git会在代码中标记出来
<<<<<<<
当前的更改
=======
merge的更改
>>>>>>>
根据需要选择保留或者遗弃代码后，再执行add, commit就可以完成merge
```

六、删除分支

```
// git push -d origin branch-name
git branch -r -d origin/branch-name  
git push origin :branch-name  
```

七、常见工作场景及问题解决

1. 在开发一个项目不同需求的时候，需要拉取不同的特性分支，此时常常会出现在4、5个特性分支上切换以配合不同开发测试联调需求，挺烦人的这种，没办法这就是工作呗，幸好Git强大一逼，stash完美的解决了这样的问题。
    ```
    git stash list #查看stashlist历史
    git stash apply 1(id) #将指定的代码pop出来，在最新代码上完成合并
    git stash pop #将最新的一次pop出来
    git stash drop #丢弃某一次stash
    ```
2. 提交的时候不小心commit了一个错误的版本，想撤销commit。

    ```
    // 撤销-可以理解为未进行push的本地代码还原
    git checkout fileName               #未执行add		
    git checkout .                      #未执行add	
    git reset HEAD fileName             #已执行add
    git commit --amend                  #已执行commit
    git reset --hard [commit | HEAD]    #已执行commit
    git reflog                          #已执行reset --hard
        
    ```
    ```
    //回滚-可以理解为已进行push的远程代码还原
    git reset --hard HEAD^          	#reset会删除commit提交历史
    git push -f			                #本地会比远程落后，所以需要加上force

    git revert HEAD		                #revert放弃指定提交，会生成一次新的提交
    git push			                #保留以前的提交历史

    ```
3. 提交的log因为某些原因比较乱，需要整理和重写log
    ```
    // 撤销-可以理解为未进行push的本地代码还原
    git rebase -i HEAD~3                #修改最近3次的提交		
    git commit -amend                   #改变最近一次提交记录	
    git rebase --continue               #自动应用其他2次提交
    //压制(Squashing)提交，将多次提交合并为单一提交
        
    ```
   