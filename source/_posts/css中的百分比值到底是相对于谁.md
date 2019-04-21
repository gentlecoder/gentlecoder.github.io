---
title: css中的百分比值到底是相对于谁
date: 2019-04-20 23:20:49
tags:
  - css
---

> 在写CSS样式的过程中常常有这样的疑问，我现在写的百分比值究竟换算成px是多少，它们又是怎么计算的呢？闲来无聊，而且总忘，感觉还是有必要总结一下。

## 初始化

### 基本HTML

```html
<div class="grandfather">
  <div class="father">
    <div class="child">
    </div>
  </div>
</div>
```

### 基本css

```css
.grandfather {
  width: 200px;
  height: 200px;
}
.father {
  width: 100px;
  height: 100px;
  padding: 20px;
  background-color: grey;
}
.child {
  background-color: red;
}
```

## 宽高百分比

### 简单情况

**父子均在文档流中**

```css
.father {
  width: 100px;
  height: 100px;
  padding: 20px;
  background-color: grey;
}
.child {
  width: 50%;							// 50px
  height: 20%;						// 20px
  background-color: red;
  /* position: relative; */
}
```

**总结**

父子均在文档流中，宽高百分比相对于父元素的content宽高。

### 脱离文档流情况

**子元素设置绝对定位**

```css
.grandfather {
  width: 200px;
  height: 200px;
  position: relative;
}
.father {
  width: 100px;
  height: 100px;
  padding: 20px;
  background-color: grey;
}
.child {
  width: 50%;							// 100px
  height: 20%;						// 40px
  background-color: red;
  position: absolute;
}
```

子元素绝对定位时，参照对象为外围第一个定位非`static`、`initial`、`unset`的父元素。

**子元素同上，该父元素存在padding的情况**

```css
.father {
  width: 100px;
  height: 100px;
  padding: 20px;
  background-color: grey;
  position: relative;
}
.child {
  width: 50%;							// 70px
  height: 20%;						// 28px
  background-color: red;
  position: absolute;
}
```

**总结**

子元素绝对定位时，参照对象为外围第一个定位非`static`、`initial`、`unset`父元素的content+padding的宽高值。

## margin、padding百分比

众所周知，他们都是相对于父元素的宽度的，当子元素脱离文档流时同样适用于上述规则。

## 总结

* 子元素在文档流中，子元素宽高百分比相对于父元素的content宽高，

  子元素margin、padding百分比相对于父元素的content宽度；

* 子元素脱离文档流时，参照对象为外围第一个定位非`static`、`initial`、`unset`父元素的content+padding的宽高值，

  子元素margin、padding百分比相对于外围第一个定位非`static`、`initial`、`unset`父元素的content+padding的宽度。