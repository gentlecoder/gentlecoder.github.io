---
title: vue-cli生成的build及config文件夹中文件解读(2)webpack.prod.conf.js
date: 2018-04-02 09:57:43
comments: true
tags: 
  - 前端
  - vue
  - vue-cli
---

> 本文`vue-cli`版本是2.9.3，低于2.0版本的build文件夹中还存在**dev-client.js**和**dev-server.js**两个文件，现版本这两个文件中的配置已经移步到`build/webpack.dev.conf.js`中进行了。

<!--more-->

本文中的文件是直接运行`vue init webpack my-project`生成的，下面是build/webpack.prod.conf.js中相关代码和配置的说明:

```js
'use strict'
const path = require('path')
// 下面是utils工具配置文件，主要用来处理css类文件的loader
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
// 用merge的方式继承base.conf里面的配置
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
// copy-webpack-plugin使用来复制文件或者文件夹到指定的目录的
const CopyWebpackPlugin = require('copy-webpack-plugin')
// html-webpack-plugin是生成html文件，可以设置模板
const HtmlWebpackPlugin = require('html-webpack-plugin')
// extract-text-webpack-plugin这个插件是用来将bundle中的css等文件生成单独的文件，比如我们看到的app.css
const ExtractTextPlugin = require('extract-text-webpack-plugin')
// 压缩css代码的，还能去掉extract-text-webpack-plugin插件抽离文件产生的重复代码，因为同一个css可能在多个模块中出现所以会导致重复代码，一般都是配合使用
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
// 压缩js代码
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

const env = process.env.NODE_ENV === 'testing'
  ? require('../config/test.env')
  : require('../config/prod.env')

const webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      // 下面就是把utils配置好的处理各种css类型的配置拿过来，和dev设置一样，就是这里多了个extract: true，此项是自定义项，设置为true表示，生成独立的文件
      sourceMap: config.build.productionSourceMap,
      extract: true,
      usePostCSS: true
    })
  },
  // devtool开发工具，用来生成个sourcemap方便调试，只用于生产环境
  devtool: config.build.productionSourceMap ? config.build.devtool : false,
  output: {
    // 和base.conf中一致，输出文件的路径：config目录下的index.js，path.resolve(__dirname, '../dist')
    path: config.build.assetsRoot,
    // 有区别，输出文件加上的chunkhash
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    // 非入扣文件配置，异步加载的模块，输出文件加上的chunkhash
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      // 利用DefinePlugin插件，定义process.env环境变量为env
      'process.env': env
    }),
    new UglifyJsPlugin({
      uglifyOptions: {
        compress: {
          warnings: false  // 禁止压缩时候的警告信息
        }
      },
      sourceMap: config.build.productionSourceMap,  // 压缩后生成map文件
      parallel: true
    }),
    // extract css into its own file，已经很清楚了就是独立css文件，文件名和hash
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css'),
      // Setting the following option to `false` will not extract CSS from codesplit chunks.
      // Their CSS will instead be inserted dynamically with style-loader when the codesplit chunk has been loaded by webpack.
      // It's currently set to `true` because we are seeing that sourcemaps are included in the codesplit bundle as well when it's `false`, 
      // increasing file size: https://github.com/vuejs-templates/webpack/issues/1110
      allChunks: true,
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    new OptimizeCSSPlugin({
      cssProcessorOptions: config.build.productionSourceMap
        ? { safe: true, map: { inline: false } }
        : { safe: true }
    }),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: process.env.NODE_ENV === 'testing'
        ? 'index.html'
        : config.build.index,
      template: 'index.html',  // 模板是index.html加不加无所谓
      inject: true,  // 将js文件注入到body标签的结尾
      minify: {  // 压缩html页面
        removeComments: true,  // 去掉注释
        collapseWhitespace: true,  // 去除无用空格
        removeAttributeQuotes: true  // 去除无用的双引号
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'  // 可以对页面中引用的chunk进行排序，保证页面的引用顺序
    }),
    // keep module.id stable when vendor modules does not change
    new webpack.HashedModuleIdsPlugin(),
    // enable scope hoisting
    new webpack.optimize.ModuleConcatenationPlugin(),
    // split vendor js into its own file
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks (module) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    // 公共模块插件，便于浏览器缓存，提高程序的运行速度（哪些需要打包进公共模块需要取舍）
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      minChunks: Infinity
    }),
    // This instance extracts shared chunks from code splitted chunks and bundles them
    // in a separate chunk, similar to the vendor chunk
    // see: https://webpack.js.org/plugins/commons-chunk-plugin/#extra-async-commons-chunk
    new webpack.optimize.CommonsChunkPlugin({
      name: 'app',
      async: 'vendor-async',
      children: true,
      minChunks: 3
    }),

    // copy custom static assets
    // 复制项目中的静态文件，忽略.开头的文件
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})

if (config.build.productionGzip) {
  const CompressionWebpackPlugin = require('compression-webpack-plugin')
  // 该插件能够将资源文件压缩为.gz文件，并且根据客户端的需求按需加载
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

// 分析 Webpack 生成的包体组成并且以可视化的方式反馈给开发者
if (config.build.bundleAnalyzerReport) {
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig

```

