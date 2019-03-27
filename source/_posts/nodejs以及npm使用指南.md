---
title: '''nodejs以及npm使用指南'''
date: 2018-03-30 15:41:31
comments: true
tags: 
  - 前端
  - npm
  - nodejs
---

### nodejs

[windows下载地址](https://nodejs.org/en/download/current/)(最新版本)

Linux参考网上其他教程

[nodejs教程](http://nodejs.cn/api/)

***

*windows下由于某些不可抗力的因素，使用命令行进行nodejs的升级比较麻烦，因此推荐直接去下载最新的安装包，直接覆盖安装即可*

<!--more-->

### npm常用命令

```javascript
npm install(i)/uninstall(remove rm r un) package_name[@xxx][-g] [--save/--save-dev]
```

安装、卸载相关的包

`-g`(可选)是否全局安装

`--save`(可选)把模块版本信息保存到dependencies（生产环境依赖）中，即你的package.json文件的dependencies字段中

`--save-dev`(可选)把模块版本信息保存到devDependencies（开发环境依赖）中，即你的package.json文件的devDependencies字段中

***

```
npm init
```

在当前目录生成一个package.json文件，这个文件中会记录一些关于项目的信息，比如：项目的作者，git地址，入口文件、命令设置、项目名称和版本号等等，一般情况下这个文件是必须要有的，方便后续的项目添加和其他开发人员的使用。

***

```
npm list/ll/ls/la -g [depth 0]
```

查看所有已安装的模块

`--depth 0`不列出嵌套依赖的模块

***

```
npm outdated
```

列出所有已经过时了的模块

***

```
npm update [-g] package_name
```

更新已经安装过的模块，一般直接`npm i`也能达到相同的效果

***

```
npm help '命令'
```

查看某条命令的详细帮助

***

```
npm config
```

设置npm命令的配置路径，这个命令一般用于设置代理、镜像

除去以上的这些命令外，经常还能见到一些`npm start`、`npm deploy`、 `npm build`等等之类的命令，这些一般都是在package.json 中自定义的一些启动、重启、停止服务之类的命令。可以在package.json文件的scripts字段里自定义。例如：

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "start": "webpack-dev-server main.js,
    "deploy": "set NODE_ENV=production"
  }
```

关于package.json的详细文档，有兴趣的同学可以参考[《package.json中文文档》](https://github.com/ericdum/mujiang.info/issues/6/)