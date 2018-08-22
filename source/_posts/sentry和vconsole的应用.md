---
title: 'sentry和vconsole的应用 '
date: 2018-08-22 19:27:28
tags:
---
## 一、关于sentry
[sentry](https://sentry.io/welcome/)业界最流行监控的开源框架，可以说是很多公司的监控系统的基础平台之一。[sentry本地搭建](https://juejin.im/post/5b226cbe51882574d02f9f62)可以自己试试哈，公司有自建的[站点](https://sentry.shopeemobile.com/sentry/)我们项目使用的react框架，接入方式查阅[官方文档](https://docs.sentry.io/clients/javascript/integrations/react/)即可。
```
import Raven from 'raven-js'
Raven.config('https://<key>@sentry.io/<project>').install()
```
## 二、关于vconsole
[vconsole](https://github.com/Tencent/vConsole)是微信前端团队出的一款可以在移动端打印log的工具，我们用户在不同的几个国家，开发面临无法复现用户的网络环境，接入vconsole应该会帮助我们解决一部分定位问题的困难，所以接入试试效果，后期看看效果。

## 三、接入vconsole
1. 方法一: 引入[vconsole-webpack-plugin](https://github.com/diamont1001/vconsole-webpack-plugin)
   问题：我们想要线上带有特定参数的链接出console控制面板，这个只能区分不同的部署环境例如test、uat、live等等，无法做到在一个环境里表现不一样，webpack动态加载打包内容是在用户访问前就已经确认的。
```
// 引入插件
var vConsolePlugin = require('vconsole-webpack-plugin'); 
module.exports = {
    plugins: [
        new vConsolePlugin({
            filter: [],  // 需要过滤的入口文件
            enable: true // 发布代码前记得改回 false
        })
    ]
}
```
2. 方法二: 在public的index.html文件里根据url不一样动态的插入script标签，引入vconsole的js库
   问题: 是可行的，只不过总觉得不是react项目的正常写法，不优雅怪怪的。
```
    // url里面带有特定参数时候，动态插入js库，js库也在public里面放着
    <script type="text/javascript" src="./vconsole.js"></script>
```
3. 方法三: 用import在需要的时候引入vconsole的代码库
```
    // 在满足条件的时候
    import('./vconsole.js').then((VConsole) => {
        const vconsole = new VConsole()
        console.log(vconsole)
    })
```
## 四、vconsole问题
1. 代码编译不过，报错变量没找到，猜想语法检查没通过，于是各种改package.json里面的standard设置，如下
```
  "standard": {
    "parser": "babel-eslint",
    "globals": [
      "it",
      "VConsole"
    ],
    "ignore": [
      "**/vendor/**/*.js",
      "**/*.test.js",
      "**/src/*.js"
    ]
  }
```
2. 依旧不对，是不是编译引入的js文件不正确，各种引入不同的js包。
3. 依旧不对，终于在一次尝试引入了modules下面的包import('vconsole')编译通过了，于是我们猜想是方向不对，应该是create-my-app在编译的过程中做了某种[语法检查](https://stackoverflow.com/questions/50672331/failed-to-compile-create-react-app-due-to-lint-warnings-after-ejecting)
4. 检查webpack.config.base.js果然发现如下代码
```
  {
        test: /\.(js|jsx|mjs)$/,
        enforce: 'pre',
        use: [
          {
            options: {
              formatter: eslintFormatter,
              eslintPath: require.resolve('eslint')

            },
            loader: require.resolve('eslint-loader')
          }
        ],
        include: paths.appSrc
      },
```
  改正后
```
    // include: paths.appSrc,
    exclude: [/vconsole.js/]
```
5. 语法的报错果然没有了，完美。
6. 打开体验的时候，发现network竟然没有抓到包，issue里面也没有提到这个问题，简单测试后，发现ajax可以抓到，我们的项目里面使用的是window.fetch竟然没有支持。
7. 额，看看怎么改下他们的代码吧。。。。
      
   
   

