---
title: 初探WebAssembly
date: 2020-05-25 11:36:59
tags:
  - JavaScript
  - WebAssembly
---

> 自从第一个网页开发出来以来，web 应用已经发生了翻天覆地的变化。应对日益复杂的页面需求，js——这种 24 年前开发出来的语言无法在解决性能问题的同时满足繁重的系统功能。每次加载网页时，我们都会加载 HTML，CSS 和 JS 数据，然后对其进行解析，创建 AST，对其进行优化，编译或解释。而这是对资源的巨大浪费。

<!--more-->

## 简介

WebAssembly 是一种新的编码方式，可以在现代的网络浏览器中运行 － 它是一种低级的类汇编语言，具有紧凑的二进制格式，可以接近原生的性能运行，并为诸如 C / C ++等语言提供一个编译目标，以便它们可以在 Web 上运行。它也被设计为可以与 JavaScript 共存，允许两者一起工作。通过使用 WebAssembly 的 JavaScript API，你可以把 WebAssembly 模块加载到一个 JavaScript 应用中并且在两者之间共享功能。这允许你在同一个应用中利用 WebAssembly 的性能和威力以及 JavaScript 的表达力和灵活性，即使你可能并不知道如何编写 WebAssembly 代码。

简而言之，WebAssembly 是对 JavaScript 的一种高性能扩展。

## 兼容性

