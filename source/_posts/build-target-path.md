---
title: build-target-path
date: 2018-08-01 16:23:55
categories: 
- shopee
tags:
- webpack
- build
---

## 前言

   项目中内置的打包编译工具在build之后生成的目录是build/dist且不支持更改配置目录，实际同一个域名挂载了不止一个项目，由于ngnix配置需要，我们期望编译后目录是build/windrunner/static/，那么如何解决这个问题呢？ 最简单的常规方法就是我们自己写操作文件脚本，下面将介绍如何通过[NODEJS](http://nodejs.cn/api/)文件操作将build的文件目录变成我们期望的。

### 解决方案
```
// package.json的配置
{
  "scripts": {
    "oldbuild": "cross-env DISABLE_ESLINT=none roadhog build",
    "build": "node scripts/build.js",
  }
}
```
```
    //child_process 模块提供了衍生子进程的功能，衍生一个 shell 并在 shell 上运行命令，当完成时会传入 stdout 和 stderr 到回调函数。
    const exec = require('child_process').exec
    //fs 模块提供了一些 API，用于以一种类似标准 POSIX 函数的方式与文件系统进行交互。
    const fs = require('fs')
    //path 模块提供了一些工具函数，用于处理文件与目录的路径。 resolve方法会把一个路径或路径片段的序列解析为一个绝对路径。
    const path = require('path')
    const config = require(path.resolve(__dirname, '../.webpackrc.js'))
    const appPath = path.resolve(__dirname, '../')
    const { outputPath } = config
    const buildPath = path.resolve(appPath, outputPath)
    const staticPath = path.resolve(appPath, outputPath + '/static')
    const cssPath = path.resolve(staticPath, 'css')
    const jsPath = path.resolve(staticPath, 'js')

    exec('npm run oldbuild', (err, stdout) => {
    if (err) {
        console.error(err)
    } else {
        console.log(stdout)

        // mkdir
        if (!fs.existsSync(staticPath)) { //如果路径存在，则返回 true，否则返回 false
            fs.mkdirSync(staticPath) //同步地创建目录。 返回 undefined
            fs.mkdirSync(cssPath)
            fs.mkdirSync(jsPath)
        }

        // move files
        const files = fs.readdirSync(buildPath) // 返回一个不包括 '.' 和 '..' 的文件名的数组
        for (let i = 0, len = files.length; i < len; i++) {
            const srcFilename = path.resolve(buildPath, files[i])
            if (files[i].indexOf('css') > -1) {
                const destFilename = path.resolve(cssPath, files[i])
                // oldpath newpath
                fs.renameSync(srcFilename, destFilename)
            }

            if (files[i].indexOf('js') > -1) {
                const destFilename = path.resolve(jsPath, files[i])
                fs.renameSync(srcFilename, destFilename)
            }
        }

        // rewrite index.html
        const indexHtmlPath = path.resolve(outputPath, 'index.html')
        let indexFile = fs.readFileSync(indexHtmlPath, 'utf8')//返回 path 的内容
        indexFile = indexFile
        .replace(/\/static\/(.*css)/, '/static/css/$1')
        .replace(/\/static\/(.*js)/, '/static/js/$1')
        fs.writeFileSync(indexHtmlPath, indexFile)
        console.log('Successful!')
    }
    })
```
部分日志打印的结果
```
{"entry":"src/index.js","extraBabelPlugins":[["import",{"libraryName":"antd","libraryDirectory":"es","style":true}]],"env":{"development":{"extraBabelPlugins":["dva-hmr"]}},"alias":{"components":"/Users/weiqian/Desktop/Windrunner/src/components","react-native":"react-native-web","@src":"/Users/weiqian/Desktop/Windrunner/src","@const":"/Users/weiqian/Desktop/Windrunner/src/constants","@utils":"/Users/weiqian/Desktop/Windrunner/src/utils","@selector":"/Users/weiqian/Desktop/Windrunner/src/selectors","@services":"/Users/weiqian/Desktop/Windrunner/src/services","@layouts":"/Users/weiqian/Desktop/Windrunner/src/layouts","@api":"/Users/weiqian/Desktop/Windrunner/src/apis"},"disableDynamicImport":true,"ignoreMomentLocale":true,"html":{"template":"./src/index.ejs"},"outputPath":"./build/windrunner","publicPath":"/digital-purchase/static","hash":true,"proxy":{"/api":{"target":"http://localhost:3003"},"/local_api":{"target":"http://localhost:3003"},"staticPublicUrl":"/static"}}
appPath /Users/weiqian/Desktop/Windrunner
buildPath /Users/weiqian/Desktop/Windrunner/build/windrunner
staticPath /Users/weiqian/Desktop/Windrunner/build/windrunner/static
cssPath /Users/weiqian/Desktop/Windrunner/build/windrunner/static/css
jsPath /Users/weiqian/Desktop/Windrunner/build/windrunner/static/js
files ["favicon.png","index.e10a2168.css","index.eadb0ebc.js","index.html","static"]
srcFilename /Users/weiqian/Desktop/Windrunner/build/windrunner/index.e10a2168.css
destFilename /Users/weiqian/Desktop/Windrunner/build/windrunner/static/css/index.e10a2168.css
srcFilename /Users/weiqian/Desktop/Windrunner/build/windrunner/index.eadb0ebc.js
destFilename /Users/weiqian/Desktop/Windrunner/build/windrunner/static/js/index.eadb0ebc.js
indexHtmlPath /Users/weiqian/Desktop/Windrunner/build/windrunner/index.html
indexFile <!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>SHOPEE DP</title>
  <link rel="icon" href="/favicon.png" type="image/x-icon">
<link href="/digital-purchase/static/index.e10a2168.css" rel="stylesheet"></head>

<body>
  <div id="root"></div>
<script type="text/javascript" src="/digital-purchase/static/index.eadb0ebc.js"></script></body>

</html>
indexFile after <!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>SHOPEE DP</title>
  <link rel="icon" href="/favicon.png" type="image/x-icon">
<link href="/digital-purchase/static/css/index.e10a2168.css" rel="stylesheet"></head>

<body>
  <div id="root"></div>
<script type="text/javascript" src="/digital-purchase/static/js/index.eadb0ebc.js"></script></body>

</html>
```