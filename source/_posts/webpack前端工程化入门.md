---
title: webpack前端工程化入门
date: 2019-08-06 18:48:08
tags: [webpack, 前端工程化]
---

**前端工程化是什么**？

这是个很大的概念，但是在我们的日常开发中又很常见。当我们对一个工程进行设计并把它拆分成各个组件和模块时，我们是在做工程化；当我们用 Webpack 构建项目，配置好各个环境的打包配置时，我们是在做工程化；当我们为项目添加了 ESLint，并在每次提交之前自动检查代码质量时，我们是在做工程化。

如果要用一句话来概括，在我的理解中前端工程化是把前端开发工作带入到更加系统和规范体系的一系列过程。这个过程会包括源代码的预编译、模块处理、代码压缩等构建方面的工作。工程化会尽可能保证开发者的开发体验更加友好，保证源代码的质量以及依赖的完整性。工程化也会尽可能高效地将构建完成后的代码送达给客户端，来追求更加良好的用户体验。所有这些都属于工程化。

## 走向正轨的第一步——模块化

一个设计良好的系统应该是模块化的。一个最简单的原因，在一个模块化的系统中，当外界的需求亦或环境变化的时候，开发者可以更快地将问题定位到相应的模块，而不必面对纠缠在一起的逻辑不知如何下手。模块化可以使系统具备更强的可维护性。

被封装良好的模块应该具备特定且单一的功能，对外界只提供接口，而将具体实现封装在内部。Webpack 中有一个核心的理念——”一切皆模块”，即 HTML、JavaScript、CSS、图片等等都是模块，在后面的文章中会展开讲。

另外在将代码提供给客户端之前，开发者可以通过 Browserify、Webpack 这些工具将工程代码进行打包，把所有依赖模块打包为单一的 JavaScript 文件。这样一来，对于开发者而言开发体验更加友好，因为开发中每次需要关注的仅仅是单个模块，而不是堆放在一起的上千行 JavaScript 文件；而对于客户端来说则只用接受单一的打包产物，解决了文件数量过多导致 HTTP 请求耗时长的问题。解决模块之间的依赖，并根据依赖树进行打包，是工程化解决的最基本的问题之一。

### 什么是模块

在软件工程设计中，我们通常将具有特定功能的代码封装为模块。比如在一个 Web 应用里，可能会有专门负责处理网络请求的模块、专门负责日期处理的模块、专门负责渲染的模块等等，每个模块各自发挥着不同的作用，都是系统不可缺少的一部分。

具体到代码层面，模块则是一个较为笼统和宽泛的概念，实现的形式并不一样。在一些情况下，一个文件可以包含多个模块，也可以与模块一一对应。比如在工程中也许会有一个 error.js 专门用于错误处理，或者通过 util.js 来放各种工具函数，由每个文件负责一个单一的功能。

### 为什么要使用模块

模块化的设计可以为系统带来很多好处，在我看来最重要的有以下几个：

**1.作用域封装**：在 JavaScript 中代码执行的顶层作用域即是全局作用域，这意味着变量和函数定义很容易冲突。而使用模块将代码封装起来，可保证内部实现不会暴露在全局作用域中，我们只需将模块的功能通过接口的方式暴露出去给其它模块调用即可，避免了污染全局命名空间的问题。

**2.重用性**：在工程中经常会出现重复的部分，比如一个 Web 应用中各个页面共同的 header、footer，最原始的开发方式是将同样的代码复制粘贴到各个地方。这种做法的缺点是当这些共同的部分发生改变时，我们需要逐一改动每个地方的代码。

而如果把实现一类功能的代码封装为模块之后，就可以提供给各个调用者。比如说应用中有一个 Dialog 组件，可以把它的结构和样式封装起来，在不同的页面中调用它。假如 Dialog 的样式需要调整，那么只要调整该模块那么相当于此修改对所有页面都生效。

