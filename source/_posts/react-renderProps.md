---
title: react/renderProps
date: 2017-07-08 16:08:25
tags: react
---

具有reader prop的组件，接受一个函数，该函数返回一个react元素，并调用它，而不是事先自己的渲染逻辑

```javascript
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

如何分享一个组件封装到其他需要相同state组件的状态或行为并不是很容易

render prop是一个用于告知组件需要渲染什么内容的函数prop

```javasccript
import React from "react";

class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <p style={{ position: "absolute", left: mouse.x, top: mouse.y }}>猫咪</p>
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handelMouseMove = this.handelMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handelMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: "100%" }} onMouseMove={this.handelMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>移动鼠标！</h1>
        <Mouse render={mouse => <Cat mouse={mouse} />} />
      </div>
    );
  }
}

export default MouseTracker;

```
