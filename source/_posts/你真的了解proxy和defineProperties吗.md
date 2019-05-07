---
title: 你真的了解proxy和defineProperties吗
date: 2019-04-10 21:52:31
tags:
  - JavaScript
---

> Vue 的响应式原理中 Object.defineProperty 有什么缺陷？为什么在 Vue3.0 采用了 Proxy，抛弃了 Object.defineProperty？少年，你真的了解他们吗？

## 一些思考

```js
const c = (...v) => console.log(...v);
let a = [1, 2, 3];
Object.entries(a).forEach(([key, value]) => {
  Object.defineProperty(a, key, {
    configurable: false,
    enumerable: false,
    get() {
      c("get:", value);
      return value;
    },
    set(newValue) {
      c("set:", newValue);
      value = newValue;
    }
  });
});
```

* 此时a[2]的值是多少？
* a.pop()后a是什么样的？
* a.push(1)等操作可以打印出对应的get和set吗？
* 哪些操作可以触发打印get和set?
* 如果不设置get，但enumerable为true，a是什么样的？

**答案**

* a[2]为3，虽然enumerable为false，但是可以通过get方法获取到值
* 无法删除，并打印`get: 3`，a保持原样
* 3和4很有意思，大家可以自己尝试下，在没有其他操作的情况下，shift操作会触发get1,2,3，以及set2,3，并且报错，然后a变成[2,3,3]。
* 可以通过in来操作a了。

其实这里主要记住三点，一、defineProperty只会对定义时的那些属性生效，后来新增的属性是不会生效的，在这里就是只对0，1，2这三个属性生效；二、改变数组的行为只要操作了这三个属性，那就会产生相应的get和set；三、enumerable和configurable的含义要搞懂。

### 以push为例，如何监听push操作

```js
let a = [1, 2, 3];
const defineReactive = (obj, key, value) => {
  Object.defineProperty(obj, key, {
    configurable: true,
    enumerable: true,
    get() {
      c("get:", value);
      return value;
    },
    set(newValue) {
      c("set:", newValue);
      value = newValue;
    }
  });
};
Object.entries(a).forEach(([key, value]) => {
  defineReactive(a, key, value);
});
Object.defineProperty(a, "push", {
  value: v => {
    c(v);
    a[a.length] = v;
    defineReactive(a, a.length - 1, v);
  },
  enumerable: true
});

a.push(4);
```

这是一个简单的实现，同时push后的属性也会被监听到。一些复杂情况需要递归绑定监听。

## Object.defineProperty(obj, prop, descriptor)

### 参数

- `obj`

  要在其上定义属性的对象。

- `prop`

  要定义或修改的属性的名称。

- `descriptor`

  将被定义或修改的属性描述符。

### 返回值

​    被传递给函数的对象。

### 属性描述符

对象里目前存在的属性描述符有两种主要形式：**数据描述符**和**存取描述符**。**数据描述符**是一个具有值的属性，该值可能是可写的，也可能不是可写的。**存取描述符**是由getter-setter函数对描述的属性。描述符必须是这两种形式之一；不能同时是两者。

**数据描述符和存取描述符均具有**以下可选键值：

- `configurable`

  当且仅当该属性的 `configurable`为` true` 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。

  **默认为 false**。

- `enumerable`

  当且仅当该属性的`enumerable`为`true`时，该属性才能够出现在对象的枚举属性中。

  在这里可能有人会有疑问，什么叫做可枚举属性呢？简单来说，如果该值为`false`，并且`get`方法未定义，那么将无法获取到该属性对应的值。在下面的一些思考中可以看到对应的实例。

  **默认为 false**。

**数据描述符同时具有以下可选键值**：

- `value`

  该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。

  **默认为 undefined**。

- `writable`

  当且仅当该属性的`writable`为`true`时，`value`才能被[赋值运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Assignment_Operators)改变。

  **默认为 false**。

**存取描述符同时具有以下可选键值**：

- `get`

  一个给属性提供 getter 的方法，如果没有 getter 则为 `undefined`。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入`this`对象（由于继承关系，这里的`this`并不一定是定义该属性的对象）。

  **默认为 undefined**。

- `set`

  一个给属性提供 setter 的方法，如果没有 setter 则为 `undefined`。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。

  **默认为 undefined**。

**如果一个描述符不具有value,writable,get 和 set 任意一个关键字，那么它将被认为是一个数据描述符。**

需要注意的是**如果一个描述符同时有(value或writable)和(get或set)关键字，将会产生一个异常。**

## proxy







## 参考

[MDN](<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty>)