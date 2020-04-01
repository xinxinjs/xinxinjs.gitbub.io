---
title: 不容错过的 Babel7 知识
date: 2020-03-31 15:41:34
tags: [babel, babel7]
---

博客原文地址：[不容错过的 Babel7 知识](https://juejin.im/post/5ddff3abe51d4502d56bd143)

简单了解Babel

## 是什么

Babel 是一个 JS 编译器。

Babel 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。

## 做什么

- 语法转换

- 通过 Polyfill 方式在目标环境中添加缺失的特性(@babel/polyfill模块)

- 源码转换(codemods)

## 如何使用

### 核心库 @babel/core

不安装 @babel/core，无法使用 babel 进行编译。

npm install --save-dev @babel/core

### CLI命令行工具 @babel/cli

babel 提供的命令行工具，主要是提供 babel 这个命令，适合安装在项目里。

@babel/node 提供了 babel-node 命令，但是 @babel/node 更适合全局安装，不适合安装在项目里。

npm install --save-dev @babel/cli

将命令配置在 package.json 文件的 scripts 字段中:

```json
"scripts": {
    "compiler": "babel src --out-dir lib --watch"
}
```

### 插件

配置在.babelrc文件中

```json
//.babelrc
{
    "plugins": ["@babel/plugin-transform-arrow-functions"], //插件发布在 npm 上，可以直接填写插件的名称，
    "plugins": ["./node_modules/@babel/plugin-transform-arrow-functions"], //也可以指定插件的相对/绝对路径
}
```

### 预设

通过使用或创建一个 preset 即可轻松使用一组插件。

@babel/preset-env 主要作用是对我们所使用的并且目标浏览器中缺失的功能进行代码转换和加载 polyfill，在不进行任何配置的情况下，@babel/preset-env 所包含的插件将支持所有最新的JS特性(ES2015,ES2016等，不包含 stage 阶段)，将其转换成ES5代码。例如，如果你的代码中使用了可选链(目前，仍在 stage 阶段)，那么只配置 @babel/preset-env，转换时会抛出错误，需要另外安装相应的插件。

```json
//.babelrc
{
    "presets": ["@babel/preset-env"]
}
```

### polyfill

垫片就是垫平不同浏览器或者不同环境下的差异，让新的内置函数、实例方法等在低版本浏览器中也可以使用。

@babel/polyfill 模块包括 core-js 和一个自定义的 regenerator runtime 模块，可以模拟完整的 ES2015+ 环境（不包含第4阶段前的提议）。

安装 @babel/polyfill 依赖: npm install --save @babel/polyfill

注意：不使用 --save-dev，因为这是一个需要在源码之前运行的垫片。

我们需要将完整的 polyfill 在代码之前加载，

@babel/polyfill 需要在其它代码之前引入，我们也可以在 webpack 中进行配置。

例如:

```json
entry: [
    require.resolve('./polyfills'),
    path.resolve('./index')
]
```

@babel/preset-env 提供了一个 useBuiltIns 参数，设置值为 usage 时，就只会包含代码需要的 polyfill 。有一点需要注意：配置此参数的值为 usage ，必须要同时设置 corejs (如果不设置，会给出警告，默认使用的是"corejs": 2) ，注意: 这里仍然需要安装 @babel/polyfill(当前 @babel/polyfill 版本默认会安装 "corejs": 2):

npm install --save core-js@3

```json
//.babelrcconst 
presets = [    [        "@babel/env",        {               "useBuiltIns": "usage",            "corejs": 3        }    ]]
```

Babel 会检查所有代码，以便查找在目标环境中缺失的功能，然后仅仅把需要的 polyfill 包含进来。

Babel 会使用很小的辅助函数来实现类似 _createClass 等公共方法。默认情况下，它将被添加(inject)到需要它的每个文件中。

### @babel/plugin-transform-runtime

@babel/plugin-transform-runtime 是一个可以重复使用 Babel 注入的帮助程序，以节省代码大小的插件。

@babel/plugin-transform-runtime 需要和 @babel/runtime 配合使用。

npm install --save-dev @babel/plugin-transform-runtime

npm install --save @babel/runtime

```json
//.babelrc
{ "presets": [
    ["@babel/preset-env", {"useBuiltIns": "usage", "corejs": 3}]
  ],
  "plugins": [["@babel/plugin-transform-runtime"]]
}
```

如果我们希望 @babel/plugin-transform-runtime 不仅仅处理帮助函数，同时也能加载 polyfill 的话，我们需要给 @babel/plugin-transform-runtime 增加配置信息。

新增依赖 @babel/runtime-corejs3: npm install @babel/runtime-corejs3 --save

```json
{ "presets": [
    ["@babel/preset-env", {"useBuiltIns": "usage", "corejs": 3}]
  ],
  "plugins": [["@babel/plugin-transform-runtime", {"corejs": 3}]]
}
```

### 插件/预设补充知识

如果两个转换插件都将处理“程序（Program）”的某个代码片段，则将根据转换插件或 preset 的排列顺序依次执行。

- 插件在 Presets 前运行。

- 插件顺序从前往后排列。

- Preset 顺序是颠倒的（从后往前）。

插件参数

插件和 preset 都可以接受参数，参数由插件名和参数对象组成一个数组。preset 设置参数也是这种格式。

### 配置文件

Babel 支持多种格式的配置文件。

所有的 Babel API 参数都可以被配置，但是如果该参数需要使用的 JS 代码，那么可能需要使用 JS 代码版的配置文件。

如果希望以编程的方式创建配置文件或者希望编译 node_modules 目录下的模块：那么 babel.config.js 可以满足你的需求。

如果只是需要一个简单的并且中用于单个软件包的配置：那么 .babelrc 即可满足你的需求。
