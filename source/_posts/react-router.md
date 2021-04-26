---
title: react-router5.0
date: 2019-012-01 19:17:09
tags: [react]
categories: [react]
---

## react-router是什么？

React Router 是一个基于 React 之上的强大路由库，它可以让你向应用中快速地添加视图和数据流，同时保持页面与 URL 间的同步。

主要包括：

1. react-router：提供路由核心路由功能

2. react-router-dom：基于react-router，加入了在浏览器运行环境下的一些功能，例如：Link组件，会渲染一个a标签，Link组件源码a标签行; BrowserRouter和HashRouter 组件，前者使用pushState和popState事件构建路由，后者使用window.location.hash和hashchange事件构建路由。

3. react-router-native：基于react-router，类似react-router-dom，加入了react-native运行环境下的一些功能。

## 为什么要使用react-router

在学习一个新的工具之前，要先了解一下为什么要学习，我们先来了解一下为什么要学习react-router，它都帮助我们解决了什么问题，带来了哪些便利等！

### 不使用react-router

我们在做一个项目的时候，通常要根据用户点击跳转的url展示不同的页面内容，当然不使用react-router我们也能实现这个

首先来看一下不使用react-router的例子

```jsx
function Home(props) {
  return <h2>我是Home</h2>
}
function About(props) {
  return <h2>我是About</h2>;
}
function Users(props) {
  return <h2>我是Users</h2>;
}
class MyApp extends React.Component{
  state = {
    route: window.location.hash.substr(1)
  }
  componentDidMount() {
    window.addEventListener('hashchange', () => {
      this.setState({
        route: window.location.hash.substr(1)
      })
    })
  }

  render() {
    let Child;
    switch (this.state.route) {
      case '/about': Child = About; break;
      case '/users': Child = Users; break;
      default:      Child = Home;
    }
    return (
      <>
        <ul>
          <li><a href="#/about">About</a></li>
          <li><a href="#/users">Users</a></li>
        </ul>
        <Child/>
      </>
    )
  }
}
```

这个例子比较简单，只有两个需要匹配的URL，可以看出，我们需要自己监听URL的哈希变化，根据URL的哈希值去匹配不同的组件在页面展示不同的内容，当我们的应用程序变的复杂，例如需要匹配的URL的数量增多并且出现嵌套路由，我们就需要更多的代码来达到URL和组建匹配。

### 使用react-router解决问题

我们用react-router重写一下上面的功能，我的demo是运行在web环境中，所以需要安装一下react-router-dom。

`yarn add react-router-dom`，当前版本是5.2.0

```jsx
function App() {
  return <BrowserRouter>
    <div>
      <nav>
        <ul>
          <li>
            <Link to={'/about'}>About</Link>
          </li>
          <li>
            <Link to={'/users'}>Users</Link>
          </li>
        </ul>
      </nav>
      {/* <Switch>通过查找所有的子<Route>并渲染与当前URL匹配的第一个<Route>的内容 */}
      <Switch>
        <Route path={'/about'}>
          <About />
        </Route>
        <Route path={'/users'} children={<Users />}/>
      </Switch>
    </div>
  </BrowserRouter>
}
```

从这个代码可以看出来，react-router可以非常方便的添加路由，并保持路由和URL的同步

## 嵌套路由

修改上面的代码实现嵌套的路由

```jsx
// 修改home组建作为嵌套的父级组件
function Home(props) {
  return (
    <>
      <h1>home</h1>
      <nav>
        <ul>
          <li>
            <Link to={'/about'}>About</Link>
          </li>
          <li>
            <Link to={'/users'}>Users</Link>
          </li>
        </ul>
      </nav>
      {props.children}
    </>
  )
}
// 修改App的组件
function App() {
  return <BrowserRouter>
    <div>
      {/* <Switch>通过查找所有的子<Route>并渲染与当前URL匹配的第一个<Route>的内容 */}
      <Switch>
        <Route path="/">
          <Home>
            <Route path={'/about'}>
              <About />
            </Route>
            <Route path={'/users'} children={<Users />}/>
          </Home>
        </Route>
      </Switch>
    </div>
  </BrowserRouter>
}
```

使用react-router-dom匹配路由差不多就是这么简单

## react-router-dom的核心组件

1. BrowserRouter HashRouter 是路由器，主要是决定使用哪种路由

