---
title: '''bootstrap-vue使用'''
date: 2018-03-07 18:13:10
comments: true
tags: 
  - 前端
  - vue
  - bootstrap
---

因项目需要要用到bootstrap的相关样式功能，相关使用教程[bootstrap-vue官方文档](https://bootstrap-vue.js.org/docs/)有详细说明就不再赘述了。

> 这里要说的是官方文档的一个坑，在目前的文档中（git issue上相关人员说在以后的文档版本里会修改），只提到了需要import bootstrap-vue，然后Vue.use(bootstrap-vue)，然而当你仅仅做了这些操作后会发现并没有自动引入bootstrap的相关样式。完整的引入如下：

```js
import BootstrapVue from 'bootstrap-vue'
import 'bootstrap-vue/dist/bootstrap-vue.css'
import 'bootstrap/dist/css/bootstrap.css'
Vue.use(BootstrapVue);
```

只需要单个组件的应用方法在[这里](https://bootstrap-vue.js.org/docs/)。