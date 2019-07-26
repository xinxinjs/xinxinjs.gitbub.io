---
title: react/refs
date: 2019-07-15 11:20:35
tags: react
---

官网文档的搬运者

## 何时使用 refs

1. 管理焦点，文本选择或媒体播放

2. 触发强制动画。

3. 集成第三方 DOM 库。

## 创建 ref

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    // 创建ref
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```

## 使用

```javascript
const node = this.myRef.current;
```

### ref 的值根据节点的类型而有所不同：

- 当 ref 属性用于 HTML 元素时，构造函数中使用 React.createRef() 创建的 ref 接收底层 DOM 元素作为其 current 属性。

- 当 ref 属性用于自定义 class 组件时，ref 对象接收组件的挂载实例作为其 current 属性。

- 你不能在函数组件上使用 ref 属性，因为他们没有实例。

## 回调 Refs

```javascript
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // 使用原生 DOM API 使 text 输入框获得焦点
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // 组件挂载后，让文本框自动获得焦点
    this.focusTextInput();
  }

  render() {
    // 使用 `ref` 的回调函数将 text 输入框 DOM 节点的引用存储到 React
    // 实例上（比如 this.textInput）
    return (
      <div>
        <input type="text" ref={this.setTextInputRef} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

可以在组件间传递回调形式的 refs，就像你可以传递通过 React.createRef() 创建的对象 refs 一样。

```javascript
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return <CustomTextInput inputRef={el => (this.inputElement = el)} />;
  }
}
```

## 转发 refs 到 DOM 组件

Ref 转发是一个可选特性，其允许某些组件接收 ref，并将其向下传递（换句话说，“转发”它）给子组件。

```javascript
const FancyButton = React.forwardRef((props, ref) => (
  // 当 ref 挂载完成，ref.current 将指向 <button> DOM 节点。
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 你可以直接获取 DOM button 的 ref：
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

## 在高阶组件(HOC)中转发 refs

```javascript
function logProps(WrappedComponent) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log("old props:", prevProps);
      console.log("new props:", this.props);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  }

  return LogProps;
}

class FancyButton extends React.Component {
  focus() {}
}

export default logProps(FancyButton);
```

调用高阶组件：logProps组件，并且传递props时，高阶组件会透传props到他的包裹组件，但refs不会传递，因为 ref 不是 prop 属性。就像 key 一样，其被 React 进行了特殊处理。如果你对 HOC 添加 ref，该 ref 将引用最外层的容器组件，而不是被包裹的组件。

### 使用React.forwardRef在高阶组件中传递refs

重写上面的高阶组件：

```javascript
function logProps(WrappedComponent) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log("old props:", prevProps);
      console.log("new props:", this.props);
    }

    render() {
      const { forwardedRef, ...rest } = this.props;

      return <WrappedComponent ref={forwardedRef} {...rest} />;
    }
  }

  // return LogProps;
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```
