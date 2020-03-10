---
title: ant-pro 替换roadhog为webpack
date: 2018-11-16 12:29:22
categories: 
- xxxxx
tags:
- webpack
- admin
- roadhog
---

### 前言
admin项目最初时间紧迫，再确定了使用react技术栈后，选择了快速产出的antd及开箱即用的框架antd-pro。当初框架用的有多爽，后来就有多哭～

9月底决定干掉框架，造一些自己的轮子，为了不影响线上业务，分为3个步骤做：1.替换打包roadhog为webpack 2.替换dva框架为开源的react技术栈 3.替换antd的UI
本篇主要记录替换roadhog的步骤。

### 一、替换步骤概览（what）
```javascript
1. 新建webpack的配置 config/webpack.base.conf.js|webpack.dev.conf.js|webpack.prod.conf.js
2. 删除webpackrc.js、paths.js，删除package.json里面的roadhog相关包和命令
3. 新增package.json里webpack的依赖包及相关命令、配置babelrc等
4. 删除原来移动文件操作的build.js，改成新的build.js，做终端的编译美化工作。
5. 删掉之前的router模块，改造为react-router4的做法，把menu和路由配置在一起
6. 精简之前的layout|pageheader|golbalheader等
7. 删除所有关于登录权限的模块，我们的权限是后台控制的相当于前端不做权限模块。
```

