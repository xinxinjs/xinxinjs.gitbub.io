---
title: React Hooks 数据流
date: 2020-08-04 10:56:34
tags: [react, hooks]
---

## 单组件数据流

使用useState，毫无争议

```jsx
function App() {
  const [count, setCount] = useState();
}
```

## 组件间共享数据流

我们首先想到的就是useContext

```tsx
// 创建Context
const CountContext = createContext<any>({});

function Index() {
  let [count, setCount] = useState(1);

  function add() {
    setCount(++count);
  }
  return (
    // 通过父级组件分发
    <CountContext.Provider value={{ count, setCount }}>
      <Card title="createContext" style={{marginBottom: 20}} type="inner">
        <Child />
        <Child2 />
      </Card>
    </CountContext.Provider>
  )
}

function Child() {
  let { count, setCount } = useContext(CountContext);

  function add() {
    setCount(--count)
  }

  return (
    <>
      <p>count:{count}</p>
      <Button onClick={add}>-</Button>
    </>
  )
}
function Child2() {
  let { count, setCount } = useContext(CountContext);

  function add() {
    setCount(count - 2)
  }

  return (
    <>
      <p>count:{count}</p>
      <Button onClick={add}>-2</Button>
    </>
  )
}
```

这个问题是数据与 UI 不解耦，我看到这里的时候，我也不是很明白，我感觉这个很好用，然后接着往下看

## 数据流与组件解耦 hooks state + unstated-next

unstated-next 是一个React轻量状态管理库，可以帮你把上面例子中，定义在 组件 中的数据单独出来，形成一个自定义数据管理 Hook：

