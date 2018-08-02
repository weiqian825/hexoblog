---
title: build-target-path
date: 2018-08-01 16:23:55
tags:
---

## 前言

   项目中内置的打包编译工具在build之后生成的目录是build/dist且不支持更改配置目录，实际同一个域名挂载了不止一个项目，由于ngnix配置需要，我们期望编译后目录是build/windrunner/static/，那么如何解决这个问题呢？ 最简单的常规方法就是我们自己写操作文件脚本，下面将介绍如何通过文件操作将build的文件目录变成我们期望的。

### 解决方案
```
    const exec = require('child_process').exec
    const fs = require('fs')
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
        if (!fs.existsSync(staticPath)) {
        fs.mkdirSync(staticPath)
        fs.mkdirSync(cssPath)
        fs.mkdirSync(jsPath)
        }

        // move files
        const files = fs.readdirSync(buildPath)
        for (let i = 0, len = files.length; i < len; i++) {
        const srcFilename = path.resolve(buildPath, files[i])
        if (files[i].indexOf('css') > -1) {
            const destFilename = path.resolve(cssPath, files[i])
            fs.renameSync(srcFilename, destFilename)
        }

        if (files[i].indexOf('js') > -1) {
            const destFilename = path.resolve(jsPath, files[i])
            fs.renameSync(srcFilename, destFilename)
        }
        }

        // rewrite index.html
        const indexHtmlPath = path.resolve(outputPath, 'index.html')
        let indexFile = fs.readFileSync(indexHtmlPath, 'utf8')
        indexFile = indexFile
        .replace(/\/static\/(.*css)/, '/static/css/$1')
        .replace(/\/static\/(.*js)/, '/static/js/$1')
        fs.writeFileSync(indexHtmlPath, indexFile)
        console.log('Successful!')
    }
    })
```