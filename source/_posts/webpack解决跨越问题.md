---
title: webpack解决跨越问题
date: 2018-09-21 18:34:25
categories: 
- react
tags:
- webpack
- cors
---
## 一、场景
  本机起一个web的服务，访问后台接口
  ```
  window.fetch('https://dp-admin.test.shopee.io/api/order/list?page=1&page_size=10', {method: 'get'}).then(response => {
    return response.json()
  }).then(data => {
    console.log(data)
  })
  ```
  跨域报错如下，可以说很日常了
  ```
  localhost/:1 Failed to load https://dp-admin.test.shopee.io/api/order/list?page=1&page_size=10: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8080' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
  ```
## 二、本地搭建一个服务器做转发
   ```
   const request = require('request')
   const express = require('express')
   const app = express()

   const testHost = 'http://dp-admin.test.shopee.io'
   const dpHost = process.env.HOST || testHost

   app.use('/api', (req, res, next) => {
    console.log('req---->', dpHost + req.originalUrl)
    const requestObj = request[req.method.toLowerCase()]({
      url: dpHost + req.originalUrl
    })
    req.pipe(requestObj)
    requestObj.pipe(res)
   })
   app.listen(3001)

   ```
   浏览器访问http://localhost:3001/api/order/list?page=1&page_size=10返回如下
   ```
    {
      "err_code": 32,
      "err_msg": "not login",
      "data": null
    }
   ```
   证明我们写的代理服务请求没问题
## 三、 设置webpack的devServer
   ```  
    proxy: {
      '/api': 'http://localhost:3001'
    }
   ```
   重新请求报错和刚才一样，为什么呢？ 页面的访问地址是http://localhost:8080/ 我们请求的地址是https://dp-admin.test.shopee.io/api/order/list?page=1&page_size=10，这里http://localhost:8080/ 和https://dp-admin.test.shopee.io明显不是一个域名，也就是说请求和devServer服务器是跨域的，则不存在devServer正常响应重新转发到'http://localhost:3001'，本地express服务是没有工作的。所以我们改下代码入下
   ```
    // http://localhost:8080//api/order/list?page=1&page_size=10等效，保证和devServer同域名就好
    window.fetch('/api/order/list?page=1&page_size=10', {method: 'get'}).then(response => {
      return response.json()
    }).then(data => {
      console.log(data)
    })
   ```
   果然跨域问题解决了，express也看到了日志进去了
   ```
   {"err_code":32,"err_msg":"not login","data":null}
   ```
   返回没有登陆，我们实际的页面已经登陆了https://dp-admin.test.shopee.io/ ，我们本地的页面地址是http://localhost:8080/ 这种情况下cookie明显写不进去，所以做如下设置

   ```
     devServer: {
        port: '8080',
        host: 'dp-admin.test.shopee.io',
        contentBase: path.join(__dirname, '../public'),
        compress: true,
        historyApiFallback: true,
        hot: true,
        https: false,
        noInfo: true,
        open: true,
        proxy: {
        '/api': 'http://localhost:3001'
    }
  }
   ```
   运行 npm run dev，报错如下
   ```
   > webpack-dev-server --inline --progress --config build/webpack.dev.conf.js

    10% building modules 1/1 modules 0 activeevents.js:167
        throw er; // Unhandled 'error' event
        ^

    Error: listen EADDRNOTAVAIL 203.117.178.74:8080
        at Server.setupListenHandle [as _listen2] (net.js:1313:19)
        at listenInCluster (net.js:1378:12)
        at GetAddrInfoReqWrap.doListen [as callback] (net.js:1491:7)
        at GetAddrInfoReqWrap.onlookup [as oncomplete] (dns.js:55:10)
    Emitted 'error' event at:
        at emitErrorNT (net.js:1357:8)
        at process._tickCallback (internal/process/next_tick.js:174:19)

   ```
   增加本地代理
   ```
   sudo vim /etc/hosts
   127.0.0.1 dp-admin.test.shopee.io
   ```
   npm run dev
   ```
   ➜  webpack-cross git:(master) ✗ npm run dev

    > webpack-cross@1.0.0 dev /Users/weiqian/Desktop/shopee/webpack-cross
    > webpack-dev-server --inline --progress --config build/webpack.dev.conf.js

    [HPM] Error occurred while trying to proxy request /api/order/list?page=1&page_size=10 from dp-admin.test.shopee.io:8080 to http://localhost:3001 (ECONNRESET) (https://nodejs.org/api/errors.html#errors_common_system_errors)
    // ECONNRESET (连接被重置): 一个连接被强行关闭。 这通常是因为连接到远程 socket 超时或重启。 常发生于 http 和 net 模块。
    // 浏览器显示错误
   index.js:9 GET http://dp-admin.test.shopee.io:8080/api/order/list?page=1&page_size=10 504 (Gateway Timeout)
   // prxoy服务错误
    node data/proxy.js

    req----> http://dp-admin.test.shopee.io/api/order/list?page=1&page_size=10
    internal/streams/legacy.js:57
        throw er; // Unhandled stream error in pipe.
        ^

    Error: connect ECONNREFUSED 127.0.0.1:80
        at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1161:14)
    npm ERR! code ELIFECYCLE
    npm ERR! errno 1
    npm ERR! webpack-cross@1.0.0 proxy: `node data/proxy.js`
    npm ERR! Exit status 1
    npm ERR!
    npm ERR! Failed at the webpack-cross@1.0.0 proxy script.
    npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

    npm ERR! A complete log of this run can be found in:
    npm ERR!     /Users/weiqian/.npm/_logs/2018-09-21T12_35_13_611Z-debug.log

    // 我们访问线上页面 https://dp-admin.test.shopee.io/ 错误
   This site can’t be reached
    dp-admin.test.shopee.io refused to connect.
    Try:

    Checking the connection
    Checking the proxy and the firewall
    ERR_CONNECTION_REFUSED
   ```
   我们设置了真实线上的域名，运行本地页面，请求被devServer转发到3001的express服务器，接着express访问了被解析到本地的机器上ECONNREFUSED，所以明显需要改配置页面不能为线上真实地址
   ```
    // webpack
    devServer{
        host: 'local.dp-admin.test.shopee.io'
    } 
    // /etc/hosts
    127.0.0.1 local.dp-admin.test.shopee.io

   ```
   重新运行npm run proxy、 npm run dev错误是一样的，正常线上页面也无法访问。哦我们忘了删掉host里面的配置，express出错，删掉host配置的   127.0.0.1 dp-admin.test.shopee.io，世界和平啊！！！
   [demo地址](https://github.com/weiqian93/react-demo/tree/master/webpack-cors)

## 四、create-my-app设置
  思路和webpack差不多 
  1. 起本地服务(服务端请求不存在跨域问题) 
  2. package.json设置proxy(跨域)和host(cookie)
  ```
  "scripts": {
    "proxy": "node data/proxy.js",
    "start": "HOST=local.dp-admin.test.shopee.io react-scripts start",
  },
  "proxy": {
    "/api": {
      "target": "http://localhost:3001/"
    }
  }
  ```
  测试代码
  ```
  componentDidMount () {
    window.fetch('/api/order/list?page=1&page_size=10', {method: 'get'}).then(response => {
      return response.json()
    }).then(data => {
      console.log(data)
    })
  }
  ```
  [demo地址](https://github.com/weiqian93/react-demo/tree/master/my-app-cors)





