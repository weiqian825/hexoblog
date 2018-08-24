---
title: 让vconsole支持fetch
date: 2018-08-24 19:36:20
tags:
---

## 序言
我们在前面说到，react项目接入vconsole后，发现w不支持fetch，于是拉取[vconsole](https://github.com/Tencent/vConsole)库，看看下如何改写源码使它支持fetch

## 一、搭建DevServer服务
源码里面只有生成dist部分的命令，不能自己调试开发，所以我们需要搭建一个DevServer支持我们开发功能。

// package.json
```
{
    "scripts": {
        "test": "mocha",
        "dist": "webpack",
        "dev": "webpack-dev-server --config webpack.dev.conf.js"
    },
    "devDependencies": {
        "webpack": "~1.12.11",
        "webpack-dev-server": "1.14.0",
        "webpack-merge": "^0.14.1",
        "html-webpack-plugin": "^2.8.1",
        "extract-text-webpack-plugin": "^1.0.1",
    }
}

```
// webpack.dev.conf.js
```
var webpack = require('webpack')
var merge = require('webpack-merge')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var baseConfig = require('./webpack.config.js')
var path = require('path')

module.exports = merge(baseConfig, {
  entry: './dev/index.js',
  devServer: {
    historyApiFallback: true,
    noInfo: true,
    overlay: true,
    open: true,
    hot: true,
    inline: true
  },
  devtool: '#eval-source-map',
  plugins: [
    new HtmlWebpackPlugin({
      template: './dev/index.html',
      filename: 'index.html',
      inject: true
    }),
    new webpack.HotModuleReplacementPlugin()
  ]
})

```
// mkdir dev ---index.html index.js data.json 等
```
// index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  
</body>
</html>
```
```
// index.js
const VConsole = require('../src/vconsole')

const vc = new VConsole()

console.log(vc)
console.log('start vconsole')

window.fetch('http://localhost:8080/dev/data.json?fetch=test',{method:'GET'}).then(data => {
    console.log(data)
})

let xhr = new XMLHttpRequest()
xhr.open('get','http://localhost:8080/dev/data.json?xhr=ajax')
xhr.send()
```
```
//data.json
{
    "test": 123,
    "testjson":{
        "name": 1
    }
}
```
试着运行下
```
npm i
npm run dev
```

## 二、如何劫持fetch呢




