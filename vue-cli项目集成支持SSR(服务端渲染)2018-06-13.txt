---
date: 2018-06-13 14:15
status: public
title: vue-cli项目集成支持SSR(服务端渲染)
---

## 约定
`vue`的项目在`node`上运行, 必须编写`通用代码`,我们必要遵守下面的这些约定
> 通用代码: 服务器渲染的 Vue.js 应用程序也可以被认为是"同构"或"通用"，因为应用程序的大部分代码都可以在`服务器`和`客户端`上运行

1. 服务器上的数据响应,
客户端和服务器端都应该是都是`全新的`、`独立的`应用程序
保证服务端渲染过程需的确定性，也就是将在服务器上`“预取”`数据- 这意味着在我们开始渲染时，我们的应用程序就已经解析完成其状态。也就是说，将数据进行响应式的过程在服务器上是多余的，所以默认情况下禁用。`禁用响应式数据`，还可以避免将「数据」转换为「响应式对象」的性能开销
2. 组件生命周期钩子函数
由于没有动态更新，所有的生命周期钩子函数中，只有 `beforeCreate` 和 `created` 会在服务器端渲染(SSR)过程中被调用。这就是说任何其他生命周期钩子函数中的代码（例如 beforeMount 或 mounted），只会在客户端执行。
3. 访问特定平台API
通用代码不可接受特定平台的 API，因此如果你的代码中，直接使用了像` window` 或 `documen`t，这种仅浏览器可用的全局变量，则会在 Node.js 中执行时抛出错误，反之也是如此。
对于共享于服务器和客户端，但用于不同平台 API 的任务，建议将平台特定实现包含在通用 API 中，或者使用为你执行此操作的 library。例如，axios 是一个 HTTP 客户端，可以向服务器和客户端都暴露相同的 API。
4. 自定义指令
大多数自定义指令直接操作 DOM，因此会在服务器端渲染(SSR )过程中导致错误。有两种方法可以解决这个问题：
 1. 推荐使用组件作为抽象机制，并运行在「虚拟 DOM 层级(Virtual-DOM level)」（例如，使用渲染函数(render function)）。
 2. 如果你有一个自定义指令，但是不是很容易替换为组件，则可以在创建服务器 renderer 时，使用 directives 选项所提供"服务器端版本(server-side version)"
