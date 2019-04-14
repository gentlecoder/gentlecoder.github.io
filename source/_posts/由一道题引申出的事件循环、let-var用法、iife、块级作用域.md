---
title: "由一道题引申出的事件循环、let\var用法、iife、块级作用域"
date: 2019-04-10 23:00:46
tags:
  - JavaScript
---

> 没有错，这道题就是：
>
> ```js
> for (var i = 0; i< 10; i++){
> 	setTimeout(() => {
> 		console.log(i);
>     }, 1000)
> } // 10 10 10 10 ...
> ```

为什么这里会出现10次10，而不是我们预期的0-9呢？我们要如何修改达到预期效果呢？

### 运行时&&事件循环

首先我们得理解`setTimeout`中函数的执行时机，这里就要讲到一个运行时的概念。

#### 可视化描述

![js运行时](/assets/blogImg/runtime.svg)

#### 栈

函数调用形成了一个栈帧。

```js
function foo(b) {
  var a = 10;
  return a + b + 11;
}

function bar(x) {
  var y = 3;
  return foo(x * y);
}

console.log(bar(7)); // 返回 42
```

当调用 `bar` 时，创建了第一个帧 ，帧中包含了 `bar` 的参数和局部变量。当 `bar` 调用 `foo`时，第二个帧就被创建，并被压到第一个帧之上，帧中包含了 `foo` 的参数和局部变量。当 `foo`返回时，最上层的帧就被弹出栈（剩下 `bar` 函数的调用帧 ）。当 `bar` 返回的时候，栈就空了。

#### 堆

对象被分配在一个堆中，即用以表示一大块非结构化的内存区域。

#### 队列

一个 JavaScript 运行时包含了一个待处理的消息队列。每一个消息都关联着一个用以处理这个消息的函数。

在[事件循环](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop#Event_loop)(Event Loop)期间的某个时刻，运行时从最先进入队列的消息开始处理队列中的消息。为此，这个消息会被移出队列，并作为输入参数调用与之关联的函数。正如前面所提到的，调用一个函数总是会为其创造一个新的栈帧。

函数的处理会一直进行到执行栈再次为空为止；然后事件循环将会处理队列中的下一个消息（如果还有的话）。

#### 与这题的关联

这里`setTimeout`会等到当前队列执行完了之后再执行，即`for`循环结束后执行，而这个时候`i`的值已经是`10`了，所以会打印出来10个10这样的结果。

要是想得到预期效果，简单的删除`setTimeout`也是可行的。当然也可以这样改`setTimeout(console.log, 1000, i);`将`i`作为参数传入函数。

#### 引申出其他

仔细查阅规范可知，异步任务可分为 `task` 和 `microtask` 两类，不同的API注册的异步任务会依次进入自身对应的队列中，然后等待 Event Loop 将它们依次压入执行栈中执行。

(macro)task主要包含：script(整体代码)、setTimeout、setInterval、I/O、UI交互事件、postMessage、MessageChannel、setImmediate(Node.js 环境)

microtask主要包含：Promise.then、MutaionObserver、process.nextTick(Node.js 环境)

附上一幅图更清楚的了解一下

![Event Loop](/assets/blogImg/event-loop.png)

每一次Event Loop触发时：

1. 执行完主执行线程中的任务。
2. 取出micro-task中任务执行直到清空。
3. 取出macro-task中一个任务执行。
4. 取出micro-task中任务执行直到清空。
5. 重复3和4。

**其实promise的then和catch才是microtask，本身的内部代码不是。**

#### ps: 再额外附上一道题

```js
new Promise(resolve => {
    resolve(1);
    Promise.resolve().then(() => console.log(2));
    console.log(4)
}).then(t => console.log(t));
console.log(3);
```

这道题比较基础，答案为4321。先执行同步任务，打印出43，然后分析微任务，2先入任务队列先执行，再打印出1。

这里还有几种变种，结果类似。

```js
let promise1 = new Promise(resolve => {
  resolve(2);
});
new Promise(resolve => {
    resolve(1);
    Promise.resolve(2).then(v => console.log(v));
  	//Promise.resolve(Promise.resolve(2)).then(v => console.log(v));
  	//Promise.resolve(promise1).then(v => console.log(v));
  	//new Promise(resolve=>{resolve(2)}).then(v => console.log(v));
    console.log(4)
}).then(t => console.log(t));
console.log(3);
```

不过要值得注意的是一下两种情况：

```js
let thenable = {
  then: function(resolve, reject) {
    resolve(2);
  }
};
new Promise(resolve => {
  resolve(1);
  new Promise(resolve => {
    resolve(promise1);
  }).then(v => {
    console.log(v);
  });
  // Promise.resolve(thenable).then(v => {
  //   console.log(v);
  // });
  console.log(4);
}).then(t => console.log(t));
console.log(3);
```

```js
let promise1 = new Promise(resolve => {
  resolve(thenable);
});
new Promise(resolve => {
  resolve(1);
  Promise.resolve(promise1).then(v => {
    console.log(v);
  });
  // new Promise(resolve => {
  //   resolve(promise1);
  // }).then(v => {
  //   console.log(v);
  // });
  console.log(4);
}).then(t => console.log(t));
console.log(3);
```

结果为4312。有人可能会说[阮老师这篇文章](<http://es6.ruanyifeng.com/#docs/promise#Promise-resolve>)里提过

```javascript
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

那为什么这两个的结果不一样呢？

请注意这里`resolve`的前提条件是参数是一个原始值，或者是一个不具有`then`方法的对象，而其他情况是怎样的呢，[stackoverflow](<https://stackoverflow.com/questions/53894038/whats-the-difference-between-resolvethenable-and-resolvenon-thenable-object>)上这个问题分析的比较透彻，我这里简单的总结一下。

**这里的RESOLVE('xxx')是new Promise(resolve=>resolve('xxx'))简写**

- `Promise.resolve('nonThenable')` 和 `RESOLVE('nonThenable')`类似；
- `Promise.resolve(thenable)` 和 `RESOLVE(thenable)`类似；
- `Promise.resolve(promise)`要根据`promise`对象的`resolve`来区分，不为`thenable`的话情况和`Promise.resolve('nonThenable')`相似；
- `RESOLVE(thenable)` 和 `RESOLVE(promise)` 可以理解为 ` new Promise((resolve, reject) => {   Promise.resolve().then(() => {     thenable.then(resolve)   }) }) `

也就是说可以理解为`Promise.resolve(thenable)`会在这一次的`Event Loop`中立即执行`thenable`对象的`then`方法，然后将外部的`then`调入下一次循环中执行。

再形象一点理解，可以理解为`RESOLVE(thenable).then`和`PROMISE.then.then`的语法类似。

**再来一道略微复杂一点的题加深印象**

```js
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
	console.log('async2');
}

console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0)

