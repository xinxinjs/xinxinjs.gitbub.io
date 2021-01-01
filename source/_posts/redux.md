---
title: redux学习
date: 2020-08-11 16:04:29
tags: redux
---

## 基础概念

Redux 是 JavaScript 状态容器，提供可预测化的状态管理。

redux三大原则：

- 单一数据源

- state是只读的

- 使用纯函数来改变state

### store

createStore()方法可以创建一个store，该方法接受一个reducer，reducer描述如何处理store的更新

store是一个对象，用来保存整个应用的状态，一个应用只能有一个store，作为数据的来源

```javascript
const store = createStore(reducer);
```

- 提供 getState() 方法获取 当前的state；

- 提供 dispatch(action) 方法更新 state；stroe是不可以随意修改的，只能通过store.dispatch(action)来修改

- 通过 subscribe(listener) 注册监听器;

- 通过 subscribe(listener) 返回的函数注销监听器。

### state

getState() 方法获取 state，state是当前的状态，state和视图层是一一对应的

### store.dispatch()

通过store.dispatch(action)方法触发更新store，action定义当前应该如何更新store，具体的更新操作有reducer完成

### Action

Action是一个对象，对象中不可缺少的type属性，描述如何更新store，对象中也可以根据需要定义其他的属性

```javascript
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```

### reducer

Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。

它的主要作用是根据action来更新state，返回一个新的state

reducer必须是一个纯函数，同样的输入必须有同样的输出，也就是说同样action和state返回的新的state也必须是一样的

### store.subscribe()

设置监听函数，一旦store发生变化，就会自动执行这个store.subscribe()，主要是用来监听store的变化更新视图层

store.subscribe()返回一个函数，调用返回的函数，可以解除监听

```javascript
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();
```

## 简单的例子

一个结合react的例子

```javascript
// 初始的store
const defaultState = 0;

const reducer = (state = defaultState, action: any) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    case 'REDUCE':
      return state - action.payload;
    default: 
      return state;
  }
};

const store = createStore(reducer);

function testRedux() {
  const [state, setState] = useState(defaultState);
  
  useEffect(() => {
    setState(store.getState());
  }, []);
  // action
  const addAction = {type: 'ADD', payload: 1};
  const reduceAction = {type: 'REDUCE', payload: 1};

  store.subscribe(() => {
    setState(store.getState());
  })
  
  return (
    <div>
      <p>{state}</p>
      <button onClick={() => {store.dispatch(addAction)}}>+</button>
      <button onClick={() => {store.dispatch(reduceAction)}}>-</button>
    </div>
  )
}
```

### combineReducers

项目特别大的时候，store的结构也会很复杂，像这样子，会有很多属性

```javascript
store = {
  count: {

  },
  todoList: {

  },
  list: {

  },
  ...
}
```

导致reducer函数也会十分庞大

```javascript
const reducer = (state = defaultState, action: any) => {
  const {count, todoList} = state;
  const {type, payload} = action;
  switch (type) {
    case 'ADD':
      return {...state, count: count + payload};
    case 'REDUCE':
      return {...state, count: count - payload};
    case 'ADD_TODO':
      return {...state, todoList: [...todoList, `这是一个待办${todoList.length}`]}
    default:
      return {...state};
  }
};
```

但是不同的action会更新store不同的属性，往往是不相关的，可以把reducer拆分成函数，不同的reducer处理不同的属性，redux提供辅助函数combineReducers来合并reducer

```javascript
import { combineReducers } from 'redux';

const countReducer = (count = defaultState.count, action: any) => {
  const {type, payload} = action;
  switch (type) {
    case 'ADD':
      return count + payload;
    case 'REDUCE':
      return count - payload;
    default: 
      return count;
  }
}

const todoReducer = (todoList = defaultState.todoList, action: any) => {
  const {type} = action;
  switch (type) {
    case 'ADD_TODO': 
      return [...todoList, `这是一个待办${todoList.length}`]
    default: 
      return [...todoList];
  }
}

const reducer = combineReducers({
  count: countReducer, todoList: todoReducer
})
```

实际开发的时候，可以把所有的reducer写在一个文件中，然后统一引入

```javascript
import { combineReducers } from 'redux'
import * as reducers from './reducers'

const reducer = combineReducers(reducers)
```

## redux的数据流方向

严格的单向数据流是 Redux 架构的设计核心

### redux的工作流程

从redux的工作流程可以看出数据的流动方向

- 首先创建store：通过createStore(reducer)方法传入一个reducer，创建一个store