详见[API](https://ssr.vuejs.org/zh/guide/universal.html)

## 借鉴官方给出的轮子上改造
官方给出的轮子[vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0/); 单纯看vue官方的文档有点不懂，配合给出的轮子食用最佳，本文参考官方的轮子对`vue-cli`进行改造. 更多的详细可以到官方文档查看
### 初始化项目
有一点要注意的是，服务端渲染(ssr)，运行的资源是项目`build`打包之后的文件, 所以一直强调`通用代码`的`重要`性
```
# 安装vue-cli
npm install -g vue-cli
# 创建项目, 命名为 ssr-vue-demo
vue init webpack ssr-vue-demo
cd vue-ssr-demo
cnpm install
npm run dev
```
好，项目启动了，会看到下面的界面

![](~/10-33-54.jpg)
### 开始改造
#### 1. 安装
`npm install vue vue-server-renderer --save`
注意
- `vue-server-renderer` 和` vue` 必须匹配版本

#### 2. 项目的src目录下新建两个js
你的src目录将会是这样。
```
src
├── components
│   ├── helloWord.vue
│   └── ssr.vue
├── App.vue
├── main.js # 通用 entry(universal entry)
├── entry-client.js # 仅运行于浏览器
└── entry-server.js # 仅运行于服务器
```
新建两个js文件, ` entry-client.js` 和 ` entry-server.js`.
` entry-client.js`写入内容:
```
import { createApp } from './main'
const { app, router } = createApp()
// 因为可能存在异步组件，所以等待router将所有异步组件加载完毕，服务器端配置也需要此操作
router.onReady(() => {
  app.$mount('#app')
})
```
` entry-server.js`写入内容:
```
// entry-server.js
import { createApp } from './main'
export default context => {
  // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise，
  // 以便服务器能够等待所有的内容在渲染前，
  // 就已经准备就绪。
  return new Promise((resolve, reject) => {
    const { app, router } = createApp()
    // 设置服务器端 router 的位置
    router.push(context.url)
    // 等到 router 将可能的异步组件和钩子函数解析完
    router.onReady(() => {
      const matchedComponents = router.getMatchedComponents()
      // 匹配不到的路由，执行 reject 函数，并返回 404
      if (!matchedComponents.length) {
        // eslint-disable-next-line
        return reject({ code: 404 })
      }
      // Promise 应该 resolve 应用程序实例，以便它可以渲染
      resolve(app)
    }, reject)
  })
}
```
ok，这里分别对浏览器和服务器所要跑的代码分别做了处理
#### 3. 配置路由
`components`目录下新建文件`ssr.vue`,并写入内容:
```
<template>
  <div>
    跳回首页
    <div>
      <router-link to="/">Home</router-link>
    </div>
    <div><h2>当前的环境是{{status}}</h2></div>
    <div><span>{{count}}</span></div>
    <div><button @click="count++">+1</button></div>
  </div>
</template>
<script>
  export default {
    data () {
      return {
	    status: process.env.VUE_ENV === 'server' ? 'server' : 'client',
        count: 2
      }
    }
  }
</script>
```

修改`route`文件的配置

> 不太懂的可以参考[官方文档](https://router.vuejs.org/zh/)学习，这里只作整体的操作。
```
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'

Vue.use(Router)
export function createRouter() {
	return new Router({
		mode: 'history',
		routes: [{
			path: '/index',
			name: 'HelloWorld',
			component: HelloWorld
		}, {
			path: '/ssr',
			name: 'ssr',
			component: () => import ('@/components/ssr') // 异步组件
		}]
	});
};


```
`HelloWorld.vue`和`ssr.vue`的引入方式为啥不一样? 因为`ssr.vue`引入使用了异步组件的方法
> 应用程序的代码分割或惰性加载，有助于减少浏览器在初始渲染中下载的资源体积，可以极大地改善大体积 bundle 的可交互时间。这里的关键在于，对初始首屏而言，"只加载所需"。
Vue 提供异步组件作为第一类的概念，将其与 [webpack 2 所支持的使用动态导入作为代码分割点](https://webpack.js.org/guides/code-splitting/)相结合

#### 4. 改造main.js
```
import Vue from 'vue'
import App from './App'
import { createRouter } from './router'

export function createApp () {
  // 创建 router 实例
  const router = new createRouter()
  const app = new Vue({
    // 注入 router 到根 Vue 实例
    router,
    render: h => h(App)
  })
  // 返回 app 和 router
  return { app, router }
}
```
我们需要为每一次的请求都创建一个新的`new Vue()`的实例, 也为了能在两端上使用, 同样为了`避免状态单例`，所以最好的办法就是创建一个`工厂函数 `进行支持

#### 5. webpack的配置 
vue相关的代码配置已经大概完成(数据遇取, 缓存等问题后面在看). 现在修改webpack的配置, 这里相对比较困难, 不过多实操几次就没事了
先来看看`vue-cli`给出的配置
```
build
├── webpack.base.conf.js	 # 基础通用配置
├── webpack.dev.conf.js
└── webpack.prod.conf.js  
```
我们依然保持这个配置, 新增多一个js文件`webpack.server.conf.js`, 用作打包(build)服务器端的文件

##### 5.1 修改webpack.base.conf.js
`entry`入口需要更改成`app: './src/entry-client.js'` ，因为前面更改了`main.js`导出的是一个工厂函数`createApp` 所以这里要把入口更改成浏览器运行的文件
##### 5.2 生成客户端构建清单client manifest
我们需要在`webpack.prod.conf.js `配置中引入一个插件, 并配置到`plugin`中即可，修改内容如下
```
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin') // 引入
// ...
// ... 
plugins: [
new webpack.DefinePlugin({
  'process.env': env,
  'process.env.VUE_ENV': 'client' // 增加process.env.VUE_ENV
}),
// 此插件在输出目录中
// 生成 `vue-ssr-client-manifest.json`。
new VueSSRClientPlugin(),
// ....
]
```
##### 5.3 webpack服务器端配置
这里需要安装一个新插件`cnpm install webpack-node-externals --save-dev`
`webpack.server.conf.js`配置如下:
```
const webpack = require('webpack')
const merge = require('webpack-merge')
const nodeExternals = require('webpack-node-externals')
const baseConfig = require('./webpack.base.conf.js')
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin')
// 去除打包css的配置
baseConfig.module.rules[1].options = ''

module.exports = merge(baseConfig, {
  // 将 entry 指向应用程序的 server entry 文件
  entry: './src/entry-server.js',
  // 这允许 webpack 以 Node 适用方式(Node-appropriate fashion)处理动态导入(dynamic import)，
  // 并且还会在编译 Vue 组件时，
  // 告知 `vue-loader` 输送面向服务器代码(server-oriented code)。
  target: 'node',
  // 对 bundle renderer 提供 source map 支持
  devtool: 'source-map',
  // 此处告知 server bundle 使用 Node 风格导出模块(Node-style exports)
  output: {
    libraryTarget: 'commonjs2'
  },
  // https://webpack.js.org/configuration/externals/#function
  // https://github.com/liady/webpack-node-externals
  // 外置化应用程序依赖模块。可以使服务器构建速度更快，
  // 并生成较小的 bundle 文件。
  externals: nodeExternals({
    // 不要外置化 webpack 需要处理的依赖模块。
    // 你可以在这里添加更多的文件类型。例如，未处理 *.vue 原始文件，
    // 你还应该将修改 `global`（例如 polyfill）的依赖模块列入白名单
    whitelist: /\.css$/
  }),
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
      'process.env.VUE_ENV': 'server'
    }),
    // 这是将服务器的整个输出
    // 构建为单个 JSON 文件的插件。
    // 默认文件名为 `vue-ssr-server-bundle.json`
    new VueSSRServerPlugin()
  ]
})
```
##### 5.4 配置打包命令
在`package.json`增加服务器端构建命令
```
"scripts": {
	//...
	"build:client": "node build/build.js",
    "build:server": "cross-env NODE_ENV=production webpack --config build/webpack.server.conf.js --progress --hide-modules",
    "build": "rimraf dist && npm run build:client && npm run build:server"
}
```
##### 5.5 修改index.html
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>vue-ssr-demo</title>
  </head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>

```
删掉原来的`<div id="app">`， 只需要在body保留一个标记`  <!--vue-ssr-outlet-->`，服务器端会在这个标记的位置自动生成一个`<div id="app" data-server-rendered="true">`，客户端会通过`app.$mount('#app')`挂载到服务端生成的元素上，并变为响应式的
> 这里修改了之后运行`npm run dev`会报错, 因为dev也是使用了这个模版, 最快的方式就是在新建一个html模版, 供dev使用
再新建一个html模版`index.dev.html`, 写入以下内容
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>vue-ssr-demo</title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

打开`webpack.dev.conf.js`文件, 将`index.html`修改成`index.dev.html`即可

##### 5.6 运行构建命令
ok， 差不多看到曙光了, 现在运行
`npm run build`
然后在dist目录下会看到生成了两个`JSON`文件, `vue-ssr-server-bundle.js`   和  `vue-ssr-client-manifest.json`  , 这两个文件都会应用在 node 端，进行服务器端渲染与注入静态资源文件。
> 如果出现cross-env找不到，请安装npm install --save cross-env
接下来我们构建服务器端

#### 6. 构建服务器
这里用到`express`进行帮助.
`cnpm install express --save`
在项目根目录创建`server.js`, 写入内容:
```
const express = require('express');
const app = express()
const fs = require('fs')
const path = require('path')
const { createBundleRenderer } = require('vue-server-renderer')
const resolve = file => path.resolve(__dirname, file)

// 生成服务端渲染函数
const renderer = createBundleRenderer(require('./dist/vue-ssr-server-bundle.json'), {
  // 推荐
  runInNewContext: false,
  // 模板html文件
  template: fs.readFileSync(resolve('./index.html'), 'utf-8'),
  // client manifest
  clientManifest: require('./dist/vue-ssr-client-manifest.json')
});
app.use(express.static('dist'));

app.get('*', (req, res) => {
    res.setHeader('Content-Type', 'text/html')
    const context = {
      title: '服务端渲染测试', // {{title}}
      url: req.url
    }
    renderer.renderToString(context, (err, html) => {
    
      res.end(html)
    });
    
});

const port = process.env.PORT || 8081
app.listen(port, () => {
  console.log(`server started at localhost:${port}`)
})
```
ok. 现在运行
```
node server.js
```
浏览器打开`http://127.0.0.1:8081/ssr`，查看网页源码

![](~/10-56-43.jpg)
ok, 服务端渲染改造成功， 接下来就是数据预取部分