[unstated-next](https://github.com/jamiebuilds/unstated-next)

简单的API介绍

1. unstated-next提供 const Counter = createContainer(useCounter)方法， 创建一个状态管理类，相当于一个容器

2. Counter.Provider，包裹共享的数据，向应用中注入状态管理实例，

3. Counter.useContainer，在子组件使用共享的数据

```tsx
import { createContainer } from "unstated-next";

// 自定义数据管理 Hook
function useCounter() {
  const [state, setState] = useState<any>({a: 100, b: 200});
  return { ...state, setState };
}
const Counter = createContainer(useCounter);

function Index() {
  return (
    <Counter.Provider>
      <CounterDisplay />
    </Counter.Provider>
  )
}

function CounterDisplay() {
  let { a, b, setState } = Counter.useContainer();

  function changeA() {
    setState({a: a + 1});
  }

  function changeB(){
    setState({b: b + 1, a: a + 2});
  }

  return (
    <Card title="unstated-next" style={{marginBottom: 20}} type="inner">
      <p>a:{a}</p>
      <Button onClick={changeA}>A加一</Button>
      <p>b:{b}</p>
      <Button onClick={changeB}>A加2，B加2</Button>
    </Card>
  )
}
```

这种方法解决了数据流与组件解耦，但是带来了新的问题，就是useState 无法合并更新

## 合并更新

为了让数据能够合并更新，我想到了useReducer 可以让数据合并更新，现在的思路就是：

hooks state + unstated-next + useReducer

```tsx
import { createContainer } from "unstated-next";

function useCounterR(){
  const [state, dispatch] = useReducer(reducer, {a: 100, b: 200});

  function reducer(state: any, action: any) {
    switch (action.type) {
      case 'add': 
        return {...state, a: state.a + 2};
      case 'dec':
        return {...state, a: state.a - 10};
      case 'double':
        return {...state, b: state.b * 2};
      case 'triple':
        return {...state, b: state.b * 3};
      default:
        return state;
    }
  }
  // 返回state和修改state的dispatch
  return { ...state, dispatch };
}
const CounterR = createContainer(useCounterR);

function Index() {
  return (
    <CounterR.Provider>
      <ViewR />
      <ViewR2 />
    </CounterR.Provider>
  )
}

function ViewR() {
  let {a, b, dispatch} = CounterR.useContainer();

  console.log('ViewR', a, b);

  return (
    <Card title="reducer" style={{marginBottom: 20}} type="inner">
      <p>a: {a}</p>
      <p>b:{b}</p>
      <Button onClick={() => dispatch({type: 'add'})}>
        a + 2
      </Button>
      <Button onClick={() => dispatch({type: 'triple'})}>
        b乘以3
      </Button>
    </Card>
  )
}

function ViewR2() {
  let {b, dispatch} = CounterR.useContainer();

  console.log('ViewR2', b);

  return (
    <Card title="reducer" style={{marginBottom: 20}} type="inner">
      <p>b:{b}</p>
      <Button onClick={() => dispatch({type: 'triple'})}>
        b乘以3
      </Button>
    </Card>
  )
}
```

现在终于能合并更新了，然后我发现了新的问题，就是子组件之间会相互影响，一个子组件修改了state，其他的子组件会一起重新渲染

这个原因是 Counter.useContainer 提供的数据流是一个引用整体，其子节点 引用变化后会导致整个 Hook 重新执行，继而所有引用它的组件也会重新渲染。

## 按需更新

所以我知道了redux为什么会比他们火的原因了

可以利用 Redux useSelector 实现按需更新

```tsx
import { createStore } from "redux";
import { Provider, useSelector, shallowEqual } from "react-redux";

const defaultState = {a: 100, b: 200, user: {name: 'xinxin'}};

function reducer(state: any = defaultState, action: any) {
  switch (action.type) {
    case 'add': 
      return {...state, a: state.a + 2};
    case 'dec':
      return {...state, a: state.a - 10};
    case 'double':
      return {...state, b: state.b * 2};
    case 'triple':
      return {...state, b: state.b * 3};
    case 'setUser':
      return {...state, user: {name: 'xinxin222'}};
    default:
      return state;
  }
}

const store = createStore(reducer);

function Index() {
  return (
    <Provider store={store}>
      <ViewR3 />
      <ViewR4 />
    </Provider>
  )
}

function ViewR3() {
  // useSelector 接受第一个参数selector函数
  // 要保持这个selector是一个纯函数
  // selector会返回任何值作为结果，并不仅仅是对象了。然后这个selector返回的结果，就会作为useSelector的返回结果。
  // 当action被dispatched的时候，useSelector()将对前一个selector结果值和当前结果值进行浅比较。如果不同，那么就会被re-render。 反之亦然
  // selector不会接收ownProps参数，但是，可以通过闭包(下面有示例)或使用柯里化selector来使用props。
  // 使用记忆(memoizing) selector时必须格外小心(下面有示例)。
  // useSelector()默认使用===(严格相等)进行相等性检查，而不是浅相等(==)。
  const { a } = useSelector(
    (state:any = {}) => ({ a: state.a }),
    shallowEqual
  );

  const setA = (a: number) => a + 2;

  console.log(1);
  return (
    <Card title="redux" style={{marginBottom: 20}} type="inner">
      <p>a: {a}</p>
      <Button onClick={() => store.dispatch({type: 'add', setA})}>
        a + 2
      </Button>
    </Card>
  )
}

function ViewR4() {
  const { b } = useSelector(
    (state:any = {}) => ({ b: state.b }),
    shallowEqual
  );

  const setB = (b: number) => b + 3;

  console.log(2);
  return (
    <Card title="redux" style={{marginBottom: 20}} type="inner">
      <p>b: {b}</p>
      <Button onClick={() => store.dispatch({type: 'setB', setB})}>
        b + 3
      </Button>
    </Card>
  )
}
```

reducer 可以让子组件之间会相互不影响，一个子组件修改了state，其他的子组件不会一起重新渲染

然后有发现了一个新的问题：但 useSelector 的作用仅仅是计算结果不变化时阻止组件刷新，但并不能保证返回结果的引用不变化。

当state的值是引用类型的时候，每次都会返回一个新的引用，虽然属性值和数量都没有发生变化，但是还是会引起组件重新渲染，如果有子组件也会重新渲染

```tsx
function Index() {
  return (
    <Provider store={store}>
      <ViewR5 />
    </Provider>
  )
}

function ViewR5() {
  const [myUser, setMyUser] = useState();
  const {user = {}} = useSelector(
    (state: any) => ({user: state.user}),
    shallowEqual
  );

  useEffect(() => {
    console.log(myUser, user, Object.is(myUser, user));
    setMyUser(user);
  }, [user])

  const setUserName = (user: any) => ({name: 'xinxin 22222'});
  return (
    <Card title="使用Redux，防止数据引用频繁变化" style={{marginBottom: 20}} type="inner">
      <p>userName: {user.name}</p>
      <Button onClick={() => store.dispatch({type: 'setUser', setUserName})}>
        setUserName
      </Button>
    </Card>
  )
}
```

这段代码console.log(myUser, user, Object.is(myUser, user));打印出来每次myUser, user这两个对象的属性数量和属性值都没发生变化，但是Object.is判断返回false，因为他们不是来自同一个引用类型的

## 防止数据引用频繁变化

为了防止数据引用频繁变化而带来的组件重新渲染，浅比较shallowEqual是不起作用的，尝试使用deepEqual

```tsx
function ViewR6() {
  const {user = {}} = useSelector(
    (state: any) => ({user: state.user}),
    deepEqual
  );

  const setUserName = (user: any) => ({name: 'xinxin 22222'});
  return (
    <Card title="使用Redux，防止数据引用频繁变化" style={{marginBottom: 20}} type="inner">
      <p>userName: {user.name}</p>
      <RChild name={user.name}  />
      <Button onClick={() => store.dispatch({type: 'setUser', setUserName})}>
        setUserName
      </Button>
    </Card>
  )
}
function RChild(props) {
  console.log(props.name);
  return <p>{props.name}</p>
}
```

运行上面的代码，发现组件不在重新渲染了，现在的问题是这段代码我们拿到的还是新的引用，只不过我们通过deepEqual对前后两次的值进行深比较，在useSelector不在返回新的引用

```ts
const {user = {}} = useSelector(
  (state: any) => ({user: state.user}),
  deepEqual  
);
```

如果换成通过机构计算函数去获取状态，像这样子

```ts
function getUser(user: any) {
  if(!Object.is(myUser, user)) {
    console.log(myUser, user, Object.is(myUser, user)); // false
    myUser = user;
  }
  return user;
}

const {user = {}} = useSelector(
  (state: any) => ({user: getUser(state.user)}),
  deepEqual  
);
```

运行代码，证明console.log(myUser, user, Object.is(myUser, user));会打印出false，说明这里计算函数不管每次接收到的参数是不是一样，每次接收到的都是一个新的引用，所以都会进行重新计算，如果计算函数是一个很复杂，消耗资源的函数，在拿到的引用类型属性值和数量是一样的时候不应该去重新计算

所以给大家介绍一个reselect 中间件，可以对引用进行缓存

## 总结

1. 了解react hooks 数据流

2. 单组件数据流，使用useState

3. 共享数据流，useContext，数据与 UI 不解耦

4. 为了数据流与组件解耦， hooks state + unstated-next， 带来了新的问题，就是useState 无法合并更新

5. 为了让数据能够合并更新，我想到了useReducer 可以让数据合并更新，问题是：子组件之间会相互影响

6. 为了让子组件不相互影响，redux按需更新， 新问题：useSelector 的作用仅仅是计算结果不变化时阻止组件刷新，但并不能保证返回结果的引用不变化。引用变化了子组件会更新

7. 防止数据引用频繁变化， deepEqual解决引用频繁变化

8. 发现新的问题，计算函数消耗资源，reselect 中间件引用进行缓存
