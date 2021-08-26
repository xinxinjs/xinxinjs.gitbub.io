---
title: 手写一个Promise，了解Promise核心原理
date: 2019-10-23 11:24:51
tags: javascript
---

使用promise：

```javascript
new Promise(
  function (resolve, reject) {
    // 一段耗时的异步操作
    resolve('成功') // 数据处理完成
    // reject('失败') // 数据处理出错
  }
).then(
  (res) => {console.log(res)},  // 成功
  (err) => {console.log(err)} // 失败
)
```

这是最简单的使用promise，我的目的就是使用自己写的MyPromise替换原生的Promise

## 定义整体的结构

### 第一步,先写出构造函数，将Promise向外暴露

```javascript
(function (window) {
  /*
  Promise构造函数
  executor:执行器函数
   */
  function Promise(executor) {
    
  }

  // 向外暴露Promise
  window.MyPromise = Promise;
})(window);
```

### 添加Promise原型对象上的方法

```javascript
/*
    Promise原型对象的then
    指定一个成功/失败的回调函数
    返回一个新的promise对象
     */
    Promise.prototype.then = function(onResolved,onRejected){

    }

    /*
    Promise原型对象的.catch
    指定一个失败的回调函数
    返回一个新的promise对象
     */
    Promise.prototype.catch = function(onRejected){

    }
```

### 添加Promise函数对象上的方法

```javascript
/*
  Promise函数对象的resovle方法
  返回一个指定结果的promise对象
   */
  Promise.resolve = function(value){
    
  }
  /*
  Promise函数对象的reject方法
  返回一个指定reason的失败状态的promise对象
  */
  Promise.reject = function(reason){
    
  }
  /*
  Promise函数对象的all方法
  返回一个promise对象，只有当所有promise都成功时返回的promise状态才成功
  */
  Promise.all = function(promises){
    
  }
  /*
  Promise函数对象的race方法
  返回一个promise对象，状态由第一个完成的promise决定
  */
  Promise.race = function(promises){
    
  }
```

## 实现Promise构造函数

首先看一下，平时是如何使用Promise的

```javascript
const promiseA = new Promise( (resolve,reject) => {
    // 一段耗时的异步操作
    resolve('resolve');
});

promiseA.then(
  (res) => {console.log(res)},  // 成功
  (err) => {console.log(err)} // 失败
)
```

通过new实例化一个Promise对象，Promise构造函数接受一个函数参数，这个函数可以叫做executor（执行器函数），执行器函数会被被立即执行，这个执行器函数接受两个函数作为参数：resolve和reject，执行器函数内部会立即执行resolve和reject，resolve和reject函数是Promise内部函数，需要逐步实现这两个函数，先声明resolve和reject函数，

```javascript
(function (window) {
  /*
  Promise构造函数
  executor:执行器函数
   */
  function Promise(executor) {
    function resovle() {

    }
    function reject() {

    }

    // 立即同步执行executor
    executor(resovle,reject)
  }

  // 向外暴露Promise
  window.MyPromise = Promise;
})(window);
```

然后程序运行到promiseA.then()，首先知道一个知识点Promise有三个状态：

1. pending[待定]初始状态
2. fulfilled[实现]操作成功
3. rejected[被否决]操作失败

并且promise状态一经改变，不会再变。只能改变一次，需要声明一个status变量表示当前状态，并且每一个Promise都有一个data值用来存储数据结果

```javascript
(function (window) {
  /*
  Promise构造函数
  executor:执行器函数
   */
  function Promise(executor) {
    var self = this;
    self.status = 'pending' // 给promise对象指定status属性，初始值为pending
    self.data = undefined // 给promise对象指定一个存储结果的data

    function resovle() {

    }
    function reject() {

    }

    // 立即同步执行executor
    executor(resovle,reject);
  }

  // 向外暴露Promise
  window.MyPromise = Promise;
})(window);
```

当程序运行到promiseA.then()的时候，从代码可以看出来，then函数接受两个函数参数，当Promise状态为fulfilled的时候执行第一个参数函数，状态为rejected的时候执行第二个函数参数，

但是当程序运行到promiseA.then()的时候，有可能所以promise的状态还是pending，这时要把then里面的回调函数保存起来，所以需要个callbacks数组

```javascript
(function (window) {
  /*
  Promise构造函数
  executor:执行器函数
   */
  function Promise(executor) {
    var self = this;
    self.status = 'pending' // 给promise对象指定status属性，初始值为pending
    self.data = undefined // 给promise对象指定一个存储结果的data
    // 新增代码
    self.callbacks = []  // 每个元素的结构：{onResolved(){}，onRejected(){}}

    function resovle() {

    }
    function reject() {

    }

    // 立即同步执行executor
    executor(resovle,reject);
  }

  // 向外暴露Promise
  window.MyPromise = Promise;
})(window);
```

