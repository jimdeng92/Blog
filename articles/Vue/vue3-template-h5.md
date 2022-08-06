---
title: vue-template-h5
date: 2021-11-25 15:06:38
permalink: /pages/82ea73/
categories: 
  - 开发者手册
  - Vue
tags: 
  - 
---

近期开发一个公众号网页工具，刚好用用 Vue3，但是没有好的脚手架模板，搭建过程花了不少时间，因此打算把它抽离出来用作 H5 开发的模板。项目托管在我的 [GitHub](https://github.com/jimdeng92/vue3-template-h5) 上，需要的直接下载即可。

**Feature**

- 样式初始化
- vant-ui 按需引入
- 多环境变量
- rem 适配
- 根据环境引入 vconsole
- axios 封装
- webpack 项目优化

### 样式初始化

**index.scss**

::: details 点击查看代码

``` scss
// index.scss
@import './variable.scss';

html, body, #app {
  width: 100%;
  height: 100%;
  color: var(--textPrimaryColor);
  font-size: 14px;
  line-height: calc(20 / 14);
  font-family: 'PingFang SC', 'PingFangSC-Medium',  Arial, Helvetica, 'STHeiti STXihei', 'Microsoft YaHei', Tohoma, sans-serif;
  background-color: var(--backgroundColor);
}

// reset
body,div,dl,dt,dd,ul,ol,li,h1,h2,h3,h4,h5,h6,
pre,code,form,fieldset,legend,input,textarea,
p,blockquote,th,td {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

ol,ul {
  list-style: none;
}

h1,h2,h3,h4,h5,h6 {
  font-size: 100%;
  font-weight: normal;
}

a {
  text-decoration: none;
  &:hover {
    text-decoration: none;
  }
}

fieldset,img {
  border: 0;
}

input, textarea, select {
  font-family: inherit;
  font-size: inherit;
  font-weight: inherit;
}

table {
  border-collapse: collapse;
  border-spacing: 0;
  th, td {
    text-align: left;
    font-weight: normal;
    word-break: break-all;
    word-wrap: break-word;
  }
}

```

:::

**variable.scss**

::: details 点击查看代码

``` scss
:root {
  --backgroundColor: #EDEDED;
  --textPrimaryColor: #2A2A2A;
  --textSecondaryColor: #888888;
  --borderColor: #EAEAEA;
  --badgeBlue: #1978FF;
  --badgeErrorColor: #FA5555;
  --loseColor: #03A16F;
  --profitColor: #F45452;
  --buttonBgColor: linear-gradient(315deg, #FFC440 0%, #FFD83C 100%);
}
```

:::

### vant-ui 按需引入

[vant-ui 引入组件](https://vant-contrib.gitee.io/vant/v3/#/zh-CN/quickstart#yin-ru-zu-jian)

下载 `babel-plugin-import` 包。

``` bash
# 安装插件
npm i babel-plugin-import -D
```

在 babel.config.js 中添加配置：

``` js
const plugins = [
  [
    'import',
    {
      libraryName: 'vant',
      libraryDirectory: 'es',
      style: true,
    },
    'vant',
  ],
]
module.exports = {
  // ...
  plugins
}
```

创建 src/plugins/vant.js 文件。

``` js
import { Button, Overlay } from 'vant'

export default (app) => {
  app.use(Button).use(Overlay)
}
```

在 src/main.js 导入并注册使用。

``` js
import useVantUI from '@/plugins/vant'

const app = createApp(App)

useVantUI(app)

app.mount('#app')
```

### 多环境变量

[Vue Cli 模式和环境变量](https://cli.vuejs.org/zh/guide/mode-and-env.html)

在 package.json 的 script 中配置模式：

``` json
"scripts": {
  "serve": "vue-cli-service serve --mode development",
  "build:test": "vue-cli-service serve --mode test",
  "build": "vue-cli-service build --mode production",
  "lint": "vue-cli-service lint"
},
```

在项目根目录分别创建对应的 `.env.[mode]` 文件并配置环境变量。

``` bash
# .env.test
NODE_ENV='production'
# must start with VUE_APP_
VUE_APP_BASE_URL='/'
VUE_APP_ENV='test'
```

*`NODE_ENV` 将决定应用运行的模式，是开发，生产还是测试，因此也决定了创建哪种 webpack 配置，因此最好不修改。*

使用方式： process.env.VUE_APP_ENV

### rem 适配

安装 `postcss-pxtorem` & `lib-flexible`。

在 vue.config.js 中添加配置：

``` js
module.exports = {
  css: {
    loaderOptions: {
      postcss: {
        plugins: [
          require('postcss-pxtorem')({ // 把px单位换算成rem单位
            rootValue: 37.5, // 换算基数，
            unitPrecision: 3, // 允许REM单位增长到的十进制数字,小数点后保留的位数。
            propList: ['*'],
            exclude: /(node_module)/,  // 默认false，可以（reg）利用正则表达式排除某些文件夹的方法，例如/(node_module)/ 。如果想把前端UI框架内的px也转换成rem，请把此属性设为默认值
            selectorBlackList: ['.van'], // 要忽略并保留为px的选择器，本项目我是用的vant ui框架，所以忽略他
            mediaQuery: false,  //（布尔值）允许在媒体查询中转换px。
            minPixelValue: 1 // 设置要替换的最小像素值
          })
        ]
      }
    }
  }
}
```

此时在页面写 px 会自动转为 rem，然后再在 main.js 引入 flexible：`import 'lib-flexible/flexible.js'` 来动态设置 html 的 `font-size` 。

### 根据环境引入 vconsole

~~不多说，下载引入即可。~~

``` js
// main.js

// const IS_PROD = process.env.VUE_APP_ENV === 'production'
// if (!IS_PROD) {
//   const VConsole = require('vconsole')
//   new VConsole()
// }
```

上面的方式可以让 `vconsole` 不初始化，但在 `webpack` 打包过程中 `vconsole` 已经被打包进去了：

![](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/微信截图_20211109114329.png)

143KB 还是挺大的，使用插件 [vconsole-webpack-plugin](https://www.npmjs.com/package/vconsole-webpack-plugin) 在打包过程中判断即可（*不过目前（2021-11-25）不管是使用 vconsole，还是 vconsole-webpack-plugin，我发现出现错误都只会报同一个错，不好排查，因此只在测试环境打包进去，开发环境就没有使用了*）：

``` js
const vConsolePlugin = require('vconsole-webpack-plugin')
const IS_PROD = process.env.VUE_APP_ENV === 'production'

module.exports = {
  configureWebpack: config => {
    //生产环境去掉vconsole调试器
    let pluginsDev = [
      new vConsolePlugin({
        filter: [],
        enable: [
          'test',
          // 'development'
        ].includes(process.env.VUE_APP_ENV)
      })
    ]
    config.plugins = [...config.plugins, ...pluginsDev]
  }
}
```

### axios 封装

下载 `axios` 并新建 src/utils/request.js：

::: details 点击查看代码

``` js
import axios from 'axios'
import qs from 'qs'

const pendingRequest = new Map()

// 生成 key
function generateReqKey(config) {
  const { method, url, params, data } = config
  return [method, url, qs.stringify(params), qs.stringify(data)].join("&")
}

// 用于把当前请求信息添加到pendingRequest对象中
function addPendingRequest(config) {
  const requestKey = generateReqKey(config)
  config.cancelToken = config.cancelToken || new axios.CancelToken((cancel) => {
    if (!pendingRequest.has(requestKey)) {
       pendingRequest.set(requestKey, cancel)
    }
  })
}

// 检查是否存在重复请求，若存在则取消已发的请求
function removePendingRequest(config) {
  const requestKey = generateReqKey(config)
  if (pendingRequest.has(requestKey)) {
    const cancelToken = pendingRequest.get(requestKey)
    cancelToken(requestKey)
    pendingRequest.delete(requestKey)
  }
}

const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_URL, // url = base api url + request url
  withCredentials: true,
  timeout: 10 * 1000,
})

// request 拦截器
service.interceptors.request.use(
  config => {
    removePendingRequest(config)
    addPendingRequest(config)
    return config
  },
  error => {
    console.log(error)
    return Promise.reject(error)
  }
)
// respone 拦截器
service.interceptors.response.use(
  response => {
    removePendingRequest(response.config)
    const responseData = response.data
    return Promise.resolve(responseData)
  },
  error => {
    removePendingRequest(error.config || {})
    if (axios.isCancel(error)) {
      console.log("已取消的重复请求：" + error.message)
    } else {
      // 添加异常处理
    }
    console.log('err' + error)
    return Promise.reject(error)
  }
)

export default service
```

:::

挂载到全局：

``` js
// main.js
import axios from '@/utils/request'

const app = createApp(App)

app.config.globalProperties.$axios = axios
```

### webpack 配置

代码略，直接看 vue.config.js 即可，主要功能：

- devServer / proxy 代理
- 别名
- webpack-bundle-analyzer 打包分析
- ...
