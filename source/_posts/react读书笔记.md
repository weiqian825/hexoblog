---
title: react读书笔记
date: 2019-08-17 13:03:39
tags:
---
## 一、react出现背景
```
1. 出现的问题根源，2013年fb的简单功能一再出现bug

2. 传统的UI操作(DOM api)关注了太多的细节（一般是用jq来进行局部更新）
react始终整理刷新页面

  新概念：组件
  4个必须的API
  单向数据流
  完善的错误提示

3. 应用程序状态分散在各处，难以追踪和维护，数据模型如何解决？
  传统的mvc难以扩展和维护
  controller -> model -> view
  Flux架构(设计模式)：单向数据流         
                               actions--------> Dispatcher----callbacks
                                  |                               |
  webApi <=> webApiUitils <=> actionCreators                     store
                                  |                               |        
                              userInteractions---reactViews<---changeEvents  +storeQueries
  
4. Flux架构的衍生项目 redux + mbox
```
## 二、以组件方式考虑ui的构建
```
1. 将UI组织成组件树的形式

2. 理解react组件
  props + state -> view
  a react组件一般不提供方法，而是某种状态机
  b react组件可以理解为纯函数
  c 单向数据绑定

3. 受控组件: 表单元素状态由外部维护
  vs 非受控组件: 表单元素状态由DOM自身维护

4. 何时创建组件： 单一职责原则
  a 每个组件只做一件事
  b 如果组件变得复杂，那么应该拆分成小组件

5. 数据状态管理：DRY原则（不要去做重复的状态保存）
  a 能计算得到的状态就不要单独存储
  b 组件尽量无状态 所需数据由props获得

```
## 三、JSX本质：不是模版语言，只是语法糖
```
JSX: 在javascript代码中直接写html标记

1. JSX本质：动态创建组件的语法糖
  eg1 jsx(是用javascript本身的特性，而不是用一种新的模版模版)
  const name ='john'
  const ele = <h1>hello</h1>
  eg2 javascript
  const name = 'john'
  const ele = React.createElement('h1',null,'hello',name)

2. 在jsx中使用表达式
  a jsx本身也是表达式
  b 在属性中使用表达式
  c 延展属性
  d 表达式作为子元素

3. jsx的优点
  a 声明式创建界面的直观
  b 代码动态创建界面的灵活
  c 不用学习新的模版语言

4. 约定： 所有的自定义组件以大写字母开头
  a react认为小写tag是原生dom节点，如div
  b 大写字母开头为自定义组件
  c jsx标记是可以直接使用属性语法的 如<item.Menu>
```

