---
title: 关于防抖debounce和截流throttle
date: 2019-04-01 21:36:06
tags:
  - JavaScript
---

## 基本定义

### debounce

去抖动。策略是当事件被触发时，设定一个周期延迟执行动作，若期间又被触发，则重新设定周期，直到周期结束，执行动作。 这是debounce的基本思想，在后期又扩展了前缘debounce，即执行动作在前，然后设定周期，周期内有事件被触发，不执行动作，且周期重新设定。

debounce的特点是当事件快速连续不断触发时，动作只会执行一次。 延迟debounce，是在周期结束时执行，前缘debounce，是在周期开始时执行。但当触发有间断，且间断大于我们设定的时间间隔时，动作就会有多次执行。

### throttling

节流的策略是，固定周期内，只执行一次动作，若有新事件触发，不执行。周期结束后，又有事件触发，开始新的周期。 节流策略也分前缘和延迟两种。与debounce类似，延迟是指 周期结束后执行动作，前缘是指执行动作后再开始周期。

throttling的特点在连续高频触发事件时，动作会被定期执行，响应平滑。

## 自己实现

```js
const debounce = (fn, time = 1000, immediate) => {
  let timer;
  return (...args) => {
    if (immediate) fn.call(this, ...args);
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => fn.call(this, ...args), time);
  };
};

const throttle = (fn, time = 1000) => {
  let startTime = new Date();
  return (...args) => {
    let timeNow = new Date();
    if (timeNow - startTime > time) {
      fn.call(this, ...args);
      startTime = new Date();
    }
  };
};
```

