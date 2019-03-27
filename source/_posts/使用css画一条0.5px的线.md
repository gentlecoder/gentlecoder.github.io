---
title: '使用css画一条0.5px的线'
date: 2018-09-28 18:26:28
comments: true
tags: 
  - 前端
  - css
---

> 理论上px的最小单位是1，但是会有几个特例，高清屏的显示就是一个特例。高清屏确实可以画0.5px，在布局方面 ， 0.5px的线看上去就比1px的线看上去要精致很多。

<!-- more -->

#### 什么是像素？

像素是屏幕显示最小的单位，在一个1080p的屏幕上，它的像素数量是1920 *1080，即横边有1920个像素，而竖边为1080个。一个像素就是一个单位色块，是由rgba四个通道混合而成。对于一个1200万像素的相机镜头来说，它有1200万个感光单元，它能输出的最大图片分辨率大约为3000* 4000。

那么像素本身有大小吗，一个像素有多大？

有的，如果一个像素越小，那么在同样大小的屏幕上，需要的像素点就越多，像素就越密集，如果一英寸有435个像素，那么它的dpi/ppi就达到了435。Macbook Pro 15寸的分辨率为2880 x 1800，15寸是指屏幕的对角线为15寸（具体为15.4），根据长宽比换算一下得到横边为13寸，所以ppi为2880 / 13 = 220 ppi. 像素越密集即ppi(pixel per inch)越高，那么屏幕看起来就越细腻越高清。

#### 如何画一条0.5PX的直线？

1、直接设置0.5px（不同浏览器效果不同，不推荐），代码如下所示：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .hr {
            width: 300px;
            background-color: #000;
        }

        .hr.half-px {
            height: 0.5px;
        }

        .hr.one-px {
            height: 1px;
        }
    </style>
</head>

<body>
    <p>0.5px</p>
    <div class="hr half-px"></div>
    <p>1px</p>
    <div class="hr one-px"></div>
</body>

</html>
```

不同的浏览器有不同的表现，其中Chrome把0.5px四舍五入变成了1px，而firefox/safari能够画出半个像素的边，并且Chrome会把小于0.5px的当成0，而Firefox会把不小于0.55px当成1px，Safari是把不小于0.75px当成1px，进一步在手机上观察iOS的Chrome会画出0.5px的边，而安卓(5.0)原生浏览器是不行的。所以直接设置0.5px不同浏览器的差异比较大，并且我们看到不同系统的不同浏览器对小数点的px有不同的处理。所以如果我们把单位设置成小数的px包括宽高等，其实不太可靠，因为不同浏览器表现不一样。

2、使用缩放

```
.hr.half-px {
   height: 1px;
   transform: scaleY(0.5);
}
```

在PC上的不同浏览器上测试，我们发现Chrome/Safari都变虚了，只有Firefox比较完美看起来是实的而且还很细，效果和直接设置0.5px一样。所以通过transform: scale会导致Chrome变虚了，而粗细几乎没有变化，所以这个效果不好。

3、使用渐变

```
.hr.gradient {
  height: 1px;
  background: linear-gradient(0deg, #fff, #000);
}
```

linear-gradient(0deg, #fff, #000)的意思是：渐变的角度从下往上，从白色#fff渐变到黑色#000，而且是线性的，在高清屏上，1px的逻辑像素代表的物理（设备）像素有2px，由于是线性渐变，所以第1个px只能是#fff，而剩下的那个像素只能是#000，这样就达到了画一半的目的。实际效果和使用缩放类似

4、使用box-shadow

```
.hr.boxshadow {
  height: 1px;
  background: none;
  box-shadow: 0 0.5px 0 #000;
}
```

设置box-shadow的第二个参数为0.5px，表示阴影垂直方向的偏移为0.5px，这个方法在Chrome和Firefox都非常完美，但是Safari不支持小于1px的boxshadow，所以完全没显示出来了。

5、使用svg

```
<style>
.hr.svg {
  background: none;
  height: 1px;
  background: url("data:image/svg+xml;utf-8,<svg xmlns='http://www.w3.org/2000/svg' width='100%' height='1px'><line x1='0' y1='0' x2='100%' y2='0' stroke='black'></line></svg>");
}
</style>
<p>svg</p>
<div class="hr svg"></div>
```

使用svg的line元素画线，stroke表示描边颜色，默认的宽度stroke-width=”1”，由于svg的1px是物理像素的px，相当于高清屏的0.5px，另外还可以使用svg的rect等元素进行绘制。要注意firefox的background-image如果是svg的话只支持命名的颜色，如”black”、”red”等。