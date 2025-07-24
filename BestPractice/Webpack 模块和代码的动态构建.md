## 需求描述
一个大型webpack + vue项目往往有上百个路由，而开发时我们只需要其中几个路由。在npm run dev时，若对所有路由都进行构建，往往耗时数分钟甚至更久，代码改动时热更新也十分缓慢。
所以需要有一种方法，可以在开发时动态地去选择哪些路由和页面是我需要去构建的。

## 思路和误区
#### 怎样才能加快构建速度？
基于vue2的项目的路由表内会引入许多的vue文件，一般一个路由/页面就是一个单独的.vue文件。
想要加快构建速度，麻烦一点的方法，手动注释掉大多数非必要的路由表，不引入这些vue文件，自然加快构建速度。
所以现在需要一种方法，在代码层面去完成动态构建，替代手动注释。

#### 哪些文件会被纳入构建？
一个文件是否会被构建，取决于这个文件是否**可能**在运行时代码里被引入。
换句话说，常规代码里所有被import的文件都会被构建，无法实现动态构建。
```js
// 语法错误，import语句必须在模块顶层
if(true){
  import './test1';
}

// 语法正确，但webpack打包时缺乏js执行能力，执行import时会因为是一个表达式而报错，两个文件都不会被构建
// Critical dependency: the request of a dependency is an expression
import (test ? "./test2.js" : "./test1.js")
```
从以上两种情况可以知道，静态import是无法动态去改变所要引入的文件的。

#### 构建和加载
在vue项目的路由文件中，动态import的写法还是比较常见的。那么考虑动态import可以吗？
这里要正确理解构建和加载的区别。参考[这个链接](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import)，动态import要求所引入的文件是一个确切存在的文件，并不意味着引入的文件动态参与构建。相反，webpack需要构建这个文件，以便运行时代码执行到此处时能够正确加载该文件。
动态import的`动态`指的是动态加载，而不是动态构建。它的优势是将引入的文件和其它运行时代码分割开来，成为独立的一块运行时代码，方便分包和按需加载（延迟加载），从而提升页面的首次加载速度。

```js
const routers = new Router({
	routes: 
  [
    {
      path: "/",
      name: "主页",
      isShow: true,
      component: Layout,
      redirect: "/sysworkbench-page",
      children: [{
        path: "/sysworkbench-page",
        name: "工作台",
        isShow: true,
        /* webpackChunkName 是引入的文件构建成的最后的chunk的命名 */
        component: () => import(/* webpackChunkName: "sysworkbenchModule" */ "@/views/staging")
      }
      ]
    }
  ]
});
```

#### 环境变量
明确以上两个点，其实就知道环境变量对这个需求无能为力，因为环境变量说到底也只是一个变量。环境变量的特殊之处在于它既可以在运行时使用，也能够被webpack在打包阶段使用。
更准确的说，环境变量是node环境的一个变量，所以理所当然在webpack构建阶段可以被使用。如果在运行时代码里有使用到某个环境变量，那么这个环境变量的值在构建阶段就被赋给了运行时内的某个变量，看起来就像在运行时内使用了环境变量。
注意，上面对于运行时内环境变量的描述是我自己的猜想，实际调试时发现存在一些问题，还没想明白。不建议在运行时内使用环境变量，可能会有未知错误。
```js
// 以下是一段在webpack打包时用的代码
module.exports = {
  context: path.resolve(__dirname, '../'),
  entry: {
    app: ['babel-polyfill', './src/main.js']
  },
  output: {
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production' // 环境变量
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
    }
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        // loader: 'vue-loader',
        use: [
          {
            loader: 'vue-loader',
            options: vueLoaderConfig
          }, 
          {
            loader: 'js-conditional-compile-loader',
            options: conditionalCompilerConfig
          }
        ],
      },
      {
        test: /\.js$/,
        use: [
          'babel-loader', 
          {
            loader: 'js-conditional-compile-loader',
            options: conditionalCompilerConfig
          }
        ],
        include: [resolve('src'), resolve('test'), resolve('node_modules/webpack-dev-server/client'),resolve('node_modules/@wangeditor')]
      },
    ]
  },
  node: {
    // prevent webpack from injecting useless setImmediate polyfill because Vue
    // source contains it (although only uses it if it's native).
    setImmediate: false,
    // prevent webpack from injecting mocks to Node native modules
    // that does not make sense for the client
    dgram: 'empty',
    fs: 'empty',
    net: 'empty',
    tls: 'empty',
    child_process: 'empty'
  },
}
```

