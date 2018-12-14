---
title: commitzen的使用
date: 2018-12-14 10:23:02
tags:
---

## 前言
commitzen的优点很多，规范、优雅、易管理等等好处不列举了，看下怎么使用

## 一、commit message 格式
```
commit message
  Header + Body + Footer(body|footer正常开发忽略)
Header
  type + scope + subject
type
  feat    : 新增加的功能（feature）
  fix     : 修复bug
  doc     : 文档，例如README
  style   : 样式
  refactor: 重构
  test    : 测试
  chore   : 构建过程或辅助工具的变动，项目版本变更等。
  scope   : 提交的影响范围
  subject : 提交内容的简要描述
```
## 二、commitizen(全局安装)
1. 全局安装执行命令
```
➜  test npm install -g commitizen
➜  test  commitizen init cz-conventional-changelog --save --save-exact
```
2. 全局安装demo
```
➜  shopee mkdir test
➜  shopee cd test
➜  test npm init
➜  test npm i
➜  test npm install -g commitizen
➜  test  commitizen init cz-conventional-changelog --save --save-exact
➜  test git init
➜  test git:(master) ✗ git status
➜  test git:(master) ✗ git add .
➜  test git:(master) ✗ git cz
cz-cli@3.0.5, cz-conventional-changelog@2.1.0


Line 1 will be cropped at 100 characters. All other lines will be wrapped after 100 characters.

? Select the type of change that you're committing: (Use arrow keys)
❯ feat:     A new feature
  fix:      A bug fix
  docs:     Documentation only changes
  style:    Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
  refactor: A code change that neither fixes a bug nor adds a feature
  perf:     A code change that improves performance
  test:     Adding missing tests or correcting existing tests
(Move up and down to reveal more choices)
```
3. 配置的改变
```
  "devDependencies": {
    "cz-conventional-changelog": "^2.1.0"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
```
## 三、commitizen(局部安装)
1. 局部安装执行命令（AB方法等效）
```
---------------方法A-------------------------
➜  test  npm i commitizen --save-dev
➜  test ./node_modules/.bin/commitizen init cz-conventional-changelog --save-dev
---------------方法B-------------------------
➜  test  npm i commitizen --save-dev
➜  test  npm i cz-conventional-changelog --save-dev
自己配置package.json
{
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}
```
2. 配置的改变
```
 "devDependencies": {
    "commitizen": "^3.0.5",
    "cz-conventional-changelog": "^2.1.0"
  },
  "config": {
    "commitizen": { 
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
  // 比全局的commitizen多了一个局部依赖commitizen
```
## 四、Change log
1. 全局安装执行脚本
```
npm install -g conventional-changelog-cli
conventional-changelog -p angular -i CHANGELOG.md -w 
```
2. 全局安装demo
```
➜  test git:(master) conventional-changelog -p angular -i CHANGELOG.md -w
# 1.0.0 (2018-12-14)
### Features
* **test commitlint and changelog:** test commitlint and changelog c6cb5f5
```
## 五、commitlint
commitzen和change log添加了后，还有一个遗留问题，git commit 依旧能使用，忘记的时候还有可能不规范提交，commitlint了解下
```
类似于 eslint commitlint支持配置文件类型
 .commitlint.config.js
 .commitlintrc.js
 .commitlintrc.json
 .commitlintrc.yml 
 ...
或者在package.json添加commitlint字段。

```
1. 配置一
```
commitlint.config.js配置
module.exports = {
  extends: ['@commitlint/config-conventional']
}
package.json添加
{
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  }
}
```
test demo
```
➜  test git:(master) ✗ git add .
➜  test git:(master) ✗ git cm 'test commitlint'
husky > commit-msg (node v10.0.0)

⧗   input:
test commitlint

✖   message may not be empty [subject-empty]
✖   type may not be empty [type-empty]
✖   found 2 problems, 0 warningshusky > commit-msg hook failed (add --no-verify to bypass)
```
2. 加强版本配置1
```
"devDependencies": {
    "@commitlint/cli": "^7.2.1",
    "@commitlint/config-conventional": "^7.1.2",
    "@commitlint/prompt": "^7.2.1",
    "commitizen": "^3.0.5",
    "cz-conventional-changelog": "^2.1.0",
    "husky": "^1.2.1"
  },
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
```
