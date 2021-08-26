---
title: React 使用 Redux
date: 2018-10-27 17:15:50
tags: react
categories: 技术
---

Redux 和 React 之间没有关系。Redux 支持 React、Angular、Ember、jQuery 甚至纯 JavaScript。

react-redux是redux为react专门封装的库，Redux 默认并不包含 React 绑定库，需要单独安装。

```bash
npm install --save react-redux
```

学习使用react-redux之前先了解一些知识

## 组件

React-Redux 将所有组件分成两大类：UI 组件（presentational component）和容器组件（container component）。

### UI组件

只负责展示UI

UI 组件有以下几个特征：

- 只负责 UI 的呈现，不带有任何业务逻辑

- 没有状态

- 所有数据都由参数（props）提供

- 不使用任何 Redux 的 API

例如：

```jsx
const Title =
  value => <h1>{value}</h1>;
```

因为不含有状态，UI 组件又称为"纯组件"，即它纯函数一样，纯粹由参数决定它的值。

### 容器组件

容器组件的特征恰恰相反:

- 负责管理数据和业务逻辑，不负责 UI 的呈现

- 带有内部状态

- 使用 Redux 的 API

## 学习使用react-redux

从一个例子，来学习如何使用react-redux：实现一个todo列表，一个 todo 项被点击后，会增加一条删除线并标记 completed。我们会显示用户新增一个 todo 字段。在 footer 里显示一个可切换的显示全部/只显示 completed 的/只显示 incompleted 的 todos。

这里使用react hooks ，ts 外加react-redux实现，整体的思路是先写出UI组件，在使用react-redux把UI组件和状态业务逻辑链接起来

### 写出UI组件

#### todo项组件

负责展示每一条todo的内容，先简单的写一个

```tsx
interface TodoProps {
  text: string
}
const Todo = ({text}: TodoProps) => (
  <li>{text}</li>
)
```

#### TodoList组件

负责展示整个todo列表

```tsx
const TodoList = ({ todos, onTodoClick }) => {
  return (
    <ul>
      {todos.map((todo, index) => (
      <Todo key={index} {...todo} onClick={() => onTodoClick(index)} />
    ))}
    </ul>
  )
}
```

#### Link组件

在footer里面用来展示todos

```tsx
const Link = ({ active, children }: LinkProps) => {
  if (active) {
    return <span>{children}</span>
  }

  return (
    <a href="" >
      {children}
    </a>
  )
}
```

### Footer组件

然后如何把UI组件和redux的state连接起来呢？这就是react-redux做的工作了，React-Redux 提供connect方法，用于从 UI 组件生成容器组件。connect的意思，就是将这两种组件连起来。

### 生成容器组件

content()使用方法，把刚才写的UI组件TodoList生成容器组件

```javascript
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)
```

content()是一个高阶组件，接受两个参数mapStatetoProps和mapDispatchToProps

使用 connect() 生成容器组件之前，需要先定义 mapStateToProps 这个函数来指定如何把当前 Redux store state 映射到展示组件的 props 中。

#### mapStateToProps

mapStateToProps是一个函数，建立一个从（外部的）state对象到（UI 组件的）props对象的映射关系。该函数执行后返回一个对象，里面每一个健值对就是一个映射。

mapStateToProps会订阅 Store，每当state更新的时候，就会自动执行，重新计算 UI 组件的参数，从而触发 UI 组件的重新渲染。

mapStateToProps的第一个参数总是state对象，还可以使用第二个参数，代表容器组件的props对象。

connect方法可以省略mapStateToProps参数，那样的话，UI 组件就不会订阅Store，就是说 Store 的更新不会引起 UI 组件的更新。

上面例子的需求中：在footer 里显示一个可切换的显示全部/只显示 completed 的/只显示 incompleted 的 todos，所以定义了根据 state.visibilityFilter 来过滤 state.todos 的方法，并在 mapStateToProps 中使用。

```tsx
const getVisibleTodos = (todos: any[], filter: string) => {
  switch (filter) {
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    case 'SHOW_ALL':
    default:
      return todos
  }
}
const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```

mapStateToProps接受state作为参数，返回一个对象，对象有一个todos属性，代表了UI组件的同名参数，后面的getVisibleTodos也是一个函数，可以从state算出 todos 的值。

#### mapDispatchToProps

除了读取 state，容器组件还能分发 action。这就要用到connect()方法的第二个参数,mapDispatchToProps可以是一个函数，也可以是一个对象。

mapDispatchToProps() 方法接收 dispatch() 方法并返回期望注入到展示组件的 props 中的回调方法

mapDispatchToProps作为函数，应该返回一个对象，该对象的每个键值对都是一个映射，定义了 UI 组件的参数怎样发出 Action。

如果mapDispatchToProps是一个对象，它的每个键名也是对应 UI 组件的同名参数，键值应该是一个函数，会被当作 Action creator ，返回的 Action 会由 Redux 自动发出。

现在我们希望 VisibleTodoList 向 TodoList 组件中注入一个叫 onTodoClick 的 props ，还希望 onTodoClick 能分发 TOGGLE_TODO 这个 action：

```tsx
const mapDispatchToProps = dispatch => {
  return {
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  }
}
```

未完。。。

## 参考文档

[react-redux文档](https://www.redux.org.cn/docs/react-redux/)

[Redux 入门教程（三）：React-Redux 的用法](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_three_react-redux.html)