2. 路由匹配器：Route Switch，通过查找所有的子Route并渲染与当前URL匹配的第一个Route的内容

3. 导航：Link，NavLink和Redirect

### Link的API有哪些

Link，在渲染到真实的DOM中的时候Link组建会被渲染成`<a>`标签:

```html
<a href="/about">About</a>
```

+ to:String

表示要链接到的地址

+ to:Object

可以具有以下任何属性的对象：

pathname: 表示要链接到的路径的字符串。search: query参数的字符串表示形式。hash: 网址中的hash值，例如#a-hash。state: 状态保留到该属性中，这个属性设置的内容会被传递到location.state中

```jsx
<Link
  to={{
    pathname: "/courses",
    search: "?sort=name",
    hash: "#the-hash",
    state: { fromDashboard: true }
  }}
/>
```

+ to:Function

将当前URL当作参数传递给这个函数，这个函数应该返回一个对象或者字符串，表示位置信息

```jsx
<Link to={location => ({ ...location, pathname: "/courses" })} />
<Link to={location => `${location.pathname}?sort=name`} />
```

+ replace:Bool

可以设置为true，则将单击链接替换为history记录堆栈中的当前条目，而不是添加一条新条目。
这样就没有回退功能了，因为它是把当前URL地址替换掉，不会产生历史记录。

```jsx
<Link to="/courses" replace />
```

+ innerRef:Function

+ innerRef: RefObject

### NavLink的API

NavLink组件是Link的特殊类型，我们可以尝试用NavLink代替上面的Link组件，会发现当当前的路由匹配的时候，它会把匹配的那个NavLink添加一个active样式

```html
<a aria-current="page" class="active" href="/about">About</a>
```

+ activeClassName:string 当元素处于active时给该元素设置的class，默认给定的class是active的，这将与className属性连接在一起。

```jsx
<NavLink to="/test" activeClassName="test">
  test
</NavLink>
```

+ activeStyle:object 元素处于active状态时应用于该元素的样式。

```jsx
<NavLink
  to="/test"
  activeStyle={{
    fontWeight: "bold",
    color: "red"
  }}
>
  test
</NavLink>
```

+ exact:bool 如果为true，则仅在locatiuon完全匹配时才应用active的class或style。

+ strict:bool 如果为true，则在确认当前的to属性的值与浏览器URL匹配时，将会考虑to属性上的斜杠，就是路由匹配的严格模式

如下的NavLink点击之后无法匹配`<Route path={'/test/'} strict children={<Test />} />`

```jsx
<NavLink strict to="/test">
  Test
</NavLink>
<Switch>
  <Route path={'/test/'} strict children={<Test />} />
</Switch>
```

+ isActive: func 一种额外的逻辑确认当前的链接是否处于active状态

+ aria-current: string 在active链接上使用的aria-current属性的值

### prompt

在用户离开页面之前提示

```jsx
import { Prompt } from 'react-router-dom';
<Route path={'/about'}>
  <About />
  <Prompt message="您确定要离开吗？" when={true} ></Prompt>
</Route>
```

prompt相关的API

1. message：string

2. message: func

3. when:true 什么时候渲染提示框

### Redirect

将会导航到新的位置，新的位置将覆盖历史记录栈中的当前位置

简单使用一下：

```jsx
<Redirect to="/dashboard" />
```

相关的API如下

+ to:string

要重定向到的url

+ to:object

```jsx
<Redirect to={{pathname: '/test-link-bbject', search: '?id=1', state: { referrer: '123' }}}></Redirect>
```

+ push:bool

是否将当前的URL推入历史记录

+ from:string

要重定向的路径名

```jsx
<Redirect from="/users/:id" to="/test-link-bbject/:id"></Redirect>
```

+ exact:bool

完全匹配

+ strict:bool

严格匹配

+ sensitive:bool

区分大小写

### Route

+ component

```jsx
<Route path={'/about'} component={Dashboard}/>
```

+ render:func

```jsx
<Route path="/home" render={props => <div>text</div>} />
```

+ children:func

```jsx
<Route path="/home" children={props => <div>text</div>} />
```

+ path:string

+ strict: bool 严格匹配

+ location：string 传入一个路径与`<Route>`的path进行匹配

+ sensitive:bool 区分大小写

### Router

所有路由器组件的通用底层接口，应该使用高级的组件代替它

+ BrowserRouter

+ HashRouter

+ MemoryRouter

