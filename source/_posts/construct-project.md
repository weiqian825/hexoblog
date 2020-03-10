---
title: construct-project
date: 2018-08-03 16:38:24
categories: 
- web
tags:
- react-init
---

### 前言
入坑react3个月，记录下一个react项目如何从头搭建


### 一、使用create-react-app搭建步骤
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
  npm i -D standard
  //package.json里面的配置
  "scripts": {
    "standard": "standard" // npm run standard
  },
  // 日常全局运行 standard --fix 会取package.json的配置
  "standard": {
    "ignore": [],
    "cwd": "",
    "fix": false,
    "globals": [
      "it"
    ],
    "plugins": [],
    "envs": [],
    "parser": "babel-eslint"
  },
```
4. 添加[husky](https://github.com/typicode/husky)，在commit前进行代码校验
```
  npm i -D husky
  "husky": {
    "hooks": {
      "pre-commit": "standard",
      "pre-push": "npm test"
    }
  },
```
5. 搭建mock server 
   
6. 添加jest

### 二、从头搭建react-webpack工程
1. 要安装的模块及功能
```
[wepack]     |   webpack-cli webpack webpack-merge
[react]      |   react react-dom
[babel]      |   babel-loader @babel/preset-core @babel/preset-env @babel/preset-core
[html]       |   html-webpack-plugin clean-webpack-plugin
[js]         |   uglifyjs-webpack-plugin 
[css]        |   style-loader css-loader  mini-css-extract-plugin less-loader sass-loader node-sass 
[css]        |   optimize-css-assets-webpack-plugin postcss postcss_loader autoprefixer
[devserver]  |   webpack-dev-server
[anlyzer]    |   webpack-bundle-analyzer
[eslint]     |   husky standardjs
[pretty]     |   ora chalk   
```

### 三、拓展CRA的webpack配置
[相关资料](https://juejin.im/post/5a5d5b815188257327399962)
大体上有4种方法
[项目 eject]()
[替换 react-scripts 包](https://auth0.com/blog/how-to-configure-create-react-app/#Test-Your-Custom-Script)
[使用 react-app-rewired]()
[scripts 包 + override 组合]()

