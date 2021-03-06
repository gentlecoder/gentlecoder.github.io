---
title: js object篇-Object.defineProperty()
date: 2018-03-20 09:48:27
comments: true
tags: 
  - 前端
  - JavaScript
---

`**Object.defineProperty()**` 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

## 语法

```javascript
Object.defineProperty(obj, prop, descriptor)
```

### 参数

- `obj`

  要在其上定义属性的对象。

- `prop`

  要定义或修改的属性的名称。

- `descriptor`

  将被定义或修改的属性描述符。

### 返回值

​    被传递给函数的对象。

### ES6

​    在ES6中，由于 Symbol类型的特殊性，用Symbol类型的值来做对象的key与常规的定义或修改不同，而`Object.defineProperty` 是定义key为Symbol的属性的方法之一。

<!-- more -->

## 描述

 

该方法允许精确添加或修改对象的属性。通过赋值来添加的普通属性会创建在属性枚举期间显示的属性（[`for...in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in) 或 [`Object.keys`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)[ ](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/keys)方法）， 这些值可以被改变，也可以被[删除](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/delete)。这种方法允许这些额外的细节从默认值改变。默认情况下，使用`Object.defineProperty()`添加的属性值是不可变的。

 

## 属性描述符

 

对象里目前存在的属性描述符有两种主要形式：**数据描述符**和**存取描述符**。**数据描述符**是一个具有值的属性，该值可能是可写的，也可能不是可写的。**存取描述符**是由getter-setter函数对描述的属性。描述符必须是这两种形式之一；不能同时是两者。

**数据描述符和存取描述符均具有**以下可选键值：

- `configurable`

  当且仅当该属性的 configurable 为 true 时，该属性`描述符`才能够被改变，同时该属性也能从对应的对象上被删除。**默认为 false**。

- `enumerable`

  当且仅当该属性的`enumerable`为`true`时，该属性才能够出现在对象的枚举属性中。**默认为 false**。

数据描述符同时具有以下可选键值：

- `value`

  该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。**默认为 undefined**。

- `writable`

  当且仅当该属性的`writable`为`true`时，`value`才能被[赋值运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Assignment_Operators)改变。**默认为 false**。

**存取描述符同时具有以下可选键值**：

- `get`

  一个给属性提供 getter 的方法，如果没有 getter 则为 `undefined`。该方法返回值被用作属性值。**默认为 undefined**。

- `set`

  一个给属性提供 setter 的方法，如果没有 setter 则为 `undefined`。该方法将接受唯一参数，并将该参数的新值分配给该属性。**默认为 undefined**。

- 描述符可同时具有的键值 configurableenumerablevaluewritablegetset数据描述符YesYesYesYesNoNo存取描述符YesYesNoNoYesYes如果一个描述符不具有value,writable,get 和 set 任意一个关键字，那么它将被认为是一个数据描述符。如果一个描述符同时有(value或writable)和(get或set)关键字，将会产生一个异常。

记住，这些选项不一定是自身属性，如果是继承来的也要考虑。为了确认保留这些默认值，你可能要在这之前冻结 [`Object.prototype`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)，明确指定所有的选项，或者将[`__proto__`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/__proto__)属性指向[`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null)。

```
// 使用 __proto__
var obj = {};
var descriptor = Object.create(null); // 没有继承的属性
// 默认没有 enumerable，没有 configurable，没有 writable
descriptor.value = 'static';
Object.defineProperty(obj, 'key', descriptor);

// 显式
Object.defineProperty(obj, "key", {
  enumerable: false,
  configurable: false,
  writable: false,
  value: "static"
});

// 循环使用同一对象
function withValue(value) {
  var d = withValue.d || (
    withValue.d = {
      enumerable: false,
      writable: false,
      configurable: false,
      value: null
    }
  );
  d.value = value;
  return d;
}
// ... 并且 ...
Object.defineProperty(obj, "key", withValue("static"));

// 如果 freeze 可用, 防止代码添加或删除对象原型的属性
// （value, get, set, enumerable, writable, configurable）
(Object.freeze||Object)(Object.prototype);
```

## 示例

如果你想了解如何使用`Object.defineProperty`方法和类二进制标记语法，看看[这篇文章](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/defineProperty/Additional_examples)。

### 创建属性

如果对象中不存在指定的属性，`Object.defineProperty()`就创建这个属性。当描述符中省略某些字段时，这些字段将使用它们的默认值。拥有布尔值的字段的默认值都是`false`。`value`，`get`和`set`字段的默认值为`undefined`。一个没有`get/set/value/writable`定义的属性被称为“通用的”，并被“键入”为一个数据描述符。

