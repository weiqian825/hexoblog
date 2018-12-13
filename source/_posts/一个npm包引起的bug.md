---
title: 一个npm包引起的小bug
date: 2018-12-13 10:32:16
tags:
---

## 一、问题
```
测试到了尾声，测试小姐姐忽然找到说filter点击不生效，一脸蒙蔽发生什么，本地测试是OK的啊
```
## 二、猜想

1. 重启大法好，node_modules，重新安装下，没卵用
2. webpack的prodution和development版本不一样？
```
a. 把dev.js的mode配置由development改为production
b. 给prod.js里添加devServer配置，运行prod.js
c. 用python起一个简单的服务器 python -m SimpleHTTPServer 8001
...
没找到问题
```
3. 点击按钮无效，A原生标签的小手也不见了，应该是被什么东西挡住了，查看dom节点
```
对比测试线上版本和本地版本，果然看见了dom节点不一样，本地的没有:before，线上有个before挡住了按钮，所以点击不到
a. 样式污染了吗？ 找配置的问题，modulecss都是有的啊
b. 去antd官网看下，它们也有:before，本地居然没有，但是没更新版本啊

```
4. antd出毛病了吧，去搜下issue [:brefore reset filter 这些关键字换着发都是OK的能搜出来的]
果然是 https://github.com/ant-design/ant-design/issues/13563

## 三、原因
是antd的问题没错了，这里还有一些疑问
1. 本地为什么是好的，按理说antd的问题本地也会有问题啊？删掉node_modules重装为什么不生效？

查看[npm文档](https://docs.npmjs.com/files/package-lock.json) https://www.zhihu.com/question/62331583  
a. npm 5.0.x 版本，不管package.json怎么变，npm i 时都会根据lock文件下载 [相关issue](https://github.com/npm/npm/issues/16866)
b. 5.1.0版本后 npm install 会无视lock文件 去下载最新的npm  [相关issue](https://github.com/npm/npm/issues/17979)
c. 如果改了package.json，且package.json和lock文件不同，那么执行`npm i`时npm会根据package中的版本号以及语义含义去下载最新的包，并更新至lock  [相关issue](https://github.com/npm/npm/issues/17979) 
```
➜  Windrunner git:(master) npm -v
6.4.1
-------------package.json为^6.5.0 第一次执行 npm i -----------------
{
    "dependencies": {
        "qs": "^6.5.0"
    }
}

{
  "requires": true,
  "lockfileVersion": 1,
  "dependencies": {
    "qs": {
      "version": "6.6.0",
      "resolved": "https://registry.npmjs.org/qs/-/qs-6.6.0.tgz",
      "integrity": "sha512-KIJqT9jQJDQx5h5uAVPimw6yVg2SekOKu959OCtktD3FjzbpvaPr8i4zzg07DOMz+igA4W/aNM7OV8H37pFYfA=="
    }
  }
}
------------改package.json为6.5.0 第二次执行npm i-----------------
{
    "dependencies": {
        "qs": "6.5.0"
    }
}
{
  "requires": true,
  "lockfileVersion": 1,
  "dependencies": {
    "qs": {
      "version": "6.5.0",
      "resolved": "https://registry.npmjs.org/qs/-/qs-6.5.0.tgz",
      "integrity": "sha512-fjVFjW9yhqMhVGwRExCXLhJKrLlkYSaxNWdyc9rmHlrVZbk35YHH312dFd7191uQeXkI3mKLZTIbSvIeFwFemg=="
    }
  }
}
------------改package.json为^6.5.0 第三次执行npm i-----------------
{
    "dependencies": {
        "qs": "^6.5.0"
    }
}
{
  "requires": true,
  "lockfileVersion": 1,
  "dependencies": {
    "qs": {
      "version": "6.5.0",
      "resolved": "https://registry.npmjs.org/qs/-/qs-6.5.0.tgz",
      "integrity": "sha512-fjVFjW9yhqMhVGwRExCXLhJKrLlkYSaxNWdyc9rmHlrVZbk35YHH312dFd7191uQeXkI3mKLZTIbSvIeFwFemg=="
    }
  }
}
------------改package.json为^6.5.0 第四次执行npm i-----------------
{
    "dependencies": {
        "qs": "^6.5.0"
    }
}
{
  "requires": true,
  "lockfileVersion": 1,
  "dependencies": {
    "qs": {
      "version": "6.5.0",
      "resolved": "https://registry.npmjs.org/qs/-/qs-6.5.0.tgz",
      "integrity": "sha512-fjVFjW9yhqMhVGwRExCXLhJKrLlkYSaxNWdyc9rmHlrVZbk35YHH312dFd7191uQeXkI3mKLZTIbSvIeFwFemg=="
    }
  }
}
------------rm package-lock.json第5次执行npm i---无变化--------------
------------rm -rf node_modules 第6次执行npm i---无变化--------------
------------rm -rf node_modules package-lock.json 第7次执行npm i---有变化--------------
回到了最初的地方，所以为了稳妥起见，如果你和我一样，不确定包的安装策略，又想安装到和服务端一样的包，那么就把node_modules和package-lock.json都删掉，再npm i
在定位问题的时候，我一直都删除node_modules重装，但实际上包一直都没有更新，本地的antd包是一个没有bug的包，线上jenkins在编译的时候没有lock版本，会一直更新，antd的bug也就被更新进来了。
```
2.  代码的package.json一直没有更新版本，为什么antd会有问题？
```
npm是围绕着语义版本控制（semver）的思想而设计的
给定一个版本号：主版本号.次版本号.补丁版本号， 以下这三种情况需要增加相应的版本号:

主版本号： 当API发生改变，并与之前的版本不兼容的时候
次版本号： 当增加了功能，但是向后兼容的时候
补丁版本号： 当做了向后兼容的缺陷修复的时候

"5.0.3" 表示安装指定的5.0.3版本
"~5.0.3" 表示安装5.0.X中最新的版本
"^5.0.3" 表示安装5.X.X中最新的版本

package.json的配置是 "antd": "^3.4.3", antd更新了，测试项目build时候也会自己更新到有bug的版本
```
## 四、疑问
[yarn](https://github.com/yarnpkg/yarn)可以解决版本非确定性的问题，我们为什么没用呢？不少讨论的，其实差别也还好～
[相关讨论](https://blog.risingstack.com/yarn-vs-npm-node-js-package-managers/)




   
