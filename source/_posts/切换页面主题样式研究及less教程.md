---
title: 切换页面主题样式研究及less教程
date: 2019-04-13 10:39:33
tags:
  - JavaScript
  - html
  - css
---

> 某一天，一个头条的大佬问我，听说你之前在项目里面弄过主题切换，你当时是怎么实现的可以详说一下吗？

<!--more-->

如果已经对less了如指掌，[直接从这里开始](#切换主题样式)。

## 从less说起

### 使用

Node.js 环境中使用 Less ：

```
npm install -g less
> lessc styles.less styles.css
```

在浏览器环境中使用 Less ：

```html
<link rel="stylesheet/less" type="text/css" href="styles.less" />
<script src="//cdnjs.cloudflare.com/ajax/libs/less.js/3.8.1/less.min.js" ></script>
```

不过比较推荐的还是通过webpack或者gulp等工具先将less编译成css，然后再引入使用。

### 变量

```less
@nice-blue: #5B83AD;
@light-blue: @nice-blue + #111;

#header {
  color: @light-blue;
}
```

### 混合

* 常用

  ```less
  .bordered {
    border-top: dotted 1px black;
    &:hover {
      border: 1px solid red;
    }  // 同样可以包含选择器混用
  }
  .post a {
    color: red;
    .bordered !important;  // 可以在属性后面都加上!important
  }
  ```

* 编译后不输出至css文件

  ```less
  .my-other-mixin() {
    background: white;
  }
  ```

* 作用域混合

  ```less
  #outer {
    .inner() {
      color: red;
    }
  }
  #outer1 {
    .inner() {
      color: blue;
    }
  }
  .c {
    #outer > .inner;
  }
  ```

  输出

  ```css
  .c {
    color: red;
  }
  ```

* 拥有前置条件的作用域混合，[更多详情](<http://lesscss.cn/features/#mixin-guards-feature>)

  ```less
  @mode: little;
  #outer when (@mode=huge) {
    .inner {
      color: red;
    }
  }
  #outer when (@mode=little) {
    .inner {
      color: blue;
    }
  }
  .c {
    #outer > .inner;
  }
  ```

  输出（可以看到没有通过前置条件的样式被过滤了）

  ```css
  #outer .inner {
    color: blue;
  }
  .c {
    color: blue;
  }
  ```

* 带参数混合，可以指定默认值，可以带多个参数，使用时可以通过名称赋值

  ```less
  .mixin(@color: black; @margin: 10px; @padding: 20px) {
    color: @color;
    margin: @margin;
    padding: @padding;
  }
  .class1 {
    .mixin(@margin: 20px; @color: #33acfe);
  }
  .class2 {
    .mixin(#efca44; @padding: 40px);
  }
  ```

* 使用`@arguments`

  ```less
  .mixin(@a; @b) {
    margin: @arguments; 
    right:extract(@arguments,2); 
    top:@b;
  }
  p {.mixin(1px; 2px);}
  ```

  输出

  ```css
  p {
    margin: 1px 2px;
    right: 2px;
    top: 2px;
  }
  ```

* 使用`@rest`参数

  ```less
  // 方式1
  .mixin(@listofvariables...) {
    border: @listofvariables;
  }
  p {
    .mixin(1px; solid; red);
  }
  // 方式2
  .mixin(@a; ...) { color: @a;}
  .mixin(@a) { background-color: contrast(@a); width:100%;}
  .mixin(@a; @b;) { background-color: contrast(@a); width:@b;}
  p {
  .mixin(red);
  }
  p.small {
  .mixin(red,50%);
  }
  ```

  输出

  ```css
  /*方式1*/
  p {
    border: 1px solid red;
  }
  /*方式2*/
  p {
    color: red;
    background-color: #ffffff;
    width: 100%;
  }
  p.small {
    color: red;
    background-color: #ffffff;
    width: 50%;
  }
  ```

* 模式匹配混合

  ```less
  .mixin(dark; @color) {
    color: darken(@color, 10%);
  }
  .mixin(light; @color) {
    color: lighten(@color, 10%);
  }
  .mixin(@_; @color) {
    display: block;
  }
  @switch: light;
  .class {
    .mixin(@switch; #888);
  }
  ```

  输出

  ```css
  .class {
    color: #a2a2a2;
    display: block;
  }
  ```

* 规则集混合

  ```less
  // declare detached ruleset
  @detached-ruleset: { background: red; };
  
  // use detached ruleset
  .top {
      @detached-ruleset(); 
  }
  ```

### 函数

类似于先函数运算计算出变量的值，再通过变量的值设置属性。

```less
.mixin() {
  @width:  100%;
  @height: 200px;
}

.caller {
  .mixin();
  width:  @width;
  height: @height;
}
```

输出

```css
.caller {
  width:  100%;
  height: 200px;
}
```

### 继承

继承另外一个选择器的属性，[更多详情](<http://lesscss.cn/features/#extend-feature>)

```less
nav ul {
  &:extend(.inline);
  background: blue;
}  // &是父类型选择器，指代的是nav ul
.inline {
  color: red;
}
```

输出

```css
nav ul {
  background: blue;
}
.inline,
nav ul {
  color: red;
}
```

### 嵌套

```less
#header {
  color: black;
  .navigation {
    font-size: 12px;
  }
  .logo {
    width: 300px;
  }
}
```

### 引入

```less
@import "foo";      // foo.less is imported
@import "foo.less"; // foo.less is imported
@import "foo.php";  // foo.php imported as a less file
@import "foo.css";  // statement left in place, as-is
```

#### 参数

`@import (keyword) "filename";`

- `reference`: use a Less file but do not output it
- `inline`: include the source file in the output but do not process it
- `less`: treat the file as a Less file, no matter what the file extension
- `css`: treat the file as a CSS file, no matter what the file extension
- `once`: only include the file once (this is default behavior)
- `multiple`: include the file multiple times
- `optional`: continue compiling when file is not found

### 循环

```less
.generate-columns(4);

.generate-columns(@n, @i: 1) when (@i =< @n) {
  .column-@{i} {
    width: (@i * 100% / @n);
  }
  .generate-columns(@n, (@i + 1));
}
```

输出

```css
.column-1 {
  width: 25%;
}
.column-2 {
  width: 50%;
}
.column-3 {
  width: 75%;
}
.column-4 {
  width: 100%;
}
```

### 合并

* 逗号合并

  ```less
  .mixin() {
    box-shadow+: inset 0 0 10px #555;
  }
  .myclass {
    .mixin();
    box-shadow+: 0 0 20px black;
  }
  ```

  输出

  ```css
  .myclass {
    box-shadow: inset 0 0 10px #555, 0 0 20px black;
  }
  ```

* 空格合并

  ```less
  .mixin() {
    transform+_: scale(2);
  }
  .myclass {
    .mixin();
    transform+_: rotate(15deg);
  }
  ```

  输出

  ```css
  .myclass {
    transform: scale(2) rotate(15deg);
  }
  ```

## 切换主题样式

### 方案1

当几种主题布局类似，几乎只是颜色不同时，可以使用这种。

通过对less的了解（可以进行许多额外的骚操作哦），我们这里最简单地编写出如下less文件。

* 基础样式模版`style.less`，里面主要通过变量设置各个样式
* 爱国红样式`patriotic-red.less`，设置变量的值
* 天空蓝样式`sky-blue.less`，设置不同的变量值
* ...

`style.less`里样式类似这样

```less
a,
.link {
  color: @link-color;
}
a:hover {
  color: @link-color-hover;
}
.widget {
  color: #fff;
  background: @link-color;
}
```

`patriotic-red.less`里样式类似这样

```less
@import "style";
@link-color: #f03818; 
@link-color-hover: darken(@link-color, 10%);
```

`sky-blue.less`里样式类似这样

```less
@import "test";
@link-color: #428bca;
@link-color-hover: darken(@link-color, 10%);
```

利用相关工具（或原始人手动lessc）提前编译成`patriotic-red.css`和`sky-blue.css`文件。

这里简单的假设页面header中只引入了默认爱国红——

`<link rel="stylesheet" href="./patriotic-red.css" />`

同时存在一个按钮，用户点击时切换主题。首先我们给按钮绑定点击事件，当用户点击时删除当前导入的样式，然后再引入另外一个样式。具体操作如下：

```js
  function toggleThemeClick() {
    let a = document.getElementsByTagName("link");
    let aHref = a[0].getAttribute("href").slice(2, -4);
    a[0].parentElement.removeChild(a[0]);
    let b = document.createElement("link");
    b.setAttribute("rel", "stylesheet");
    if ("patriotic-red" === aHref) {
      b.setAttribute("href", "./sky-blue.css");
    } else {
      b.setAttribute("href", "./patriotic-red.css");
    }
    document.getElementsByTagName("head")[0].appendChild(b);
  }
```

这样我们就可以自由的切换主题啦。

### 方案2

我们还可以通过给body添加类标签来进行主题的切换。

新建一个`style-mixin.less`，里面内容如下——

```less
.patriotic-red {
  @import "patriotic-red";
}
.sky-blue {
  @import "sky-blue";
}
```

这里需要注意的是，在`sky-blue.less`或者`patriotic-red.less`中引入`style.less`时，需要使用`(multiple)`参数，否则会只编译出一份样式。编译出来的样式会自动加上`.patriotic-red`和`sky-blue`前缀。

这个时候只需要引入一份`style-mixin.css`就行了，要切换样式的时候修改body的类标签。

`document,getElementsByTagName('body')[0].className='xxx'`

## 题外话

头条大佬似乎并不是很认可方案1，他认为方案1会导致之前引入的样式和后来的冲突？我没有明白具体是什么意思，由于时间关系他也没有解释得很清楚，希望知道问题出在哪里的同学告诉我一下哦。

ps，有没有更好的切换主题的方案呢，求大佬指点～



## 参考

[http://lesscss.cn](http://lesscss.cn/)