```
var o = {}; // 创建一个新对象

// 在对象中添加一个属性与数据描述符的示例
Object.defineProperty(o, "a", {
  value : 37,
  writable : true,
  enumerable : true,
  configurable : true
});

// 对象o拥有了属性a，值为37

// 在对象中添加一个属性与存取描述符的示例
var bValue;
Object.defineProperty(o, "b", {
  get : function(){
    return bValue;
  },
  set : function(newValue){
    bValue = newValue;
  },
  enumerable : true,
  configurable : true
});

o.b = 38;
// 对象o拥有了属性b，值为38

// o.b的值现在总是与bValue相同，除非重新定义o.b

// 数据描述符和存取描述符不能混合使用
Object.defineProperty(o, "conflict", {
  value: 0x9f91102, 
  get: function() { 
    return 0xdeadbeef; 
  } 
});
// throws a TypeError: value appears only in data descriptors, get appears only in accessor descriptors
```

### 修改属性

如果属性已经存在，`Object.defineProperty()`将尝试根据描述符中的值以及对象当前的配置来修改这个属性。如果旧描述符将其`configurable` 属性设置为`false`，则该属性被认为是“不可配置的”，并且没有属性可以被改变（除了单向改变 writable 为 false）。当属性不可配置时，不能在数据和访问器属性类型之间切换。

当试图改变不可配置属性（除了`writable` 属性之外）的值时会抛出{jsxref("TypeError")}}，除非当前值和新值相同。

#### Writable 属性

当`writable`属性设置为`false`时，该属性被称为“不可写”。它不能被重新分配。

```
var o = {}; // Creates a new object

Object.defineProperty(o, 'a', {
  value: 37,
  writable: false
});

console.log(o.a); // logs 37
o.a = 25; // No error thrown
// (it would throw in strict mode,
// even if the value had been the same)
console.log(o.a); // logs 37. The assignment didn't work.

// strict mode
(function() {
  'use strict';
  var o = {};
  Object.defineProperty(o, 'b', {
    value: 2,
    writable: false
  });
  o.b = 3; // throws TypeError: "b" is read-only
  return o.b; // returns 2 without the line above
}());
```

如示例所示，试图写入非可写属性不会改变它，也不会引发错误。

#### Enumerable 特性

 `enumerable`定义了对象的属性是否可以在 [`for...in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in) 循环和 [`Object.keys()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys) 中被枚举。

```
var o = {};
Object.defineProperty(o, "a", { value : 1, enumerable:true });
Object.defineProperty(o, "b", { value : 2, enumerable:false });
Object.defineProperty(o, "c", { value : 3 }); // enumerable defaults to false
o.d = 4; // 如果使用直接赋值的方式创建对象的属性，则这个属性的enumerable为true

for (var i in o) {    
  console.log(i);  
}
// 打印 'a' 和 'd' (in undefined order)

Object.keys(o); // ["a", "d"]

o.propertyIsEnumerable('a'); // true
o.propertyIsEnumerable('b'); // false
o.propertyIsEnumerable('c'); // false
```

#### Configurable 特性

`configurable`特性表示对象的属性是否可以被删除，以及除`writable`特性外的其他特性是否可以被修改。

```
var o = {};
Object.defineProperty(o, "a", { get : function(){return 1;}, 
                                configurable : false } );

// throws a TypeError
Object.defineProperty(o, "a", {configurable : true}); 
// throws a TypeError
Object.defineProperty(o, "a", {enumerable : true}); 
// throws a TypeError (set was undefined previously) 
Object.defineProperty(o, "a", {set : function(){}}); 
// throws a TypeError (even though the new get does exactly the same thing) 
Object.defineProperty(o, "a", {get : function(){return 1;}});
// throws a TypeError
Object.defineProperty(o, "a", {value : 12});

console.log(o.a); // logs 1
delete o.a; // Nothing happens
console.log(o.a); // logs 1
```

如果`o.a`的`configurable`属性为`true`，则不会抛出任何错误，并且该属性将在最后被删除。

### 添加多个属性和默认值

考虑特性被赋予的默认特性值非常重要，通常，使用点运算符和`Object.defineProperty()`为对象的属性赋值时，数据描述符中的属性默认值是不同的，如下例所示。

```
var o = {};

o.a = 1;
// 等同于 :
Object.defineProperty(o, "a", {
  value : 1,
  writable : true,
  configurable : true,
  enumerable : true
});


// 另一方面，
Object.defineProperty(o, "a", { value : 1 });
// 等同于 :
Object.defineProperty(o, "a", {
  value : 1,
  writable : false,
  configurable : false,
  enumerable : false
});
```

### 一般的 Setters 和 Getters

下面的例子展示了如何实现一个自存档对象。 当设置`temperature` 属性时，`archive` 数组会获取日志条目。

```
function Archiver() {
  var temperature = null;
  var archive = [];

  Object.defineProperty(this, 'temperature', {
    get: function() {
      console.log('get!');
      return temperature;
    },
    set: function(value) {
      temperature = value;
      archive.push({ val: temperature });
    }
  });

  this.getArchive = function() { return archive; };
}

var arc = new Archiver();
arc.temperature; // 'get!'
arc.temperature = 11;
arc.temperature = 13;
arc.getArchive(); // [{ val: 11 }, { val: 13 }]
```