### 二、为什么要做替换呢(why)
当初选择的理由只有一个，是框架里自带的，适合快速产出，替换的理由却有很多
```javascript
1. roadhog jenkins build经常失败，原因未知，解决办法，换个版本号呵呵
2. roadhog 阉割了的webpack，一些自定义的功能接口不能使用，不太方便
3. roadhog 相当于又学了一个不通用的轮子，开源的东西可能更好一些
4. 我们想做自己的脚手架，从打包到ui最终都尽量用自己的东西，更好掌控一些
```
### 三、首次迁移的遗留问题
```javascript
1. 打包过大，按需加载，优化
2. 其他的更酷的配置，我们目前只用了webpack一些最常见的配置
3. 有一些有问题的配置，例如引不进react，强行插入了，先运行起来
4. 在layout和router menu中需要做的细节工作很多
```
后面回持续跟进解决这些问题
### 四、打包工具替换具体细节
1. webpack相关配置
```javascript
// webpack.base.conf.js
const path = require('path')
const SRC_PATH = path.resolve(__dirname, '../src')
const config = {
  entry: {
    src: './src/index.js'
  },
  output: {
    path: path.resolve(__dirname, '../build/windrunner'),
    filename: 'js/bundle.js',
    publicPath: '/'
  },
  resolve: {
    alias: {
      '@src': SRC_PATH,
      '@assets': path.resolve(SRC_PATH, 'assets'),
      '@comp': path.resolve(SRC_PATH, 'components'),
      '@common': path.resolve(SRC_PATH, 'common'),
      '@models': path.resolve(SRC_PATH, 'models'),
      '@api': path.resolve(SRC_PATH, 'apis'),
      '@helper': path.resolve(SRC_PATH, 'helpers'),
      '@pages': path.resolve(SRC_PATH, 'pages'),
      '@const': path.resolve(SRC_PATH, 'constants'),
      '@utils': path.resolve(SRC_PATH, 'utils'),
      '@selector': path.resolve(SRC_PATH, 'selectors'),
      '@services': path.resolve(SRC_PATH, 'services'),
      '@layouts': path.resolve(SRC_PATH, 'layouts')
    }
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader?cacheDirectory',
        include: SRC_PATH
      }
    ]
  }
}

if (process.env.ANALYZE) {
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  config.plugins = config.plugins || []
  config.plugins.push(new BundleAnalyzerPlugin())
}
module.exports = config

```
webpack.dev.conf.js
```javascript
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf.js')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const webpack = require('webpack')
module.exports = merge(baseWebpackConfig, {
  mode: 'development',
  devtool: 'cheap-module-source-map',
  output: {
    filename: 'js/[name].[hash:16].js'
  },
  module: {
    rules: [
      {// CSS处理
        test: /\.css$/,
        loader: 'style-loader!css-loader?modules',
        exclude: /node_modules/
      },
      {// antd样式处理
        test: /\.css$/,
        exclude: /src/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1
            }
          }
        ]
      },
      {
        test: /\.(less)$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
              localIdentName: '[local]__[hash:7]'
            }
          }, {
            loader: 'postcss-loader'
          }, {
            loader: 'less-loader',
            options: {
              javascriptEnabled: true
            }
          }
        ]
      },
      {
        test: /\.(scss)$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
              localIdentName: '[local]__[hash:7]'
            }
          }, {
            loader: 'postcss-loader'
          }, {
            loader: 'sass-loader'
          }
        ]
      },
      {
        test: /\.(png|jpg|gif)$/,
        use: [{
          loader: 'url-loader',
          options: {
            limit: 8192
          }
        }]
      },
      {
        test: /\.svg$/,
        use: [
          {
            loader: 'react-svg-loader'
          }
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: 'public/index.html',
      inject: 'body',
      chunksSortMode: 'none'
    }),
    new webpack.ProvidePlugin({
      'React': 'react'
    }),
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {
    port: 80,
    host: 'local.dp-admin.test.xxxxx.io',
    historyApiFallback: true,
    open: true,
    proxy: {
      '/api': 'http://localhost:3001'
    }
  }
})
```
```javascript
// webpack.prod.conf.js
const merge = require('webpack-merge')
const webpack = require('webpack')
const baseWebpackConfig = require('./webpack.base.conf.js')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CleanWebpackPlugin = require('clean-webpack-plugin')
const UglifyJSWebpackPlugin = require('uglifyjs-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin')

const config = merge(baseWebpackConfig, {
  mode: 'production',
  output: {
    filename: 'static/js/[name].[chunkhash:16].js'
  },
  module: {
    rules: [
      {// CSS处理
        test: /\.css$/,
        loader: 'style-loader!css-loader?modules',
        exclude: /node_modules/
      },
      {// antd样式处理
        test: /\.css$/,
        exclude: /src/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1
            }
          }
        ]
      }, {
        test: /\.(less)$/,
        use: [
          MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: true,
              localIdentName: '[local]__[hash:7]'
            }
          }, {
            loader: 'postcss-loader'
          }, {
            loader: 'less-loader',
            options: {
              javascriptEnabled: true
            }
          }
        ]
      }, {
        test: /\.(scss)$/,
        use: [
          MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: true,
              localIdentName: '[local]__[hash:7]'
            }
          }, {
            loader: 'postcss-loader'
          }, {
            loader: 'sass-loader'
          }
        ]
      },
      {
        test: /\.(png|jpg|gif)$/,
        exclude: /node_modules/,
        use: [{
          loader: 'url-loader',
          options: {
            limit: 8192
          }
        }]
      },
      {
        test: /\.svg$/,
        use: [
          {
            loader: 'react-svg-loader'
          }
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: 'public/index.html',
      inject: 'body',
      chunksSortMode: 'none',
      minify: {
        minimize: true,
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true,
        minifyCSS: true,
        minifyJS: true
      }
    }),
    new webpack.ProvidePlugin({
      'React': 'react'
    }),
    new MiniCssExtractPlugin({
      filename: 'static/css/[name].[hash].css',
      chunkFilename: 'static/css/[id].[hash].css'
    }),
    new CleanWebpackPlugin(['../build'], { allowExternal: true })
  ],
  optimization: {
    minimizer: [
      new UglifyJSWebpackPlugin({
        uglifyOptions: {
          ecma: 8,
          compress: {
            warnings: false
          }
        }
      }),
      new OptimizeCSSAssetsPlugin({
        cssProcessorOptions: true // eslint-disable-line
          ? {
            map: { inline: false }
          }
          : {}
      })
    ],
    splitChunks: {
      chunks: 'initial', // 代码块类型 必须三选一： "initial"（初始化） | "all"(默认就是all) | "async"（动态加载）
      minSize: 0, // 最小尺寸，默认0
      minChunks: 1, // 最小 chunk ，默认1
      maxAsyncRequests: 1, // 最大异步请求数， 默认1
      maxInitialRequests: 1, // 最大初始化请求书，默认1
      name: () => {}, // 名称，此选项课接收 function
      cacheGroups: { // 缓存组会继承splitChunks的配置，但是test、priorty和reuseExistingChunk只能用于配置缓存组。
        priority: '0', // 缓存组优先级 false | object |
        vendor: { // key 为entry中定义的 入口名称
          chunks: 'initial', // 必须三选一： "initial"(初始化) | "all" | "async"(默认就是异步)
          test: /react|antd|dva|dva-loading/, // 正则规则验证，如果符合就提取 chunk
          name: 'vendor', // 要缓存的 分隔出来的 chunk 名称
          minSize: 0,
          minChunks: 1,
          enforce: true,
          reuseExistingChunk: true // 可设置是否重用已用chunk 不再创建新的chunk
        },
        common: { // key 为entry中定义的 入口名称
          chunks: 'initial', // 必须三选一： "initial"(初始化) | "all" | "async"(默认就是异步)
          test: /moment/, // 正则规则验证，如果符合就提取 chunk
          name: 'common', // 要缓存的 分隔出来的 chunk 名称
          minSize: 0,
          minChunks: 1,
          enforce: true,
          reuseExistingChunk: true // 可设置是否重用已用chunk 不再创建新的chunk
        }
      }
    }
  }
})

module.exports = config
```

