---
title: js三种模块化方式对比以及编译打包机制介绍
date: 2018-03-21 10:44:24
comments: true
tags: 
  - 前端
  - JavaScript
  - es6
---

## 一些术语: 

模块: 可以理解为一个js文件, 就像你以前需要import的那个文件一样: module不一定非要是一个外部文件, 你可以动态创建一个module。

loaded into global context, module中的变量或者函数都是封装起来的, 不会被暴露到global, 只有export出来的才会暴露;

你不能使用`<script>`来直接引用, 只能通过systemjs来加载;

每个模块只有一个实例signleton;

模块就是一个黑盒子。

Package: cards package可以包含card.js module,  deck.js module,  而index.js却是一个入口, 将多个模块打包在一起。

<!-- more -->

app也可以看做一个package

![img](https://images2015.cnblogs.com/blog/737565/201605/737565-20160519003848951-1196299578.png)

module loader:

![img](https://images2015.cnblogs.com/blog/737565/201605/737565-20160519004306060-2113926469.png)

## 什么是模块化

当我们说一个应用是modular的, 我们通常意思是: 这个应用是由一系列高度解耦的具有不同功能的分布在不同module里面的功能模块组成的。正如你可能知道的: 轻量级解耦往往具有下面的好处: 你可以轻松地删除部分依赖而不会影响到其他模块的功能。

不像其他的编程语言, javascript并不原生具备模块化的能力。正因为如此, 程序员只能使用第三方开发的模块依赖处理库来实现模块化开发模式: AMD,  COMMONJS, ES6

### script loader

在探讨AMD,  COMMONJS之前我们必须来谈谈script loader这个概念。

script loading is a means to a goal,  that goal being modular JavaScript that can be used in applications today - for this,  use of a compatible script loader is unfortunately necessary.

有几种广泛应用的Loaders来处理AMD, COMMJS格式的模块加载, 比如requireJS,  curl.js, dojo, 甚至system.js

从产品角度来看, 使用优化工具, 比如RequireJS Optimizer来拼接所有的脚本文件也是很有必要的。但是另外一个可行的应用场景是: 应用程序可以在页面page load后动态加载需要的脚本, 而RequireJS就能很好地满足到这一点。

# AMD,  COMMONJS,  ES6三种模块化方案简单对比: 

AMD: define + require

CMD: exports + require

ES6: export + import

### AMD

AMD(Asynchronous Module Defination)format的目标是提供一个可供js开发人员当下使用的模块化js方案。

AMD module的格式本身建议定义定义的模块以及该模块的依赖dependencies可以被异步地加载。

#### AMD模块定义和使用

在AMD模块哲学中, 有两个重要的概念:  define 方法用于定义一个模块, 而require 方法则用于处理dependency loading. 

```
define(
    module_id /*optional:如果该参数不存在, 我们就成为匿名模块*/,  
    [dependencies] /*optional*/,  
    definition function /*function for instantiating the module or object*/
);
```

实例: 

```
// A module_id (myModule) is used here for demonstration purposes only
 
define('myModule',  
    ['foo',  'bar'],  
    // module definition function
    // dependencies (foo and bar) are mapped to function parameters
    function ( foo,  bar ) {
        // return a value that defines the module export
        // (i.e the functionality we want to expose for consumption)
    
        // create your module here
        var myModule = {
            doStuff:function(){
                console.log('Yay! Stuff');
            }
        }
 
        return myModule;
});
 
// An alternative example could be..
define('myModule',  
    ['math',  'graph'],  
    function ( math,  graph ) {
 
        // Note that this is a slightly different pattern
        // With AMD,  it's possible to define modules in a few
        // different ways due as it's relatively flexible with
        // certain aspects of the syntax
        return {
            plot: function(x,  y){
                return graph.drawPie(math.randomGrid(x, y));
            }
        }
    };
});
```

#### 理解AMD:require()

```
// Consider 'foo' and 'bar' are two external modules
// In this example,  the 'exports' from the two modules loaded are passed as
// function arguments to the callback (foo and bar)
// so that they can similarly be accessed
 
require(['foo',  'bar'],  function ( foo,  bar ) {
        // rest of your code here
        foo.doSomething();
});
```

动态加载dependecies

```
define(function ( require ) {
    var isReady = false,  foobar;
 
    // note the inline require within our module definition
    require(['foo',  'bar'],  function (foo,  bar) {
        isReady = true;
        foobar = foo() + bar();
    });
 
    // we can still return a module
    return {
        isReady: isReady, 
        foobar: foobar
    };
});
```

#### 理解AMD Plugin

```
// With AMD,  it's possible to load in assets of almost any kind
// including text-files and HTML. This enables us to have template
// dependencies which can be used to skin components either on
// page-load or dynamically.
 
define(['./templates',  'text!./template.md', 'css!./template.css'], 
    function( templates,  template ){
        console.log(templates);
        // do some fun template stuff here.
    }
});
```

#### 通过两种方式来加载AMD模块: 

\1. require.js

```
require(['app/myModule'],  
    function( myModule ){
        // start the main module which in-turn
        // loads other modules
        var module = new myModule();
        module.doStuff();
});
```

 

2.curl.js

```
curl(['app/myModule.js'],  
    function( myModule ){
        // start the main module which in-turn
        // loads other modules
        var module = new myModule();
        module.doStuff();
});
```

### 为什么说AMD是javascript模块化的一个比较好的选择？

我们来看看他解决的问题和优势: 

\1. 定义了一个清晰的建议去如何实现灵活的模块;

\2. 相比于当前通过全局引入一个对象, 和通过`<script>`标签加载文件的方案要清晰地多。清晰地定义一个模块以及这个模块的所有依赖;

\3. 模块定义是封装好的, 这样我们就不会污染global namespace。（比如常用的jquery, 实际上我们就污染全局空间, 引入了$这个变量)

\4. 比一些替代方案可能更好用（比如commonjs), 没有cross-domain的问题, 没有local/debugging问题, 也没有对server side工具有任何依赖。大部分AMD Loaders支持并不需要任何build process的AMD modules

