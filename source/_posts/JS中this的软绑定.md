---
title: JS中this的软绑定
date: 2019-04-15 00:24:47
tags:
  - JavaScript
---

> 还记得之前手写的`bind`函数吗？
>

```js
Function.prototype.myBind = function(obj) {
  obj.fn = this;
  return function(...args) {
    return obj.fn(...args);
  };
};
```

这个函数和原生`bind`函数类似，都属于***硬绑定***。这个词是在[You-Dont-Know-JS](<https://github.com/CuiFi/You-Dont-Know-JS-CN>)看到的，在JavaScript中，this的绑定是动态的，在函数被调用的时候绑定，它指向什么完全取决于函数在哪里调用，情况比较复杂，光是绑定规则就有默认绑定、隐式绑定、显式绑定、new绑定等，而硬绑定是显式绑定中的一种，通常情况下是通过调用函数的 apply() 、 call() 或者ES5里提供的 bind() 方法来实现硬绑定的。

上述三个方法好是好，可以按照自己的想法将函数的this强制绑定到指定的对象上（除了使用new绑定可以改变硬绑定外），但是硬绑定存在一个问题，就是会降低函数的灵活性，并且在硬绑定之后无法再使用隐式绑定或者显式绑定来修改this的指向。

在这种情况下，被称为软绑定的实现就出现了，也就是说，通过软绑定，我们希望this在默认情况下不再指向全局对象（非严格模式）或 undefined （严格模式），而是指向两者之外的一个对象（这点和硬绑定的效果相同），但是同时又保留了隐式绑定和显式绑定在之后可以修改this指向的能力。

下面是一个简单的软绑定实现

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
fn.myBind(obj).myBind(obj1)();		// xiaoC 硬绑定的话则还为xiaoD
```