2. package.json 删除原有的roadhog部分script，替换成新的webpack的相关命令
```javascript
"scripts": {
    "debug": "webpack-dev-server --config config/webpack.dev.conf.js",
    "analysis": "ANALYZE=true node scripts/build.js",
    "build": "node scripts/build.js",
    "proxy": "node data/proxy.js",
    "mock": "node data/mock.js",
    "standard": "standard"
},
"devDependencies": {
      "@babel/core": "^7.1.2",
    "@babel/plugin-proposal-class-properties": "^7.1.0",
    "@babel/plugin-proposal-decorators": "^7.1.2",
    "@babel/plugin-proposal-object-rest-spread": "^7.0.0",
    "@babel/plugin-syntax-dynamic-import": "^7.0.0",
    "@babel/plugin-transform-runtime": "^7.1.0",
    "@babel/polyfill": "^7.0.0",
    "@babel/preset-env": "^7.1.0",
    "@babel/preset-react": "^7.0.0",
    "@babel/runtime": "^7.1.2",
    "@babel/runtime-corejs2": "^7.1.2",
    "autoprefixer": "^9.2.1",
    "babel-eslint": "^10.0.1",
    "babel-loader": "^8.0.4",
       "babel-plugin-syntax-dynamic-import": "^6.18.0",
    "chalk": "^2.4.1",
    "clean-webpack-plugin": "^0.1.19",
    "compression-webpack-plugin": "^2.0.0",
    "cross-env": "^5.2.0",
    "css-loader": "^1.0.0",
    "enzyme": "^3.1.0",
    "express": "^4.16.4",
    "gh-pages": "^1.0.0",
    "html-webpack-plugin": "^3.2.0",
    "husky": "^1.1.2",
    "less": "^3.8.1",
    "less-loader": "^4.1.0",
    "lint-staged": "^6.0.0",
    "meepojs-loader": "0.0.5",
    "mini-css-extract-plugin": "^0.4.4",
    "mockjs": "^1.0.1-beta3",
    "node-sass": "^4.9.4",
    "optimize-css-assets-webpack-plugin": "^5.0.1",
    "ora": "^3.0.0",
    "postcss": "^7.0.5",
    "postcss-loader": "^3.0.0",
    "prettier": "1.11.1",
    "react-svg-loader": "^2.1.0",
    "redbox-react": "^1.5.0",
    "regenerator-runtime": "^0.11.1",
    "request": "^2.88.0",
    "sass-loader": "^7.1.0",
    "standard": "^12.0.1",
    "style-loader": "^0.23.1",
    "uglifyjs-webpack-plugin": "^1.1.6",
    "url-loader": "^1.1.2",
    "webpack": "^4.23.1",
    "webpack-bundle-analyzer": "^3.0.3",
    "webpack-cli": "^3.1.2",
    "webpack-dev-server": "^3.1.10",
    "webpack-merge": "^4.1.4"  
  },

```
3. babelrc.js   postcss.config.js 
```javascript
module.exports = {
  presets:[
    [
      "@babel/preset-env",
      {
        "shippedProposals": true,
        "forceAllTransforms": true,
        "targets": {
          "browsers": [
            "> 1%",
            "last 5 versions",
            "ie >= 8"
          ]
        }
      }
    ],
    "@babel/preset-react"
  ],
  plugins: [
    ["import", {
      "libraryName": "antd",
      "libraryDirectory": "es",
      "style": "css" 
    }], // `style: true` 会加载 less 文件
    [
      "@babel/plugin-transform-runtime"
    ],
    "@babel/transform-arrow-functions",
    ["@babel/plugin-proposal-decorators", { "legacy": true }],
    ["@babel/plugin-proposal-class-properties", { "loose" : true }],
    "@babel/plugin-proposal-object-rest-spread",
    "@babel/plugin-syntax-dynamic-import"
  ]
}
```
postcss.config.js 
```javascript
module.exports = () => ({
  plugins: {
    autoprefixer: { browsers: ['last 5 version', '>1%', 'ie >= 8'] }
  }
})
```
4. build脚本美化终端展示
```javascript
const ora = require('ora')
const chalk = require('chalk')
const webpack = require('webpack')
const webpackConfig = require('../config/webpack.prod.conf')
const spinner = ora('编译中...\n').start()
webpack(webpackConfig, function (err, stats) {
  if (err) {
    spinner.fail('编译失败')
    console.log(err)
    return
  }
  spinner.succeed('编译已结束. \n')

  process.stdout.write(stats.toString({
    colors: true,
    modules: false,
    children: false,
    chunks: false,
    chunkModules: false
  }) + '\n\n')
  console.log(chalk.cyan('  编译成功！\n'))
  console.log(chalk.yellow(
    '  提示: 编译后的文件可以上传并且部署到服务器\n' +
        '  通过file:// 打开index.html不会起作用.\n'
  ))
})
```
### 五、路由相关地方替换细节
1. 删除原有router配置，将menu和router的配置合并在一起
```javascript
import { countryCodeID } from '@const/index'
export const menuData = [
  {
    name: 'Order',
    icon: 'book',
    title: 'Order List',
    path: '/order/list',
    model: ['order'],
    component: 'Order/List'
  },
  {
    name: 'Refund',
    icon: 'book',
    path: '/refund/list',
    title: 'Refund List',
    model: ['refund'],
    component: 'Refund/List'
  },
  {
    name: 'Order',
    icon: 'profile',
    title: 'Order Detail',
    path: '/order/detail/:id',
    model: ['order'],
    component: 'Order/Detail',
    hideInMenu: true
  },
  {
    name: 'Refund',
    icon: 'profile',
    title: 'Refund Detail',
    path: '/refund/detail/:id',
    model: ['refund', 'order'],
    component: 'Refund/Detail',
    hideInMenu: true
  },
  {
    name: 'Product',
    icon: 'schedule',
    children: [
      {
        name: 'Product List',
        path: '/product/list',
        title: 'Product List',
        model: ['product'],
        component: 'Product/List'
      },
      {
        name: 'Product MassEdit',
        path: '/product/massedit',
        title: 'Product MassEdit',
        model: ['product'],
        component: 'Product/MassEdit'
      }]
  },
  {
    name: 'Product',
    icon: 'profile',
    title: 'Product Detail',
    path: '/product/detail/:id',
    model: ['product'],
    component: 'Product/Detail',
    hideInMenu: true
  },
  {
    name: 'Coins',
    icon: 'form',
    path: 'coins',
    children: [
      {
        name: 'Coins Spend',
        path: '/coins/spend',
        title: 'Coins Spend',
        model: ['coins'],
        component: 'Coins/Spend'
      },
      {
        name: 'Coins Earn',
        path: '/coins/earn',
        title: 'Coins Earn',
        model: ['coins'],
        component: 'Coins/Earn'
      }
    ]
  },
  {
    name: 'Report',
    icon: 'fund',
    title: 'Report Download',
    path: '/report/download',
    model: ['report'],
    component: 'Report/Download'
  },
  {
    name: 'Banner',
    icon: 'form',
    title: 'Banner List',
    path: '/banner/list',
    model: ['banner'],
    component: 'Banner/List',
    hideInMenu: (countryCodeID !== 'id')
  }
]

const getRouterData = (menuData) => {
  let routerList = []
  const getMenuRouter = (menuData) => {
    (menuData || []).forEach(item => {
      if (item.title && item.component && !item.children) {
        routerList.push({
          model: item.model,
          path: item.path,
          title: item.title,
          component: item.component
        })
      } else if (item.children && item.children.length) {
        getMenuRouter(item.children)
      }
    })
  }
  getMenuRouter(menuData)
  return routerList
}

export const routerData = getRouterData(menuData)
```
2. BasicLayout
```javascript
import React, { createElement } from 'react'
import { Layout } from 'antd'
import { connect } from 'dva'
import { Route, Redirect, Switch } from 'dva/router'
import { ContainerQuery } from 'react-container-query'
import { routerData } from '@common/menu'
import { setCountry } from '@utils/utils'
import { accountLogout } from '@services/api'
import { layoutData } from '@common/default'
import DocumentTitle from 'react-document-title'
import classNames from 'classnames'
import GlobalHeader from '@comp/GlobalHeader'
import SiderMenu from '@comp/SiderMenu/index.js'
import NotFound from '@pages/Exception/500'
import logo from '@assets/favicon.jpg'
import dynamic from 'dva/dynamic'
const { Content, Header, Footer } = Layout

// wrapper of dynamic
const dynamicWrapper = (app, models, component) => {
  return dynamic({
    app,
    models: () => ((models && models.map(m => import(`@models/${m}.js`))) || [import(`@models/refund.js`)]),
    component: () => {
      return component().then(raw => {
        const Component = raw.default || raw
        return props =>
          createElement(Component, {
            ...props
          })
      })
    }
  })
}
class BasicLayout extends React.PureComponent {
  constructor (props) {
    super(props)
    const { app } = this.props
    this.state = {
      collapsed: false
    }
    this.routerConfig = routerData.map((item, index) => {
      const component = dynamicWrapper(app, item.model, () => import('@pages/' + item.component))
      return (
        <Route
          key={index}
          path={item.path}
          title={item.name}
          component={component}
        />
      )
    })
  }

  handleMenuCollapse = collapsed => {
    this.setState({
      collapsed: !this.state.collapsed
    })
    this.props.dispatch({
      type: 'global/changeLayoutCollapsed',
      payload: !collapsed
    })
  }

  handleMenuClick = ({ key }) => {
    if (key === 'logout') {
      accountLogout().then((data) => {
        window.sessionStorage.setItem('dp-admin-auth', null)
        window.location.reload()
      })
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

export default connect(({ login, global }) => ({
  collapsed: global.collapsed
}))(BasicLayout)
```
删掉那些多的模块一直做减法，遇到报错就google大概就差不多了

### 六、 要安装的基础包大概如下

```javascript
[wepack]     |   webpack-cli webpack webpack-merge
[react]      |   react react-dom
[babel]      |   babel-loader @babel/preset-core @babel/preset-env @babel/preset-core
[html]       |   html-webpack-plugin clean-webpack-plugin
[js]         |   uglifyjs-webpack-plugin 
[css]        |   style-loader css-loader  mini-css-extract-plugin less-loader sass-loader node-sass 
[css]        |   optimize-css-assets-webpack-plugin postcss postcss_loader autoprefixer
[devserver]  |   webpack-dev-server
[anlyzer]    |   webpack-bundle-analyzer
[eslint]     |   husky standardjs
[pretty]     |   ora chalk
```

