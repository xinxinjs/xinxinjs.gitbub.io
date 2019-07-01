---
title: react/state
date: 2019-06-28 10:57:39
tags: react
---

setState方法可以接受一个函数

```javascript
this.setState((state, props) => ({
  // counter: state.counter + props.increment
}));
```

上一个状态的state作为第一个参数，此次更新的props作为第二个参数