async1();

new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
});
console.log('script end');
```

[题目来源](<https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/7>)

**答案**

```js
/*
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
*/
```

### let、var和const用法和区别

总结一下[阮老师](<http://es6.ruanyifeng.com/#docs/let>)的介绍。

* ES6 新增了`let`命令，用来声明变量。它的用法类似于`var`，但是所声明的变量，只在`let`命令所在的代码块内有效。而`var`全局有效。
* `var`命令会发生“变量提升”现象，即变量可以在声明之前使用，值为`undefined`。`let`命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。
* 在代码块内，使用`let`命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。
* `let`不允许在相同作用域内，重复声明同一个变量。
* `const`声明的变量不得改变值，这意味着，`const`一旦声明变量，就必须立即初始化，不能留到以后赋值。`const`其他用法和`let`相同。

#### 与这题关联

上面代码中，变量`i`是`var`命令声明的，在全局范围内都有效，所以全局只有一个变量`i`。每一次循环，变量`i`的值都会发生改变，而循环内被赋给数组`a`的函数内部的`console.log(i)`，里面的`i`指向的就是全局的`i`。也就是说，所有数组`a`的成员里面的`i`，指向的都是同一个`i`，导致运行时输出的是最后一轮的`i`的值，也就是 10。

要是想得到预期效果，可以简单的把`var`换成`let`。

### iife和块级作用域

`let`实际上为 JavaScript 新增了块级作用域。

```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

上面的函数有两个代码块，都声明了变量`n`，运行后输出 5。这表示外层代码块不受内层代码块的影响。如果两次都使用`var`定义变量`n`，最后输出的值才是 10。

块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

```javascript
// IIFE 写法
(function () {
  var tmp = ...;
  ...
}());

// 块级作用域写法
{
  let tmp = ...;
  ...
}
```

#### 与这题的关联

如果不用`let`，我们可以使用iife将`setTimeout`包裹，从而达到预期效果。

```js
for (var i = 0; i < 10; i++) {
  (i =>
    setTimeout(() => {
      console.log(i);
    }, 1000))(i);
}
```

### 其他骚气方法

```js
for (var i = 0; i < 10; i++) {
  try {
    throw i;
  } catch (i) {
    setTimeout(() => {
      console.log(i);
    }, 1000);
  }
}
```



### 参考

[阮老师es6](<http://es6.ruanyifeng.com/>)

[What's the difference between resolve(thenable) and resolve('non-thenable-object')?](https://stackoverflow.com/questions/53894038/whats-the-difference-between-resolvethenable-and-resolvenon-thenable-object)

[Daily-Interview-Question](<https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/7>)

[并发模型与事件循环](<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop>)

