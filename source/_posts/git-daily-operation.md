---
title: "git daily operation "
date: 2018-07-30 15:41:28
categories:
  - tools
tags:
  - git
---

一、新建代码库

```
git init  #在当前目录新建一个git仓库
git clone url  #下载一个历史和整个代码
```

二、在一个 master 分支下工作

```
git pull  #拉取代码
git status #查看本地仓库修改状态
git add . #暂存文件
git commit #提交文件
git pull  #拉取最新代码
git push  #提交代码
git log #查看日志
```

三、创建一个本地分支 feature

```
git checkout -b test master  #新建分支并切换到新建分支
git branch  #本地分支
git branch -d #删除分支(删除分支之前先checkout分支)
git checkout #切换分支
git branch -a #查看所有分支
git branch -r #查看远程分支
```

四、合并代码到 release 分支

方案一: 合并 master 到 test，然后在把 test 合并到 master 这个时候比较绕进行了 2 次操作，以防止意外。

```
git pull origin master #假设在当前在test分支，git pull合并代码，如果有冲突，解决重突。解决完冲突后直接正常add. commit pull push 同样的代码在执行一次。checkout master去拉取test分支。
git reset --hard HEAD #撤销merge
```

方案二：直接从 master 拉取指定的 test 分支

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
git branch -r -d origin/branch-name // 删除本地的origin/branch-name游标
git branch -d branch-name //删除本地分支，发生过merge的才可以
git branch -D branch-name //强制删除本地分支
git push origin :branch-name  //删除远程分支
git push -d origin branch-name //删除远程分支
```

七、命令缩写等小技巧

```
git config -e --global
vscode GitLens
git cherry-pick xxxxx
```

八、常见工作场景及问题解决

1. 在开发一个项目不同需求的时候，需要拉取不同的特性分支，此时常常会出现在 4、5 个特性分支上切换  以配合不同开发测试联调需求，挺烦人的这种，没办法这就是工作呗，幸好 Git 强大一逼，stash 完美的解决了这样的问题。
   ```
   git stash list #查看stashlist历史
   git stash apply 1(id) #将指定的代码pop出来，在最新代码上完成合并
   git stash pop #将最新的一次pop出来
   git stash drop #丢弃某一次stash
   ```
2. 提交的时候不小心 commit 了一个错误的版本，想撤销 commit。

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

3. 提交的 log 因为某些原因比较乱，需要整理和[重写 log](https://git-reference.readthedocs.io/zh_CN/latest/Git-Tools/Rewriting-History/)

   ```
   // 撤销-可以理解为未进行push的本地代码还原
   git rebase -i HEAD~3                #修改最近3次的提交
   git commit -amend                   #改变最近一次提交记录
   git rebase --continue               #自动应用其他2次提交
   git rebase --abort                  #做了一波眼花缭乱的操作之后，想要放弃回到原点
   //压制(Squashing)提交，将多次提交合并为单一提交

   ```

   demo

   ```sh
   ➜  Windrunner git:(test_notice) git log --oneline
   ➜  Windrunner git:(test_notice) git rebase -i HEAD~10
   ➜  Windrunner git:(fdd5404) ✗ git rebase --abort
   ➜  Windrunner git:(test_notice) git rebase -i HEAD~10
   Auto-merging src/components/Common/FieldForm/Comp/Form.js
   CONFLICT (content): Merge conflict in     src/components/Common/FieldForm/Comp/Form.js
   Auto-merging src/components/Common/FieldForm/Comp/Fields/ArrayField.js
   CONFLICT (content): Merge conflict in     src/components/Common/FieldForm/Comp/Fields/ArrayField.js
   Auto-merging config/webpack.dev.conf.js
   CONFLICT (content): Merge conflict in config/webpack.dev.conf.js
   error: could not apply bb1121f... fix: 🐛 fix the tree state

   Resolve all conflicts manually, mark them as resolved with
   "git add/rm <conflicted_files>", then run "git rebase --continue".
   You can instead skip this commit: run "git rebase --skip".
   To abort and get back to the state before "git rebase", run "git rebase     --abort".

   Could not apply bb1121f... fix: 🐛 fix the tree state
   ➜  Windrunner git:(aab68bb) ✗ git rebase --continue
   config/webpack.dev.conf.js: needs merge
   src/components/Common/FieldForm/Comp/Fields/ArrayField.js: needs merge
   src/components/Common/FieldForm/Comp/Form.js: needs merge
   You must edit all merge conflicts and then
   mark them as resolved using git add
   ➜  Windrunner git:(aab68bb) ✗ git add -A
   ➜  Windrunner git:(aab68bb) ✗ git rebase --continue
   [detached HEAD a9c1a3c] fix: 🐛 fix the tree state
   Author: Qian Wei <qian.wei@xxxxx.com>
   8 files changed, 121 insertions(+), 157 deletions(-)
   rewrite src/components/Common/FieldForm/Comp/Fields/ObjectField.js (67%)
   Auto-merging src/reducers/notice/fetch.js
   Auto-merging src/pages/Notice/Setting.js
   CONFLICT (content): Merge conflict in src/pages/Notice/Setting.js
   error: could not apply 4036dcc... fix: 🐛 fix the state

   Resolve all conflicts manually, mark them as resolved with
   "git add/rm <conflicted_files>", then run "git rebase --continue".
   You can instead skip this commit: run "git rebase --skip".
   To abort and get back to the state before "git rebase", run "git rebase     --abort".

   Could not apply 4036dcc... fix: 🐛 fix the state
   ➜  Windrunner git:(3e22969) ✗ git add -A
   ➜  Windrunner git:(3e22969) ✗ git rebase --continue
   [detached HEAD 42346cb] fix: 🐛 fix the state
    Author: Qian Wei <qian.wei@xxxxx.com>
    3 files changed, 41 insertions(+), 39 deletions(-)
   Successfully rebased and updated refs/heads/test_notice.
   ➜  Windrunner git:(test_notice) git log
   ➜  Windrunner git:(test_notice) git push
   To ssh://git.garena.com:2222/xxxxx/digital-purchase/Windrunner.git
    ! [rejected]        test_notice -> test_notice (non-fast-forward)
   error: failed to push some refs to     'ssh://gitlab@git.garena.com:2222/xxxxx/digital-purchase/Windrunner.git'
   hint: Updates were rejected because the tip of your current branch is behind
   hint: its remote counterpart. Integrate the remote changes (e.g.
   hint: 'git pull ...') before pushing again.
   hint: See the 'Note about fast-forwards' in 'git push --help' for details.
   ➜  Windrunner git:(test_notice) git pull
   Auto-merging src/utils/utils.js
   Auto-merging src/reducers/notice/fetch.js
   CONFLICT (content): Merge conflict in src/reducers/notice/fetch.js
   Auto-merging src/pages/Notice/Setting.js
   CONFLICT (content): Merge conflict in src/pages/Notice/Setting.js
   Automatic merge failed; fix conflicts and then commit the result.
   ➜  Windrunner git:(test_notice) ✗ git add -A
   ➜  Windrunner git:(test_notice) ✗ npm run commit
   ```

4. 日常新建一个仓库之后要做的

   ```sh
   Git global setup
   git config --global user.name "Qian Wei"
   git config --global user.email "qian.wei@xxxxx.com"

   Create a new repository
   git clone ssh://gitlab@git.garena.com:2222/qian.wei/sentry.git
   cd sentry
   touch README.md
   git add README.md
   git commit -m "add README"
   git push -u origin master

   Existing folder
   cd existing_folder
   git init
   git remote add origin ssh://gitlab@git.garena.com:2222/qian.wei/sentry.git
   git add .
   git commit -m "Initial commit"
   git push -u origin master

   Existing Git repository
   cd existing_repo
   git remote rename origin old-origin
   git remote add origin ssh://gitlab@git.garena.com:2222/qian.wei/sentry.git
   git push -u origin --all
   git push -u origin --tags

   Existing Git repository2
   cd existing_repo
   git remote set-url origin https://github.com/weiqian93/vconsole.git
   git push
   ```

5. cherry-pick、reset、revert
6. git 对文件名的大小写不敏感，修改了文件的大小写，git st 并没有改动
   ```sh
   //方法一 git设置忽略文件
   git config core.ignorecase false
   //方法二 git rm/add/commit xxx
   git rm xxx
   git add xxx
   git commit xxx
   ```
7. 更改 git 提交记录的 author 和 email[方法](https://help.github.com/en/articles/changing-author-info)

   执行脚本

   ```sh
    #!/bin/sh

    git filter-branch --env-filter '

    OLD_EMAIL="qian.wei@shopee.com"
    CORRECT_NAME="weiqian93"
    CORRECT_EMAIL="540518665@qq.com"

    if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
    then
        export GIT_COMMITTER_NAME="$CORRECT_NAME"
        export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
    fi
    if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
    then
        export GIT_AUTHOR_NAME="$CORRECT_NAME"
        export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
    fi
    ' --tag-name-filter cat -- --branches --tags
   ```

   git push --force --tags origin 'refs/heads/\*'
