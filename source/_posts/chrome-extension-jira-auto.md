---
title: chrome-extension-jira-auto
date: 2018-09-06 16:38:11
categories: 
- web
tags:
- chrome-extension
- jira
---
[乞丐版的GUI](https://weiqian93.github.io/2018/08/30/electron%E5%81%9Ago%E7%88%AC%E8%99%AB%E7%9A%84gui/)可以抓取jira，生成本地的csv，我们希望这个东西能直接写到chrome drive里面去，每个人安装一个exe，是有些大材小用了。于是开始重新规划搞一个[goole插件](https://developer.chrome.com/extensions)，负责抓取数据和写入drive。[源码](https://github.com/weiqian93/jira-auto/tree/chrome-extension-demo)

### 一、 jira的数据处理

1. [jira官网](https://developer.atlassian.com/cloud/jira/platform/)有相关接口文档，我们自己的jira平台也可抓取请求包。demo测试下
插件的demo我们参考了一些[简单的教程](https://github.com/welearnmore/chrome-extension-book)和[书籍资料](http://www.ituring.com.cn/book/1421 )
```json
//manifest.json
{
  "version": "0.0.1",
  "name": "jira-demo",
  "manifest_version": 2,
  "description": "jira-demo",
  "browser_action": {
    "default_title": "jira-demo"
  },
  "content_scripts": [
    {
      "matches": ["https://jira.garenanow.com/*"],
      "js": ["content_scripts.js"]
    }
  ],
  "permissions": [
    "cookies", 
    "<all_urls>"
  ],
  "content_security_policy": "script-src 'self' 'unsafe-eval'; object-src 'self';"
}
```
```javascript
// content_scripts.js
var searchUrl = 'https://jira.garenanow.com/rest/api/2/search'
window.fetch(searchUrl)
    .then(function (response) {
      if (response.status === 200) {
        return response.json()
      } else if ([400, 401].indexOf(response.status) > -1) {
        window.alert('您还没登陆jira呦～去登陆')
        window.chrome.tabs.create({ url: 'https://jira.garenanow.com/secure/Dashboard.jspa' })
      }
    })
    .then(function(myJson){
        console.log(myJson)
    })

```
测试ok，根据jql语法完善下查询条件，demo测试ok，预言这部分完全可以替代go爬虫，更简单一些
```sql
var searchUrl = `https://jira.garenanow.com/rest/api/2/search?jql=
                project IN ("SPDGT", "Test")
            AND issuetype IN ("Task", "Sub-task")
            AND resolution IN ("Unresolved", "Done")
            AND component IN ("FE","BE")
            AND status IN ("To Do", "In Progress", "Closed")
            AND assignee IN ("qian.wei@xxxxx.com")`
```
这里有一个遗留的问题，就是jira的登陆态问题，刚开始猜想是否可以直接从插件部分输入一次用户的密码和账户，测试发现同样的请求接口，cookie里面的succLogin一直返回false，并且缺少了正常登陆态cookie里面的sessionid，估计这里平台这里做了一些登陆的处理，所以我们登陆模块暂时处理为，引导用户登陆jira，然后再通过插件正常查询jira里面的数据，体验上也不会有太大问题。

### 二、 写入chrome drive文档

[谷歌开发者文档](https://developers.google.com/sheets/api/guides/values)里面支持多种开发方式，在快速引导里面分别体验了 browser、google apps script、node.js及python，发现一些问题，貌似并不能和插件有些不一样，插件是纯css html的静态脚本，文档里的例子显然相对复杂。

1. google apps script类似在谷歌文档里面保存里一段script，需要生成一些模版选择从文档页面插入，有点类似编代码时候快捷键快速生成代码。
2. node也起了一个服务，起了一个服务端资源，我们的初衷希望是一个纯插件，并不想引入服务端的概念，尽可能简单。
3. browser和python方法类似，问题同2。中间尝试是否可以用python写代码编译成一个纯插件，后来并没有开始做因为我们找到了看起来更简单的方法，即4。
4. ayou在翻阅chrome sheets文档测试时候，发现插件可以调用，如下。
```json
// manifest.json
{
    "version": "0.0.1",
    "name": "sheets-demo",
    "manifest_version": 2,
    "description": "sheets-demo",
    "browser_action": {
      "default_title": "sheets-demo"
    },
    "background": {
      "page": "background.html",
      "persistent": false
    },
    "permissions": [
      "cookies",
      "<all_urls>",
      "background",
      "identity",
      "notifications",
      "storage",
      "alarms",
      "https://spreadsheets.google.com/",
      "https://docs.google.com/"
    ],
    "oauth2": {
      "client_id": "624254244178-9f527anv43dhfs5ghii10bdgo2jbii3e.apps.googleusercontent.com",
      "scopes": [
        "https://spreadsheets.google.com/feeds",
        "https://docs.google.com/feeds",
        "https://www.googleapis.com/auth/plus.login",
        "https://www.googleapis.com/auth/drive",
        "https://www.googleapis.com/auth/spreadsheets"
      ]
    }
  }
```
 ``` 
// background.html
<script type="module" src="background.js"></script>
 ```
```
//background.js
import GSheet from './gsheet.js'
const gSheet = new GSheet()
var  data = [['title1','title2'],[11,12],[21,22]]
generateSheet(data)
async function generateSheet (data) {
  await gSheet.getAuthToken()
  const res = await gSheet.createSheet({
    'properties': {
        title:'test-demo'
    }
  })
  const { spreadsheetId } = res
  await gSheet.writeToSheet({
    spreadsheetId,
    range: 'Sheet1!A1:A',
    values: data
  })
  window.chrome.tabs.create({ url: 'https://drive.google.com/drive/u/0/my-drive' })
}
```
```javascript

//gsheet.js
const identity = window.chrome.identity

export default class GSheet {
  constructor () {
    this._token = ''
  }

  _fetch ({
    url,
    method = 'GET',
    body
  }) {
    url = 'https://content-sheets.googleapis.com/v4/spreadsheets' + url

    const params = {
      method,
      headers: {
        'Authorization': 'Bearer ' + this._token,
        'Content-Type': 'application/json'
      }
    }

    if (body) {
      params.body = JSON.stringify(body)
    }

    let status = 0
    return new Promise((resolve, reject) => {
      window.fetch(url, params)
        .then(rsp => {
          status = rsp.status
          switch (status) {
            case 200:
              return rsp.json()
            case 204:
              return null
            case 500:
              return {
                message: 'Internal server error.'
              }
            default:
              return {
                message: rsp.statusText
              }
          }
        })
        .then(body => {
          if (status === 200) {
            resolve(body)
          }
        })
    })
  }

  getAuthToken () {
    return new Promise((resolve, reject) => {
      identity.getAuthToken({
        'interactive': true
      }, token => {
        console.log(token,token)
        identity.removeCachedAuthToken({ token }, () => {
          identity.getAuthToken({
            'interactive': true
          }, token => {
            this._token = token
            resolve()
          })
        })
      })
    })
  }

  writeToSheet ({
    spreadsheetId,
    range,
    values
  }) {
    const url = `/${spreadsheetId}/values/${encodeURIComponent(range)}:append?insertDataOption=OVERWRITE&valueInputOption=RAW&alt=json`
    return this._fetch({
      url,
      method: 'POST',
      body: {
        range,
        values,
        'majorDimension': 'ROWS'
      }
    })
  }

  createSheet (params) {
    return this._fetch({
      url: '/',
      method: 'POST',
      body: params
    })
  }
}

```
background.js代码稍微有点多，我们简单的模块化下，引入[es6-modules-in-chrome-extensions](https://medium.com/front-end-hacking/es6-modules-in-chrome-extensions-an-introduction-313b3fce955b)

### 三、合并抓取数据和写入drive

[源码](https://github.com/weiqian93/jira-auto/tree/chrome-extension-demo)，再试用的时候发现问题，永远写入了一个人的账户，不管谁操作，猜想是client_id的问题，每个用户是否都在本地有唯一的授权id，毕竟操作google doc也算是比较敏感操作。测试后发现的确是这样，manifest.json里面oauth2部分的client_id是专属的，每个安装插件的人都需要申请google权限并且在manifest.json里面设置自己的专属client_id

```json
    "oauth2": {
      "client_id": "624254244178-9f527anv43dhfs5ghii10bdgo2jbii3e.apps.googleusercontent.com",
      "scopes": [
        "https://spreadsheets.google.com/feeds",
        "https://docs.google.com/feeds",
        "https://www.googleapis.com/auth/plus.login",
        "https://www.googleapis.com/auth/drive",
        "https://www.googleapis.com/auth/spreadsheets"
      ]
    }
```
### 四、配置google权限

1. 获取插件[代码](https://github.com/weiqian93/jira-auto/tree/chrome-extension-demo)
2. 获取插件 id
    打开[chrome插件](chrome://extensions/) 
    点击加载已解压的拓展程序，选择jira_auto整个目录，得到插件的id
3. 添加 google sheet api 库
   打开[Dashboard](https://console.developers.google.com/)
   输入 google sheet api => MANAGE
4. 修改 client id
    打开[credentials](https://console.developers.google.com/apis/credentials) 
    Credentials => Create Credentials => oAuth Client Id => Chrome App
    将第一步得到的 id 复制到 Application ID 一栏，点击创建得到 client ID
5. 将得到的 client id 拷贝下来覆盖掉项目下 manifest.json 中的 client_id 字段，然后刷新插件，接着就可以愉快的使用了。

### 五、小结

这里的实现比乞丐版的gui的开发代码是简单了许多，不过写chrome drive的操作真的比较繁琐了，权衡下是否可以考虑在插件里做更加丰富的查询生成比较好的本地模版内容，然后复制到google drive里面，更优呢？

