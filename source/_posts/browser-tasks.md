---
title: 浏览器的事件循环和任务队列
date: 2018-08-17 15:08:32
tags: browser
---

参考文档：

[浏览器中的事件循环机制](https://segmentfault.com/a/1190000012748907)

[浏览器的微任务MicroTask和宏任务MacroTask](https://segmentfault.com/a/1190000019059045?utm_source=tag-newest)

## 异步

js在浏览器运行是单线程的，一般情况下浏览器会按顺序同步执行js代码，但是有些情况下需要延迟执行js代码

最常见的就是等待接口返回数据之后再去执行回调函数，不可能一直在那里等着接口有返回了再去执行后面的，这就是会阻塞后面js执行，请求接口是同步执行的，请求成功之后的回调是异步的，所以会先找一个地方把异步操作存起来，等其他的js执行完成了再去执行，这个存贮的地方就是浏览器的任务队列

## 事件循环（event loop）机制

浏览器的事件循环机制，可以理解为任务队列的机制

### 任务队列的分类

- 同步任务（Task），即为立即执行的js代码队列

- 宏任务（macroTask），setTimeout、setInterval、I/O、UI渲染

- 微任务（microTask），promise、object.obsever、MutationObsever

- 用户交互事件，点击事件等

### promise 与 setTimeout

我们用的比较多的是这两个，先看一下这两个的执行顺序

```javascript
function test() {
  let a = 10 + 10;
  console.log(a);

  setTimeout(function () {
    console.log("setTimeout");
  }, 0);

  const promise = new Promise(function(resolve, reject) {
    console.log('before resolve');
    resolve('promise');
    console.log('after resolve');
  });

  promise.then((value) => {
    console.log(value);
  })

  console.log('end');
}
```

执行结果：

```javascript
20
before resolve
after resolve
end
promise
setTimeout
```

为什么‘promise’会比‘延迟任务’先打印出来，明明按照代码的编写顺序延迟任务setTimeout是先进入任务队列的，这就是因为promise进入的是微任务（microTask）队列，而延迟任务进入的是宏任务（macroTask）队列，从上可看出，微任务（microTask）队列是优先于宏任务（macroTask）队列执行的

执行顺序说明：

1. test函数进入执行队列

2. 执行同步任务，console.log(a)，同步任务都是立即执行的

3. setTimeout任务进入任务队列，没有立即执行，而是放在了宏任务（macroTask）队列

4. 代码继续往下走，promise进入任务队列，new promise会立即执行传入的回调函数，resolve('promise)前后和它的两个console.log()，都是同步任务，所以会立即执行，执行到resolve('promise');这一句，把promise.then()放入任务队列，这个是放入微任务（microTask）队列，

5. 执行同步任务console.log('end');

6. 浏览器闲下来，优先查找微任务（microTask）队列，执行promise.then()，控制台打印console.log(value);

7. 微任务（microTask）队列为空，查找宏任务（macroTask）队列，控制台打印console.log("延迟任务");

### 只要有 Microtasks 插入，就会不断执行 Microtasks 队列直到结束，在结束前都不会执行到 Tasks

```html
<body>
  <div class="outer">
    <div class="inner">click me</div>
  </div>
  <script>
    var outer = document.querySelector(".outer");
    var inner = document.querySelector(".inner");
    function test() {
      // 创建并返回一个新的 MutationObserver 它会在指定的DOM发生变化时被调用。
      new MutationObserver(function () {
        console.log("检测到dom改变");
      }).observe(outer, {
        attributes: true,
      });

      function onClick(target) {
        console.log("click" + ' ' + target);

        setTimeout(function () {
          console.log("延迟任务");
        }, 0);

        const promise = new Promise(function(resolve, reject) {
          resolve('promise');
        });

        promise.then((value) => {
          console.log(value);
        })

        outer.setAttribute("data-random", Math.random());
      }

      inner.addEventListener("click", () => onClick('inner'));
      outer.addEventListener("click", () => onClick('outer'));
    }
    test();
  </script>
</body>
```

执行结果：

```javascript
click inner
promise
检测到dom改变
click outer
promise
检测到dom改变
延迟任务
延迟任务
```

### 一个事件循环执行过程

先执行

```javascript
function test() {
      const interval = setInterval(() => {
        console.log('setInterval')
      }, 0)

      setTimeout(() => {  
        console.log('setTimeout 1')
        Promise.resolve().then(() => {
          console.log('promise 2')
        }).then(() => {
          setTimeout(() => {
            console.log('setTimeout 2')
            Promise.resolve().then(() => {
              console.log('promise 3')
            }).then(() => {
                clearInterval(interval)
            })
          }, 0)
        })
      }, 0)

      Promise.resolve().then(() => {
        console.log('promise 1')
      })
    }
```

```javascript
promise 1
setInterval
setTimeout 1
promise 2
setInterval
setTimeout 2
promise 3
```