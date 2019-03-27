---
title: vue-cli生成的build及config文件夹中文件解读(4)build.js
date: 2018-04-02 11:33:26
comments: true
tags: 
  - 前端
  - vue
  - vue-cli
---

> 本文`vue-cli`版本是2.9.3，低于2.0版本的build文件夹中还存在**dev-client.js**和**dev-server.js**两个文件，现版本这两个文件中的配置已经移步到`build/webpack.dev.conf.js`中进行了。

<!--more-->

本文中的文件是直接运行`vue init webpack my-project`生成的，下面是build/build.js中相关代码和配置的说明:

```js
'use strict'
// npm和node版本检查
require('./check-versions')()

process.env.NODE_ENV = 'production'

const ora = require('ora')  // ora是一个命令行转圈圈动画插件，好看用的
const rm = require('rimraf')  // rimraf插件是用来执行UNIX命令rm和-rf的用来删除文件夹和文件，清空旧的文件
const path = require('path')  // chalk插件，用来在命令行中输出不同颜色的文字
const chalk = require('chalk')
const webpack = require('webpack')
const config = require('../config')
const webpackConfig = require('./webpack.prod.conf')

const spinner = ora('building for production...')
spinner.start()  // 开启转圈圈动画

// 调用rm方法，第一个参数和第二个参数分别是 ../dist 和 static，表示删除路径下面的所有文件
rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err  // 如果删除的过程中出现错误，就抛出这个错误，同时程序终止
  // 调用webPack执行构建
  webpack(webpackConfig, (err, stats) => { // stats对象中保存着编译过程中的各种消息
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,  // 增加控制台颜色开关
      modules: false,  // 不增加内置模块信息
      children: false, // If you are using ts-loader, setting this to true will make TypeScript errors show up during build.  // 不增加子级信息
      chunks: false,  // 允许较少的输出
      chunkModules: false  // 不将内置模块的信息加到包信息
    }) + '\n\n')

    if (stats.hasErrors()) {
      console.log(chalk.red('  Build failed with errors.\n'))
      process.exit(1)
    }

    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})

```

