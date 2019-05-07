---
title: segmentfault答题汇总
date: 2019-04-16 00:06:28
tags:
  - SegmentFault
---

## 19-01-17

### 问题

```js
function Foo(){
    getName = function(){
        console.log(1);
    };
    return this;
}
Foo.getName = function() {
    console.log(2);
}
Foo.prototype.getName = function(){
    console.log(3);
}

var getName = function(){
    console.log(4);
};

function getName() {
    console.log(5);
}

Foo.getName(); // 
getName(); // 
Foo().getName(); // 
getName(); // 
new Foo.getName(); // 
new Foo().getName(); //  
new new Foo().getName(); //
```

### 回答

2 4 1 1 2 3 3

1、第5问和第6问结果不一样呢
优先级不同
`new Foo.getName();`就类似于`new (Foo.getName)()`
`new Foo().getName();`类似于`(new Foo()).getName()`
2、最后一问这样看就比较好理解了`new ((new Foo()).getName)()`

另外，[运算符优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)