## 四、react组件的生命周期
```
                                  |  创建时        更新时      卸载时
  render阶段                       |  constructor
  [纯净且没有副作用，可中止重启]       |  getDerivedStateFromProps
                                   |  shouldComponentUpdate(此处可以做性能优
  -----------------------------------------------------------------                 
   Pre-commit阶段[读取Dom]             | getSnapshotBeforeUpdate        
  -------------------------------------------------------------------
  commit阶段[使用Dom，运行副作用，安排更新] |   componentDidMount|  componentDidUnmount|componentDidUnmount                                          
```
```
1. constructor 
  a 初始化内部状态，比较少使用
  b 唯一可以直接修改state的地方

2. getDerivedStateFromProps 
  a 当state需要从Props初始化时候使用
  b 尽量不要使用： 维护2者状态一致性会增加复杂度
  c 每次render都会调用
  e 典型场景： 表单控件获取默认值

3. componentDidMount
  a UI渲染完成后调用
  b 只执行一次
  c 典型场景：获取外部资源

4. componentDidUnmount
  a 组件移除时候调用
  b 典型场景：释放资源

5. getSnapshotBeforeUpdate
  a 在页面render之前会调用，state已更新
  b 典型场景： 获取render之前的dom状态

6. componentDidUpdate
  a 每次UI更新时被调用
  b 典型场景： 页面需要根据Props的变化重新获取数据

7. shouldComponentUpdate
  a 决定Virtual Dom是否需要重绘
  b 一般可以由PureComponent自动实现
  c 典型场景：性能优化
```
## 五、Virtual Dom (Diff算法)
```
1. 虚拟Dom工作原理
  标准的2个tree diff 0n3次方
  react Diff算法O(n)
2. 广度优先分层比较
3. 节点的跨层移动
  删除原来的，创建新的节点(性能问题？)
4. 虚拟dom的2个假设
  A组件dom结构其实相对稳定
  B类型相同的兄弟节点可以唯一被标识（key不仅是消除warning，而 且是提升性能）

```
## 六、高阶组件（HOC）
```
组件复用的另外2种形式：高阶组件和函数作为子组件

1. 高阶组件
  const EnhancedComponent = higerOrderComponent (WrapperdComponent)
  a 对已有组件进行封装，形成一个新组建，新组件形成包含自己的应  用逻辑和新状态传给已有组件。
  b 高阶组件一般不会有自己的UI展现，而是为它封装的组件提供一些新的方法和功能。
c 高阶组件接受组件作为参数，返回新的组件

2. 函数作为子组件
  <MyComponent>{(name) => (<div>{name}</div>)} </MyComponent>
  一个组件如何render的内容可以很大程度上由使用它的让你来决定
```
## 七、contextApi
```
1. contextApis使用方法
  根节点 provide
  使用contextApi consume
  const TestContext = React.createContext('light')
  class App extends React.Component {
    render() {
      return (<TestContext.Provider value='dark'>   <TestButton /><TestContext.Provider />)
    }
  }
  function TestButton(props) {
    return (<TestContext.Consumer>{theme => <button   {...props} theme={theme}/>}</  TestContext.Consumer>)
  }
  Context.Provider一定要包住Context.Consumer
2. 使用场景： i18n

```
## 八、使用脚手架工具

```
1. 使用脚手架工具创建React应用
  create react app 
  codesandbox.io
  rekit

2. 为什么需要脚手架工具
  react ui开发
  redux 状态管理
  react/router 路由管理
  babel 将最新的javascript特性翻译成浏览器兼容的旧  javascript
  webpack进行打包
  eslint进行语法检查

3. 各个脚手架对比
  create react app [babel+webpack config +testing + eslint]适合简单的新学习的，简单的application
  rekit[redux+reactRouter+less/sass  +featureOrientedArchitecture+DedicatedIDE]更丰富的配置
  codesandbox 在线的IDE脚手架，不需要搭建本地环境就可以直接开发，webpack打包的是运行在浏览器端的，很牛逼，平常都是在node端运行webpack

```
## 九、打包和部署
```
1. 为什么要打包
  a 编译ES6语法特性，编译JSX
  b 整合资源，例如图片，Less/Sass
  c 优化代码的体积

2.使用webpack进行打包
  a nodejs环境为production
  b 禁用开发时候的专用代码，比如logger
  c 设置应用跟路径

```

