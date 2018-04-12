---
title: 【构建】webpack4.5升级踩坑
date: 2017-4-10 20:49:03
tags: 
  - webpack
  - 填坑
categories: 
  - webpack
---

> 正好离webpack4的升级也过了一个多月了，大佬们已经踩完第一波雷，官方也强推现在的稳定版本4.5，加上最近在看服务端渲染，正好弄个demo升级玩玩

<!-- more -->

## 相关依赖调整

node 版本 8.9.4 + 为佳，之前8.9.0升级了好久也没见什么问题，node版本升级比飞还快，能升就跟着升吧

webpack命令优化， 要装上
各种loader跟着升级到最新先- - 不然报错有点头疼，这个放后面说

"webpack": "^4.5.0",
"webpack-cli": "^2.0.14",
"webpack-dev-server": "^3.1.2",
"webpack-merge": "^4.1.2"

html-webpack-plugin file-loader url-loader style-loader 各种loader该升级的都升级吧，要相信社区的实力和效率

## webpack相关更新

### mode模式

```js
// webpack.config.js
const isDev = process.env.NODE_ENV === 'development'

module.exports = {
  config: {
    // 省略大部分代码
    mode: isDev ? 'development' : 'production'
  }
}
```

现在可以这么写。以前 dev 和 prd 环境分开设置的一些东西，在webpack 4官方已经为我们做好了预配置，不用麻烦些一堆东向了（感谢parcel）

**development模式下，将侧重于功能调试和优化开发体验：**

1. 浏览器调试工具
2. 开发阶段的详细错误日志和提示
3. 快速和优化的增量构建机制

**production模式下，将侧重于模块体积优化和线上部署**

1. 开启所有的优化代码
2. 更小的bundle大小
3. 去除掉只在开发阶段运行的代码
4. Scope hoisting和Tree-shaking
5. 自动启用uglifyjs对代码进行压缩

### 移除commonChunk 改为 optimization

之前为了将公共代码提取出来，我们会将入口中配置相关依赖 vendor。webpack打包时，会给每一个模块单独加上一个ID， 在有新的模块加入时，模块插入的顺序在中间，导致后面的模块顺序发生变化，后面的chunk就是为解决这一问题的，这样顺序不会变动，前面部分不会重复打包了

```js
// 省略
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor'
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'runtime'
    })
  ]
```

下面的代码即是optimize.splitChunks 中的一些配置参考，

```js
module.exports = {
  optimization: {
    runtimeChunk: {
      name: 'manifest'
    },
    minimizer: true, // [new UglifyJsPlugin({...})]
    splitChunks:{
      chunks: 'async',
      minSize: 30000,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      name: false,
      cacheGroups: {
        vendor: {
          name: 'vendor',
          chunks: 'initial',
          priority: -10,
          reuseExistingChunk: false,
          test: /node_modules\/(.*)\.js/
        },
        styles: {
          name: 'styles',
          test: /\.(scss|css)$/,
          chunks: 'all',
          minChunks: 1,
          reuseExistingChunk: true,
          enforce: true
        }
      }
    }
  }
}
```

从webpack4开始官方移除了commonchunk插件，改用了optimization属性进行更加灵活的配置，这也应该是从V3升级到V4的代码修改过程中最为复杂的一部分。其中变化有如下几方面：

* commonchunk配置项被彻底去掉，之前需要通过配置两次new webpack.optimize.CommonsChunkPlugin来分别获取vendor和manifest的通用chunk方式已经做了整合， 直接在optimization中配置runtimeChunk和splitChunks即可 ，提取功能也更为强大，具体配置见：splitChunks

* runtimeChunk可以配置成true，single或者对象，用自动计算当前构建的一些基础chunk信息，类似之前版本中的manifest信息获取方式。

* webpack.optimize.UglifyJsPlugin现在也不需要了，只需要使用optimization.minimize为true就行，production mode下面自动为true，当然如果想使用第三方的压缩插件也可以在optimization.minimizer的数组列表中进行配置

### ExtractTextWebpackPlugin调整

如果升级到 最新的 extract-text-webpack-plugin": "^4.0.0-beta.0" 也没问题，如果遇到了下面的错误，你懂的：

```js
 (node:4728) DeprecationWarning: Tapable.plugin is deprecated. Use new API on `.hooks` instead
(node:4728) DeprecationWarning: Tapable.apply is deprecated. Call apply on the plugin directly instead
/Users/x-kxem/myWorkSpace/vue-ssr-tech/node_modules/webpack/lib/Chunk.js:460
```

建议选用新的CSS文件提取插件mini-css-extract-plugin，配置如下：

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin")

module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css',
      chunkFilename: '[id].css'
    })
  ],
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader
        ]
      }
    ]
  }
}
```

### 生产环境下的配置优化

```js
const UglifyJsPlugin = require("uglifyjs-webpack-plugin")
const MiniCssExtractPlugin = require("mini-css-extract-plugin")
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin")

module.exports = {
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        cache: true,
        parallel: true,
        sourceMap: true 
      }),
      new OptimizeCSSAssetsPlugin({
        // use OptimizeCSSAssetsPlugin
      }) 
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'css/app.[name].css',
      // use contenthash *
      chunkFilename: 'css/app.[contenthash:12].css'
    })
  ]
  // ...
}
```

### 将多个 css chunk 合并成一个 css文件

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin")

module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        styles: {            
          name: 'styles',
          test: /\.scss|css$/,
          // 下面的设置就是将 css chunk合并为一个文件的
          chunks: 'all',    
          enforce: true
        }
      }
    }
  }
}
```

### 其他调整

```js
module.exports = {
  config: {
    // plugins: [
    //   new webpack.NoEmitOnErrorsPlugin()
    //   new webpack.NamedChunksPlugin()
    // ]
    optimization: {
      // noEmitOnErrors
      // namedModules
    }
  }
}
```

## 参考资料

感谢 webpack4升级完全指南 https://segmentfault.com/a/1190000014247030

webpack4更新日志 https://github.com/webpack/webpack/releases/tag/v4.0.0

《webpack 4: mode and optimization》 https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a

splitChunks  https://webpack.js.org/plugins/split-chunks-plugin/#optimization-splitchunks

mini-css-extract-plugin  https://github.com/webpack-contrib/mini-css-extract-plugin

webpack4配置工程实例  https://github.com/taikongfeizhu/react-mobx-react-router4-boilerplate

---

重点参考资料
 
webpack4 https://blog.csdn.net/qq_20334295/article/details/79401231

webpack4发布概览 https://zhuanlan.zhihu.com/p/34028750

webpack 4: mode and optimization https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a

webpack4新特性介绍 https://segmentfault.com/a/1190000013970017

webpack4升级指北 https://segmentfault.com/a/1190000013420383

webpack4升级指南以及从webpack3.x迁移 https://blog.csdn.net/qq_26733915/article/details/79446460