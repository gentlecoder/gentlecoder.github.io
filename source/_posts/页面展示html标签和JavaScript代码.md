---
title: 页面展示html标签和JavaScript代码
date: 2018-03-06 19:27:35
comments: true
tags: 
  - 前端
  - html
---

在前端日常编码的过程中经常会遇到这样的问题：比如说某个博客留言里面，用户输入了一段html或者JavaScript代码，而我们又要考虑到xss的问题，那么我们如何将用户输入的内容原样展示出来呢？`<textarea>`是个选择，有人说`<pre>`以及`<code>`也可以，但是在实际操作过程中，我们会发现如`<svg/onclick=alert(1)>`这样的代码仍会被浏览器解释成html代码，很让人头疼。

这里给大家介绍一种简单的方法，把用户输入的内容用下面的函数进行HTML编码后再显示到页面上就行了：

```javascript
// HTML转义函数
function encodeHtml(s){
    return s.replace(
        /"|&|'|<|>|[\x00-\x20]|[\x7F-\xFF]|[\u0100-\u2700]/g,
        function($0){
            var c = $0.charCodeAt(0);
            switch(c){
            case 13: return "<br />";
            case 32: return "&#160;";
            default: return "&#"+c+";";
            }
        }
    );
};
```