这是通过[W3C WebAssembly Community Group](https://www.w3.org/community/webassembly/)开发的一项网络标准，并得到了来自各大主要浏览器厂商的积极参与。除了 IE 之外，大部分浏览器兼容性很好。

[兼容性详情](https://developer.mozilla.org/zh-CN/docs/WebAssembly#Browser_compatibility)

## WebAssembly 的目标

作为 [W3C WebAssembly Community Group](https://www.w3.org/community/webassembly/)中的一项开放标准，WebAssembly 是为下列目标而生的：

- 快速、高效、可移植——通过利用[常见的硬件能力](http://webassembly.org/docs/portability/#assumptions-for-efficient-execution)，WebAssembly 代码在不同平台上能够以接近本地速度运行。
- 可读、可调试——WebAssembly 是一门低阶语言，但是它有确实有一种人类可读的文本格式，通常被保存为.wat/.wast 扩展名（其标准即将得到最终版本），这允许通过手工来写代码，看代码以及调试代码。最终在转换为.wasm 汇编格式文件。
- 保持安全——WebAssembly 被限制运行在一个安全的沙箱执行环境中。像其他网络代码一样，它遵循浏览器的同源策略和授权策略。
- 兼容现有平台——与其他网络技术很好集成并保持向后兼容，与 js 在相同语义环境中执行，允许 js 同步调用。
- 包含对 C/C++以外语言的支持，如 Java、kotlin 等 40 余总语言。Java 可以使用[JWebAssembly](https://github.com/i-net-software/JWebAssembly)将 Java 字节码转换为 wasm。
- 支持非浏览器嵌入，如数据服务器、IOT 设备、移动终端等（参考[非网络嵌入](http://webassembly.org/docs/non-web/)）。wasm 可以运行在一个特定的虚拟机中，也可以通过特定的编译器编译成相关平台的字节码来运行。

## 应用场景

3D 游戏、虚拟现实、增强现实、计算机视觉、图像/视频编辑以及大量的要求原生性能的其他领域的时候，我们遇到了性能问题（参考 [WebAssembly 使用案例](http://webassembly.org/docs/use-cases/) 获取更多细节）。

### WebAssembly 关键概念

为了理解 WebAssembly 如何在浏览器中运行，需要了解几个关键概念。所有这些概念都是一一映射到了[WebAssembly 的 JavaScript API](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly)中。

- **模块**：表示一个已经被浏览器编译为可执行机器码的 WebAssembly 二进制代码。一个模块是无状态的，并且像一个[二进制大对象](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)（Blob）一样能够[被缓存到 IndexedDB](https://developer.mozilla.org/zh-CN/docs/WebAssembly/Caching_modules)中或者在 windows 和 workers 之间进行共享（通过[postMessage()](https://developer.mozilla.org/zh-CN/docs/Web/API/MessagePort/postMessage)函数）。一个模块能够像一个 ES2015 的模块一样声明导入和导出。
- **内存**：ArrayBuffer，大小可变。本质上是连续的字节数组，WebAssembly 的低级内存存取指令可以对它进行读写操作。
- **表格**：带类型数组，大小可变。表格中的项存储了**不能作为原始字节存储在内存里的对象**的引用（为了安全和可移植性的原因）。
- **实例**：一个模块及其在运行时使用的所有状态，包括内存、表格和一系列导入值。一个实例就像一个已经被加载到一个拥有一组特定导入的特定的全局变量的 ES2015 模块。

JavaScript API 为开发者提供了创建模块、内存、表格和实例的能力。给定一个 WebAssembly 实例，JavaScript 代码能够调用普通 JavaScript 函数暴露出来的导出代码。通过把 JavaScript 函数导入到 WebAssembly 实例中，任意的 JavaScript 函数都能被 WebAssembly 代码同步调用。

因为 JavaScript 能够完全控制 WebAssembly 代码如何下载、编译运行，所以，JavaScript 开发者甚至可以把 WebAssembly 想象成一个高效地生成高性能函数的 JavaScript 特性。

将来，WebAssembly 模块将会[像 ES2015 模块那样加载](http://webassembly.org/docs/modules/#integration-with-es6-modules)（使用`<script type='module'>`)，这也就意味着 JavaScript 代码能够像轻松地使用一个 ES2015 模块那样来获取、编译和导入一个 WebAssembly 模块。

## 如何在实际项目中应用

### 从 C/C++移植

将 C/C++文件转换为 wasm 有两种方式可供选择：

1、许多在线 WASM 转换页面，例如：

- [WasmFiddle](https://wasdk.github.io/WasmFiddle/)
- [WasmFiddle++](https://anonyco.github.io/WasmFiddle/)
- [WasmExplorer](https://mbebenita.github.io/WasmExplorer/)

2、[Emscripten](https://developer.mozilla.org/zh-CN/docs/Mozilla/Projects/Emscripten)

Emscripten 工具能够将一段 C/C++代码，编译出

- 一个.wasm 模块
- 用来加载和运行该模块的 JavaScript”胶水“代码
- 一个用来展示代码运行结果的 HTML 文档

![img](https://mdn.mozillademos.org/files/14647/emscripten-diagram.png)

简而言之，工作流程如下所示：

1. Emscripten 首先把 C/C++提供给 clang+LLVM——一个成熟的开源 C/C++编译器工具链，比如，在 OSX 上是 XCode 的一部分。
2. Emscripten 将 clang+LLVM 编译的结果转换为一个.wasm 二进制文件。
3. 就自身而言，WebAssembly 当前不能直接的存取 DOM；它只能调用 JavaScript，并且只能传入整形和浮点型的原始数据类型作为参数。这就是说，为了使用任何 Web API，WebAssembly 需要调用到 JavaScript，然后由 JavaScript 调用 Web API。因此，Emscripten 创建了 HTML 和 JavaScript 胶水代码以便完成这些功能。

> 下载安装最新 emscripten
>
> ```shell
> # Get the emsdk repo
> git clone https://github.com/emscripten-core/emsdk.git
>
> # Enter that directory
> cd emsdk
>
> # Fetch the latest version of the emsdk (not needed the first time you clone)
> git pull
>
> # Download and install the latest SDK tools.
> ./emsdk install latest
>
> # Make the "latest" SDK "active" for the current user. (writes ~/.emscripten file)
> ./emsdk activate latest
>
> # Activate PATH and other environment variables in the current terminal
> source ./emsdk_env.sh
>
> ```

注意：计划将来[允许 WebAssembly 直接调用 Web API](https://github.com/WebAssembly/gc/blob/master/README.md)。

### 直接编写 WebAssembly 代码

让我们看一个简单的例子——下面的程序从一个叫做 imports 的模块中导入了一个叫做 imported_func 的函数并且导出了一个叫做 exported_func 的函数：

```
(module
  (func $i (import "imports" "imported_func") (param i32))
  (func (export "exported_func")
    i32.const 42
    call $i
  )
)
```

WebAssembly 函数 exported_func 是被导出供我们的环境（比如，使用了 WebAssembly 模块的网络应用）使用。当被调用的时，它进而调用了一个被导入的叫做 imported_func 的函数并且向该函数传递了一个值（42）作为参数。

编写完成后通过这些工具（[WebAssemby text-to-binary tools](http://webassembly.org/getting-started/advanced-tools/)、[WasmExplorer](http://mbebenita.github.io/WasmExplorer/)）将 wat 文件转换为二进制格式。

[理解 WebAssembly 文本格式](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format)——详细解释文本格式语法。

[语义](http://webassembly.org.cn/docs/semantics/)

### LLVM

to be done

## 实际操作

### 生成 HTML 和 JavaScript

我们先来看一个最简单的例子，通过这个，你可以使用 Emscripten 来将任何代码生成到 WebAssembly，然后在浏览器上运行。

1. 首先我们需要编译一段样例代码。将下方的 C 代码复制一份然后命名为 hello.c 保存在一个新的文件夹内。

   ```cpp
   #include <stdio.h>

   int main(int argc, char ** argv) {
     printf("Hello World\n");
   }
   ```

2. 现在，转到一个已经配置过 Emscripten 编译环境的终端窗口中，进入刚刚保存 hello.c 文件的文件夹中，然后运行下列命令：

   ```bash
   emcc hello.c -s WASM=1 -o hello.html
   ```

下面列出了我们命令中选项的细节：

- `-s WASM=1` — 指定我们想要的 wasm 输出形式。如果我们不指定这个选项，Emscripten 默认将只会生成[asm.js](http://asmjs.org/)。
- `-o hello.html` — 指定这个选项将会生成 HTML 页面来运行我们的代码，并且会生成 wasm 模块，以及编译和实例化 wasm 模块所需要的“胶水”js 代码，这样我们就可以直接在 web 环境中使用了。

这个时候在您的源码文件夹应该有下列文件:

- `hello.wasm` 二进制的 wasm 模块代码
- `hello.js` 一个包含了用来在原生 C 函数和 JavaScript/wasm 之间转换的胶水代码的 JavaScript 文件
- `hello.html` 一个用来加载，编译，实例化你的 wasm 代码并且将它输出在浏览器显示上的一个 HTML 文件

### 在 JavaScript 中加载.wasm 模块

```javascript
function fetchAndInstantiate(url, importObject) {
  return fetch(url)
    .then((response) => response.arrayBuffer())
    .then((bytes) => WebAssembly.instantiate(bytes, importObject))
    .then((results) => results.instance)
}
```

目前正在进行加载的改进，目标是像 js 模块一样加载.wasm 模块。

需要注意的是.wasm 模块中的函数返回值只支持数字类型（整数或者浮点数），如果需要其他类型，需使用 WebAssembly 的[内存模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/WebAssembly/Memory)。举个例子，如果需要在两者之间传递字符串，我们需要先将字符串转换为相应的字符串代码，然后写入内存数组，最后将数组索引传递。

### 字节码

我们将下面的 c 函数编译为.wasm 文件

```c
int add42(int num) {
  return num + 42;
}
```

我们得到如下的字节码

```
00 61 73 6D 0D 00 00 00 01 86 80 80 80 00 01 60
01 7F 01 7F 03 82 80 80 80 00 01 00 04 84 80 80
80 00 01 70 00 00 05 83 80 80 80 00 01 00 01 06
81 80 80 80 00 00 07 96 80 80 80 00 02 06 6D 65
6D 6F 72 79 02 00 09 5F 5A 35 61 64 64 34 32 69
00 00 0A 8D 80 80 80 00 01 87 80 80 80 00 00 20
00 41 2A 6A 0B
```

具体分析一下

```
20 00 41 2A 6A
```

转换为二进制

```
10000000000000010000010010101001101010
```

人类可读格式

```
get_local 0			// 得到第一个参数的值并推入栈中
i32.const 42		// 将一个常数42推入栈中
i32.add					// 将栈顶部两个数字相加
```

这里我们可以看到 add 并没有指定参数来源，这是因为 WebAssembly 被称为一种堆栈机器。这意味着在执行操作之前，操作需要的所有值都在堆栈上排队。需要注意的是在实际的物理机器上却不是这样工作的。当浏览器将 WebAssembly 转换为浏览器所运行的机器的机器码时，它将使用寄存器。而由于 WebAssembly 代码没有指定寄存器，所以它提供了更大的灵活性来分配最佳的寄存器。

## PS

由于`application/wasm`不是一个正式的 MIME 类型[WebAssembly/spec#573](https://github.com/WebAssembly/spec/issues/573)，所以我们需要在服务器中指明 wasm 类型。

```
application/wasm    wasm
```

> 点击[官网](https://webassembly.org/)了解更多。