resolve和reject函数，resolve函数执行callbacks里的函数，并保存data，并将当前promise状态改为resolved。reject函数也是这个道理

```javascript
(function (window) {
  /*
  Promise构造函数
  executor:执行器函数
   */
  function Promise(executor) {
    var self = this;
    self.status = 'pending' // 给promise对象指定status属性，初始值为pending
    self.data = undefined // 给promise对象指定一个存储结果的data
    // 新增代码
    self.callbacks = []  // 每个元素的结构：{onResolved(){}，onRejected(){}}

    // 新增代码
    function resovle(value) {
      // 如果当前状态不是pending，则不执行
      if(self.status !== 'pending'){
        return;
      }
      // 将状态改为resolved
      self.status = 'resolved';
      // 保存value的值
      self.data = value
      // 如果有待执行的callback函数，立即异步执行回调函数onResolved
      if (self.callbacks.length>0){
        self.callbacks.forEach(callbackObj=>{
          callbackObj.onResolved(value)
        })
      }
    }
    function reject(value) {
      // 如果当前状态不是pending，则不执行
      if(self.status !== 'pending'){
        return;
      }
      // 将状态改为rejected
      self.status = 'rejected';
      // 保存value的值
      self.data = value;

      // 如果有待执行的callback函数，立即异步执行回调函数onResolved
      if (self.callbacks.length>0){
        self.callbacks.forEach(callbackObj=>{
            callbackObj.onRejected(value);
        })
      }
    }

    // 立即同步执行executor
    executor(resovle,reject);
  }

  // 向外暴露Promise
  window.MyPromise = Promise;
})(window);
```

try catch函数捕获executor执行器函数运行异常

```javascript
(function (window) {
  /*
  Promise构造函数
  executor:执行器函数
   */
  function Promise(executor) {
    var self = this;
    self.status = 'pending' // 给promise对象指定status属性，初始值为pending
    self.data = undefined // 给promise对象指定一个存储结果的data
    // 新增代码
    self.callbacks = []  // 每个元素的结构：{onResolved(){}，onRejected(){}}

    // 新增代码
    function resovle(value) {
      // 如果当前状态不是pending，则不执行
      if(self.status !== 'pending'){
        return;
      }
      // 将状态改为resolved
      self.status = 'resolved';
      // 保存value的值
      self.data = value
      // 如果有待执行的callback函数，立即异步执行回调函数onResolved
      if (self.callbacks.length>0){
        self.callbacks.forEach(callbackObj=>{
          callbackObj.onResolved(value)
        })
      }
    }
    function reject(value) {
      // 如果当前状态不是pending，则不执行
      if(self.status !== 'pending'){
        return;
      }
      // 将状态改为rejected
      self.status = 'rejected';
      // 保存value的值
      self.data = value;

      // 如果有待执行的callback函数，立即异步执行回调函数onResolved
      if (self.callbacks.length>0){
        self.callbacks.forEach(callbackObj=>{
            callbackObj.onRejected(value);
        })
      }
    }

    try{
      // 立即同步执行executor
      executor(resolve,reject);
    }catch (e) { // 如果执行器抛出异常，promise对象变为rejected状态
      reject(e);
    }
  }

  // 向外暴露Promise
  window.MyPromise = Promise;
})(window);
```

## 实现then方法

Promise.then接受两个参数onResolved,onRejected，分别在Promise状态为pending和rejected的时候执行，当Promise状态是pending的时候，会把onResolved和onRejected函数push进之前声明的callbacks数组

```javascript
Promise.prototype.then = function(onResolved,onRejected){
  var self = this;
  if(self.status === 'pending'){
      // promise当前状态还是pending状态，将回调函数保存起来
      self.callbacks.push({
          onResolved(){onResolved(self.data)},
          onRejected(){onRejected(self.data)}
      })
  }else if(self.status === 'resolved'){

  }else{

  }
}
```

当Promise状态是resolved或者rejected状态，此时就不用把回调保存起来，直接执行onResolved或onRejected方法。注意是异步执行。而且是做为微任务的，这里我们简单的用setTimeout来实现就好了。

```javascript
Promise.prototype.then = function(onResolved,onRejected){
  var self = this;
  if(self.status === 'pending'){
      // promise当前状态还是pending状态，将回调函数保存起来
      self.callbacks.push({
          onResolved(){onResolved(self.data)},
          onRejected(){onRejected(self.data)}
      })
  }else if(self.status === 'resolved'){
    // 新增代码
    setTimeout(()=>{
      onResolved(self.data)
    })
  }else{
    // 新增代码
    setTimeout(()=>{
      onResolved(self.data)
    })
  }
}
```

执行完then是要返回一个新的promise的，而新的promise的状态则由当前then的执行结果来确定。所以先来return一个新的Promise

