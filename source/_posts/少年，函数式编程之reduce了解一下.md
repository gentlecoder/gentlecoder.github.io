---
title: 少年，函数式编程之reduce了解一下
date: 2019-04-04 23:42:45
tags:
  - JavaScript
  - 函数式编程
---

> 今天主要想说两种实现数组reduce的方法

### 迭代式

```js
const myReduce = (arr, fn, initial) => {
  let i = 0;
  let acc = initial !== undefined ? initial : arr[i++];
  for (; i < arr.length; i++) {
    acc = fn(acc, arr[i], i, arr);
  }
  return acc;
};
```

### 递归式

```js
const myReduce = (arr, fn, initial) => {
  if (arr.length === 0) return initial;
  let i = 0;
  let acc = initial !== undefined ? initial : arr[i++];
  acc = fn(acc, arr[i]);
  arr = arr.slice(++i);
  return myReduce(arr, fn, acc);
};
```

### 验证一下

```js
const fn = (acc, cur) => acc + cur;
const a = [1, 2, 3];
c(a.reduce(fn, 0), myReduce(a, fn, 0));  // 6 6
```

