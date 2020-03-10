---
title: commitzen的使用
date: 2018-12-14 10:23:02
tags:
---

### 前言
[commitzen](https://github.com/commitizen/cz-cli)是一个帮助规范commit message的工具，优点很规范、优雅、易管理拓展等等好处不列举了，看下怎么使用

### 一、commit message 格式
```html
commit message
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>

Header
  type + scope + subject
type
  feat    : 新增加的功能（feature）
  fix     : 修复bug
  doc     : 文档，例如README
  style   : 格式（不影响代码运行的变动）
  refactor: 重构
  test    : 测试
  chore   : 构建过程或辅助工具的变动，项目版本变更等。
scope   : 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同
subject : 提交内容的简要描述
```
### 二、commitizen(全局安装)
1. 全局安装执行命令
```javascript
➜  test npm install -g commitizen
➜  test  commitizen init cz-conventional-changelog --save --save-exact
（或者 echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc）
```
2. 配置的改变
```json
  "devDependencies": {
    "cz-conventional-changelog": "^2.1.0"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
```
3. 全局安装demo
```js
➜  xxxxx mkdir test
➜  xxxxx cd test
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

### 三、commitizen(局部安装)
1. 局部安装执行命令（AB方法等效）
```js
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
```json
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
3. cz-customizable和cz-conventional-changelog差不多，也是commitizen的adapter。

### 四、Change log

1. 全局安装执行脚本
```js
npm install -g conventional-changelog-cli
conventional-changelog -p angular -i CHANGELOG.md -w 
为了方便起见我们把这个log的命令写到package.json
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -w -r 0"
  }
}

```
2. 全局安装demo
```
如果你的提交都符合规范，那么执行脚本npm run changelog将生成文档包括下面三个部分
New features
Bug fixes
Breaking changes.
➜  test git:(master) conventional-changelog -p angular -i CHANGELOG.md -w
# 1.0.0 (2018-12-14)
### Features
* **test commitlint and changelog:** test commitlint and changelog c6cb5f5
```
### 五、commitlint
commitzen和change log添加了后，还有一个遗留问题，git commit 依旧能使用，忘记的时候还有可能不规范提交，[commitlint安装](https://marionebl.github.io/commitlint/#/guides-local-setup)了解下
```json
类似于 eslint commitlint支持配置文件类型
 .commitlint.config.js
 .commitlintrc.js
 .commitlintrc.json
 .commitlintrc.yml 
 ...
或者在package.json添加commitlint字段。

```
1. 配置一
```json
# Install commitlint cli and conventional config
npm install --save-dev @commitlint/{config-conventional,cli}
# Configure commitlint to use conventional config
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
npm install --save-dev husky
{
 "devDependencies": {
    "@commitlint/cli": "^7.2.1",
    "@commitlint/config-conventional": "^7.1.2",
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
}
```
test demo
```json
➜  test git:(master) ✗ git add .
➜  test git:(master) ✗ git cm 'test commitlint'
husky > commit-msg (node v10.0.0)

⧗   input:
test commitlint

✖   message may not be empty [subject-empty]
✖   type may not be empty [type-empty]
✖   found 2 problems, 0 warningshusky > commit-msg hook failed (add --no-verify to bypass)
```

### 六、实践问题
在加上了commitzen之后遇到一个问题，不同人提交终端提示不一样，最终的提交记录也不一样，有的有小图标，有的没有小图标？
1. 有图标
```
➜  sniper git:(feature-train-tickets) npm run commit

> sniper@0.1.0 commit /Users/weiqian/Desktop/sniper
> npx git-cz

npx: 1 安装成功，用时 3.681 秒
? Select the type of change that you're committing:
  🎡 ci:         CI related changes
  ⚡️ perf:       A code change that improves performance
  💍 test:       Adding missing tests
❯ 🎸 feat:       A new feature
  🐛 fix:        A bug fix
  🤖 chore:      Build process or auxiliary tool changes
  ✏️ docs:       Documentation only changes
```
2. 没图标
```
➜  sniper git:(feature-train-tickets) ✗ npm run commit

> sniper@0.1.1 commit /Users/weiqian/Desktop/sniper
> npx git-cz

cz-cli@3.0.5, cz-conventional-changelog@2.1.0


Line 1 will be cropped at 100 characters. All other lines will be wrapped after 100 characters.

? Select the type of change that you're committing: (Use arrow keys)
❯ feat:     A new feature
  fix:      A bug fix
  docs:     Documentation only changes
  style:    Changes that do not affect the meaning of the code (white-space, formatting, missing semi
-colons, etc)
  refactor: A code change that neither fixes a bug nor adds a feature
  perf:     A code change that improves performance
```
从执行的命令可以看到，执行npx git-cz，有图标的实际上是在安装git-cz的adpater。没有图标的实际上是用的cz-cli@3.0.5的adapter，为什么执行同样的命令会发生不一样的结果，是因为npm的bin目录不一致导致，最简单的方法就是把依赖安装到项目本身，而不是用全局命令，保证所有人命令是统一的。


### 七、npm 一些其他命令
```
npm list -g --depth 0
npm cache clean -f
sudo chown -R $USER /Users/weiqian/Desktop/test
```

➜  sniper git:(feature-train-tickets) ✗ npm list -g --depth 0
/usr/local/lib
├── @xxxxx/regen@0.3.0
├── ant-design-pro-cli@1.0.0
├── babel-eslint@8.2.5
├── commitizen@3.0.5
├── conventional-changelog-cli@2.0.11
├── create-react-app@1.5.2
├── cz@1.8.2
├── depcheck@0.6.11
├── download-csv@1.1.1
├── eslint@5.0.1
├── hexo@3.7.1
├── hexo-cli@1.1.0
├── husky@0.14.3
├── n@2.1.11
├── node-pre-gyp@0.11.0
├── npm@6.4.1
├── rapydscript@0.5.58
├── rimraf@2.6.2
├── sqlite3@4.0.2
├── standard@11.0.1
├── webpack@4.8.3
├── webpack-cli@2.1.3
├── webpack-dev-server@3.1.4
├── whistle@1.10.10
├── whistle.vase@0.1.0
└── yarn@1.9.4


