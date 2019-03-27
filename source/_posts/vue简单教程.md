---
title: '''vue简单教程'''
date: 2018-03-02 17:33:16
comments: true
tags: 
  - 前端
  - vuejs
---

> vue生命周期 

![vuejs](/assets/blogImg/vuejs.png)      


<!-- more -->

## VUE简单教程

### 1 准备工作

#### 开发工具

webstorm(当然可以使用vscode+相关插件或者其他自己比较熟悉的开发工具，这里先介绍下webstorm)

[webstorm下载地址](http://www.jetbrains.com/webstorm/download/download-thanks.html)

[注册码](http://idea.lanyus.com/)

安装注册完成后进行相关配置：

- 安装vuejs插件 (setting  -->  plugin，点击plugin，在内容部分的左侧输入框输入vue，选择插件进行安装，若已安装请忽略)

- 设置vue新建文件模板(settings  -->  file and code templates ，在内容区域左侧点击vue file，修改对应的模板内容即可，若没有相关模板则新建)

  > 我的模板配置:

  ```html
  <template>
  #[[$END$]]#
  </template>

  <script>
  export default {
  name: "${KEBAB_CASE_NAME}"
  }
  </script>

  <style scoped>

  </style>
  ```

- 设置js版本为es6(settings --> languages& frameworks -- > javascript ，选择javascript，修改内容区域的javascript language version: ECMAScript 6 即可。)

- ~~vue文件高亮设置~~

#### 环境配置

安装配置nodejs

1. [nodejs下载地址](http://nodejs.cn/download/)

2. 设置npm的默认安装路径，和缓存路径

   ```shell
   npm config set prefix "你想设置的npm包安装路径"
   npm config set cache "你想设置的npm缓存路径"
   ```

   也可以不配，不配的话默认安装在用户路径下

3. 配置代理和镜像仓库(u know what happened)

   简单一点的方法，直接修改用户路径下的.npmrc文件（没有就新建一个），添加下列代码

   ```properties
   proxy=http://server:port
   registry=https://registry.npm.taobao.org/
   https-proxy=http://server:port
   ```

   命令行方法

   ```shell
   npm config set registry http://registry.npm.taobao.org/
   npm config set proxy http://server:port
   npm config set https-proxy http://server:port
   ```

   如果代理需要认证

   ```shell
   npm config set proxy http://username:password@server:port
   npm confit set https-proxy http://username:password@server:port
   ```

   镜像仓库也可以配其他的

4. 安装vue脚手架(CLI)

   Vue 提供一个[官方命令行工具](https://github.com/vuejs/vue-cli)，可用于快速搭建大型单页应用。该工具为现代化的前端开发工作流提供了开箱即用的构建配置。只需几分钟即可创建并启动一个带热重载、保存时静态检查以及可用于生产环境的构建配置的项目:

   ```shell
   # 全局安装 vue-cli
   $ npm install --global vue-cli
   # 创建一个基于 webpack 模板的新项目
   $ vue init webpack my-project
   # 安装依赖，走你
   $ cd my-project
   $ npm install
   $ npm run dev
   ```

   从头开始搭建vue项目的时候需要了解一下

#### 储备知识

html、css、js、webpack、[babel](https://babeljs.cn/docs/setup/)、[axios](https://www.kancloud.cn/yunye/axios/234845)、scss、less等。

为了更好、更快地进行开发，比较推荐了解下[es6](http://es6.ruanyifeng.com/#README)\7\8的一些新特性以及写法。

###2 开始开发

#### 项目目录结构

| 目录/文件    | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| build        | 项目构建(webpack)相关代码                                    |
| config       | 配置目录，包括端口号等。我们初学可以使用默认的。             |
| node_modules | npm 加载的项目依赖模块                                       |
| src          | 这里是我们要开发的目录，基本上要做的事情都在这个目录里。里面包含了几个目录及文件：assets: 放置一些图片，如logo等。components: 目录里面放了一个组件文件，可以不用。App.vue: 项目入口文件，我们也可以直接将组件写这里，而不使用 components 目录。main.js: 项目的核心文件。 |
| static       | 静态资源目录，如图片、字体等。                               |
| test         | 初始测试目录，可删除                                         |
| .xxxx文件    | 这些是一些配置文件，包括语法配置，git配置等。                |
| index.html   | 首页入口文件，你可以添加一些 meta 信息或统计代码啥的。       |
| package.json | 项目配置文件。                                               |
| README.md    | 项目的说明文档，markdown 格式                                |