---
title: css-overflow
date: 2021-03-09 16:17:10
tags:
  - css
---

说到`overflow`，大家都耳熟能详，`auto`、`visible`、`hidden`、`scroll`，而这里主要聚焦以下几点：
1、clip 属性
2、`overflow`冒泡是什么意思，html 元素和 body 元素设置 overflow 具体表现如何
3、`overflow`配置导致的滚动条究竟属于哪个元素，滚动条占的空间怎么算
4、设置了`overflow`为什么不生效

[w3c 关于 overflow 的文档](https://www.w3.org/TR/css-overflow-3/)，英文好的小伙伴可以直接去看。

## clip 属性

首先看下兼容性：

![clip属性兼容性](/assets/blogImg/overflow:clip.png)

可以看到兼容性并不是很乐观，毕竟是新出的一个属性嘛。

`clip`的作用大致与`hidden`相仿，会将元素内容超出填充盒子部分裁剪掉，而与`hidden`不同的是，`clip`不可以使用编程的方法如设置`Element.scrollTop`的值来使内容滚动。与此同时呢，声明`overflow: clip`并不会自动创建一个 BFC，想要达到和声明`hidden`一样的效果，可以和`display: flow-root`配合使用。

## 冒泡

根据 w3c 文档中的描述可以提炼出以下几点：

- 当`html`元素的`overflow`属性不是`visible`的时候，viewport 的`overflow`属性取`html`元素的`overflow`属性值；
- 当`html`元素的`overflow`属性是`visible`的时候，viewport 的`overflow`属性取`body`元素的`overflow`属性值；
- 冒泡的元素的`overflow`属性值会被隐式的设置为`visible`
- 当需要冒泡`overflow`属性给 viewport 的元素没有加载，那该属性值不会被冒泡。
- 如果最终 viewport 的`overflow`属性值为`visible`，在实际表现上将会变成`auto`；如果为`clip`，那将表现为`hidden`。

从此我们可以推断出：
1、一般来说只有元素才能拥有滚动条（更准确地说，只有产生[block container box](https://www.w3.org/TR/CSS2/visuren.html#block-boxes)的元素才能拥有滚动条）。但 viewport 是个例外。它虽然不是一个元素，但是也可以拥有滚动条。如果在`<html>`和`<body>`上都没有设置 overflow 属性而使用默认值 visible（大部分场景都是这样），那么，viewport 的 overflow 就是 auto：当网页中有内容超出 viewport 时，viewport 上会出现滚动条。
2、`<html>`的最终 overflow 永远都是 visible。也就是说，`<html>`元素永远不可能拥有滚动条。
3、如果你想要为`<body>`设置非 visible 的 overflow，需要先为`<html>`设置一个非 visible 的值来冒泡，从而`<body>`的 overflow 不会被冒泡。