#### 条件编译和webpack loader
现在，我们只能在webpack构建阶段想办法，让webpack在构建时，选择性地执行/忽略某些代码。
参考uniapp的多平台条件编译，其实我们想实现的也就是让webpack对路由的代码进行条件编译，符合条件的路由表就引入，不符合的就视作被注释的代码忽略掉。
如何实现条件编译？webpack提供了plugin和loader，都可以实现在构建前修改代码文件。
1. [conditional-webpack-plugin ](https://www.npmjs.com/package/conditional-webpack-plugin)
	一个插件，可以实现vue, js等文件的条件编译。这个插件非常新，用的人也非常少，应该只有作者自己在用；实测兼容性不好，在webpack 3.6.0下，vue文件的条件编译失效；在webpack5下也有问题。后续可以考虑我自己改一下。
2. [js-conditional-compile-loader](https://www.npmjs.com/package/js-conditional-compile-loader)
	参考资料: 
	https://github.com/hzsrc/js-conditional-compile-loader/blob/master/readme-cn.md
	https://cloud.tencent.com/developer/news/893340
3. [ifdef-loader](https://github.com/nippur72/ifdef-loader)
4. uglify-js-plugin + define-plugin
	参考资料：https://cloud.tencent.com/developer/news/893340

## 实现
基于`js-conditional-compile-loader`实现webpack动态构建

#### 安装
`npm i -D js-conditional-compile-loader`
#### webpack配置
vue文件和js文件需要使用这个loader，这个loader需要第一个执行，所以放在loader数组的最后一项。

```js
/* webpack config */
const conditionalCompilerConfig = require("./conditional-compiler.conf")
// ...
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.vue$/,
        use: [
          {
            loader: 'vue-loader',
            options: vueLoaderConfig
          }, 
          // 放在最后一项
          {
            loader: 'js-conditional-compile-loader',
            options: conditionalCompilerConfig
          }
        ],
      },
      {
        test: /\.js$/,
        use: [
          'babel-loader', 
          // 放在最后一项
          {
            loader: 'js-conditional-compile-loader',
            options: conditionalCompilerConfig
          }
        ],
        include: [resolve('src'), resolve('test'), resolve('node_modules/webpack-dev-server/client'),resolve('node_modules/@wangeditor')]
      },
      // ...
    ]
  },
  // ...
}
```

#### webpack条件编译所使用的变量
1. 条件编译所使用到的变量提取为一个文件单独维护
2. 导入到webpack配置文件中作为`js-conditional-compile-loader`的option。
``` js
/* loader option */
const RouteEnableMap = {
  IndexRoutes: true, // 主页模块
  SalesRoutes: false, // 销售模块
  PurchaseRoutes: false, // 采购模块
  WareHouseRoutes: false, // 仓库模块
  ProduceRoutes: false, // 生产模块
  MachiningRoutes: false, // 加工模块
  MosRoutes: false, // Mos模块
  PlatFormRoutes: false, // 平台级模块
  BoardRoutes: false, // 看板模块
  FinancialRoutes: false, // 财务模块
  BaseRoutes: false, // 基础模块
  AndonRoutes: false, // 安灯模块
}

const otherVars = {}

const config = {
  ...otherVars,
  ...RouteEnableMap
}

// 在生产环境(npm run build)时将所有路由变量置为true，因为在打包线上环境时，所有路由都是必要的。
for (const route in RouteEnableMap) {
  config[route] = process.env.NODE_ENV === 'production' ? true : RouteEnableMap[route] 
}

module.exports = config
```
#### router.js
```js
import Vue from "vue";
import Router from "vue-router";
import Layout from "@/Layout/index";
import App from '../App'
Vue.use(Router);

function genRoutes(){
  const routes = []

  /* 基础路由 */
  /* IFTRUE_IndexRoutes */
  const IndexRoutes = {
    // ...
  }
  routes.push(IndexRoutes)
  /* FITRUE_IndexRoutes */

  /* 销售模块路由 */
  /* IFTRUE_SalesRoutes */
  const SalesRoutes = {
    // ...
  }
  routes.push(SalesRoutes)
  /* FITRUE_SalesRoutes */

  // ...
  return routes
}

const routers = new Router({
	routes: genRoutes()
});
export default routers;
```

## 引申
这个方案所使用到的条件编译是十分强大的，可以根据业务有其它的用处，比如同一份开发代码，根据商户权限的不同，生成不同的线上环境代码供运维部署。
这里用的这个插件功能上还不是很强大，仅能够识别全局变量，不能识别表达式和对象特定键值。条件编译的嵌套不知道是否可以。