**3.解除耦合**：试想一下如果有一个几千行代码的文件在你的工程里，内部实现了各种各样的功能并且互相调用，这样的代码调试起来有多痛苦。将系统分解为模块的一个很重要意义就是解除各部分之间的耦合。当系统的某个部分需要发生改变的时候，通过模块我们可以快速定位问题。由于模块把功能的具体实现封装在了内部，只要模块间的接口不变，模块内部的变化对于外面的其它部分并没有感知。因此通过模块化可以提升系统的可维护性。

**4.按需加载**：如果没有模块，所有的代码将被放在一个大文件里面统统塞给用户。当页面不断地增加功能，不断地添加代码，最终的文件只会越来越大，而页面也打开地越来越慢，对于用户来说非常不友好。使用模块化来拆分逻辑可以使页面需要的资源最先被加载，而后续的模块在恰当的时机再进行异步加载，从而让页面加载速度更快，用户也得到更好的体验。

### CommonJS 与 Node.js 模块系统

CommonJS 是于 2009 年提出的 JavaScript 规范，它最开始是为了定义服务端标准，而非用于浏览器环境。之后 Node.js 采用并实现了它的部分规范，在模块系统上进行了一些调整。一般来说，我们不会严格区分 CommonJS 与 Node.js 的模块标准，详述两种标准的区别超出了本文的范围，在下文中我会直接使用 CommonJS 来进行表述。

Browserify 的出现带来了浏览器环境模块的变革。它是一个运行在 Node 环境下的模块打包工具，可以把模块按照 Node.js 的模块规则合并为浏览器支持的形式，这使得浏览器端的框架类库也可以按照 CommonJS 的形式编写。随着 Node.js 以及 npm 流行，近两年来对于开发者来说遵循 CommonJS 标准来编写和使用模块已经成为了一个基本通识。

### ES6 Module

ES6 Module 是目前比较推荐开发者使用的模块标准。之所以在过去我们有各种不同的模块化标准是因为 JavaScript 这门语言本身不具备模块化的特性，而现在 ES6 中已经具备了。ES6 Module 的模块语法和 CommonJS 很像，它通过 `import` 和 `export` 来进行模块的导入和导出。

```js
import math from './math';

export function sum(a, b) {
  return a + b;
}
```

在 ES6 Module 中也是每个文件作为一个模块。和 CommonJS 不同的是，ES6 Module 的模块的依赖是静态的，或者说是在编译时确定的，而不是运行时确定的。

举个例子，我们可以在 CommonJS 中的 `if` 语句中 `require` 模块，根据代码运行时 if 的判断条件决定是否要引入该模块。

```js
// 根据运行时条件确定是否引入
if(Date.now() > new Date('2019-01-01')) {
  require('./my_module');
}
```

而在 ES6 Module 中则不允许这样做，`import` 必须在代码的顶层作用域，这意味着你不能把它放在 `if` 等代码块中。ES6 Module 这样规定的原因在于可以使编译器在编译阶段就可以获取到整个依赖树，从而进行代码静态分析层面的优化，比如检测出哪些模块是从来没有被使用过的，然后从打包结果中优化掉等等。

### 动态加载模块

有些场景下我们希望能够动态地去加载一些模块，在 CommonJS 中可以直接使用 `require` 实现。

```js
if(condition) {
    require('moduleA');
} else {
    require('moduleB');
}
```

但是在 ES6 Module 中，由于上面我们提到的 `import` 是在编译时被处理而非运行时，因此无法实现动态加载的特性。

```js
// 报错
if(condition) {
    require('moduleA');
}

// 报错
var foo = 'foo';
var bar = 'bar';
import foobar from (foo + bar);
```

为了解决这个问题，tc39 提出了一个 `import()` 函数提案。它可以接受一个参数，指定所加载的模块，并且返回一个 `Promise` 对象。

```js
var foo = 'foo';
var bar = 'bar';
import(foo + bar).then(module => {
    console.log('foobar loaded:', module);
}).catch(err => {
    console.log(err);
});
```

目前 Webpack 从 2.0.0 版本开始已经支持该动态加载形式，在后面的文章中会更加详细地进行介绍。