## 十、Redux（1）JS状态管理框架
```
1. 为什么需要redux
  react组件工作：把state转换成DOM结构（组件内部发生）
  redux工作：store->DOM(把状态放到组件外部管理)
  redux store里面的结构也是一个类似Tree的结构
  [链接](https://css-tricks.com/learning-react-redux/)
  解决组件通信: 组件之前的通信通过store来完成

2. Redux特性
  a single source of truth
    view
    view --> store
    view
  b 可预测性 state + action = new sate
  c 纯函数[输出结果完全取决于输入参数，不依赖任何外部]更新store

```
## 十一、Redux（2）深入理解store action
```
1. 理解Store
  const store = createStore(reducer)
  a getState
  b dispatch(action)
  c subscribe(listener)
  |---------------------| 
  |  dispatcher action  | 
  |     |               | 
  |   reducer  处理action| 
  |     |      改变store | 
  |   state              | 
  |----------------------| 

2. 理解action(描述行为的数据结构)
  {
      type: ADD_TODO
      text: 'build my app'
  }

3. 理解reducer
  function todoApp(state = intialState, action) {
    switch (action.type) {
      case ADD_TODO:
        return Object.assign({}, state, {
          todos: [
            ...state.todos,
            {
              text: action.text,
              completed: false
            }
          ]
        })
      default: 
        return state
    }
  }
(state,action) => new state

4. 理解combineReducers 
function bindActionCreator(actionCreator, dispatch) {
  return function () {
    return dispatch(actionCreator.apply(this, arguments))
  }
}
function bindActionCreators(actionCreator, dispatch) {
  const keys = Object.keys(actionCreators)
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++ >) {
    const key = keys[i]
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}

5. pureRedux Demos
// pureRedux
import React from 'react'
import { createStore, boundActionCreators } from 'redux'

function run() {
  //Store initial state
  const initialState = { count: 0 }
  // reducer
  const counter = (state = initialState, action) => {
    switch (action.type) {
      case: 'PLUS_ONE':
        return { count: state.count + 1 }
      case: 'MINUS_ONE':
        return { count: state.count - 1 }
      default:
        break;
    }
    return state
  }
  const todos = state => state
  const re
  //create store
  //const store = createStore(counter)
  const store = creatorStore(combineReducers({
      todos,
      counter
  }))
  //action creator
  function plusOne() {
    return { type: 'PLUS_ONE' }
  }
  function minusOne() {
    return { type: 'PLUS_ONE' }
  }
  

  store.subscribe(() => console.log(store.getState()))
  store.dispatch(plusOne)
  //store.dispatch(minusOne)
  minusOne = boundActionCreators(minusOne)
  minusOne()
}
```
## 十二、Redux（3）在react中使用redux

