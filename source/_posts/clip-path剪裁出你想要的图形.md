---
title: clip-path剪裁出你想要的图形
date: 2019-04-07 21:31:53
tags:
  - css
---

> 虽然[CSS Shapes Module Level 1（CSS形状模块标准1）](http://www.w3.org/TR/css-shapes/)的规范出现，可以[打破矩形设计的限制](http://www.w3cplus.com/css3/css-shapes-breaking-rectangular-design.html)。但仍需要一些不规则的图形。而早前实现一些不规则的图形，都需借助其它的元素功能，比如[CSS绘制图形](http://www.w3cplus.com/blog/tags/112.html)，很多时候就依赖于伪元素，或多个元素。如此一来，[CSS Shapes](http://www.w3cplus.com/blog/tags/418.html)依旧无法发挥其强大的功能，让我们的Web打破常规的矩形布局。不过值得庆幸的是，[CSS的`clip-path`出现](https://www.w3.org/TR/css-masking/#the-clip-path)，它可以帮助我们绘制很多特殊的图形（不规则的图形），地址是  <http://bennettfeely.com/clippy/> 。

<!--more-->

## 基本概念

`clip-path`从单词"clip path"的直译上来说，表示的就是裁剪路径。既然有裁剪，咱们就来了解这里面的几个简单的概念。

**裁剪**就是从某样东西剪切一块。比如说，我们在`<img>`元素上，根据需要，剪切一部分需要留下的区域。而在整个裁剪中，将会碰到两个相关的概念：**裁剪路径(Clipping Path)**和**裁剪区域(Clipping Region)**。

**裁剪路径**是我们用来裁剪元素的路径，它标记了我们需要裁剪的区域。它可以是个简单的形状（比如Web中常见的矩形），也可以是一个复杂的多边形（不规则的多边形）。

**裁剪区域**是裁剪路径闭合后所包含的全部区域。

这样一来，元素分为两部分，裁剪区域和裁剪区域外。浏览器会裁剪掉裁剪区域以外的区域，不仅是背景及其它类似的内容，也包括`border`、`text-shadow` 等。更赞的是，浏览器不会捕获元素裁剪区域以外的 `hover`、`click` 等事件。

 即使如今一些特定元素不受长方形限制，但实际上元素周围的内容还是会认为元素是原始形状（长方形）的，并按此进行文档流的布局。要想使周围元素根据元素裁剪后的形状进行布局，可以使用 `shape-outside`属性。有关于`shape-outside`相关详细的介绍，可以[阅读有关于CSS Shapes相关的教程](http://www.w3cplus.com/blog/tags/418.html)，这里不进行过多阐述。

## clip-path语法

[W3C官方规范](https://www.w3.org/TR/css-masking/#the-clip-path)提供的`clip-path`语法：

```
clip-path: <clip-source> | [ <basic-shape> || <geometry-box> ] | none
其默认值是none。另外简单介绍clip-path几个属性值：
```

- **clip-source**: 可以是内、外部的SVG的`<clipPath>`元素的URL引用
- **basic-shape**: 使用一些基本的形状函数创建的一个形状。主要包括`circle()`、`ellipse()`、`inset()`和`polygon()`。具体的说明可以看[CSS Shapes](https://www.w3.org/TR/css-shapes-1/#typedef-basic-shape)中有关于说明。另外在[CSS Shapes 101](http://www.w3cplus.com/css3/css-shapes-101.html)一文中也有详细介绍。
- **geometry-box**: 是可选参数。此参数和`basic-shape`函数一起使用时，可以为`basic-shape`的裁剪工作提供[参考盒子](http://www.w3cplus.com/css3/css-shapes-reference-boxes.html)。如果`geometry-box`由自身指定，那么它会使用指定盒子形状作为裁剪的路径，包括任何(由`border-radius`提供的)的角的形状。

## 开始使用clip-path

在开始使用`clip-path`绘制图形，或者说裁剪图形之前，有两点需要大家注意：

- 使用`clip-path`要从同一个方向绘制，如果顺时针绘制就一律顺时针，逆时针就一律逆时针，因为`polygon`是一个连续线段，若线段彼此有交集，裁剪区域就会有相减的情况发生，当然如果你特意需要这样的效果除外。
- 如果绘制时采用比例的方式绘制，长宽就必须要先行设定，不然有可能绘制出来的长宽和我们想像的就会有差距，使用像素绘制就不会有这样的现象。

先来看一个使用`polygon()`函数绘制的示例：

```
img {
  clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);
}
```

这段代码会将所有的图片裁剪为菱形。但是为什么图片会被裁剪为菱形而不是梯形或平行四边形之类的呢？这主要**取决于函数顶点的值**。下图将说明一切：

![img](https://images2015.cnblogs.com/blog/841880/201612/841880-20161216211503417-348088929.png)

每个点的第一个坐标值决定了它在 `x` 轴上的位置，第二个坐标值指定了它在 `y` 轴的位置，所有点是顺时针绘制的。比如菱形最右边的点，它位于 `y` 轴下方一半处，所以它的 `y` 坐标是 `50%`。同时这个点位于 `x` 轴的最右侧，所以它的 `x` 坐标是 `100%`。其它点的坐标同理可得。

最后效果如下所示：

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

效果图如下：

![img](https://images2015.cnblogs.com/blog/841880/201612/841880-20161216215353808-2135613293.png)

 

 

记得以前[CSS绘制图形](http://www.w3cplus.com/blog/tags/112.html)总得束手束脚，而且还得想法设法，使用`clip-path`绘制什么六边形、八边形、五角形、心形等，都不再是难事了。[OXXO.STUDIO](http://www.oxxostudio.tw/)有一篇文章《[運用 clip-path 的純 CSS 形狀變換](http://www.oxxostudio.tw/articles/201503/css-clip-path.html)》详细介绍了这些图形是如何绘制的。当然除此之外，在线的[CSS clip-path maker](http://bennettfeely.com/clippy)提供了很多不规则的图形案例：

## 利用 geometry-box 裁剪元素

在具体使用`geometry-box`来裁剪元素之前，对`geometry-box`做一下相关的了解。

`geometry-box`可以是`shape-box`、`fill`、`stroke`或者`view-box`。其中`shape-box`应用于HTML元素，它具有四种值：`margin-box`、`border-box`、`padding-box`和`content-box`。

![shape-box](http://cdn1.w3cplus.com/cdn/farfuture/u5gxQl6FQF_qsdSZLviva2YfG6art6ePaWGAjgR5iWA/mtime:1407158693/sites/default/files/blogs/2014/1408/reference-boxes-1.jpg)

来看个简单的示例：

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

在上例中，元素的 `margin-box` 会作为参考，来决定裁剪点的实际位置。点（`10%，10%`）是 `margin-box` 的左上角，所以`clip-path` 的定位会根据此点进行计算。

其实`shape-box`和CSS Shapes中的**引用框**概念非常类似，有关于这方面的介绍，可以花点时间阅读《[理解CSS Shapes的引用框](http://www.w3cplus.com/css3/css-shapes-reference-boxes.html)》一文。

如果`geometry-box`和`basic-shape`一起使用，可以引用`basic-shape`提供的引用框。其作用和`shape-outside`属性类似，更多的细节可以看看`shape-outside`的[属性介绍](http://tympanus.net/codrops/css_reference/shape-outside)。

如果`geometry-box`由自身指定，那么它会使用指定盒子形状作为裁剪的路径，包括任何(由`border-radius`提供的)的角的形状。

除了`shape-box`值，还可以运用SVG元素上，它具有另外三个值：`fill`、`stroke`和`view-box`。

## clipPath 和clip-path

在SVG中有一个`clipPath`元素。`<clipPath>`元素不会直接在页面上呈现，他唯一的作用就是可以通过`clip-path`来引用。它和CSS的`clip-path`还是有很大的区别。有关于两者的详细介绍可以阅读《[CSS和SVG中的剪切:`clip-path`属性和``元素](http://www.w3cplus.com/css3/css-svg-clipping.html)》一文。

而很多时候两者可以结合一起使用。

你不需要在CSS中定义`clip-path`的值，因为它能够引用SVG中定义的 `<clipPath>`标签元素。下面是它的使用示例：

HTML

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

CSS

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

效果如下图

![img](https://images2015.cnblogs.com/blog/841880/201612/841880-20161216213918229-924201212.png)

## clip-path和masking

剪裁和遮罩都是用来隐藏元素的一些部分、显示其他部分的。当然了，这两者还是有区别的。区别主要在于这几方面：他们能做的东西，不同的语法，涉及到的不同技术，是新的还是旧的，以及浏览器支持的差异。

两者最主要的区别：**遮罩使用的是图像，剪裁使用的是路径**。

想象一张从左到右、从黑到白渐变的正方形图像，它可以是一个遮罩。对于应用了这个渐变遮罩图像的元素，它在遮罩图像的黑色部分是透明（透视）的，而在遮罩图像的白色的部分是不透明（正常）的。所以作出的结论是：这个元素是从左到右淡入的。

而剪裁一直都是矢量路径的。路径之外的部分是透明的，路径里边的部分是不透明的。

个人觉得有点混乱。因为很多时候可能会碰到某个关于遮罩的教程用的是一个在黑色上有白色矢量形状的遮罩图像，这和剪裁基本是同一个原理。但这还好，它只是混淆了一点东西。

有关于两者相关的详细介绍可以[点击这里阅读](http://www.w3cplus.com/css3/clipping-masking-css.html)。

## clip-path和CSS Shapes

前面已经多次提到CSS Shapes了，是的，因为CSS Shapes可以帮助我们[打破常规则的Web排版](http://www.w3cplus.com/css3/css-shapes-breaking-rectangular-design.html)，让Web页面可以像媒体杂志一样布局，这将是激动人心的一件事情。

而在CSS Shapes中同样会有`clip-path`的身影。

`clip-path`接收与`basic-shape`相同的形状函数和值(前面提到过)。如果我们定义相同的多边形形状，同时用于`shape-outside`与`clip-path`属性上,它将裁掉图像上你定义的形状之外的图像。

 

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

结果如下：

![img](https://images2015.cnblogs.com/blog/841880/201612/841880-20161216214145589-1599748629.png)

下面有个示例

HTML

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

CSS

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

效果图如下

 ![img](https://images2015.cnblogs.com/blog/841880/201612/841880-20161216215730276-245274448.png)

## clip-path示例和工具

前面内容简单的提到过了，`clip-path`是一个强大的属性，除了自身能实现一些特殊效果之外，还可以和SVG结合在一起。另外还可以和Masking以及CSS Shapes在一起，做出我们意想不到的效果。那么有关于`clip-path`相关的案例，网上已经有大把了。除此之外，`clip-path`还有一些在线的工具，可以直接帮助我们做一些事情。比如[Chrome插件CSS Shapes 编辑器](http://razvancaliman.com/writing/css-shapes-editor-chrome/)、[Clip Path生成器](http://cssplant.com/clip-path-generator)和[CSS clip-path Maker: Clippy](http://bennettfeely.com/clippy/)。

最后强列建议大家收藏好下面[这篇文章](http://bashooka.com/coding/18-css-clip-path-tutorials-examples-tools/)，因为这篇文章整理了[18个有关于`clip-path`的教程、案例和工具](http://bashooka.com/coding/18-css-clip-path-tutorials-examples-tools/)：

 ![img](https://images2015.cnblogs.com/blog/841880/201612/841880-20161216215453698-906423044.png)

## 参考资料

- [W3C官方规范](https://www.w3.org/TR/css-masking/#the-clip-path)
- [clip-path(WBP)](https://docs.webplatform.org/wiki/css/properties/clip-path)
- [clip-path(CSS Reference)](http://tympanus.net/codrops/css_reference/clip-path/)
- [Introducing the CSS clip-path Property](https://www.sitepoint.com/introducing-css-clip-path-property/)