```javascript
Promise.prototype.then = function(onResolved,onRejected){
  var self = this;
  return new Promise((resolve,reject) => {
    if(self.status === 'pending'){
      // promise当前状态还是pending状态，将回调函数保存起来
      self.callbacks.push({
          onResolved(){onResolved(self.data)},
          onRejected(){onRejected(self.data)}
      })
    }else if(self.status === 'resolved'){
      // 新增代码
      setTimeout(()=>{
        onResolved(self.data)
      })
    }else{
      // 新增代码
      setTimeout(()=>{
        onResolved(self.data)
      })
    }
  })
}
```

Promise有一个优点：链式调用，如下代码里面的：`promise.then().then()`，在创建Promise的时候，执行到第一个then函数，由于then函数里面也会返回一个Promise，所以创建的第一个Promise就称他为当前Promise，当前的Promise状态为resolved的时候，会运行到then函数里面的`else if(self.status === 'resolved')`语句，然后会执行`onResolved(self.data)`

```javascript
let promise = new Promise((resolve,reject)=>{
  resolve(1)
})

promise.then(
  
).then(

)
```

当then的回调函数返回的不是promise，那么then函数中return的新promise的状态是则是resolved，value就是返回的值。例如：

```javascript
let promise = new Promise((resolve,reject)=>{
  resolve(1)
})

promise.then(
  value=>{
    return value //返回的不是promise，是value
  }
)
```

所以then函数可以这样子写

```javascript
Promise.prototype.then = function(onResolved,onRejected){
  var self = this;
  return new Promise((resolve,reject) => {
    if(self.status === 'pending'){
      // promise当前状态还是pending状态，将回调函数保存起来
      self.callbacks.push({
          onResolved(){onResolved(self.data)},
          onRejected(){onRejected(self.data)}
      })
    }else if(self.status === 'resolved'){
      // 新增代码
      setTimeout(()=>{
        const result = onResolved(self.data);
        if (result instanceof Promise){

        } else {
          // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
          // resolve函数将当前新的promise的状态改为resolved，同时将value保存到当前新的promise的data中。
          resolve(result)
        }
      })
    }else{
      // 新增代码
      setTimeout(()=>{
        onResolved(self.data)
      })
    }
  })
}
```

当then的回调函数返回的是promise，then函数中return的promise的结果就是这个promise的结果，如下代码所示，then函数的回调函数返回一个新的promise。如果这个promise执行了resolve，then函数中返回的新的promise的状态则是resolved的。否则为rejected

```javascript
let promise = new Promise((resolve,reject)=>{
    resolve(1)
})

promise.then(
    value=>{
        return new Promise((resolve,reject)=>{
            resolve(2)
            //或者
            //reject(error)
        })
    }
)
```

所以then函数可以这样子写，try catch来实现如果执行then回调函数这段代码的时候抛出错误，则返回的promise的状态为rejected，

```javascript
Promise.prototype.then = function(onResolved,onRejected){
  var self = this;
  return new Promise((resolve,reject) => {
    if(self.status === 'pending'){
      // promise当前状态还是pending状态，将回调函数保存起来
      self.callbacks.push({
          onResolved(){onResolved(self.data)},
          onRejected(){onRejected(self.data)}
      })
    }else if(self.status === 'resolved'){
      // 新增代码
      setTimeout(()=>{
        try{
          const result = onResolved(self.data);
          if (result instanceof Promise){
            // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
            result.then(
              value => {resolve(value)},
              reason => {reject(reason)}
            )
          } else {
            // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
            // resolve函数将当前新的promise的状态改为resolved，同时将value保存到当前新的promise的data中。
            resolve(result)
          }
        }catch (e) {
          //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
          reject(e)
        }
      })
    }else{
      // 新增代码
      setTimeout(()=>{
        onResolved(self.data)
      })
    }
  })
}
```

创建的第一个Promise就称他为当前Promise，当前的Promise状态为rejected的时候，道理和状态为resolved的时候一样，所以代码也是一样的，
可以封装一下代码

```javascript
Promise.prototype.then = function(onResolved,onRejected){
  var self = this;
  return new Promise((resolve,reject) => {
    /*
    调用指定回调函数的处理，根据执行结果。改变return的promise状态
     */
    function handle(callback) {
      try{
        const result = callback(self.data)
        if (result instanceof Promise){
          // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
          result.then(
            value => {resolve(value)},
            reason => {reject(reason)}
          )
        } else {
          // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
          resolve(result)
        }
      }catch (e) {
        //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
        reject(e)
      }
    }

    if(self.status === 'pending'){
      // promise当前状态还是pending状态，将回调函数保存起来
      // promise当前状态还是pending状态，将回调函数保存起来
      self.callbacks.push({
        onResolved(){
          handle(onResolved);
        },
        onRejected(){onRejected(self.data)}
      });
    }else if(self.status === 'resolved'){
      setTimeout(()=>{
        handle(onResolved);
      });
    }else{
      setTimeout(()=>{
        handle(onRejected);
      });
    }
  })
}
```

