---
title: 【Vue】vue-cli改多页配置及踩坑笔记
date: 2017-08-24 19:49:03
tags: 
  - vue
  - webpack
categories: 
  - vue
  - 配置
---

> vue写起项目来很爽，效率也着实高，加上官方的vue-cli脚手架也是各种爽。不过直接搭出来是单页，在最近的项目实践上加上网上的参考。弄了套demo出来，可以参考我的项目，要是感兴趣记得start+fork下，谢谢

<!-- more -->

那么还是一步步来来吧，也算是记下笔记，方便以后搭建新项目。

npm install vue-cli -g

vue init webpack my-project

注意，`node`和`npm`的版本，越高越好。vue-cli的配置就跳过不说了，根据需要选择ESLint，在这里单元测试就先去掉了，router记得选上

npm install  
npm run dev

然后可以看到Hello Vue

怎么写我们知道了，这是一个单页组件。那么接下来，我们开始正式配置我们的多页路由项目吧

## 开始配置我们的多页路由项目吧

### 第一步 调整项目结构

|-- build
|-- config
|-- node_modules
|-- src
|-- static
|-- ... 省略了
|-- ... index.html

既然是多页，我们把 index.html 换个地方吧。这是我目前的vue项目架构（不够好，但慢慢提升学习吧，以后会更改）

你们参考下图片吧… 大致是这样


### 第二步 修改webpack配置：

需要安装一个很重要的包

    npm install glob --save-dev

接着配置config下面的index.js文件：(增加的地方我下面标注了)

```js
var path = require('path')
var glob = require('glob')  // - add -

var build = {
  env: require('./prod.env'),
  // - remove - index: path.resolve(__dirname, '../dist/index.html'),
  assetsRoot: path.resolve(__dirname, '../dist'),
  assetsSubDirectory: 'static',
  assetsPublicPath: '/',
  productionSourceMap: true,
  productionGzip: false,
  productionGzipExtensions: ['js', 'css'],
  bundleAnalyzerReport: process.env.npm_config_report
}

//根据getEntry获取所有入口主页面
var pages = getEntry('src/pages/**/*.html');

//每个入口页面生成一个入口添加到build中
for (var pathname in pages) {
  build[pathname] = path.resolve(__dirname, '../dist/' + pathname + '.html')
}

module.exports = {
  build: build,//生成的配置build
  dev: {
    env: require('./dev.env'),
    port: 8080,
    autoOpenBrowser: false,
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {
      // 设置代理跨域 - 根据需要来就好
      '/api': {
        // 我用了 easy-mock 意思为 项目/api 的请求实际上 会转发到 target的地址上
        target: 'https://www.easy-mock.com/mock/597a9bf0a1d30433d8401855/api/edr3/api',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    },
    cssSourceMap: false
  }
}

//获取所有入口文件
function getEntry(globPath) {
  var entries = {};
  var basename;

  glob.sync(globPath).forEach(function(entry) {
    basename = path.basename(entry, path.extname(entry));
    entries[basename] = entry;
  });

  return entries;
}
```

这样就配置完了index.js了。然后是配置 `build/webpack.base.conf.js`

```js
var path = require('path')
var glob = require('glob') //  - add -
var utils = require('./utils')
var config = require('../config')
var vueLoaderConfig = require('./vue-loader.conf')
var entries = getEntry('./src/pages/**/*.js')

function resolve(dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = {
  // entry: {
  //  app: './src/main.js'
  // },
  entry: entries,  // 改成这样
  output: {
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production' ? config.build.assetsPublicPath : config.dev.assetsPublicPath
  },
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    modules: [
      resolve('src'),
      resolve('node_modules')
    ],
    alias: {
      'vue$': 'vue/dist/vue.common.js',
      '@': resolve('src')
      // 可自行来配， 比如若配了src的alias话， vue里 写 地址可 'src/*.js' 就是 /src/*.js了
    }
  },
  module: {
    rules: [{
      test: /\.vue$/,
      loader: 'vue-loader',
      options: vueLoaderConfig
    }, {
      test: /\.js$/,
      loader: 'babel-loader',
      include: [resolve('src'), resolve('test')]
    }, {
      test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
      loader: 'url-loader',
      query: {
        limit: 10000,
        name: utils.assetsPath('img/[name].[hash:7].[ext]')
      }
    }, {
      test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
      loader: 'url-loader',
      query: {
        limit: 10000,
        name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
      }
    }]
  }
}

//获取入口js文件
function getEntry(globPath) {
  var entries = {},
    basename, tmp, pathname;

  glob.sync(globPath).forEach(function(entry) {
    basename = path.basename(entry, path.extname(entry));
    pathname = basename.split("_")[0];
    entries[pathname] = entry;
  });
  return entries;
}
```

