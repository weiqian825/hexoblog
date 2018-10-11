---
title: construct-project
date: 2018-08-03 16:38:24
categories: 
- web
tags:
- react-init
---

## 前言
入坑react3个月，记录下一个react项目如何从头搭建

## 搭建步骤
1. 为了快速学习和方便开发，跳过项目配置和安装的繁琐步骤，我们这里选择了react官方推荐的[create-react-app](https://github.com/facebook/create-react-app)生成项目框架。
```
npx create-react-app my-app
cd my-app
npm start
```
2. 默认的配置会比较简单，执行eject脚本，弹出更丰富的配置。
```
npm run eject
```
3. 我们代码的规范是[stardardjs](https://standardjs.com/)，规范开发工作。
```
  //package.json里面的配置
  "scripts": {
    "standard": "standard" // npm run standard
  },
  // 日常全局运行 standard --fix 会取package.json的配置
  "standard": {
    ignore: [],   // file globs to ignore (has sane defaults)
    cwd: '',      // current working directory (default: process.cwd())
    fix: false,   // automatically fix problems
    globals: ["it"],  // global variables to declare
    plugins: [],  // eslint plugins
    envs: [],     // eslint environment
    parser: 'babel-eslint'    // js parser (e.g. babel-eslint)
  },
```
4. 添加[husky](https://github.com/typicode/husky)，在commit前进行代码校验
```
  npm install husky@next --save-dev
  "husky": {
    "hooks": {
      "pre-commit": "standard",
      "pre-push": "npm test"
    }
  },
```
5. 搭建mock server 
   
6. 添加jest
   