- 提前写好reducer，reducer是一个函数，接受一个当前的state和action作为参数，并在函数中做我们要做的数据处理

- 通过store.getState() 方法获取当前的state，state和视图层是一一对应的

- 用户想要更新界面，也就是要更新state，需要调用store.dispatch(action)，来改变state，action也是提前写好的

- store接受到dispatch之后会自动调用reducer方法，并把当前最新的state和dispatch方法接收到的action传入其中

- reducer负责根据action 更新state。并返回一个新的state

- state一旦有变化，store就会调用store.subscribe(listener)，store.subscribe()接受一个函数listener，在listener中可以通过store.getState() 方法获取被reducer处理过的state，然后更新界面

- 如果结合react，可以把setState()方法放在listener函数中去调用，然后以此更新界面

## 异步action

刚才写的全部都是同步更新state，store发出action之后自动调用reducer方法，state立即更新，然后就拿新的数据更新UI层

但是，很多情况下，都需要从接口拉取数据，根据异步请求的知识，等接口返回数据之后，在执行reducer，然后才会用接口返回的数据更新state，再拿新的state更新UI层，这就需要异步数据流来实现，来看一下redux是如何实现异步数据流的

这就涉及到一个知识点，如何让reducer在接口返回了数据之后自动执行，这就需要先了解一个知识点，叫中间件

### Middleware（中间件）

Middleware可以增强store.dispatch()，可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

以异步获取接口数据为例子使用中间件：

异步action，就是发出三种action

1. 在发起请求接口，发出一个action，去请求接口

2. 接口返回数据成功后，发出一个action，调用reducer，更新store，继而获取最新的state，然后更新UI

3. 接口返回失败的时候，发出一个action，

用户触发第一个action是没有问题的，那么如何才能在接口有返回数据的时候触发第二个action去更改state呢？就需要先知道一个知识点：Action Creator，这个东西呢是用来生成action的，一般同步的action 生成比较简单，也好理解，就是返回一个对象：

```javascript
export function addAction(payload) {
  return {
    type: add,
    payload
  }
}

// 使用的时候
store.dispatch(addAction)
```

异步的action Creator，返回一个函数，这个函数首先dispatch一个同步action，然后进行异步操作，也就是请求接口的数据，这里使用原生的fetch方法，等到接口有返回数据之后，在then（）函数里调用第二个dispatch，提前写好要用的reducer，我还写了一个按钮来调用action，简单的代码示例：

```javascript
const tagsReducer = (tags = defaultState.tags, action: any) => {
  const {type, payload = {}} = action;
  switch (type) {
    case 'requestData': 
      return {loading: true, list: []};
    case 'reciveData':
      return {loading: false, list: [...payload.list]}
    default: 
      return {...tags};
  }
}

const requestData = () => {
    return {
      type: 'requestData',
    }
  }

const reciveData = (payload: any) => {
    return {
      type: 'reciveData',
      payload
    }
  }

const fetchData = () => (dispatch: any, getState: any) => {
    dispatch(requestData());
    return fetch('/api/tags').then(response => response.json()).then(json => dispatch(reciveData(json)))
  }

<button onClick={() => {store.dispatch(fetchData)}}>异步获取数据</button>
```

点了按钮之后，发现代码报错：`Error: Actions must be plain objects. Use custom middleware for async actions.`，意思就是action必须是纯对象，如果是异步 action，需要使用自定义的中间件

现在就知道中间件的意义了，本来action只能是一个对象，store.dispatch（action）方法也只能接受一个对象，所以现在需要一个中间件，来让store.dispatch()能够接受一个函数作为参数

redux-thunk就是具有这样子功能的一个中间件，首先安装一下 `yarn add redux-thunk`，然后配合redux提供的applyMiddleware使用：

```javascript
import { applyMiddleware } from 'redux';
import thunk from 'redux-thunk';

const store = createStore(reducer, applyMiddleware(thunk));
```

createStore可以接受applyMiddleware作为第二个参数，applyMiddleware是一个方法，接受我们要使用的中间件，然后再次点击按钮，可以正常的使用了，没有报错，现在可以正常的使用redux处理异步数据流了

当然支持异步能支持异步数据流的中间件不只有redux-thunk，redux-promise也是一样可以的

关于redux的知识点看了这些之后差不多就会使用redux了

参考链接：

[阮一峰老师：Redux 入门教程（二）：中间件与异步操作](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)

[Redux 中文文档](https://www.redux.org.cn/)

[redux教程](https://juejin.im/post/6856791557035524109#heading-14)
