---
title: Intersection Observer详解
date: 2021-01-02 19:22:50
tags:
  - JavaScript
---

## 基本定义

> Intersection Observer API提供了一种异步检测目标元素与祖先元素或 [viewport](https://developer.mozilla.org/zh-CN/docs/Glossary/Viewport) 相交情况变化的方法。
>
> `IntersectionObserver`对象的 observe()方法向`IntersectionObserver`对象监听的目标集合添加一个元素。一个监听者有一组阈值和一个根，但是可以监视多个目标元素，以查看这些目标元素可见区域的变化。调用 IntersectionObserver.unobserve()方法可以停止观察元素。

以上是mdn对`IntersectionObserver`的定义，通俗一点的话来讲就是，该API可以通过设置一组阈值和根元素，来对目标元素相对于根元素的位置进行监听，从而达到响应变化的目的。

<!--more-->

## 简介

在以前要检测一个元素是否可见或者两个元素是否相交并不容易，很多解决办法不可靠或性能很差。然而随着互联网的发展，这种需求却与日俱增，比如说下面这些情况都需要用到相交检测：

- 图片懒加载——当图片滚动到可见时才进行加载
- 内容无限滚动——也就是用户滚动到接近内容底部时直接加载更多
- 检测广告的曝光情况——为了计算广告收益，需要知道广告元素的曝光情况
- 在用户看见某个区域时执行任务或播放动画

通常要实现这样的效果都要用到事件监听，并且需要调用[`Element.getBoundingClientRect()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect) 方法或者通过`clientHeight`、`offsetTop`、`scrollTop`等频繁计算得到元素的边界信息。而事件监听和调用这些API 都是在主线程上运行，因此频繁触发、调用可能会造成性能问题。这种检测方法极其怪异且不优雅。

在某次研发过程中，我们有注意到在一些页面上进行滚动的时候明显出现了一些卡顿的现象，起初觉得是由于非原生开发引起的性能问题，后来进一步定位发现是几个组件争相监听`scroll`事件，不停触发调用，造成了性能问题，用户体验很差。

而`IntersectionObserver`API的出现再一定程度上很好的解决了这些问题。



## 基本语法

```javascript
const options = {
  root: document.body,
  rootMargin: '0px',
  threshold: 0
}

function callback (entries, observer) {
  console.log(observer);
  entries.forEach(entry => {
    console.log(entry);
  });
}

let observer = new IntersectionObserver(callback, options);
observer.observe(targetElement);
```

### options相关属性

**`root`**

用来检查是否和目标元素发生交集的根元素。该选项接受任何合法元素，但是根元素必须是目标元素的祖先，这一点很重要。如果不指定根元素，或设为 `null`，则浏览器视口就作为默认的根元素。

**`rootMargin`**

该属性被用来扩展或缩减根元素的尺寸。接受和 CSS 中的 margin 相同格式的值，比如一个单独的值 `10px` 或定义不同边的多个值 `10px 11% -10px 25px`。默认值为`0px 0px 0px 0px`。

**`threshold`**

最后，threshold（阈值）选项指定了一个最小量，表示目标元素和根元素交集时，其自身满足该最小量才会触发回调。取值为 0.0 – 1.0 之间的一个浮点数，所以 75% 左右的交集率应该写成 0.75。如果希望在多个点触发回调，也可以传入一个值的数组，如  `[0.33, 0.66, 1.0]`。

看到这里可能会有些疑问，这个交集又是怎么计算的呢？在`IntersectionObserver API`中所有的图形都会被看成矩形来计算，那些不规则的图形会被转化为能包裹它的的最小矩形参与交集计算。

![calcIntersection](/assets/blogImg/calcIntersection.gif)

### 回调函数

当目标元素和根元素的相交区域达到阈值时，将会触发回调函数，注意这里和监听事件不同的是，回调函数不会一直触发，只会在达到阈值的时候触发。

```typescript
callback IntersectionObserverCallback = void (
    sequence<IntersectionObserverEntry> entries, 
    IntersectionObserver observer
);
```

回调的参数，entries 是一个`IntersectionObserverEntry`数组，数组的长度是我们监控目标元素的个数，另一个参数是 `IntersectionObserver` 对象。

**`IntersectionObserverEntry`**

`IntersectionObserverEntry`属性如下：

```typescript
interface IntersectionObserverEntry {
  readonly attribute DOMHighResTimeStamp time;
  readonly attribute DOMRectReadOnly? rootBounds;
  readonly attribute DOMRectReadOnly boundingClientRect;
  readonly attribute DOMRectReadOnly intersectionRect;
  readonly attribute boolean isIntersecting;
  readonly attribute double intersectionRatio;
  readonly attribute Element target;
};
```

**time**：从首次创建观察者到触发此交集改变的时间（以毫秒为单位）

**rootBounds**：根元素矩形区域的信息，如果没有设置根元素则返回null。

**boundingClientRect**：目标元素的矩形区域的信息。

**intersectionRect**：目标元素与视口（或根元素）的交叉区域的信息。

**isIntersecting**：目标元素与根元素是否相交

**intersectionRatio**：目标元素与视口（或根元素）的相交比例。

**target**：目标元素

## 应用场景

除了一些常见的页面滚动时元素动画或导航变化之外额外出两个应用的例子如下

1、确定相交时目标元素相对于根元素的位置

[demo1](https://codepen.io/gentlecoder/pen/MWjVVoE)

2、为粘性布局创建相应的效果

[demo2](https://codepen.io/gentlecoder/pen/yLaKKPz)

## 兼容性

![IntersectionObserverCompatibility](/assets/blogImg/IntersectionObserverCompatibility.png)