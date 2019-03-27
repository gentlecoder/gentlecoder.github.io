---
title: vue-cli生成的build及config文件夹中文件解读(1)webpack.base.conf.js
date: 2018-03-30 17:25:23
comments: true
tags: 
  - 前端
  - vue
  - vue-cli
---

> 本文`vue-cli`版本是2.9.3，低于2.0版本的build文件夹中还存在**dev-client.js**和**dev-server.js**两个文件，现版本这两个文件中的配置已经移步到`build/webpack.dev.conf.js`中进行了。

<!--more-->

本文中的文件是直接运行`vue init webpack my-project`生成的，下面是build/webpack.base.conf.js中相关代码和配置的说明:

```javascript
'use strict'
const path = require('path') // 使用nodejs path模块
const utils = require('./utils') // css类的loader配置集。主要用来处理css-loader和vue-style-loader
const config = require('../config') // // 项目自定义的配置文件
const vueLoaderConfig = require('./vue-loader.conf') // vue-loader.conf配置文件是用来解决各种css文件的

// 获取路径的函数，因为该文件在项目的二级目录build下，要找到src这样的二级目录，需要加上../
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}

// eslint校验
const createLintingRule = () => ({
  test: /\.(js|vue)$/,
  loader: 'eslint-loader',
  enforce: 'pre',
  include: [resolve('src'), resolve('test')],
  options: {
    formatter: require('eslint-friendly-formatter'),
    emitWarning: !config.dev.showEslintErrorsInOverlay
  }
})


module.exports = {
  context: path.resolve(__dirname, '../'),
  entry: {
    // 入口文件是src目录下的main.js
    app: './src/main.js'
  },
  output: {
    // 输出文件的路径：config目录下的index.js，path.resolve(__dirname, '../dist')
    path: config.build.assetsRoot,
    // 输出文件的名称：name是保持和entry入口的文件名一致，由于使用了 CommonsChunkPlugin 这样的插件，通过使用占位符(substitutions)来确保每个文件具有唯一的名称
    filename: '[name].js',
    // 文件引用路径，例如index.html中引用vendor.js的src
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    // resolve是webpack的内置选项，顾名思义，决定要做的事情，如何处理 import
    extensions: ['.js', '.vue', '.json'],
    // 创建 import 或 require 的别名，来确保模块引入变得更简单。例如，一些位于 src/ 文件夹下的常用模块：
    alias: {
      // 后面的$符号指精确匹配，也就是说 import vue from 'vue' 将直接引入 vue/dist/vue.esm.js
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
    }
  },
  module: {
    rules: [
      ...(config.dev.useEslint ? [createLintingRule()] : []),
      {
        // 正则表达式，需要执行规则的文件
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test'), resolve('node_modules/webpack-dev-server/client')]
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
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
  }
}

```

