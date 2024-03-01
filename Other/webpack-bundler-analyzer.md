webpack-bundle-analyzer 是 webpack 的插件，需要配合 webpack 和 webpack-cli 一起使用。这个插件可以读取输出文件夹（通常是 dist）中的 stats.json 文件，把该文件可视化展现，生成代码分析报告，可以直观地分析打包出的文件有哪些，及它们的大小、占比情况、各文件 Gzipped 后的大小、模块包含关系、依赖项等，对应做出优化，从而帮助提升代码质量和网站性能。

## 使用方法
#### webpack插件配置
```js 
// webpack构建配置的配置文件
module.exports = {
  build: {
    //...

    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  }
}
```
```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
const webpackConfig = merge(baseWebpackConfig, {
  module: {
    // ...
  },
  devtool: config.build.productionSourceMap ? config.build.devtool : false,
  output: {
    // ...
  },
  plugins: [
    // plugins...
  ]
})

// `npm run build --report`时为true
if (process.env.npm_config_report) { 
    // 默认配置的具体配置项
    // new BundleAnalyzerPlugin({
    //   analyzerMode: 'server',
    //   analyzerHost: '127.0.0.1',
    //   analyzerPort: '8888',
    //   reportFilename: 'report.html',
    //   defaultSizes: 'parsed',
    //   openAnalyzer: true,
    //   generateStatsFile: false,
    //   statsFilename: 'stats.json',
    //   statsOptions: null,
    //   excludeAssets: null,
    //   logLevel: info
    // })
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig
```
#### 查看报告
```
npm run build --report
```
#### 参数配置
![[Pasted image 20240301144327.png]]
![[Pasted image 20240301144701.png]]
![[Pasted image 20240301144739.png]]