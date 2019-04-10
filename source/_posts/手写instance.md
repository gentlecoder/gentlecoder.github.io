---
title: 手写instance
date: 2019-04-07 22:52:58
tags:
  - JavaScript
---

### 迭代

```js
const myInstance = (left, right) => {
  while (left) {
    if (left.__proto__ === right.prototype) return true;
    left = left.__proto__;
  }
  return false;
};
```

### 递归

```js
const myInstance = (left, right) => {
  if (!left) return false;
  if (left.__proto__ === right.prototype) return true;
  return myInstance(left.__proto__, right);
};
```

