---
title: 让vconsole支持fetch
date: 2018-08-24 19:36:20
categories: 
- web
tags:
- vconsole
- fetch
---


我们[前面](https://weiqian93.github.io/2018/08/22/sentry%E5%92%8Cvconsole%E7%9A%84%E5%BA%94%E7%94%A8/)说到，react项目接入vconsole后，发现不支持fetch，于是拉取[vconsole](https://github.com/Tencent/vConsole)库，看看下如何改写源码使它支持fetch

### 一、搭建DevServer服务
源码里面只有生成dist部分的命令，不能自己调试开发，所以我们需要搭建一个DevServer支持我们开发功能。

```javascript
// package.json
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
```javascript
// webpack.dev.conf.js

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
```html
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
```js
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
```json
//data.json
{
    "test": 123,
    "testjson":{
        "name": 1
    }
}
```
试着运行下
```sh
npm i
npm run dev
```

### 二、如何拦截fetch呢
vconsole里面可以抓到ajax的请求包，是在network里面对XMLHttpRequest进行了重写，拦截到相关信息记录下来并在日志面板显示。依照这个思路，我们初步想的是如何重写fetch又不影响它原来的功能，于是搜索到一篇很有意思的[博客](http://undefinedblog.com/how-to-intercept-fetch/)，利用fetch里面的respone.clone()这个特性在中间做一些拦截记录的工作如下。
代码思路

```js
mockFetch(){
    let _fetch = window.fetch
    let prevFetch = (url,init) => {
        //做一些发请求前的记录工作
        return _fetch(url,init).then(response => {
            //利用response.clone()这个特性真正的做一些事情
            response.clone().json().then(json => {
                // 具体在这里
            })
            return response
        })
    }
}
```
具体代码
```js
   mockFetch() {
    let _fetch = window.fetch;
    if(!_fetch){ return; }
    let that = this;

    let prevFetch = (url,init) => {
      let id = that.getUniqueID()
      that.reqList[id] = {}
      let item = that.reqList[id] || {};
      let query = url.split('?'); 
      item.id = id;
      item.method = init.method||'GET';
      item.url = query.shift(); 

      if (query.length > 0) {
        item.getData = {};
        query = query.join('?'); // => 'b=c&d=?e'
        query = query.split('&'); // => ['b=c', 'd=?e']
        for (let q of query) {
          q = q.split('=');
          item.getData[ q[0] ] = q[1];
        }
      }

      if (item.method == 'POST') {
        // save POST data
        if (tool.isString(data)) {
          let arr = data.split('&');
          item.postData = {};
          for (let q of arr) {
            q = q.split('=');
            item.postData[ q[0] ] = q[1];
          }
        } else if (tool.isPlainObject(data)) {
          item.postData = data;
        }
      }
      // UNSENT
      if (!item.startTime) {
        item.startTime = (+new Date());
      }
      return _fetch(url,init).then(response => {
        response.clone().json().then(json => {
          item.endTime = +new Date(),
          item.costTime = item.endTime - (item.startTime || item.endTime);
          item.status = response.status;
          item.header = {}
          // 拿到header里面的东西
          for (let pair of response.headers.entries()) {
            item.header[pair[0]] = pair[1]
            console.log(pair[0])
          }
          item.response = json;
          item.readyState = 4;
          item.responseType = response.type
          that.updateRequest(id, item)
          return json
        })
        return response
      })
    }
    window.fetch = prevFetch
  }
```

### 三、生成目标js
```json
//webpack.config.js
var pkg = require('./package.json');
var webpack = require('webpack');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  devtool: false,
  entry: {
    vconsole : './src/vconsole.js'
  },
  output: {
    path: './dist',
    filename: '[name].min.js',
    library: 'VConsole',
    libraryTarget: 'umd'
    // umdNameDefine: true
  },
  module: {
    loaders: [
      {
        test: /\.html$/, loader: 'html?minimize=false'
      },
      { 
        test: /\.js$/, loader: 'babel'
      },
      {
        test: /\.less$/,
        loader: 'style!css!less'
        // loader: ExtractTextPlugin.extract('style-loader', 'css-loader!less-loader') // 将css独立打包
      },
      {
        test: /\.json$/, loader: 'json'
      }
    ]
  },
  plugins: [
    new webpack.BannerPlugin([
        'vConsole v' + pkg.version + ' (' + pkg.homepage + ')',
        '',
    ].join('\n'))
    // 开发调试项目的时候先去掉压缩命令，确认没有问题，生产版本再加上
    // ,new webpack.optimize.UglifyJsPlugin({
    //   compress: {
    //     warnings: false
    //   }
    // })
    // ,new ExtractTextPlugin('[name].min.css') // 将css独立打包
  ]
};
//package.json
{
  "main": "dist/vconsole.min.js",
  "scripts": {
    "test": "mocha",
    "dist": "webpack",
    "dev": "webpack-dev-server --config webpack.dev.conf.js"
  },
}

```

### 四、问题记录
1. header在window.XMLHtmlRequest里获取的方法是getAllResponseHeaders()。在fetch里面需要去遍历entries
```js
for (let pair of response.headers.entries()) {
  item.header[pair[0]] = pair[1]
  console.log(pair[0])
}
```
2. response是json的时候，面板打印不出来具体数据，打印的是[object object]，看了下是vconsole里面重写了JSON.stringify()，重写的作用是？


