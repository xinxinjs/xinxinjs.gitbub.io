---
title: react/context
date: 2018-02-28 15:12:29
tags: react
---

context 提供了一个无需为每层组件手动添加 props，就可以在组件树之间进行数据传递的方法

## API

- React.createContext

```javascript
// 创建Context对象，并传递一个默认的值： defaultValue， 其他组件可以订阅这个Context对象，称为消费组件
const MyContext = React.createContext("defaultValue");
```

- Context.Provider

```javascript
// 每个Context对象都返回一个Provider React 组件，
<MyContext.Provider value={/* 某个值 */}>
  <ThemedButton />
</MyContext.Provider>
```

Provider 接收一个 value 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

当 Provider 的 value 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 shouldComponentUpdate 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。

- 订阅 Context

```javascript
// 通过指定contextType订阅Context
class ThemedButton extends React.Component {
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

- Class.contextType

挂在在 class 的 contextType 属性会被重新赋值为一个由 React.createContext() 创建的 Context 对象

在 class 内部可以使用 this.context 来消费最近的 Context 上的那个值，可以在任何生命周期中访问到它，包括 render 函数中。

- Context.Consumer

在函数式组件中完成订阅 Context，这需要函数作为子元素，这个函数接受当前的 context 值，返回一个 React 节点

```javascript
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

传递给函数的 value 值等同于往上组件树离这个 context 最近的 Provider 提供的 value 值。如果没有对应的 Provider，value 参数等同于传递给 createContext() 的 defaultValue。

## 动态 Context

## 嵌套组件中更新 Context

要在嵌套很深的组件中更新 Context，可以通过 context 传递一个函数，使得 consumers 组件更新 context

```javascript
// theme-context.js
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext({
  theme: themes.dark,
  // 默认值，函数
  toggleTheme: () => {
    console.log("default click");
  }
});

// theme-toggler-button.js
function ThemeToggleButton() {
  return (
    // 利用函数式组件和Consumer订阅Context
    <ThemeContext.Consumer>
      {({ theme, toggleTheme }) => (
        <button
          onClick={toggleTheme}
          style={{ backgroundColor: theme.background, color: theme.foreground }}
        >
          Toggle Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
}

class App extends React.Component {
  constructor(props) {
    super(props);

    this.toggleTheme = () => {
      console.log("click");
      this.setState(state => ({
        theme: state.theme === themes.dark ? themes.light : themes.dark
      }));
    };

    this.state = {
      theme: themes.light,
      toggleTheme: this.toggleTheme
    };
  }

  render() {
    return (
      <ThemeContext.Provider value={this.state}>
        <Content />
      </ThemeContext.Provider>
    );
  }
}

export default App;

function Content() {
  return (
    <div>
      <ThemeToggleButton />
    </div>
  );
}
```

## 消费多个 Context

```JSX
<ThemeContext.Consumer>
  {theme => (
    <UserContext.Consumer>
      {user => <ProfilePage user={user} theme={theme} />}
    </UserContext.Consumer>
  )}
</ThemeContext.Consumer>
```