promise会发生值传透，所以最终的then函数

```javascript
/*
  Promise原型对象的then
  指定一个成功/失败的回调函数
  返回一个新的promise对象
   */
  Promise.prototype.then = function(onResolved,onRejected){
    // 当then中传入的不算函数，则这个then返回的promise的data，将会保存上一个的promise.data。这就是发生值穿透的原因。而且每一个无效的then所返回的promise的状态都为resolved。
    onResolved = typeof onResolved === 'function'? onResolved: value => value
    onRejected = typeof onRejected === 'function'? onRejected: reason => {throw reason}
    var self = this;

    return new Promise((resolve,reject)=>{
      /*
      调用指定回调函数的处理，根据执行结果。改变return的promise状态
       */
      function handle(callback) {
        try{
          const result = callback(self.data)
          if (result instanceof Promise){
            // 2. 如果回调函数返回的是promise，return的promise的结果就是这个promise的结果
            result.then(
              value => {resolve(value)},
              reason => {reject(reason)}
            )
          } else {
            // 1. 如果回调函数返回的不是promise，return的promise的状态是resolved，value就是返回的值。
            resolve(result)
          }
        }catch (e) {
          //  3.如果执行onResolved的时候抛出错误，则返回的promise的状态为rejected
          reject(e)
        }
      }

      if(self.status === 'pending'){
        // promise当前状态还是pending状态，将回调函数保存起来
        self.callbacks.push({
          onResolved(){
            handle(onResolved);
          },
          onRejected(){onRejected(self.data)}
        });
      }else if(self.status === 'resolved'){
        setTimeout(()=>{
          handle(onResolved);
        });
      }else{
        setTimeout(()=>{
          handle(onRejected);
        });
      }
    })
  }
```

## 实现catch方法

catch方法的作用跟then里的第二个回调函数一样，因此我们可以这样来实现

```javascript
/*
  Promise原型对象的.catch
  指定一个失败的回调函数
  返回一个新的promise对象
   */
  Promise.prototype.catch = function(onRejected){
    return this.then(undefined,onRejected)
  }
```

## 实现Promise.resolve

Promise.resolve方法可以传三种值

1. 不是promise
2. 成功状态的promise
3. 失败状态的promise

```javascript
Promise.resolve(1)
Promise.resolve(Promise.resolve(1))
Promise.resolve(Promise.reject(1))
```

具体的实现

```javascript
/*
  Promise函数对象的resovle方法
  返回一个指定结果的promise对象
 */
Promise.resolve = function(value){
  return new Promise((resolve,reject)=>{
    if (value instanceof Promise){
        // 如果value 是promise
        value.then(
            value => {resolve(value)},
            reason => {reject(reason)}
        )
    } else{
        // 如果value不是promise
        resolve(value)
    }
  })
}
```

## 实现Promise.reject

```javascript
/*
  Promise函数对象的reject方法
  返回一个指定reason的失败状态的promise对象
*/
Promise.reject = function(reason){
  return new Promise((resolve,reject)=>{
    reject(reason);
  })
}
```

## 实现Promise.all

```javascript
/*
  Promise函数对象的all方法
  返回一个promise对象，只有当所有promise都成功时返回的promise状态才成功
*/
Promise.all = function(promises){
  const values = new Array(promises.length)
  var resolvedCount = 0 //计状态为resolved的promise的数量
  return new Promise((resolve,reject)=>{
    // 遍历promises，获取每个promise的结果
    promises.forEach((p,index)=>{
      Promise.resolve(p).then(
        value => {
          // p状态为resolved，将值保存起来
          values[index] = value
          resolvedCount++;
          // 如果全部p都为resolved状态，return的promise状态为resolved
          if(resolvedCount === promises.length){
            resolve(values)
          }
        },
        reason => { //只要有一个失败，return的promise状态就为reject
          reject(reason);
        }
      )    
    })    
  })
}
```

## 实现Promise.race

```javascript
/*
  Promise函数对象的race方法
  返回一个promise对象，状态由第一个完成的promise决定
*/
Promise.race = function(promises){
  return new Promise((resolve,reject)=>{
    // 遍历promises，获取每个promise的结果
    promises.forEach((p,index)=>{
        Promise.resolve(p).then(
            value => {
                // 只要有一个成功，返回的promise的状态九尾resolved
                resolve(value)
            },
            reason => { //只要有一个失败，return的promise状态就为reject
                reject(reason)
            }
        )
    })
  })
}
```