1. component =>connect=> store
```
import { connect } from 'react-redux'
class SidePanel extends Component{

}

function mapStateToProps(state){
    return {
        nextGen: state.nextGen,
        router: state.router
    }
}
function mapDispatchToProps(dispatch){
    return {
        actions: bindActionCreators({...actions},dispatch)
    }
}

export default connect(mapStateToProps,mapDispatchToProps)(SildePannel)

```
2. connect 工作原理： 高阶组件
```
    connect(Mycomp) <---store<---
Mycomp <--传入props             reducers
                    <---actions--->
```
## 十三、Redux（4）理解异步Action Redux中间件
[异步ReduxAction](/images/异步redux.png)
```
1. redux异步请求：到Middleware进行预处理，再形成最后的action然后dispatch。异步ReduxAction比同步ReduxAction多了一个Middleware，Middleware帮助实现异步action流程
异步action并不是redux的概念，而是一个action的设计模式，是几个同步action的组合

2. redux中间件Middleware
a 截获action(ajax由redux-thunk截获，是否截获是判断action是否是一个promise)
b 发出action
典型例子：logger
谷歌插件redux

3.demo
import axios from 'axios'
import {
  FETCH_LIST_BEGIN,
  FETCH_LIST_SUC,
  FETCH_LIST_FAIL
} from './actions'

export function fetchList(args = {}) {
  return dispatch => {
    dispatch({
      type: FETCH_LIST_BEGIN
    })
    const promise = new Promise((resolve, reject) => {
      const doRequest = axios.get('http://www.baidu.com')
      doRequest.then(
        res => {
          dispatch({
            type: FETCH_LIST_SUC,
            data: res.data
          })
          resolve(res)
        },
        err => {
          dispatch({
            type: FETCH_LIST_FAIL,
            data: { error: err }
          })
          reject(err)
        }
      )
    })
    return promise
  }
}
export function reducer(state, action) {
  switch (action.type) {
    case FETCH_LIST_BEGIN:
      return {
        ...state,
        fetchListPending: true
      }
    case FETCH_LIST_SUC:
      return {
        ...state,
        fetchListPending: false,
        list: action.data
      }
    case FETCH_LIST_FAIL:
      return {
        ...state,
        fetchListPending: false,
        list: action.data.error
      }
    default:
      return state
  }
}
```
## 十四、Redux（5）如何组织actions和reducer
```
1. 标准形式的redux action的问题
  a 所有action放一个文件，会无限拓展
  b action reducer 分开，实现业务逻辑需要来回切换
  c 系统中有哪些action不够直观

2. 新的方式： 单个action和reducer放在同一个文件
  a 易于开发： 不用再action和reducer切换
  b 易于维护： 每个action文件都很小，容易理解 
  c 易于测试： 每个业务逻辑只需要对应一个测试文件
  d 易于理解： 文件名就是action名字，文件列表就是action列表
```
## 十五、Redux（6）不可变数据（immutable data）
```
1. 为何需要不可变数据
  a 性能优化（只需要比较引用）
  b 易于调试和跟踪（看前一个数据，后一个数据状态）
  c 易于推测
2. 如何操作不可变数据
  a 原生写法 {...}, Object.assign({}, state, {xxx: 1})
  b immutability-helper
    import update from 'imutablility-helper'
    const state = { filter: 'completed', todos: ['learn react']}
    const newState = update(state, {todo: {$push: ['Learn redux']}})
  c immer (用很原生的写法简单操作，性能稍微差一点)
    import produce from 'immer'
    const state = { filter: 'completed', todos: ['learn react']}
    const newState = product(state, draft => {
        draft.todos.push('learn redux')
    })
    product接受旧状态，新生成新数据的一个代理draft，对draft的任何造作都会再immer内部以不可变数据操作
    当节点层数很多很多，会为每个属性建立一个代理，操作会有一定的性能问题。
```
## 十六、React Router(1): 路由不只是页面切换，更是代码的组织方式
```
1. 为什么需要路由
  a 单页面应用需要进行页面切换
  b 通过url可以定位到页面
  c 更有语义的组织资源

2. 路由实现的基础架构
  路由定义                Router      页面布局
  /topic/:id -> Topic
  /topics -> List        =>          组件容器
  /abount -> About
  
  import React from "react";
  import { BrowserRouter as Router, Route, Link } from "react-router-dom";
  
  function Index() {return <h2>Home</h2>;}
  
  function About() { return <h2>About</h2>;}
  
  function Users() { return <h2>Users</h2>;}
  function AppRouter() {
    return (
      <Router>
        <div>
          <nav>
            <ul>
              <li><Link to="/">Home</Link></li>
              <li><Link to="/about/">About</Link></li>
              <li><Link to="/users/">Users</Link></li>
            </ul>
          </nav>
          <Route path="/" exact component={Index} />
          <Route path="/about/" component={About} />
          <Route path="/users/" component={Users} />
        </div>
      </Router>
    );
  }
  export default AppRouter;
  
2. react router的特性（和后端对比，通过路由配置表）
  a 声明式路由定义
    [路由其实是像使用react标记一样的，作为一个tag定义的，所以使用非常灵活，想放到哪里]
  b 动态路由
    路由是在页面render的时候实时解析的

3. 三种路由实现方式
  a url路径(BrowserRouter)
  b hash路由(HashRouter)（兼容低版本浏览器）
  c 内存路由（MemeryRouter）路由并不会反应到url上，一般是服务器渲染

4. 基于路由配置进行资源组织
  a 实现业务逻辑的松耦合
  b 易于扩展，重构和维护
  c 路由层面实现lazyload
```
5. React Router Api
```
import {Link， NavLink } from 'react-router-dom'
<Link /> => 类似<a>，不会触发浏览器刷新
NavLink => 类似Link但是会添加当前选中的状态，<NavLink to='/fag' xxxClassName={xxxClass}>
```
```
import { Prompt, Route, Redirect, Switch } from 'react-router'
<Prompt when={formIsOk} message='are you sure you want to leave'> 满足条件时候提醒用户是否离开当前页面
<Redirect />重定向单前页面
<Route exact path='/' render={() => (
    loggedIn ? (<Redirect to='/dashboard'>): (<PublicHomePage />)
)}>
<Route />路径匹配时候显示对应组件
<Switch />只显示第一个匹配的路由
```
## 十七、React Router(2): 参数定义，嵌套路由的使用场景
```
1. 通过url传递参数
  a 如何通过url传递参数 <Route path='/topic/:id' />
  b 如何获取参数 this.props.match.params
  c https://github.com/pillarjs/path-to-regexp

2. 何时需要url参数，页面状态尽量通过Url参数定义

3. 嵌套路由
  a 每个react组件都可以是路由容器
  b React Router声明式语法可以方便的定义嵌套路由
  react route标记是可以多个匹配的，因为他只是react的一个组件
```
## 十八、 UI组件库对比介绍 ant.Design, Material UI, Semantic UI
```
1. 当前比较流行的
 ant.Design => 企业级应用，数据密集合理，复杂场景
 Material UI => 更花哨消费者
 Semantic UI => jq时代，ui作为一种languageOfWeb

2. 考虑选择UI库的因素
  a  组件库是否齐全 antDesign > Material > Semantic
  b  样式风格是否符合业务需求 
  c  api设计是否更加便捷灵活 antDesign > Material > Semantic
     例如table （antd 数据驱动型）>（Material|和写html一样）
     antd table功能非常强大
  d 技术支持是否完善（文档issue等）
  c 开发是否完善（稳定团队）
```
## 十九、 使用next.js创建react的同构应用
```
1. 什么是同构应用
  浏览器                 服务器
      -->初次发送请求-->   
                       服务器端逻辑
      <--返回渲染后的app页面<--
  客户端操作

2. 创建页面
  a 页面就是pages目录下的一个组件
  b static目录映射静态文件
  c page具有特殊的静态方法getInitialProps

3. 页面中使用其他的react组件
  a 页面也是标准的node模块，可使用其它React组件
  b 页面会针对性打包，仅包含其引入的组件

4. 使用Link实现同构的路由
  a 使用next/link定义链接
  b 点击链接时页面不会刷新
  c 使用prefetch预加载目标资源
  d 使用replace属性替换Url

5. 动态加载页面
  import dynamic from 'next/dynamic'
  const DynamicHello = dynamic(import('./xxx'))
```
## 二十、 使用Jest, Enzyme等工具进行单元测试

