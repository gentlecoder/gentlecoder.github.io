---
title: vue-cli生成的build及config文件夹中文件解读(3)webpack.dev.conf.js
date: 2018-04-02 11:04:36
comments: true
tags: 
  - 前端
  - vue
  - vue-cli
---

> 本文`vue-cli`版本是2.9.3，低于2.0版本的build文件夹中还存在**dev-client.js**和**dev-server.js**两个文件，现版本这两个文件中的配置已经移步到`build/webpack.dev.conf.js`中进行了。

<!--more-->

本文中的文件是直接运行`vue init webpack my-project`生成的，下面是build/webpack.dev.conf.js中相关代码和配置的说明:

```js
'use strict'
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const path = require('path')
const baseWebpackConfig = require('./webpack.base.conf')
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
// 下面这个插件是用来把webpack的错误和日志收集起来，漂亮的展示给用户
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
const portfinder = require('portfinder')

const HOST = process.env.HOST
const PORT = process.env.PORT && Number(process.env.PORT)

const devWebpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap, usePostCSS: true })
  },
  // cheap-module-eval-source-map is faster for development
  devtool: config.dev.devtool,

  // these devServer options should be customized in /config/index.js
  devServer: {
    // 日志级别
    clientLogLevel: 'warning',
    // 使用H5的history来做返回，而不是浏览器url
	// 用来解决单页面应用，点击刷新按钮和通过其他search值定位页面的404错误
    historyApiFallback: {
      rewrites: [
        { from: /.*/, to: path.posix.join(config.dev.assetsPublicPath, 'index.html') },
      ],
    },
    // 热加载
    hot: true,
    contentBase: false, // since we use CopyWebpackPlugin.
    // 如果为 true ，开启虚拟服务器时，为你的代码进行压缩。加快开发流程和优化的作用
    compress: true,
    host: HOST || config.dev.host,
    port: PORT || config.dev.port,
    // 编译完成打开浏览器
    open: config.dev.autoOpenBrowser,
    // 在浏览器上全屏显示编译的errors或warnings
    overlay: config.dev.errorOverlay
      ? { warnings: false, errors: true }
      : false,
    publicPath: config.dev.assetsPublicPath,
    proxy: config.dev.proxyTable,
    // true，则终端输出的只有初始启动信息。 webpack 的警告和错误是不输出到终端的。
    quiet: true, // necessary for FriendlyErrorsPlugin
    // 监听改动，每隔（你设定的）多少时间查一下有没有文件改动过
    watchOptions: {
      poll: config.dev.poll,
    }
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': require('../config/dev.env')
    }),
    // 热替换
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NamedModulesPlugin(), // HMR shows correct file names in console on update.
    new webpack.NoEmitOnErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.dev.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})

module.exports = new Promise((resolve, reject) => {
  portfinder.basePort = process.env.PORT || config.dev.port
  portfinder.getPort((err, port) => {
    if (err) {
      reject(err)
    } else {
      // publish the new Port, necessary for e2e tests
      process.env.PORT = port
      // add port to devServer config
      devWebpackConfig.devServer.port = port

      // Add FriendlyErrorsPlugin
      devWebpackConfig.plugins.push(new FriendlyErrorsPlugin({
        compilationSuccessInfo: {
          messages: [`Your application is running here: http://${devWebpackConfig.devServer.host}:${port}`],
        },
        onErrors: config.dev.notifyOnErrors
        ? utils.createNotifierCallback()
        : undefined
      }))

      resolve(devWebpackConfig)
    }
  })
})

```

