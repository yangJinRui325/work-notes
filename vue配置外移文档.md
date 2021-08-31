# 配置外移

## 应用场景：

不同运行环境下部署前端项目，需要打包不同路径的前端包。配置外移可以实不同环境下部署项目，只需要修改配置文件即可。

## 具体实施：

### 1、安装插件

```
npm i generate-asset-webpack-plugin -S
```

### 2、配置 vue.config.js

在 vue.config.js 中添加如下配置

```js
const GenerateAssetPlugin = require('generate-asset-webpack-plugin')

var createServerConfig = function (compilation) {
  // 配置需要的api接口
  const cfgJson = {
    // oss域名
    oss_base_url:
      'https://shujucang.oss-cn-hangzhou-zwynet-d01-a.internet.cloud.zj.gov.cn/',

    // 是否加密 true => 加密  false => 不加密
    is_encrypt: false,

    // 接口地址
    serve_base_url: 'http://192.168.1.24/zzyqddservice',
  }
  return JSON.stringify(cfgJson)
}

module.exports = {
  chainWebpack: (config) => {
    // ...
    // 配置外移
    config.plugin(GenerateAssetPlugin).use('generate-asset-webpack-plugin', [
      {
        // 此处是打包形成的配置文件名称，可自定义
        filename: 'serverconfig.json',
        fn: (compilation, cb) => {
          cb(null, createServerConfig(compilation))
        },
        extraFiles: [],
      },
    ])
  },
}
```

### 3、配置 main.js

```js
// 在本地添加一个serverconfig.js文件
import serverconfig from './serverconfig.js'

if (process.env.NODE_ENV === 'development') {
  // 本地开发时候只需要变更本地的serverconfig.js即可
  Vue.prototype.$configApiUrl = serverconfig
  initVue()
} else {
  getConfigJson()
}

// 获取外放的配置文件
function getConfigJson() {
  // 此处调用封装的axios请求
  request
    .get('serverconfig.json')
    .then((result) => {
      // 挂载到vue原型上面，这样就可以在项目中调用了
      Vue.prototype.$configApiUrl = result
      // 等待必要的环境变量请求回来了，再渲染页面
      initVue()
    })
    .catch((error) => {
      console.log('getConfigJson Error!', error)
    })
}

// 实例化vue
function initVue() {
  new Vue({
    router,
    store,
    render: (h) => h(App),
  }).$mount('#app')
}
```

### 4、本地新增 serverconfig.js 文件。 配置如下，保持和 vue.config.js 一致即可，本地开发时候调试方便

```js
// 自己本地维护，不提交代码库
export default {
  // oss域名
  oss_base_url:
    'https://shujucang.oss-cn-hangzhou-zwynet-d01-a.internet.cloud.zj.gov.cn/',

  // 是否加密 true => 加密  false => 不加密
  is_encrypt: false,

  // 接口地址
  serve_base_url: 'http://192.168.2.83:8022/npc-media-service',
}
```

### 5、调用方式

```js
// js文件中需要实例化vue再调用
new Vue().$configApiUrl.serve_base_url

// vue文件中调用方式：
this.$configApiUrl.serve_base_url
```