```
1. React让单元测试变得容易
  a React应用很少需要访问浏览器API
  b 虚拟DOM可以在NodeJS环境运行和测试
  c Redux隔离了状态管理，纯数据层单元测试

2. 单元测试涉及的工具
  a Jest: FB开源的JS单元测试框架
  b JS DOM: 浏览器环境的NodeJS模拟
  c Enzyme: React组件渲染和测试（airbnb）
  d nock: 模拟Http请求
  e sinon: 函数的模拟和调用跟踪
  f istanbul: 单元测试覆盖率
```
## 二十一、 常用的开发调试工具 Eslint、Prettier、React DevTool、Redux DevTool
```
1. Eslint 语法风格检查
  a 使用.eslintrc进行规则配置
  b 使用airbnb的javascript代码风格
  {
      "extends": './node_modules/eslint-config-airbnb/.eslintrc',
       "parser": "babel-eslint",
       "globals": {
           "document": true,
           "window": true
       },
       "settings": {
           "import/resolver": {
               "babel-modules": {}
           }
       },
       rules: {
          "prettier/prettier": "error",
          "strict": 0,
          "function-paren-newline": 0,
          "guard-for-in": 0,
          "max-len": 0,
          "no-nested-ternary": 0,
          "no_underscore_dangle": 0,
          "no-console": 0,
          "no-shadow": 0,
          "class-methods-use-this": 0,
       }
  }

2. Prettier 
  a 代码格式化的神器
  b 保证大家写出风格一致的代码

3. React Dev Tools
  a select
  b highlight updates

4. Redux DevTool
  a 观察store的完整状态
  b diff引起的变化
  c timeMachine jump(可以停止到某个状态，不用debugger)
  d test 测试某个action是否正确，拷贝到单元测试文件中，就可以形成一个完整的testcase
```
## 二十二、 前端理想架构
```
1.理想架构
  易于开发--> 开发工具是否完善、生态圈是否繁荣、社区是否活跃
  易于维护--> 增加新功能是否容易、新功能是否会显著的增加系统的复杂度
  易于扩展--> 代码是否容易理解、文档是否健全
  易于测试--> 功能的分层是否清晰、副作用（外部变量）少、尽量使用纯函数
  易于构建--> 使用通用技术和架构、构建工具的选择（webpack rollup）

```
## 二十三、 拆分复杂度rekit 
```
1. 按照领域模型（feature）组织代码，降低耦合度
   eg 文件夹结构
   a 按照feature组织源文件
   b 组件和样式文件
   c Redex 单独文件夹
   d 单元测试保持同样的目录结构在单独文佳佳

2. 在每个feature中单独定义自己的路由
   根路由分散到每个子路由
   使用json定义顶级路由

3. rekit如何工作
   定义了基于feature的可扩展的文件夹结构
   基于最佳实践生成代码和管理项目元素
   提供工具和IDE确保代码和文件夹结构遵循最佳结构

4. 通过代码生成保持一致性
   文件夹结构、文件名、变量名、代码逻辑一致性
```
## 二十四、使用react router来管理登陆

