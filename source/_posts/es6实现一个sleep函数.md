---
title: es6实现一个sleep函数
date: 2019-04-18 21:02:14
tags:
  - JavaScript
---

### 简单实现

```js
var sleep = async (duration) => {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, duration);
    });
};
(async () => {
    console.log("start!"+new Date().toISOString());
    console.time("timer1");
    await sleep(2000);//阻塞该async方法的执行线程，直到sleep()返回的Promise resolve
    console.timeEnd("timer1");
    console.log("end"+new Date().toISOString());
})();
console.log("after sleep test!");
```

### 输出

```js
start!2017-11-11T11:25:28.834Z
after sleep test!
timer1: 2000.845947265625ms
end2017-11-11T11:25:30.838Z
```

