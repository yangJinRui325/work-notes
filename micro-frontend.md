# 浅析微前端框架 single-spa

## 一、应用场景

将一个单体大型应用，转化为多个可以独立运行、独立开发、独立部署、独立维护的应用的聚合。

解决的问题：  
1、随着项目迭代应用越来越庞大，难以维护。  
2、跨团队或跨部门协作开发项目导致效率低下的问题。

## 二、基于 single-spa 二次开发的微前端解决方案（蚂蚁金服的 qiankun）

> (qinkun 文档地址)：https://qiankun.umijs.org/zh/guide

## 三、使用方法

以 single-spa 为例，我们一起配置一下

### 1、配置子应用 app1

- 用 vue-cli 脚手架搭建一个轻量级的 vue 工程

- 改造 vue.config.js

```js
const package = require('./package.json')
module.exports = {
  // 告诉子应用在这个地址加载静态资源，否则会去基座应用的域名下加载
  publicPath: '//localhost:8091',
  // 开发服务器
  devServer: {
    port: 8081,
  },
  configureWebpack: {
    // 导出umd格式的包，在全局对象上挂载属性package.name，基座应用需要通过这个全局对象获取一些信息，比如子应用导出的生命周期函数
    output: {
      // library的值在所有子应用中需要唯一
      library: package.name,
      libraryTarget: 'umd',
    },
  },
}
```

- 安装 single-spa-vue

  > single-spa-vue 负责为 vue 应用生成通用的生命周期钩子，在子应用注册到 single-spa 的基座应用时需要用到

```js
npm i single-spa-vue -S
```

- 改造入口文件 main.js

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import singleSpaVue from 'single-spa-vue'

Vue.config.productionTip = false

const appOptions = {
  el: '#microApp', //基座上面用来挂载子应用的dom节点
  router,
  render: (h) => h(App),
}

// 支持应用独立运行、部署，不依赖于基座应用
if (!window.singleSpaNavigate) {
  delete appOptions.el
  new Vue(appOptions).$mount('#app')
}

// 基于基座应用，导出生命周期函数
const vueLifecycle = singleSpaVue({
  Vue,
  appOptions,
})

export function bootstrap(props) {
  console.log('app1 bootstrap')
  return vueLifecycle.bootstrap(() => {})
}

export function mount(props) {
  console.log('app1 mount')
  return vueLifecycle.mount(() => {})
}

export function unmount(props) {
  console.log('app1 unmount')
  return vueLifecycle.unmount(() => {})
}
```

- 更改试图文件

```html
<!-- /views/Home.vue -->
<template>
  <div class="home">
    <h1>app1 home page</h1>
  </div>
</template>
```

```html
<!-- /views/About.vue -->
<template>
  <div class="about">
    <h1>app1 about page</h1>
  </div>
</template>
```

- 环境配置文件

```
# .env文件：应用独立运行时的开发环境配置
NODE_ENV=development
VUE_APP_BASE_URL=/
```

```
# .env.micro文件：作为子应用运行时的开发环境配置
NODE_ENV=development
VUE_APP_BASE_URL=/
```

- 修改路由文件

```js
// /src/router/index.js
// ...
const router = new VueRouter({
  // 此处子应用1先用history模式
  mode: 'history',
  // 通过环境变量来配置路由的 base url
  base: process.env.VUE_APP_BASE_URL,
  routes,
})
// ...
```

- 在 package.json 的 script 中添加命令

```json
{
  "name": "app1",
  // ...
  "scripts": {
    // 独立运行
    "serve": "vue-cli-service serve",
    // 作为子应用运行
    "micro": "vue-cli-service serve --mode micro"
  }
  // ...
}
```

- 启动应用，此处演示子应用嵌入所以用子应用模式运行项目

```js
npm run micro
```

### 2、配置子应用 app2。配置一个子应用即可，这里两个用来做对比

> 和 app1 一样的配置，把 app1 修改成 app2。区别：这里把 app2 配置成 hash 模式

- 修改路由文件

```js
// /src/router/index.js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../views/Home.vue'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home,
  },
  {
    path: '/about',
    name: 'About',
    component: () =>
      import(/* webpackChunkName: "about" */ '../views/About.vue'),
  },
]

let newRoutes = routes
// 子应用模式下动态拼接路由
if (window.singleSpaNavigate) {
  newRoutes = routes.map((route) => {
    route.path = `${process.env.VUE_APP_BASE_URL}${route.path}`
    return route
  })
}
const router = new VueRouter({
  mode: 'hash',
  // history模式下base配置有用，hash模式下不可以，只能动态拼接路由
  // base: process.env.VUE_APP_BASE_URL,
  routes: newRoutes,
})

// hash模式下还要动态的在路由守卫地方拼接路径
router.beforeEach((to, from, next) => {
  if (window.singleSpaNavigate) {
    if (!to.path.includes(`${process.env.VUE_APP_BASE_URL}`)) {
      next({
        ...to,
        path: `${process.env.VUE_APP_BASE_URL}${to.path}`,
      })
    } else {
      next()
    }
  } else {
    next()
  }
})

export default router
```

### 3、配置基座应用 layout

- 通过脚手架搭建 layout
- 安装 single-spa

```node
npm i single-spa -S
```

- 修改入口文件

```js
// src/main.js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import { registerApplication, start } from 'single-spa'

Vue.config.productionTip = false

// 远程加载子应用
function createScript(url) {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script')
    script.src = url
    script.onload = resolve
    script.onerror = reject
    const firstScript = document.getElementsByTagName('script')[0]
    firstScript.parentNode.insertBefore(script, firstScript)
  })
}

// 记载函数，返回一个 promise
function loadApp(url, globalVar) {
  // 支持远程加载子应用
  return async () => {
    await createScript(url + '/js/chunk-vendors.js')
    await createScript(url + '/js/app.js')
    // 这里的return很重要，需要从这个全局对象中拿到子应用暴露出来的生命周期函数
    return window[globalVar]
  }
}

// 子应用列表
const apps = [
  {
    // 子应用名称
    name: 'app1',
    // 子应用加载函数，是一个promise
    app: loadApp('http://localhost:8091', 'app1'),
    // 当路由满足条件时（返回true），激活（挂载）子应用。
    // 如果基座是hash模式，此处判断要更改为：location.hash.startsWith('#/app1'),
    activeWhen: (location) => location.pathname.startsWith('/app1'),
    // 传递给子应用的对象
    customProps: {},
  },
  {
    name: 'app2',
    app: loadApp('http://localhost:8092', 'app2'),
    activeWhen: (location) => location.pathname.startsWith('/app2'),
    customProps: {},
  },
]

// 注册子应用
for (let i = apps.length - 1; i >= 0; i--) {
  registerApplication(apps[i])
}

new Vue({
  router,
  mounted() {
    // 启动
    start()
  },
  render: (h) => h(App),
}).$mount('#app')
```

- App.vue 文件修改

```html
<template>
  <div id="app">
    <div id="nav">
      <router-link to="/app1">app1</router-link> |
      <router-link to="/app2">app2</router-link>
    </div>
    <!-- 子应用容器 -->
    <div id="microApp">
      <router-view />
    </div>
  </div>
</template>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }

  #nav {
    padding: 30px;
  }

  #nav a {
    font-weight: bold;
    color: #2c3e50;
  }

  #nav a.router-link-exact-active {
    color: #42b983;
  }
</style>
```

- 路由修改

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const routes = []

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes,
})

export default router
```

- 启动项目

```
npm run serve
```

---

到这里利用 single-spa 搭建的小型微前端集成功能就好了

## 四、整体智治项目使用 qiankun 框架的踩坑记录
