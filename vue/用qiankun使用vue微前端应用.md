# qiankun + vue 简单生成微前端应用

常见的项目 主体页面通过`iframe`去链接对应的tag页面
`iframe`的体验不太友善，不管用户层面 还是开发层面 会有一些陆陆续续的奇怪问题

`qiankun`是个不错的解决微前端方案 具体的可以到官网了解[qiankun](https://qiankun.umijs.org/zh)



## 创建主应用



创建主应用vue-app
```
vue create vue-app
```

主应用程序安装`qinkun`

```
$ yarn add qiankun  # or npm i qiankun -S

```

`qiankun`很简单，开箱即用 修改主应用的`main.js`

```
import { registerMicroApps, setDefaultMountApp, start } from "qiankun";
const startsWith = (app) => location => location.pathname.startsWith(app)

// 注册子应用
registerMicroApps([
  {
    name: 'vueapp',            // 子应用名称
    entry: '//localhost:8101',  // 子应用入口
    container: '#container',    // 子应用所在容器
    activeRule: startsWith('/vue'),         // 子应用触发规则（路径）
  }],
  {
    beforeLoad: [
      app => {
        console.log("before load", app);
      }
    ], // 挂载前回调
    beforeMount: [
      app => {
        console.log("before mount", app);
      }
    ], // 挂载后回调
    afterUnmount: [
      app => {
        console.log("after unload", app);
      }
    ] // 卸载后回调
  })

// 启动默认应用
setDefaultMountApp('/')

// 开启服务
start()
```

只需要用到`qiankun`的三个方法，就完成了主应用的配置

然后在新增微应用挂载的容器节点 和 跳转到微应用的链接
修改`App.vue`

```
<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/vue">微应用</router-link> |
      <router-link to="/about">About</router-link>
    </div>
      <!-- 主应用容器 -->
    <router-view/>
     <!-- 子应用容器 -->
    <div id="container"></div>
  </div>
</template>
```

这里就完成了主应用的简单配置



## 创建微应用

创建主应用vue-app
```
vue create vue-app
```
（微应用不需要install qiankun）

新增`public-path.js` :
```
if (window.__POWERED_BY_QIANKUN__) {
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```
> 运行`qiankun`的时候 会注入一个全局变量 能用到判断一些信息，比如是否在主应用打开

修改`main.js`
```
import './public-path';
import Vue from 'vue';
import VueRouter from 'vue-router';
import App from './App.vue';
import routes from './router';
import store from './store';

Vue.config.productionTip = false;

let router = null;
let instance = null;
function render(props = {}) {
  const { container } = props;
  router = new VueRouter({
    base: window.__POWERED_BY_QIANKUN__ ? '/vue/' : '/', // 通过window.__POWERED_BY_QIANKUN__判断是不是在微应用环境 如果是就匹配主应用的activeRule规则 如果不是就打开默认的页面
    mode: 'history',
    routes,
  });

  instance = new Vue({
    router,
    store,
    render: h => h(App),
  }).$mount(container ? container.querySelector('#app') : '#app');
}

// 独立运行时
if (!window.__POWERED_BY_QIANKUN__) {
  render();
}

export async function bootstrap() {
  console.log('[vue] vue app bootstraped');
}
export async function mount(props) {
  console.log('[vue] props from main framework', props);
  render(props);
}
export async function unmount() {
  instance.$destroy();
  instance.$el.innerHTML = '';
  instance = null;
  router = null;
}
```

最后修改打包的配置 和 端口
```
const { name } = require('./package');
module.exports = {
  devServer: {
    port: 8101,
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  },
  configureWebpack: {
    output: {
      library: `${name}-[name]`,
      libraryTarget: 'umd',// 把微应用打包成 umd 库格式
      jsonpFunction: `webpackJsonp_${name}`,
    },
  },
};
```


最后运行两个项目 在主应用上点击`微应用`的时候 就会跳转挂载到对应的微应用页面

(完)