或

```
var pattern = {
    get: function () {
        return 'I alway return this string,whatever you have assigned';
    },
    set: function () {
        this.myname = 'this is my name string';
    }
};


function TestDefineSetAndGet() {
    Object.defineProperty(this, 'myproperty', pattern);
}


var instance = new TestDefineSetAndGet();
instance.myproperty = 'test';

// 'I alway return this string,whatever you have assigned'
console.log(instance.myproperty);
// 'this is my name string'
console.log(instance.myname);
```

## 规范

| Specification                                                | Status   | Comment                                              |
| ------------------------------------------------------------ | -------- | ---------------------------------------------------- |
| [ECMAScript 5.1 (ECMA-262)Object.defineProperty](https://www.ecma-international.org/ecma-262/5.1/#sec-15.2.3.6) | Standard | Initial definition. Implemented in JavaScript 1.8.5. |
| [ECMAScript 2015 (6th Edition, ECMA-262)Object.defineProperty](https://www.ecma-international.org/ecma-262/6.0/#sec-object.defineproperty) | Standard |                                                      |
| [ECMAScript Latest Draft (ECMA-262)Object.defineProperty](https://tc39.github.io/ecma262/#sec-object.defineproperty) | Draft    |                                                      |

## 浏览器兼容

[新的兼容性表格正在测试中 **](https://developer.mozilla.org/docs/New_Compatibility_Tables_Beta)

|               | Desktop**     | Mobile**        | Server**      |                            |                  |                              |                   |                      |                 |                       |                     |                 |                    |                 |
| ------------- | ------------- | --------------- | ------------- | -------------------------- | ---------------- | ---------------------------- | ----------------- | -------------------- | --------------- | --------------------- | ------------------- | --------------- | ------------------ | --------------- |
|               | Chrome**      | Edge**          | Firefox**     | Internet Explorer**        | Opera**          | Safari**                     | Android webview** | Chrome for Android** | Edge Mobile**   | Firefox for Android** | Opera for Android** | iOS Safari**    | Samsung Internet** | Node.js**       |
| Basic support | Full support5 | Full supportYes | Full support4 | Full support9Notes**打开** | Full support11.6 | Full support5.1Notes**打开** | Full supportYes   | Full supportYes      | Full supportYes | Full support4         | Full support11.5    | Full supportYes | ?                  | Full supportYes |

### Legend

- Full support 

  Full support

- Compatibility unknown 

  Compatibility unknown

- See implementation notes.**

  See implementation notes.

## 兼容性问题

### 重定义数组对象的 length 属性

数组的 [`length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/length) 属性重定义是可能的，但是会受到一般的重定义限制。（[`length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/length) 属性初始为 non-configurable，non-enumerable 以及 writable。对于一个内容不变的数组，改变其 [`length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/length) 属性的值或者使它变为 non-writable 是可能的。但是改变其可枚举性和可配置性或者当它是 non-writable 时尝试改变它的值或是可写性，这两者都是不允许的。）然而，并不是所有的浏览器都允许 `Array.length` 的重定义。

在 Firefox 4 至 22 版本中尝试去重定义数组的 length 属性都会抛出一个 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError) 异常。

有些版本的Chrome中，`Object.defineProperty()` 在某些情况下会忽略不同于数组当前[`length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/length)属性的length值。有些情况下改变可写性并不起作用（也不抛出异常）。同时，比如[`Array.prototype.push`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/push)的一些数组操作方法也不会考虑不可读的length属性。

有些版本的Safari中，`Object.defineProperty()`  在某些情况下会忽略不同于数组当前[`length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/length)属性的length值。尝试改变可写性的操作会正常执行而不抛出错误，但事实上并未改变属性的可写性。

只在Internet Explorer 9及以后版本和Firefox 23及以后版本中，才完整地正确地支持数组[`length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/length)属性的重新定义。目前不要依赖于重定义数组[`length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/length) 属性能够起作用，或在特定情形下起作用。与此同时，即使你能够依赖于它，你也[没有合适的理由这样做](http://whereswalden.com/2013/08/05/new-in-firefox-23-the-length-property-of-an-array-can-be-made-non-writable-but-you-shouldnt-do-it/)。

### Internet Explorer 8 具体案例

Internet Explorer 8 实现了 `Object.defineProperty()` 方法，但 [只能在 DOM 对象上使用](http://msdn.microsoft.com/en-us/library/dd229916%28VS.85%29.aspx)。 需要注意的一些事情：

- 尝试在原生对象上使用 `Object.defineProperty()会`报错。
- 属性特性必须设置一些特定的值。对于数据属性描述符，`configurable`, `enumerable`和 `writable `特性必须全部设置为 `true；`对于访问器属性描述符，`configurable `必须设置为 `true`，`enumerable `必须设置为 `false`。(?) 任何试图提供其他值(?)将导致一个错误抛出。
- 重新配置一个属性首先需要删除该属性。如果属性没有删除，就如同重新配置前的尝试。