+ NativeRouter

+ StaticRouter

API：

+ history:object 用于导航的history对象

```jsx
<Router history={customHistory} />
```

+ children：node

### StaticRouter

永远不会变更位置的router

+ basename:string

+ location

+ context

+ children

### Switch

渲染与位置匹配的第一个子元素

+ location:object

+ children

`<Switch>`的所有子元素，应该都是`<Route>`或者`<Redirect>`元素，`<Route>`元素使用其path属性进行匹配，而`<Redirect>`元素使用其from属性进行匹配。没有path属性的`<Route>`或没有from属性的`<Redirect>`将始终与当前位置匹配。

如果给`<Switch>`一个location属性，它将覆盖匹配的子元素上的location属性。

### history

历史记录对象

history对象通常具有以下属性和方法：

+ length -（number）历史记录堆栈中的条目数
+ action - (string)当前操作（PUSH，REPLACE或POP）
+ location - (object)当前位置。可能具有以下属性：
+ pathname - (string)URL的路径
+ search - (string)URL查询字符串
+ hash - (string)URL哈希片段
+ state - (object)提供给例如当此位置被压入堆栈时，push（path，state）。仅在browser和memory history中可用。
+ push(path, [state]) - (function)将新条目推入历史记录堆栈
+ replace(path, [state]) - (function)替换历史记录堆栈上的当前条目
+ go(n) - (function)将历史记录堆栈中的指针移动n个条目
+ goBack() - (function)相当于go(-1)
+ goForward() - (function)相当于go(1)
+ block(prompt) - (function)防止导航（请参阅history文档）

history是可变的

history对象是可变的，因此，建议从`<Route>`的渲染属性中访问location，而不是从history.location中访问。这确保了您对React的假设在生命周期钩子中是正确的

### location

location表示该应用程序现在的位置，console log打印出来应该是长这个样子的

```javascript
{
  key: 'ac3df4', // not with HashHistory!
  pathname: '/somewhere',
  search: '?some=search-string',
  hash: '#howdy',
  state: {
    [userDefined]: true
  }
}
```

router将在几个地方为您提供location对象：

+ Route component 为 this.props.location
+ Route render 为 ({ location }) => ()
+ Route children 为 ({ location }) => ()
+ withRouter 为 this.props.location

您可以将location传递给以下组件：

+ Route
+ Switch

这样可以防止他们在路由器状态下使用实际位置。这对于动画和待处理的导航很有用，或者在您想要诱使组件在与真实位置不同的位置进行渲染时，这很有用。

### match

match对象包含有关`<Route path>`如何与URL匹配的信息。

## 设计思路

react-router V4和V5的设计思路有所不同，V4使用的是静态路由，V5使用的是动态路由

## 响应式路由

## Hooks

React Router附带了一些钩子，可让您访问路由器的状态并从组件内部执行导航。

### useHistory

useHistory钩子使您可以访问可用于导航的history实例。

修改上面的Home组件

```jsx
import { BrowserRouter, Switch, Route, Link, NavLink, useHistory } from 'react-router-dom';

function Home(props) {
  const history = useHistory();
  const click = () => {
    history.push('/about')
  }
  return (
    <>
      <h1>home</h1>
      <nav>
        <ul>
          <li>
            <NavLink to={'/about'}>About</NavLink>
          </li>
          <li>
            <NavLink to={'/users'}>Users</NavLink>
          </li>
        </ul>
      </nav>
      <button onClick={click}>click me</button>
      {props.children}
    </>
  )
}
```

除了history.push()方法，还有history.go()， history.goBack()等

### useLocation

useLocation返回代表当前URL的location对象。您可以像useState一样使用它，只要URL更改，它就会返回一个新位置。

暂时还不知道在什么场景下使用，只知道一般是和useEffect()一起使用

### useParams

返回URL的key/value对象

修改一下上面的About组件，使用场景还是比较常见的

```jsx
import { useParams } from 'react-router-dom';
function About(props) {
  const { id } = useParams();
  return <h2>我是About， ID：{id}</h2>;
}

// 配置路由的时候要这样子配，useParams不能拿到？问号后面的值
<Route path={'/about/:id'}>
    <About />
</Route>
```

### useRouteMatch

useRouteMatch钩子尝试以与`<Route>`相同的方式匹配当前URL。它主要用于在不实际渲染`<Route>`的情况下访问匹配数据。