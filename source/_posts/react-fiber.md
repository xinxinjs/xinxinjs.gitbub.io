---
title: react-fiber 原理介绍
date: 2020-12-30 15:30:37
tags: react
categories: 技术
---

## 为什么会出现react-fiber

react15的问题：当我们调用setState更新页面的时候，React 会遍历应用的所有节点，计算出差异，然后再更新 UI。整个过程是一气呵成，不能被打断的

默认情况下，js运算，页面布局，页面绘制都在浏览器的主线程运行，如果页面元素很多，在大量的频繁计算之下，js运算持续占用主线成，页面就没有办法即使的更新

## 解决思路

要解决上面的问题，就是要解决js运算占用主线成大量时间的问题，将运算切割成多个部分，分批完成

15以下的react使用的是递归的方式进行渲染，使用js引擎自身调用函数栈，是不能被中断的，fiber使用window.requestIdleCallback()方法实现切片运行

window.requestIdleCallback()会在浏览器空闲时期依次调用函数，

## react是怎么做的

react框架内部的运作可以分为三层

+ virtual DOM层，描述页面长什么样子

+ react reconciler（调节器） 负责调用组件生命周期方法，进行diff运算

+ Renderer，根据不同的平台，渲染出响应的页面，比较常见的是reactDom 和reactNative

fiber 是一个新的reconciler,它有一个新的名字叫fiber-reconsiler

fiber-reconsiler运行一段时间就会把控制权交给浏览器

Fiber Reconciler 在执行过程中，会分为 2 个阶段。

1. 生成 Fiber 树，得出需要更新的节点信息。这一步是一个渐进的过程，可以被打断。

2. 将需要更新的节点一次过批量更新，这个过程不能被打断。

第一个阶段是Diff 计算的时候生成一个fiber树，这个阶段是可以被打断的，这颗新树每生成一个新的节点，都会将控制权交回给主线程，去检查有没有优先级更高的任务需要执行。如果没有，则继续构建树的过程：

如果过程中有优先级更高的任务需要进行，则 Fiber Reconciler 会丢弃正在生成的树，在空闲的时候再重新执行一遍。