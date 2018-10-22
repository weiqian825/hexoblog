---
title: express和mockjs搭建mock服务
date: 2018-10-22 20:54:49
categories: 
- react
tags:
- mockjs
- express
---

## 一、express起一个本地服务
1. [express](http://www.expressjs.com.cn/guide/using-middleware.html) 设置中间件起一个server端服务 mockjs.js
```
const express = require('express')
const app = express()
const banner = require('./routes/banner')

app.use('', banner)

app.listen(3001, function () {
  console.log('app listening')
})

```
2. mock具体数据
```
const express = require('express')
const Mock = require('mockjs')
const router = express.Router()

router.get('/display_plan', (req, res) => {
  let ret = {
    data: [{
      imgInfo: []
    }],
    err_code: 0,
    err_msg: 'success'
  }
  for (let index = 0; index < 4; index++) { // eslint-disable-line
    var data = Mock.mock({
      name: Mock.Random.name(),
      url: Mock.Random.url(),
      location: Mock.Random.image(),
      ordinal: index + 1
    })
    ret.data[0].imgInfo.push(data)
  }
  res.json(ret)
})

module.exports = router

```
3. package.json里面添加
```
"scripts": {
  "mock": "node index.js"
}
```
4. npm run mock 访问 http://localhost:3001/digital-product/api/banner/display_plan正常，则服务端的代码写的没问题

## 二、 webpack devServer设置proxy解决跨域
1. 设置devServer
```
  devServer: {
    proxy: {
      '/digital-product/api': 'http://localhost:3001'
    }
  }
```
这样localhost:8080访问接口/digital-product/api/banner/display_plan时候，会被devServer转发到3001端口，访问到了我们的mock server，这里并不存在浏览器的跨域问题，正常拿到我们mock的数据。
2. 如果cgi你的访问的路径不是localhost，记得页面相应的url设置成和cgi一样的域名，再配置host即可
