---
title: css position:sticky
date: 2021-03-09 11:17:31
tags:
  - css
---

## 定义

`position: sticky`可以被理解为 relative 布局和 fixed 布局的混合。一个`position: sticky`的元素在与*离它最近的滚动祖先*的某个方向上的偏移量达到设置的某个阈值之前会表现为 relative 布局，但达到这个阈值后将会表现为 fixed 布局，直到包裹`position: sticky`元素的父元素滚出*离它最近的滚动祖先*。

例如下面的例子：

```html
<div class="scroll-container">
  <div class="item">
    <div class="sticky">sticky111</div>
  </div>
  <div class="item">
    <div class="sticky">sticky222</div>
  </div>
</div>
```

```CSS
.scroll-container {
  height: 200px;
  width: 200px;
  background: #f8cecc;
  overflow: auto;
}
.item {
  height: 200px;
  background: #ffe6cc;
}
.item:last-child {
  height: 200px;
  background: #d5e8d4;
}
.sticky {
  position: sticky;
  top: 10px;
}
```

`sticky`元素的上边在距离`scroll-container`元素里 content area 的上边超过 10px 时，将表现为 relative，会随着包裹它的父元素滚动而滚动；而当达到 10px 阈值后，`sticky`元素将会保持与滚动元素 10px 的上边距，不会随着其父元素的滚动而滚动，直到父元素滚动出滚动元素。

## 最近的滚动祖先

所谓的*最近的滚动祖先*是从定义`sticky`的元素开始，向上查找，直到找到`overflow`属性不是`visible`的父元素，如果找到根元素都没有找到这种父元素，那么滚动祖先就是视口。

### 关于 viewport

1、在这里可能有人会有疑问，那视口的`overflow`属性是`visible`怎么办呢？
在 w3c 规范中有定义 [Overflow Viewport Propagation](https://www.w3.org/TR/css-overflow-3/#overflow-propagation)

> UAs must apply the overflow-_ values set on the root element to the viewport. However, when the root element is an [HTML] html element (including XML syntax for HTML) whose overflow value is visible (in both axes), and that element has a body element as a child, user agents must instead apply the overflow-_ values of the first such child element to the viewport. The element from which the value is propagated must then have a used overflow value of visible.

总结下来就是以下三点

- 当`html`元素的`overflow`属性不是`visible`的时候，viewport 的`overflow`属性取`html`元素的`overflow`属性值；
- 当`html`元素的`overflow`属性是`visible`的时候，viewport 的`overflow`属性取`body`元素的`overflow`属性值；
- 冒泡的元素的`overflow`属性值会被隐式的设置为`visible`

> The overflow values are not propagated to the viewport if no boxes are generated for the element whose overflow values would be used for the viewport (for example, if the root element has display: none).

- 当需要冒泡`overflow`属性给 viewport 的元素没有加载，那该属性值不会被冒泡。

> If visible is applied to the viewport, it must be interpreted as auto. If clip is applied to the viewport, it must be interpreted as hidden.

- 如果最终 viewport 的`overflow`属性值为`visible`，在实际表现上将会变成`auto`；如果为`clip`，那将表现为`hidden`。

更加详细的 overflow 内容在另外一篇文章中。

综上所述，_视口的`overflow`属性在表现上将不会是`visible`_，这其实也算是一个最终的兜底方案。

2、overflow: hidden 为什么也可以？
其实即使将 overflow 设置为 hidden，也可以使用 JavaScript Element.scrollTop 属性来滚动 HTML 元素。所以依然可以滚动，`position: sticky`仍然有用武之地。

### 获取滚动祖先元素

一个简单的方法，并没有考虑`overflow`冒泡的情况

```js
function getStickyParent(node) {
  const parent = node.parentElement
  if (!parent) return null
  const style = window.getComputedStyle(parent)
  const overflows = style.overflow.split(' ')
  if (overflows.some((o) => o !== 'visible')) {
    return parent
  }
  return getStickyParent(parent)
}
```

## 距离计算

我们在上面提到了滚动祖先的`content area`，需要注意的是这里的比较都是相对于`content area`来进行的。
以上的 demo 为例，当`scroll-container`元素存在`padding-top: 10px`，这时`sticky`元素在滚动时会固定在距离`scroll-container`元素上边缘 20px 处。

## 失效原因

- 没有设置 top, right, bottom 或 left 四个阈值其中之一；
- 父元素`overflow`属性值不为`visible`(默认)，且父元素在设计上并不是`sticky`元素所想要相对位置固定的元素；
- 父元素高度小于等于`sticky`元素，导致没有多余的空间来`sticky`，从而没有 sticky 效果；
