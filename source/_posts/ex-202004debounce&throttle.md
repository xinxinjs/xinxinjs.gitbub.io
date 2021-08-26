---
title: 每日一题之防抖节流
date: 2019-04-13 20:03:10
tags: [数据结构与算法]
categories: [每日一题]
---

防抖和节流都是优化高频率执行代码的一种手段，例如浏览器的resize，scroll，input等事件，

这些事件在被触发的时候会不断的调用绑定在事件上的回调函数，这个就会极大的浪费资源，降低前端的性能

为了优化体验就需要对这些代码的调用次数做限制，所使用的方法就是防抖或者节流算法

## 防抖

防抖的概念：连续触发事件N秒后执行该函数，如果在N秒内事件被重复触发，则重新计时

防抖算法：使用闭包

```javascript
const debounce = (delay, callback) => {
    let timer;
    return function (value) {
      clearTimeout(timer);
      timer = setTimeout(() => {
        callback(value);
      }, delay);
    }
  }

function func (value) {
  console.log(value)
}
// 保存在外部调用
const debounceFunc = debounce(1000, func);
const click = (e) => {
  debounceFunc(e.target.value);
}
```

使用场景：防止重复的执行

- 搜索框输入查询，用户连续触发input事件，在停止输入后的N秒执行函数

- 手机号码验证输入，用户连续触发input事件，在停止输入的N秒之后做校验

- 窗口大小改变，用户连续拖动改变窗口的大小，在停止拖动N秒之后执行函数

## 节流

N秒内之运行一次

```javascript
const throttle = (delay, callback) => {
  let timer;
  return function (value) {
    if(!timer) {
      timer = setTimeout(() => {
        callback(value);
        timer = null;
      }, delay);
    }
  }
}
```

使用场景：

- 滚动加载

- 加载更多

- 滚到底部监听

- 搜索联想功能

## 防抖和节流的相同之处

1. 实现方法，都是通过setTimeout实现的

2. 目的一样，都是想要降低高频代码的执行顺序

## 不同之处

1. 防抖是在一段连续操作结束后处理回调，利用clearTimeout和setTimeout实现

2. 节流，是在一段连续操作中，每一段事件执行一次