\5. 提供了一种在单个文件中包含多个模块的'transport'方案。而比如commonjs却要求必须统一一个transport format

\6. 可以轻松实现lazy load script

#### AMD Module Design patterns

传统的js设计模式也可以方便地以AMD方式传承和使用: 

 

#### AMD modules with jQuery

使用Jquery(注意这种AMD模块引入jquery的方式来使用没有在全局命名空间中引入$或者jquery!):

```
define(['js/jquery.js', 'js/jquery.color.js', 'js/underscore.js'], 
    function($,  colorPlugin,  _){
        // Here we've passed in jQuery,  the color plugin and Underscore
        // None of these will be accessible in the global scope,  but we
        // can easily reference them below.
 
        // Pseudo-randomize an array of colors,  selecting the first
        // item in the shuffled array
        var shuffleColor = _.first(_.shuffle(['#666', '#333', '#111']));
 
        // Animate the background-color of any elements with the class
        // 'item' on the page using the shuffled color
        $('.item').animate({'backgroundColor': shuffleColor });
        
        return {};
        // What we return can be used by other modules
    });
```

上面是jquery以模块方式来使用, 我们还得看看jquery为了可以被以AMD方式加载应用, 我们应该做什么来改造jquery: 

## CommonJS

require()和exports

这一点和AMD类似的,  exports指示本模块需要暴露给其他模块使用的对象;require则表明本模块的依赖模块

```
// package/lib is a dependency we require
var lib = require('package/lib');
 
// some behaviour for our module
function foo(){
    lib.log('hello world!');
}
 
// export (expose) foo to other modules
exports.foo = foo;
```

更复杂一点的例子: 

```
// define more behaviour we would like to expose
function foobar(){
        this.foo = function(){
                console.log('Hello foo');
        }
 
        this.bar = function(){
                console.log('Hello bar');
        }
}
 
// expose foobar to other modules
exports.foobar = foobar;
 
 
// an application consuming 'foobar'
 
// access the module relative to the path
// where both usage and module files exist
// in the same directory
 
var foobar = require('./foobar').foobar, 
    test   = new foobar();
 
test.bar(); // 'Hello bar'
```