这个文件主要是配置entry: entries入口，剩下的两个配置文件类似。直接复制了，参考以下代码：

build/webpack.dev.conf.js文件的配置

```js
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
var glob = require('glob') // 新增这两行
var path = require('path')

Object.keys(baseWebpackConfig.entry).forEach(function(name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})

module.exports = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
    new FriendlyErrorsPlugin(),
  ]
})

function getEntry(globPath) {
  var entries = {},
    basename;

  glob.sync(globPath).forEach(function(entry) {

    basename = path.basename(entry, path.extname(entry));
    entries[basename] = entry;
  });
  return entries;
}

var pages = getEntry('src/pages/**/*.html');

for (var pathname in pages) {
  // 配置生成的html文件，定义路径等
  var conf = {
    filename: pathname + '.html',
    template: pages[pathname], // 模板路径
    inject: true, // js插入位置
    chunks: [pathname]
  };
  module.exports.plugins.push(new HtmlWebpackPlugin(conf));
}
```

webpack.prod.conf.js文件的配置

```js
var path = require('path')
var glob = require('glob') // - add - 
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
var env = config.build.env
// 增加的
var plugins = [
  new webpack.DefinePlugin({
    'process.env': env
  }),
  new webpack.optimize.UglifyJsPlugin({
    compress: {
      warnings: false
    },
    sourceMap: true
  }),
  new ExtractTextPlugin({
    filename: utils.assetsPath('css/[name].[contenthash].css')
  }),
  new OptimizeCSSPlugin({
    cssProcessorOptions: {
      safe: true
    }
  }),
  new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks: function (module, count) {
      return (
        module.resource &&
        /\.js$/.test(module.resource) &&
        module.resource.indexOf(
          path.join(__dirname, '../node_modules')
        ) === 0
      )
    }
  }),
  new webpack.optimize.CommonsChunkPlugin({
    name: 'manifest',
    chunks: ['vendor']
  }),
  new CopyWebpackPlugin([
    {
      from: path.resolve(__dirname, '../static'),
      to: config.build.assetsSubDirectory,
      ignore: ['.*']
    }
  ])
];
function getEntry(globPath) {
  var entries = {},
    basename;

  glob.sync(globPath).forEach(function(entry) {

    basename = path.basename(entry, path.extname(entry));
    entries[basename] = entry;
  });
  return entries;
}
var pages = getEntry('src/pages/**/*.html');
for (var pathname in pages) {
  var conf = {
    filename: process.env.NODE_ENV === 'testing' ? pathname + '.html' : config.build[pathname],
    template: pages[pathname],
    inject: true,
    minify: {
      removeComments: true,
      collapseWhitespace: true,
      removeAttributeQuotes: true
    },
    // 关键一行 加上，不错到时会报 webpackJsonp not defin的错
    chunks: ['manifest', 'vendor', pathname]
  }
  plugins.push(new HtmlWebpackPlugin(conf));
}
// ----------增加的--
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: plugins
})
if (config.build.productionGzip) {
  var CompressionWebpackPlugin = require('compression-webpack-plugin')
  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}
if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}
module.exports = webpackConfig
```

好了，所有配置文件写完。但现在还跑不起来，我们再把 `src/pages` `src/router` 目录下的文件改一改，具体参考我的项目

就可以跑起来我们的vue多页项目了