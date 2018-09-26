---
title: react-i18next的应用及工作原理
date: 2018-09-26 10:37:57
tags:
---
## 一、相关资料
我理解的i18n的原理就是字符串替换。根据语言拉取不同的json文件，再根据key做一个映射取到预期的结果。
[i18next官方文档](https://www.i18next.com/)
[react-i18next官方文档](https://react.i18next.com/)

## 二、i18next的基本API
1. init 默认的的i18next模块是一个已经被init初始化后的i18next实例
```
// i18next.init(option,callback) 
// 具体的[option](https://www.i18next.com/overview/configuration-options)
// i18next instance
const resources = process.env.LANGUAGE.resources
const language = Object.keys(resources)[0]
i18next
  .init({
    lng: language, // language to use
    fallbackLng: language, // language to use if translations in user language are not available
    defaultNS: 'common',// default namespace used if not passed to translation function​
    keySeparator: false, //char to separate keys
    debug: process.env.NODE_ENV === 'development',
    resources, // resources to initialize with (if not using loading or not appending using addResourceBundle)
    interpolation: {
      escapeValue: false
    },
    react: {
      wait: true,
      bindI18n: 'languageChanged loaded',// which events trigger a rerender, can be set to false or string of events
      bindStore: 'added removed',// which events on store trigger a rerender, can be set to false or string of events
      nsMode: 'default' // default: namespaces will be loaded an the first will be set as default or fallback: namespaces will be used as fallbacks used in order provided
    }
  })
```
2. t
```
//i18next.t(keys,options)
//i18next.t('my.key') // will return value in set language
const firstLetterUpper = (str, allWords = true) => {
  let tmp = str.replace(/^(.)/g, $1 => $1.toUpperCase())
  if (allWords) {
    tmp = tmp.replace(/\s(.)/g, $1 => $1.toUpperCase())
  }
  return tmp
}
export const tUpper = (str, allWords = true) => {
  return firstLetterUpper(i18next.t(str), allWords)
}
```
3. changeLanguage 切换语言
```
//i18next.changeLanguage(lng, callback)
i18next.changeLanguage('en', (err, t) => {
  if (err) return console.log('something went wrong loading', err);
  t('key'); // -> same as i18next.t
});
export const changeLanguage = (locale) => {
  i18next.changeLanguage(locale)
}
```
## 三、react部分
1. 引入react-i18next,用I18nextProvider将i18注入到componet(例子里面是App)
```
ReactDOM.render(
  <I18nextProvider i18n={i18n}>
    <App />
  </I18nextProvider>,
  document.querySelector('#root')
)
```
2. 在遇到需要国际化的地方调用translation即可{tUpper('xxx_key')}
```
class App extends React.Component {
  render () {
    return <h1>{tUpper('order_id')}</h1>
  }
}
```
## 四、本地翻译文件部分 
上面部分展示了i18n是怎么工作的，在实际的项目中，我们会有很多的需要翻译的文件，如何工程化的组织和加载这些文件呢，我们的目录结构如下
i18n
├── index.js //自动拉取线上的翻译文件，避免手动导入。 script里面可以放脚本 node i18n/index.js 先执行这个，在执行正常工作目录
├── locales //翻译文件具体内容
│   ├── en.json
│   ├── id-ID.json
│   ├── ms-MY.json
│   ├── th-TH.json
│   ├── vi-VN.json
│   ├── zh-Hans.json
│   └── zh-Hant.json
├── localesHash.js //国家对应的语言hash
└── resourcesHash.js //语言对应的资源文件包

## 五、webpack加载配置
```
//国家对应的语言hash
const localesHash = require('./i18n/localesHash')
//语言对应资源hash
const resourcesHash = require('./i18n/resourcesHash')
//webpack.config.js
const config = (env = {}, argv) => {
  const isProductionMode = argv.mode === 'production'
  const country = (env.country || 'en').toUpperCase()
  const locales = (localesHash[country] || []).concat(['en'])
  const resources = locales.reduce(
    (previous, current) => {
      previous[current] = {
        common: resourcesHash[current]
      }
      return previous
    }, {})
  return {
    plugins: [
      new webpack.EnvironmentPlugin({
        NODE_ENV: isProductionMode ? 'production' : 'development',
        SERVER_ENV: env.server || 'test',
        COUNTRY: country,
        LANGUAGE: {
          resources
        }
      })
    ]
  }
}
```
## 六、相应的package.json配置
```
{
  "scripts": {
    "debug:id": "webpack-dev-server  --progress  --env.country=id   --config  ./webpack.config.js",
    "debug:th": "webpack-dev-server  --progress   --env.country=th  --config  ./webpack.config.js",
    "start": "webpack-dev-server --mode  development  --config  ./webpack.config.js",
    "i18n": "node i18n/index.js",
    "build": "webpack --config  ./webpack.config.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```
## 七、问题记录
1. 每次正常启动访问http://localhost:9000之后总是自动跳到http://localhost:9000/auth/login，看了代码里面并没有任何关于权限的跳转逻辑，怀疑是本机ngnix或者别的程序影响的
```
➜  init-i18n git:(master) ✗ lsof -i :9000
COMMAND   PID    USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
com.docke 655 weiqian   18u  IPv4 0x673df5045a8519a5      0t0  TCP *:cslistener (LISTEN)
com.docke 655 weiqian   19u  IPv6 0x673df5045531595d      0t0  TCP localhost:cslistener (LISTEN)
➜  init-i18n git:(master) ✗ kill -9 655
➜  init-i18n git:(master) ✗ lsof -i :9000
➜  init-i18n git:(master) ✗ npm run start
```
果然，docker的进程影响到，在重启世界和平。

2. 刚开始怎么写代码都只能翻译英语的，智商捉鸡，orz
```
//有bug的代码
//webpack.config.js里面
const country = (env.country || 'en').toUpperCase()
const locales = ['en'].concat(localesHash[country]) // wrong
// const locales = (localesHash[country] || []).concat(['en']) // right
const resources = locales.reduce(
    (previous, current) => {
        previous[current] = {
        common: resourcesHash[current]
        }
        return previous
    }, {})
//i18n里面选择的
const resources = process.env.LANGUAGE.resources
//这里选择默认选择的是第一个，所以webpack里面的resource顺序就很重要
const language = Object.keys(resources)[0]
```
  错误地方好弱智
```
const locales = ['en'].concat(localesHash[country])
// locales的第一个永远是en那么选择策略 Object.keys(resources)[0] 永远都选第一个
//改正到正确代码
const locales = (localesHash[country] || []).concat(['en'])
```

   



 
