---
title: css-虚线边框滚动效果
date: 2019-05-05 23:02:09
tags:
  - css
---

> 常常看到一种酷炫的效果，鼠标hover一片区域后，区域显示出虚线边框，并且还有线条动画，那么这种效果具体是怎么实现的呢，本文提供了几种思路仅供参考。

**基本HTML**

```html
<div class="box">
  <p>测试测试</p>
</div>
```

## Easy-way

通过背景图片实现。

p得垂直居中哦，还记得如何垂直居中吗？详见另一篇博客～

```less
.box {
  width: 100px;
  height: 100px;
  position: relative;
  background: url(https://www.zhangxinxu.com/study/image/selection.gif);
  p {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    margin: auto;
    height: calc(100% - 2px);
    width: calc(100% - 2px);
    background-color: #fff;
  }
}
```

## repeating-linear-gradient

135度repeating线性渐变，p撑开高度，白色背景覆盖外层div渐变。

```less
.box {
  width: 100px;
  height: 100px;
  background: repeating-linear-gradient(
    135deg,
    transparent,
    transparent 4px,
    #000 4px,
    #000 8px
  );
  overflow: hidden;				// 新建一个BFC，解决margin在垂直方向上折叠的问题
  animation: move 1s infinite linear;
  p {
    height: calc(100% - 2px);
    margin: 1px;
    background-color: #fff;
  }
}
@keyframes move {
  from {
    background-position: -1px;
  }
  to {
    background-position: -12px;
  }
}
```

## linear-gradient&&background

通过线性渐变以及background-size画出虚线，然后再通过background-position将其移动到四边。这种方式比较好的地方在于可以分别设置四条边的样式以及动画的方向，细心的同学应该会发现上一种方式的动画并不是顺时针或者逆时针方向的。

```less
.box {
  width: 100px;
  height: 100px;
  background: linear-gradient(0deg, transparent 6px, #e60a0a 6px) repeat-y,
    linear-gradient(0deg, transparent 50%, #0f0ae8 0) repeat-y,
    linear-gradient(90deg, transparent 50%, #09f32f 0) repeat-x,
    linear-gradient(90deg, transparent 50%, #fad648 0) repeat-x;
  background-size: 1px 12px, 1px 12px, 12px 1px, 12px 1px;
  background-position: 0 0, 100% 0, 0 0, 0 100%;
  animation: move2 1s infinite linear;
  p {
    margin: 1px;
  }
}
@keyframes move2 {
  from {
  }
  to {
    background-position: 0 -12px, 100% 12px, 12px 0, -12px 100%;
  }
}
```

## linear-gradient&&mask

mask属性规范已经进入候选推荐规范之列，会说以后进入既定规范标准已经是板上钉钉的事情，大家可以放心学习，将来必有用处。

这里同样可以使用mask来实现相同的动画，并且可以实现虚线边框渐变色这种效果，与background不同的是mask需要在中间加上一块不透明的遮罩，不然p元素的内容会被遮盖住。

```less
.box {
  width: 100px;
  height: 100px;
  background: linear-gradient(0deg, #f0e, #fe0);
  -webkit-mask: linear-gradient(0deg, transparent 6px, #e60a0a 6px) repeat-y,
    linear-gradient(0deg, transparent 50%, #0f0ae8 0) repeat-y,
    linear-gradient(90deg, transparent 50%, #09f32f 0) repeat-x,
    linear-gradient(90deg, transparent 50%, #fad648 0) repeat-x,
    linear-gradient(0deg, #fff, #fff) no-repeat;		// 这里不透明颜色随便写哦
  -webkit-mask-size: 1px 12px, 1px 12px, 12px 1px, 12px 1px, 98px 98px;
  -webkit-mask-position: 0 0, 100% 0, 0 0, 0 100%, 1px 1px;
  overflow: hidden;
  animation: move3 1s infinite linear;
  p {
    height: calc(100% - 2px);
    margin: 1px;
    background-color: #fff;
  }
}
@keyframes move3 {
  from {
  }
  to {
    -webkit-mask-position: 0 -12px, 100% 12px, 12px 0, -12px 100%, 1px 1px;
  }
}
```



[具体demo点这里](https://codepen.io/gentlecoder/pen/RmwZYp)

## PS

今天有一个人来我们这边面试，五年工作经验，上来头头是道说自己参与过多少项目，独自一人架构一个什么前端项目，熟练css、js、html、vue等等等，听得我瑟瑟发抖。

然后问了个简单的如何垂直居中，答曰：用flex；再问怎么用flex，答：不太了解；问还有什么方式，答曰：transform；再问为何transform，答：不太了解。。。

等等等，问了许多基础的东西，大多不了解，更不用说vue的相关原理，diff算法之流。

最后问期望薪资，答：已有18k的offer，期望不要低于18k。

是我不了解行情了，还是现在压根就不是寒冬了，拿着微薄薪水的我求个内推～～南京或杭州地区。

[个人博客](http://gentlecoder.cn/)求友链～

[电子简历](http://gentlecoder.cn/my-resume/)～

## 参考

[张哥虚线边框](https://www.zhangxinxu.com/wordpress/2018/08/css-gradient-dashed-border/)

[mask教程](https://www.zhangxinxu.com/wordpress/2017/11/css-css3-mask-masks/#mask-repeat)