```
1. 使用react router管理路由授权
   实现基础是react router的动态路由机制
   区分受保护路由和公开路由
   访问未授权路由时重定向到登陆页面
  （方法： 动态修改路由配置）
```
## 二十五、实现表单、列表页面、react-router实现分步操作

```
1. form = validator + jsonschema

2. 设计list 
  a redux的store模型设计
  {
     listItems: array
     keyword: string
     page:numner 
     byId: Object
     fetchListPending:bool
     fetchListError: object
     listNeedReload: bool
  }
  b url的设计以及和store同步

3. 向导页面考虑的技术要点
  a 使用url进行导航的好处
  b 表单内容存放的位置
  c 页面状态如何切换
  
```
## 二十六、常见的布局方式实现、ReactPortals
```
1. 常见页面布局的实现
  a 从0开始用css实现
  b 使用css Grid系统[boostrap等]
  c 使用组件库，例如antd

2. demo
  上中下布局
  左右布局
3. dialog（ant Model）
  react 16.3引入的Api
  可以将虚拟DOM映射到任何真实的DOM节点
  解决了漂浮层的问题，比如：Dialog、ToolTip

```
## 二十七、 集成第三方js库，例如d3.js
```
1. 集成第三方JS库的技术要点
  a 使用ref获取原生DOM节点引用
  b 手动将组件状态更新到DOM节点
  c 组件销毁时移除原生节点DOM事件
  核心：这些js库都是在操作真实的dom节点

2. demo
```
## 二十八、 基于路由实现菜单导航
```
1. 技术要点
  a 菜单导航只是改变url状态
  b 根据当前url显示菜单的当前状态

2. demo NavLink
```
## 二十九、 react中拖放的实现
```
1. 技术要点
  a 如何使用React的鼠标事件系统
  b 如何判断拖放开始和拖放结束
  c 如何实现拖放元素的位置移动
  d 拖放状态在组件中如何维护

2. demo
  react-beautiful-dnd虽然好，但是太繁琐了，自己需要明白怎么搞dnd的原理
```
## 三十、性能优化（如何避免应用出现性能问题）
```
1.了解常见的性能问题场景[例如键盘输入、鼠标移动例如滚动页面]
2. 自动按需加载[loadable|webpack按需加载]
3. 使用reselect避免重复计算
4. 异步渲染
```
## 三十一、使用Chrome DevTool
```
1. 使用react DevTool 找到多余的渲染
2. 使用Chrome DevTool 定位性能瓶颈
```



