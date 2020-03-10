---
title: ant-pro替换-dva框架
date: 2018-11-16 16:39:24
categories: 
- xxxxx
tags:
- admin
- dva
- redux
- yml
---

背景参考[ant-pro 替换roadhog为webpack](https://weiqian93.github.io/2018/11/16/ant-pro-%E6%9B%BF%E6%8D%A2roadhog%E4%B8%BAwebpack/)


### 一、替换步骤概览（what）
```javascript
1. 添加meepjs相关loader webpack配置
2. 新建apis文件配置yml文件，删除service/api模块
3. 新建store|reducer模块、删除model模块
4. 新建页面Loadable加载、删除dva/dynamic的创建方式
5. page里面引入react-redux的connent、删除dva的connect
6. 结合yml引入新的error模块，删除原有的error模块
7. selector重新写一下，util和helper模块规划的更细一些
8. 重新替换上自己的layout布局
9. 新增mock、i18n（这2个模块作用现在不是很大，为了后面提取更完善的框架，先把这些标配的东西放进去）
```

### 二、为什么要做替换呢(why)
```javascript
1. dva的框架和现在有sniper的框架有一些差距，很容易走到一个人的小圈子，不够开放，不能及时更新同步新的东西
2. 代码的可读性没有sniper高，sniper模块就很繁琐强迫证一样，但也使的代码的可读性更好
3. 我们做自己的脚手架，从打包到ui最终都尽量用自己的东西，更好掌控一些，这样就一定要去掉这些别人家的轮子，做自家轮子
4. 把这些模块慢慢替换掉后，就可以慢慢打磨代码，从线上代码很快提取手架的，相互磨合改造的一个过程。
```
### 三、首次迁移的遗留问题
```javascript
1. reducer的模块多繁琐，代码都差不多，如何精简
2. 关于数据的处理，有时候时在action 有时候时在saga，这里有没有更好的规范
3. 各个组件之间的loading和全局的loading组织关系，怎么优雅组织代码
```

### 四、dva替换部分具体细节
1. 添加meepjs相关loader webpack配置 
```javascript
 // webpack.base.conf.js
 module: {
    noParse: [/\.min\.js$/],
    rules: [
      {
        test: /\.yml$/,
        use: [
          { loader: require.resolve('meepojs-loader') }
        ]
      }
    ]
 }
```
2. 新建apis文件配置yml文件，删除service/api模块
src/apis/banner.yml
```javascript
apis:
  fetchBannerGroup:
    url: /api/banner/group
    options:
      method: GET
      timeout: 6000
    needs:
     group_id: number

  saveBannerGroup:
    url: /api/banner/save
    options:
      method: POST
      timeout: 6000
    needs:
      images: array
      group_id: number
      group_name: string

config:
  meta:
```
src/apis/index.js
```javascript
import './init.js'
import { registerApi } from 'meepojs'
import bannerApiConf from './banner.yml'
import { memoizeAsync, hashObj } from '@utils/memoize'

const registerApis = apiConfObj => {
  const apiMap = {}
  for (let name in apiConfObj) {
    apiConfObj[name].url = apiConfObj[name].url.replace('/api', '/api/id')
    apiMap[name] = params =>
      (memoizeAsync(
        registerApi(apiConfObj[name]),
        ({ query, body, params }) => apiConfObj[name].url + hashObj(query) + hashObj(params) + hashObj(body)
      ))(params, apiConfObj[name].meta)
  }
  return apiMap
}
export const bannerApi = registerApis(bannerApiConf)

```
src/apis/init.js
```javascript
import { notification } from 'antd'
import { accountLogin } from '@utils/authority'
import { clearLoginSession } from '@helper/login'
import {
  configDefaultFetchOptions,
  registerApiSuccChecker,
  onHttpError,
  onApiError
} from 'meepojs'

configDefaultFetchOptions({
  credentials: 'include',
  prefix: ''
})

registerApiSuccChecker(
  ({ body, status }) => body.err_code === 0 && status === 200
)

onHttpError(({ status, apiConfig }) => {
  if (!apiConfig.meta.showError) return
  if (status === 404) {
    window.location.href = '/error/404'
  } else if (status >= 500 && status < 600) {
    window.location.href = `/error/${status}`
  }
})

onApiError(({ body, status, apiConfig }) => {
  if (body.err_code !== 0) {
    notification.error({
      message: `Request Error ${body.err_code}: ${body.err_msg}`,
      description: body.err_msg
    })
    if (body.err_code === 31) {
      return window.alert(body.err_code)
    } else if (body.err_code === 32) {
      clearLoginSession()
      accountLogin()
    }
  }
})
```
src/utils/momoize.js
```javascript
export const hashObj = obj =>
  obj ? Object.keys(obj).map(key => `${key}=${obj[key]}`).join('&')
    : ''

const MAX_CACHE_SIZE = 1000
const DEFAULT_OPTIONS = {
  isFlush: false,
  expiry: 1000 * 60
}

let cache = {}

export const memoizeAsync = (func, keyCreator = hashObj) => {
  const that = this
  const setCache = (key, data) => {
    cache[key] = {
      data,
      updateTime: Date.now()
    }
  }

  return async function (params, opt) {
    const key = keyCreator(params)
    const options = Object.assign({}, DEFAULT_OPTIONS, opt)
    const { isFlush, expiry } = options

    let result

    if (Object.keys(cache).length > MAX_CACHE_SIZE) cache = {}

    if (isFlush || !cache[key] || cache[key].updateTime + expiry < Date.now()) {
      result = await func.call(that, params)
      setCache(key, result)
    } else {
      result = cache[key].data
    }

    return Promise.resolve(result)
  }
}

```
3. 新建store|reducer模块、删除model模块
src/store.js
```javascript
import { createStore, applyMiddleware, combineReducers } from 'redux'
import createHistory from 'history/createBrowserHistory'
import createSagaMiddleware from 'redux-saga'
import logger from 'redux-logger'
import { routerReducer, routerMiddleware } from 'react-router-redux'
import { reducers, sagas } from './reducers'

const history = createHistory({})
const _routerMiddleware = routerMiddleware(history)
const sagaMiddleware = createSagaMiddleware()
const middlewares = [sagaMiddleware, _routerMiddleware, logger]

const store = createStore(
  combineReducers({
    ...reducers,
    router: routerReducer
  }),
  applyMiddleware(...middlewares)
)
sagaMiddleware.run(sagas)
export default store

```
src/reducer/banner/fetch.js
```javascript
import { createActions } from 'redux-actions'
import { takeLatest, call, put } from 'redux-saga/effects'
import { bannerApi } from '@apis/index.js'

export const {
  fetchBannerGroupReq,
  fetchBannerGroupSucc,
  fetchBannerGroupFailed
} = createActions({
  FETCH_BANNER_GROUP_REQ: null,
  FETCH_BANNER_GROUP_SUCC: (rsp) => ({ banner: rsp }),
  FETCH_BANNER_GROUP_FAILED: null
})

function * fetchBannerGroupReqSaga ({ payload }) {
  const { groupId, callback } = payload
  try {
    const rsp = yield call(bannerApi.fetchBannerGroup, {
      query: {
        group_id: groupId
      }
    })
    rsp.data = (rsp.data || []).map((item, idx) => {
      return {
        key: idx + 1,
        name: item.name,
        banner_link: item.location,
        redirect_url: item.url,
        start_time: item.start_time,
        end_time: item.end_time
      }
    })
    yield put(fetchBannerGroupSucc(rsp.data))
    if (callback) {
      callback(rsp.data)
    }
  } catch (err) {
    yield put(fetchBannerGroupFailed())
  }
}

export function * watchFetchBannerGroup () {
  yield takeLatest(fetchBannerGroupReq, fetchBannerGroupReqSaga)
}

/****************************************************************************
 *                        saveBannerGroup
 ****************************************************************************/

export const {
  saveBannerGroupReq,
  saveBannerGroupSucc,
  saveBannerGroupFailed
} = createActions({
  SAVE_BANNER_GROUP_REQ: null,
  SAVE_BANNER_GROUP_SUCC: (rsp) => ({ banner: rsp }),
  SAVE_BANNER_GROUP_FAILED: null
})

function * saveBannerGroupReqSaga ({ payload }) {
  try {
    const rsp = yield call(bannerApi.saveBannerGroup, {
      body: payload
    })
    yield put(saveBannerGroupSucc(rsp.data))
    if (typeof payload.callback === 'function') {
      payload.callback(rsp.data)
    }
  } catch (err) {
    console.log(err)
    yield put(saveBannerGroupFailed())
  }
}

export function * watchSaveBannerGroup () {
  yield takeLatest(saveBannerGroupReq, saveBannerGroupReqSaga)
}

/****************************************************************************
 *                        watchFetch
 ****************************************************************************/

export const watchFetch = [ watchFetchBannerGroup, watchSaveBannerGroup ]

export default {
  [fetchBannerGroupSucc]: (state, { payload: { banner } }) => {
    return {
      ...state
    }
  },
  [fetchBannerGroupFailed]: state => ({
    ...state,
    isError: true
  }),
  [saveBannerGroupSucc]: (state, { payload: { banner } }) => {
    return {
      ...state
    }
  },
  [saveBannerGroupFailed]: state => ({
    ...state,
    isError: true
  })
}

```
src/reducer/banner/index.js
```javascript
import { handleActions } from 'redux-actions'
import fetchReducer, { watchFetch } from './fetch'

export const watchBannerSagas = [
  ...watchFetch
]

export default handleActions(fetchReducer, {})
```
4. 新建页面Loadable加载、删除dva/dynamic的创建方式
src/index.js
```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'
import { ConnectedRouter } from 'react-router-redux'
import BasicLayout from '@layouts/BasicLayout'
import createHistory from 'history/createBrowserHistory'
import { setCountry } from '@utils/utils'
import store from './store'
import './index.less'

let country = 'id'
  country = window.location.pathname.split('/')[1]
  setCountry(country)
}
const history = createHistory({ basename: '/' + (country || 'id') })

ReactDOM.render(
  <Provider store={store}>
    <ConnectedRouter history={history}>
      <BasicLayout />
    </ConnectedRouter >
  </Provider>,
  document.getElementById('root')
)

```
src/layout/BasicLayout.js
```javascript
import React from 'react'
import { Layout } from 'antd'
import { Route, Redirect, Switch } from 'dva/router'
import { ContainerQuery } from 'react-container-query'
import { routerData } from '@common/menu'
import { setCountry } from '@utils/utils'
import { accountLogout } from '@utils/authority.js'
import Loading from '@comp/Loading'
import Loadable from 'react-loadable'
import { layoutData } from '@common/default'
import DocumentTitle from 'react-document-title'
import classNames from 'classnames'
import GlobalHeader from '@comp/GlobalHeader'
import SiderMenu from '@comp/SiderMenu/index.js'
import NotFound from '@pages/Exception/500'
import logo from '@assets/favicon.jpg'
const { Content, Header, Footer } = Layout

const genOptions = (loader) => {
  return Object.assign({
    loading: Loading
  }, {
    loader
  })
}

class BasicLayout extends React.PureComponent {
  constructor (props) {
    super(props)
    this.state = {
      collapsed: false
    }
    this.routerConfig = routerData.map((item, index) => (
      (
        <Route
          key={index}
          path={item.path}
          title={item.name}
          component={Loadable(genOptions(() => import('@pages/' + item.component)))}
        />
      )
    ))
  }

  handleMenuCollapse = collapsed => {
    this.setState({
      collapsed: !this.state.collapsed
    })
  }

  handleMenuClick = ({ key }) => {
    if (key === 'logout') {
      accountLogout()
    }
  }
  handleCountryClick = (key) => {
    console.log(`selected ${key}`)
    setCountry(key)
    window.location = window.location.origin + '/' + key
  }
  render () {
    const { collapsed } = this.state
    const layout = (
      <Layout>
        <SiderMenu
          collapsed={collapsed}
          isMobile={this.state.isMobile}
          onCollapse={this.handleMenuCollapse}
        />
        <Layout>
          { <Header style={{ padding: 0 }}>
            <GlobalHeader
              logo={logo}
              collapsed={collapsed}
              isMobile={this.state.isMobile}
              onCountryClick={this.handleCountryClick}
              onMenuClick={this.handleMenuClick}
              onCollapse={this.handleMenuCollapse}
            />
          </Header> }
          <Content style={{ margin: '24px 24px 0', height: '100%' }}>
            <Switch>
              { this.routerConfig }
              <Redirect exact from='/' to='order/list' />
              <Route render={NotFound} />
            </Switch>
          </Content>
          <Footer style={{ padding: 0 }} />
        </Layout>
      </Layout>
    )
    return (
      <DocumentTitle title={layoutData.docTitle}>
        <ContainerQuery query={layoutData.query}>
          {params => <div className={classNames(params)}>{layout}</div>}
        </ContainerQuery>
      </DocumentTitle>
    )
  }
}

export default BasicLayout
```
5. page里面引入react-redux的connent、删除dva的connect
```javascript
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { bindActionCreators } from 'redux'
import PageHeaderLayout from '@layouts/PageHeaderLayout'
import { fetchOrderDetailReq } from '@reducers/order/fetch'
import { fetchRefundDetailReq } from '@reducers/refund/fetch'
import { createCardDescriptList } from '@utils/utils'
import { refundDetailSelector, orderDetailSelector } from '@selector/index'

class RefundDetail extends Component {
  componentDidMount () {
    const { location: { pathname } } = this.props
    const urlparam = pathname.split('/')[3].split('_')
    const param = {
      order_id: urlparam[0],
      user_id: urlparam[1] * 1
    }
    this.props.fetchOrderDetailReq(param)
    this.props.fetchRefundDetailReq(param)
  }

  render () {
    const { refund, order } = this.props
    const refundProfile = refundDetailSelector(refund.detail)
    const refundDetail = createCardDescriptList(refundProfile, 'Refund Detail')
    const orderProfile = orderDetailSelector(order.detail)
    const orderDetail = createCardDescriptList(orderProfile, 'Order Detail')
    return (
      <PageHeaderLayout>
        { refundDetail }
        { orderDetail }
      </PageHeaderLayout>
    )
  }
}

const mapStateToProps = state => {
  return {
    ...state
  }
}

const mapDispatchToProps = dispatch => bindActionCreators({
  fetchOrderDetailReq,
  fetchRefundDetailReq
}, dispatch)

export default connect(mapStateToProps, mapDispatchToProps)(RefundDetail)

```

