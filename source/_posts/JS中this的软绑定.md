---
title: JS中this的软绑定
date: 2019-04-15 00:24:47
tags:
  - JavaScript
---

> 还记得之前手写的`bind`函数吗？
>
> ```js
> Function.prototype.myBind = function(obj) {
>   obj.fn = this;
>   return function(...args) {
>     return obj.fn(...args);
>   };
> };
> ```

这个函数和原生`bind`函数类似，都属于***硬绑定***。这个词是在





```js
Function.prototype.myBind = function(obj) {
  let fn1 = this;
  obj.fn = this;
  return function(...args) {
    if (!this || this === (typeof window == "undefined" ? global : window)) {
      return obj.fn(...args);
    }
    this.fn = fn1;
    return this.fn(...args);
  };
};
const obj = {
  name: "xiaoD"
};
const obj1 = {
  name: "xiaoC"
};
const fn = function(...args) {
  c(this.name, ...args);
};
fn.myBind(obj).myBind(obj1)();
```