同时使用多个依赖: 

```
var modA = require('./foo');
var modB = require('./bar');
 
exports.app = function(){
    console.log('Im an application!');
}
 
exports.foo = function(){
    return modA.helloWorld();
}
```

注意commonJS module export时的以下情况: 

```
// CMD 模式下的合法exports
exports.someFunc = someFunc;
module.exports.someFunc = someFunc;

module.exports = {};
module.exports = function(){};

//相反下面就是非法的！
exports = {};
exports = function(){};
```

 

## ES6

由于ES6本身是原生语言支持实现的模块化, 但是现代浏览器大多都还未支持, 因此必须使用相应的transpiler工具转换成ES5的AMD, CMD模块, 再借助于systemjs/requirejs等模块加载工具才能使用。

![img](https://images2015.cnblogs.com/blog/737565/201608/737565-20160823231558995-586731097.png)

上图中直到生成AMD,  CommonJS的ES5模块都属于dev/build流程, 而从ES5 CMD, AMD代码到SystemJS/RequireJS加载则属于runtime过程 

```
module 'math' {
 // named exports
    export default function sum(x,  y) { //注意default export是当import 
        return x + y;
    }
    export var pi = 3.141593;
}
//或者使用下面的literal export方法
export { sum as sumFunc,  pi }

// we can import in script code,  not just inside a module
import {sum,  pi} from 'math';

```

```
alert("2π = " + sum(pi,  pi));
```

```
// 或者这样import和调用
import *as mathmodule from 'math';
```

```
alert("2π = " + mathmodule.sum(mathmodule.pi,  mathmodule.pi));

// 下面的语法则同时import了default export和自选export: sum是上面export default定义的
// 而pi则没有export default关键字！
import sum,  { pi } from 'math' 
```

## 关于babel

babel是一个transpiler, 代码转换器, 它起到了es6和es5的桥梁的作用, 作为build step而存在于工具链的,  支持所有的模块格式:module formats. Typescript和babel是类似的, 它支持es6并且transpile到es5

## module bundler

在上面ES6模块章节提到要让build step将es6模块转化出来的es5 cmd模块能在浏览器中运行, 有一个解决方案就是: 在浏览器中加载一个module loader(requireJS/SystemJS), 由加载器来动态加载相应的js文件。和这个方案对应的有另外一个方案就是使用module bundler(browserify/webpack), 这个bundler工具作为build step的延伸。实际上如果是ES6 module, webpack将调用babel来transpile成CMD模块, 并且最后打包成一个或者多个大的chunk直接加载到浏览器运行（注意: 这时无需systemjs/requirejs加载器, 因为webpack本身已经能够认识CMD的require/exports, AMD的define!!）

![img](https://images2015.cnblogs.com/blog/737565/201608/737565-20160823234424667-2008662880.png)

## 模块化工具链

我们已经知道js语言本身并不支持模块化, 同时浏览器中js和服务端nodejs中的js运行环境是不同的, 如何实现浏览器中js模块化, 不是一个很简单的事情, 主流有两种方案: 

\1. requirejs/seajs:他们是一种在线“编译”模块的方案, 相当于在页面上加载一个CommonJS/AMD模块格式解释器。这样浏览器就认识了上面讲到的define,  exports, module这些东西, 也就实现了模块化。

2.browserify/webpack: 是一个预编译模块打包的方案, 相比于第一种方案, 这个方案更加智能。由于是预编译的, 不需要在浏览器中加载解释器。你在本地直接写JS, 不管是AMD/CMD/ES6风格的模块化, 它都能认识, 并且编译成浏览器认识的JS

注意:  browerify打包器本身只支持Commonjs模块, 如果要打包AMD模块, 则需要另外的plugin来实现AMD到CMD的转换！！https://github.com/jaredhanson/deamdify

gulp一般作为task runner, 在前端构建流程中起到非常重要的作用, 它在软件工程界和C语言的make类似。

Browserify: It provides a way to bundle CommonJS modules together,  adheres to the Unix philosophy,  is in fact a good alternative to Webpack.