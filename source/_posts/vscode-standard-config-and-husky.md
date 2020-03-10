---
title: 'vscode standard config and husky'
date: 2018-07-18 16:48:59
type: "web"
tags:
- vscode
- standardjs
- husky
---
### 前言
目前的项目代码遵循的规范是[standardjs](https://standardjs.com/)，用的IDE是[vscode](https://code.visualstudio.com/)，在代码提交的时候用[husky](https://github.com/typicode/husky)检查代码规范是否符合standards标准，配置如下

### 配置选项
> 1. 安装插件 JavaScript Standard Style，全局配置选项，解决[es7语法报错的问题](https://standardjs.com/#how-do-i-use-experimental-javascript-es-next-features) 刚开始项目安装的语法检查总不起作用，发现是代码里的装饰器@报错，导致其他的检查不能正常进行，安装babel-eslint可以正常检查装饰器@认为是合理的，从而进一步检查其他地方错误。
```json
"standard.options": {
    "parser": "babel-eslint"
}
```
>2. [husky](https://github.com/typicode/husky)在react项目中的提交检查配置，设置package.json
```json
"scripts": {
    "test": "standard --fix"
},
"husky": {
    "hooks": {
        "pre-commit": "npm test"
    }
},
"devDependencies": {
    "husky": "^1.0.0-rc.13"
}
```