---
layout: interview
title: interView - React
date: 2021-01-07 14:25:17
tags: [interview,题,React]
---

### 1、`setState`

#### 1、异步还是同步

- 合成事件和钩子函数中 是异步的
- 原生事件 和 setTimeout 中是同步
相关链接：[你真的理解setState吗？](https://juejin.cn/post/6844903636749778958)

#### 2、调用 setState 后的过程

在 `setState` 的时候，`React` 会为当前节点创建一个 `updateQueue` 的更新列队。
然后会触发 `reconciliation` 过程，在这个过程中，会使用名为 `Fiber` 的调度算法，开始生成新的 `Fiber` 树， `Fiber` 算法的最大特点是可以做到异步可中断的执行。
然后 `React Scheduler` 会根据优先级高低，先执行优先级高的节点，具体是执行 `doWork` 方法。
在 `doWork` 方法中，`React` 会执行一遍 `updateQueue` 中的方法，以获得新的节点。然后对比新旧节点，为老节点打上 更新、插入、替换 等 `Tag`。
当前节点 `doWork` 完成后，会执行 `performUnitOfWork` 方法获得新节点，然后再重复上面的过程。
当所有节点都 `doWork` 完成后，会触发 `commitRoot` 方法，`React` 进入 `commit` 阶段。
在 `commit` 阶段中，`React` 会根据前面为各个节点打的 `Tag`，一次性更新整个 `dom` 元素。

### 2、生命周期

#### 1、钩子函数

> [生命周期图谱](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

- v16.0 之前:
  - 第一阶段: 组件挂载阶段 `constructor` => `componentWillMount` => `render` => `componentDidMount`
  - 第二阶段: 组件更新阶段 分为两种情况：
    - New props: `componentWillReceiveProps` => `shouldComponentUpdate` => `componentWillUpdate` => `render` => `componentDidUpdate`
    - set­State(): `shouldComponentUpdate` => `componentWillUpdate` => `render` => `componentDidUpdate`
  - 第三阶段: 组件卸载阶段 `componentWillUnmount`
- v16.0 - v16.3 :
  - 第一阶段: 组件挂载阶段 `constructor => getDerivedStateFromProps => render => componentDidMount`
  - 第二阶段: 组件更新阶段
    - New props: `getDerivedStateFromProps => shouldComponentUpdate => render => getSnapshotBeforeUpdate => componentDidUpdate`
    - set­State(): `shouldComponentUpdate => render => getSnapshotBeforeUpdate => componentDidUpdate`
    - force­Update(): `render => getSnapshotBeforeUpdate => componentDidUpdate`
  - 第三阶段: 组件卸载阶段 `componentWillUnmount`
- v16.4 之后:
  - 第一阶段: 组件挂载阶段 `constructor => getDerivedStateFromProps => render => componentDidMount`
  - 第二阶段: 组件更新阶段 `getDerivedStateFromProps => shouldComponentUpdate => render => getSnapshotBeforeUpdate => componentDidUpdate`
  - 第三阶段: 组件卸载阶段 `componentWillUnmount`

**`React < V16.0`的生命周期为**
![React _ 14 lifeCycle.png](https://i.loli.net/2021/03/01/oxNYw1WzvXpcLym.png)

**`React V16.0 - V16.3`的生命周期为**
![React 16.3lifeCycle.png](https://i.loli.net/2021/03/01/bdk62qIZhVNPDvn.png)

**`React V16.4+`的生命周期为**
![React V16.4+生命周期](https://i.loli.net/2020/12/09/gEyt6U785HqOIVd.png)

#### 2、新增/删除哪些钩子

废弃了三个钩子函数: ~~`componentWillMount`~~、~~`componentWillReceiveProps`~~、~~`componentWillUpdate`~~
新增了两个钩子函数: `static getDerivedStateFromProps`、`getSnapshotBeforeUpdate`

- `static getDerivedStateFromProps`: 这个生命周期的功能实际上就是将传入的 `props` 映射到 `state` 上面, 为了替代`componentWillReceiveProps`而存在, 静态方法, 所以不能通过 `this` 访问到 `class` 的属性。

```javaScript
  // 它返回一个对象表示新的 state。如果不需要更新组件，返回 null 即可
  // 参数是组件接收到的新的 props(nextProps) 和组件当前的state数据(prevState)
  static getDerivedStateFromProps (nextProps, prevState) {
      // 当 props 的值 发生变化时，更新state
      if (nextProps.color !== prevState.color) {
          return {
              color: nextProps.color,
          };
      }
      return null; // 否则，对于state不进行任何操作
  }
  // 与 componentWillReceiveProps 触发机制的区别:
  // 自身组件 setState 时, getDerivedStateFromProps 会被调用,而 componentWillReceiveProps 不会
  //  - getDerivedStateFromProps: 在每次组件被重新渲染前被调用.
  //     这意味着无论是父组件的更新, props 的变化, 或是组件内部执行了 setState(), 它都会被调用.
  //  - componentWillReceiveProps: 接收到新的参数时被触发
  //     当父组件导致子组件更新的时候, 即使接收的 props 并没有变化, 这个函数也会被调用

```

- `getSnapshotBeforeUpdate`: 在 `React` 对视图做出实际改动（如 DOM 更新）发生前被调用，返回值将作为 `componentDidUpdate` 的第三个参数。
  `getSnapshotBeforeUpdate` 配合 `componentDidUpdate` 可以取代 `componentWillUpdate`.

废除三个钩子函数的原因：

- 为何移除 ~~`componentWillReceiveProps`~~ : 并不是因为功能问题，而是性能问题。当外部多个属性在很短的时间间隔之内多次变化，就会导致`componentWillReceiveProps`被多次调用。这个调用并不会被合并，如果这次内容都会触发异步请求，那么可能会导致多个异步请求阻塞。
- 为何移除 ~~`componentWillMount`~~: 更新后 React 的异步渲染机制可能会导致单个组件实例可以多次调用该方法,很多开发者目前会将事件绑定、异步请求等写在 `componentWillMount` 中，一旦异步渲染时 `componentWillMount` 被多次调用
  - 进行重复的时间监听，无法正常取消重复的 `Listener`，更有可能导致内存泄漏
  - 发出重复的异步网络请求，导致 IO 资源被浪费
  - 在服务端渲染时,`componentWillMount` 会被调用，但是会因忽略异步获取的数据而浪费 IO 资源
- 为何移除 ~~`componentWillUpdate`~~: 大多数开发者使用 `componentWillUpdate` 的场景是配合 `componentDidUpdate`,分别获取 `rerender` 前后的视图状态，进行必要的处理。但随着 React 新的 `suspense`、`time slicing`、异步渲染等机制的到来，`render` 过程可以被分割成多次完成，还可以被暂停甚至回溯，这导致 `componentWillUpdate` 和 `componentDidUpdate` 执行前后可能会间隔很长时间，足够使用户进行交互操作更改当前组件的状态. 而 `getSnapshotBeforeUpdate` 方法是在 `componentWillUpdate` 后，并且它调用的结果会作为第三个参数传入 `componentDidUpdate`，数据用完即被销毁，效率更高。
  
[React 生命周期 我对 React v16.4 生命周期的理解](https://juejin.cn/post/6844903655372488712)
[谈谈新的 React 新的生命周期钩子](https://imweb.io/topic/5b8cacaa7cd95ea863193572)

### 3、Hooks

#### 1 `useEffect(fn, [])` 和 `componentDidMount` 的什么差异

`useEffect` 会捕获 `props` 和 `state`，形成闭包。所以即便在回调函数里，你拿到的还是初始的 `props` 和 `state`。
如果想得到“最新”的值，可以使用 `ref`（`useRef`）。

#### 2、为什么不能放在条件判断里？

以 `setState` 为例，在 `react` 内部，每个组件(`Fiber`)的 `hooks` 都是以链表的形式存在 `memoizeState` 属性中：

<img src="https://i.loli.net/2021/02/02/Rvz1y4EKVZwGCM6.jpg" >

`update` 阶段，每次调用 `setState`，链表就会执行 `next` 向后移动一步。如果将 `setState` 写在条件判断中，假设条件判断不成立，没有执行里面的 `setState` 方法，会导致接下来所有的 `setState` 的取值出现偏移，从而导致异常发生。

参考链接：[烤透 React Hook](https://juejin.cn/post/6867745889184972814)

### 4、组件

#### （1）函数组件

- 纯函数： 输入props，输出JSX
- 没有实例，没有生命周期，因此没有state
- 不能扩展其他方法

#### （2）受控组件 和 非受控组件

  1> 在 constructor 中创建 ref
  `this.fileInputRef = React.createRef();`
  2> 非受控组件
  `<input type="file" ref={this.fileInputRef} />`
  3> 通过ref获得DOM节点，完成需要的操作
  `const elem = this.fileInputRef.current;`
  `alert(elem.files[0].name);`

使用场景：

- 必须手动操作DOM元素，setState实现不了
- 比如文件上传 `<input type=file>`
- 某些富文本编辑器，需要传入DOM元素

如何选择：

- 优先使用受控组件，符合React设计原则 （数据驱动视图）
- 必须操作DOM的时候，再使用非受控组件

#### （3）Portals

Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。
`ReactDOM.createPortal(child, container)`

Portals 使用场景

- overflow: hidden
- 父组件 z-index 值太小
- fixed 需要放在body的第一层

#### （4）异步组件

1. import()
2. React.lazy
  通过React.lazy()方法 对import引入的组件进行懒加载
  `const ContextDemo = React.lazy(() => import('./ContextDemo'))`
3. React.Suspense
  React.Suspense 使用懒加载引入的组件
  `<React.Suspense fallback={<div>Loading...</div>}>`
    `<ContextDemo/>`
  `</React.Suspense>`

### 5、 高阶组件

高阶组件不是一种功能，而是一种模式 HOC、Render Props、redux connect

（1）HOC （使用了高阶函数的思想）

（2）Render Props

  <img src="https://i.loli.net/2020/12/30/iLBsy15Y6ZTP9Gu.png" >

  HOC与 Render Props 的对比：

- HOC： 模式简单，但是会增加组件层级
- Render Props： 代码简洁，学习成本较高

（3）redux connect

  react-redux 的核心组件只有两个：Provider组件 和 connect
  **`Provider`** 存放 Redux 里 store 的数据到 context 里，通过 connect 从 context 拿数据，通过 props 传递给 connect 所包裹的组件。
 `connect([mapStateToProps], [mapDispatchToProps], [mergeProps],[options])`
  作用：连接React组件与 Redux store。
  **`connect`** 是一个高阶函数，首先传入 `mapStateToProps`、`mapDispatchToProps`，然后返回一个生产`Component`的函数(`wrapWithConnect`)，然后再将真正的`Component`作为参数传入`wrapWithConnect`，这样就生产出一个经过包裹的`Connect`组件

### 6、Redux

（1）单向数据流：

1. dispatch(action): 通过 dispatch 一个 action 改变 state 的值（组件改变state的唯一方法）
2. reducer->newState: 通过 reducer 生成一个新的 state
3. subscribe 触发通知: 通过 subscribe 触发通知，进行视图的更新

<img src="https://i.loli.net/2020/12/30/1XfzRaJ8dYcnT6x.png" >

中间件的调用顺序：

<img src="https://i.loli.net/2020/12/30/luqEbTaCHQL3Iwf.png" >

redux-saga基本用法总结：

1. 使用 createSagaMiddleware 方法创建 saga 的 Middleware ，然后在创建的 redux 的 store 时，使用 applyMiddleware 函数将创建的 saga Middleware 实例绑定到 store 上，最后可以调用 saga Middleware 的 run 函数来执行某个或者某些 Middleware 。
2. 在 saga 的 Middleware 中，可以使用 takeEvery 或者 takeLatest 等 API 来监听某个 action ，当某个 action 触发后， saga 可以使用 call 发起异步操作，操作完成后使用 put 函数触发 action ，同步更新 state ，从而完成整个 State 的更新。

redux 有哪些异步中间件:

1. redux-thunk - 源代码简短优雅，上手简单

2. redux-saga - 借助 JS 的 generator 来处理异步，避免了回调的问题

3. redux-observable - 借助了 RxJS 流的思想以及其各种强大的操作符，来处理异步问题

（2）Redux 的发布订阅模式：

1. subscribe 订阅：Redux 通过 subscribe 接口注册订阅函数，并将这些用户提供的订阅函数添加到闭包中的 nextListeners 中
2. dispatch 发布：通过 Redux 的 dispatch 接口，我们可以发布一个 action 对象，去通知状态需要做一些改变

Redux 与 发布订阅模式的区别：
redux 其实也是一个发布订阅，但是 redux 可以做到数据的可预测和可回溯

（3）redux 与 react 关联

react-redux 的核心组件只有两个，Provider 和 connect，Provider 存放 Redux 里 store 的数据到 context 里，通过 connect 从 context 拿数据，通过 props 传递给 connect 所包裹的组件。

（4）Redux、mobx、Vuex：

Redux 与 Mobx：

- Redux面向函数式编程，Mobx面向对象
- Redux单一数据源（只有一个store），而 Mobx 容许多个
- Redux的状态对象不可直接修改，而Mobx可以
- Mobx更自由，更容易上手，Redux约束更多，有一定学习成本
- 自由需要代价，Mobx在更复杂的应用中将难管理，更难维护

Redux 与 Vuex：

- Redux： view——>actions——>reducer——>state变化——>view变化（同步异步一样）
- Vuex-同步： view——>commit——>mutations——>state变化——>view变化（同步操作）
- Vuex-异步：view——>dispatch——>actions——>mutations——>state变化——>view变化（异步操作）

### 7、fiber

react 在进行组件渲染时，从setState 开始到渲染完成整个过程是同步的。
如果需要渲染的组件比较庞大，js执行会占据主线程时间较长，会导致页面响应度变差，使得react在动画、手势等应用中效果比较差，出现卡顿的效果。

为了解决这个问题，React团队重写了react中核心算法。之前的reconciler['rekənsailə] 称为stack reconciler，重写后的称为fiber reconciler，简称为Fiber。

`React Fiber` 是一种基于浏览器的单线程调度算法。

将 客户端线程执行任务 以帧的形式划分（切片），大部分设备控制在30-60帧是不会影响用户体验；在两个执行帧之间，主线程通常会有一小段空闲时间，`requestIdleCallback`可以在这个空闲期（`Idle Period`）调用空闲期回调（`Idle Callback`），执行一些任务.

- 低优先级任务由 `requestIdleCallback` 处理；
- 高优先级任务，如动画相关的由 `requestAnimationFrame` 处理；
- `requestIdleCallback` 可以在多个空闲期调用空闲期回调，执行任务；
- `requestIdleCallback` 方法提供 `deadline` ，即任务执行限制时间，以切分任务，避免长时间执行，阻塞UI渲染而导致掉帧；

这就可能会产生两个问题：

- 饿死：正在实验中的方案是重用，也就是说高优先级的操作如果没有修改低优先级操作已经完成的节点，那么这部分工作是可以重用的。
- 一次渲染可能会调用多次声明周期函数

`React Fiber` 用类似 `requestIdleCallback` 的机制来做异步 `diff`。但是之前数据结构不支持这样的实现异步 `diff`，于是 `React` 实现了一个类似链表的数据结构，将原来的 **递归diff** 变成了现在的 **遍历diff**，这样就能做到异步可更新了。

<img src="https://i.loli.net/2021/02/02/gweAjQS5osflqX1.jpg" >

相关链接：[React Fiber 是什么？](https://github.com/WangYuLue/react-in-deep/blob/main/02.React%20Fiber%20%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F.md)
[React Fiber](https://juejin.cn/post/6844903582622285831)

### 8、diff 算法

将两颗树中所有的节点一一对比需要`O(n^2)`的复杂度。
再紧接着找到不同之后，再计算最短修改距离然后修改（添加、删除）节点，复杂度是`O(n)`.
合起来diff两个树的复杂度就是`O(n^3)`

传统 `diff` 算法的时间复杂度是 `O(n^3)`，这在前端 `render` 中是不可接受的。
为了降低时间复杂度，`react` 的 `diff` 算法做了一些妥协，放弃了最优解，最终将时间复杂度降低到了 `O(n)`。

`vue` 的更新策略就是：深度优先、同层比较。就是只比较同层级，也就是 `O(n)`

那么 `react diff` 算法做了哪些妥协呢？，参考如下：

1、`tree diff`：只对比同一层的 `dom` 节点，忽略 `dom` 节点的跨层级移动

如下图，`react` 只会对相同颜色方框内的 `DOM` 节点进行比较，即同一个父节点下的所有子节点。当发现节点不存在时，则该节点及其子节点会被完全删除掉，不会用于进一步的比较。

这样只需要对树进行一次遍历，便能完成整个 `DOM` 树的比较。

<img src="https://i.loli.net/2021/02/02/j79gK3iQVwWyENr.jpg" >

这就意味着，如果 `dom` 节点发生了跨层级移动，`react` 会删除旧的节点，生成新的节点，而不会复用。

2、`component diff`：如果不是同一类型的组件，会删除旧的组件，创建新的组件

因此， React 允许用户通过 `shouldComponentUpdate()` 来判断该组件是否需要大量 `diff` 算法分析。

<img src="https://i.loli.net/2021/02/02/YSNq9gemUvCKV5G.jpg" >

3、`element diff`：对于同一层级的一组子节点，需要通过唯一 `id`(即`key`') 进行来区分
当节点处于同一层级时，diff 提供三种节点操作：插入、移动、删除
如果没有 `id` 来进行区分，一旦有插入动作，会导致插入位置之后的列表全部重新渲染。

这也是为什么渲染列表时为什么要使用唯一的 `key`。

Vue 和 React 的 diff 算法有什么区别：
(1) Vue进行diff时，调用patch打补丁函数，一边比较一边给真实的DOM打补丁
(2) 当对比 `element` (元素) 发现 `className` 不同时，Vue 认为是不同类型元素，删除重建；React 认为是同类型节点，只修改节点属性
(3) 遍历同一层级的元素进行比较的方式不同：
  Vue 采用从两端到中间的方式，旧集合和新集合两端各存在两个指针，两两进行比较，如果匹配上了就按照新集合去调整旧集合，每次对比结束后，指针向队列中间移动。
  React 是从左往右依次对比，利用元素的index和标识lastIndex进行比较，如果满足index < lastIndex就移动元素，删除和添加则各自按照规则调整；
  当一个集合把最后一个节点移动到最前面，react会把前面的节点依次向后移动，而Vue只会把最后一个节点放在最前面，这样的操作来看，Vue的diff性能是高于react的

Vue 2.x vs Vue 3.x：
（1）Vue2的核心Diff算法采用了双端比较的算法，同时从新旧children的两端开始进行比较，借助key值找到可复用的节点，再进行相关操作。相比React的Diff算法，同样情况下可以减少移动节点次数，减少不必要的性能损耗，更加的优雅。
（2）Vue3.x借鉴了 ivi算法和 inferno算法（运用了动态规划的思想求解 最长递归子序列 ）。在创建VNode时就确定其类型，以及在mount/patch的过程中采用位运算来判断一个VNode的类型，在这个基础之上再配合核心的Diff算法，使得性能上较Vue2.x有了提升。

### 9、 React性能优化

1、shouldComponentUpdate

2、PureComponent 和 memo

- PureComponent(纯组件) shouldComponentUpdate 中实现了浅比较
- memo 函数组件中的 PureComponent
- 浅比较已经适用大部分场景（尽量不要做深度比较）

3、immutable.js

- 彻底拥抱**不可变值**
- 基于共享数据（不是深拷贝），速度好
- 缺点是 有一定的学习和迁移成本，按需使用

### 10、错误边界是什么？它有什么用？

在 `React` 中，如果任何一个组件发生错误，它将破坏整个组件树，导致整页白屏。这时候我们可以用错误边界优雅地降级处理这些错误。

例如下面封装的组件：

```javascript
class ErrorBoundary extends React.Component<IProps, IState> {
  constructor(props: IProps) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 可以将错误日志上报给服务器
    console.log('组件奔溃 Error', error);
    console.log('组件奔溃 Info', errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return this.props.content;
    }
    return this.props.children;
  }
}
```

### 10、什么是 Portals？

`Portal` 提供了一种将子节点渲染到存在于父组件以外的 `DOM` 节点的优秀的方案。

`ReactDOM.createPortal(child, container)`

### 11、React 组件间有那些通信方式?

父组件向子组件通信
1、 通过 `props` 传递

子组件向父组件通信
1、 主动调用通过 `props` 传过来的方法，并将想要传递的信息，作为参数，传递到父组件的作用域中

跨层级通信
1、 使用 `react` 自带的 `Context` 进行通信，`createContext` 创建上下文， `useContext` 使用上下文。

参考下面代码：

```javascript
import React, { createContext, useContext } from 'react';

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

const ThemeContext = createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}

export default App;
```

2、使用 `Redux` 或者 `Mobx` 等状态管理库

3、使用订阅发布模式

相关链接：[React Docs](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)

### 12、React 父组件如何调用子组件中的方法？

1、如果是在方法组件中调用子组件（>= react@16.8），可以使用 useRef 和 useImperativeHandle:

```javascript
const { forwardRef, useRef, useImperativeHandle } = React;

const Child = forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({
    getAlert() {
      alert("getAlert from Child");
    }
  }));
  return <h1>Hi</h1>;
});

const Parent = () => {
  const childRef = useRef();
  return (
    <div>
      <Child ref={childRef} />
      <button onClick={() => childRef.current.getAlert()}>Click</button>
    </div>
  );
};
```

2、如果是在类组件中调用子组件（>= react@16.4），可以使用 createRef:

```javascript
const { Component } = React;

class Parent extends Component {
  constructor(props) {
    super(props);
    this.child = React.createRef();
  }

  onClick = () => {
    this.child.current.getAlert();
  };

  render() {
    return (
      <div>
        <Child ref={this.child} />
        <button onClick={this.onClick}>Click</button>
      </div>
    );
  }
}

class Child extends Component {
  getAlert() {
    alert('getAlert from Child');
  }

  render() {
    return <h1>Hello</h1>;
  }
}
```

参考阅读：[Call child method from parent](https://stackoverflow.com/questions/37949981/call-child-method-from-parent)

### 13、React有哪些优化性能的手段?

类组件中的优化手段
1、使用纯组件 PureComponent 作为基类。

2、使用 React.memo 高阶函数包装组件。

3、使用 shouldComponentUpdate 生命周期函数来自定义渲染逻辑。

方法组件中的优化手段
1、使用 useMemo。

2、使用 useCallBack。

其他方式
1、在列表需要频繁变动时，使用唯一 id 作为 key，而不是数组下标。

2、必要时通过改变 CSS 样式隐藏显示组件，而不是通过条件判断显示隐藏组件。

3、使用 Suspense 和 lazy 进行懒加载，例如：
/ [səˈspens]/ 悬念；悬疑；焦虑；悬而不决

```javascript
import React, { lazy, Suspense } from "react";

export default class CallingLazyComponents extends React.Component {
  render() {
    var ComponentToLazyLoad = null;

    if (this.props.name == "Mayank") {
      ComponentToLazyLoad = lazy(() => import("./mayankComponent"));
    } else if (this.props.name == "Anshul") {
      ComponentToLazyLoad = lazy(() => import("./anshulComponent"));
    }

    return (
      <div>
        <h1>This is the Base User: {this.state.name}</h1>
        <Suspense fallback={<div>Loading...</div>}>
          <ComponentToLazyLoad />
        </Suspense>
      </div>
    )
  }
}
```

Suspense 用法可以参考[官方文档](https://zh-hans.reactjs.org/docs/concurrent-mode-suspense.html)

相关阅读：[21个React性能优化技巧](https://www.infoq.cn/article/KVE8xtRs-uPphptq5LUz)

### 14、为什么 React 元素有一个 $$typeof 属性？

<img src="https://i.loli.net/2021/02/02/9QgBSIZsl3u8Mat.jpg" >

目的是为了防止 XSS 攻击。因为 Synbol 无法被序列化，所以 React 可以通过有没有 $$typeof 属性来断出当前的 element 对象是从数据库来的还是自己生成的。

如果没有 $$typeof 这个属性，react 会拒绝处理该元素。

在 React 的古老版本中，下面的写法会出现 XSS 攻击：

```javascript
// 服务端允许用户存储 JSON
let expectedTextButGotJSON = {
  type: 'div',
  props: {
    dangerouslySetInnerHTML: {
      __html: '/* 把你想的搁着 */'
    },
  },
  // ...
};
let message = { text: expectedTextButGotJSON };

// React 0.13 中有风险
<p>
  {message.text}
</p>
```

相关阅读：[为什么React元素有一个$$typeof属性？](https://overreacted.io/zh-hans/why-do-react-elements-have-typeof-property/)

### 15、React 如何区分 Class组件 和 Function组件？

一般的方式是借助 typeof 和 Function.prototype.toString 来判断当前是不是 class，如下：

```javascript
function isClass(func) {
  return typeof func === 'function'
    && /^class\s/.test(Function.prototype.toString.call(func));
}
```

但是这个方式有它的局限性，因为如果用了 babel 等转换工具，将 class 写法全部转为 function 写法，上面的判断就会失效。

React 区分 Class组件 和 Function组件的方式很巧妙，由于所有的类组件都要继承 React.Component，所以只要判断原型链上是否有 React.Component 就可以了：

AComponent.prototype instanceof React.Component
相关阅读：[React 如何区分 Class 和 Function？](https://overreacted.io/zh-hans/how-does-react-tell-a-class-from-a-function/)

### 16、HTML 和 React 事件处理有什么区别?

在 HTML 中事件名必须小写：

```html
<button onclick='activateLasers()'>
而在 React 中需要遵循驼峰写法：

<button onClick={activateLasers}>
在 HTML 中可以返回 false 以阻止默认的行为：

<a href='#' onclick='console.log("The link was clicked."); return false;' />
而在 React 中必须地明确地调用 preventDefault()：
```

```javascript
function handleClick(event) {
  event.preventDefault()
  console.log('The link was clicked.')
}
```

### 17、什么是 suspense 组件?

Suspense 让组件“等待”某个异步操作，直到该异步操作结束即可渲染。在下面例子中，两个组件都会等待异步 API 的返回值：
/ [səˈspens]/ 悬念；悬疑；焦虑；悬而不决

```javascript
const resource = fetchProfileData();

function ProfilePage() {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails />
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline />
      </Suspense>
    </Suspense>
  );
}

function ProfileDetails() {
  // 尝试读取用户信息，尽管该数据可能尚未加载
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline() {
  // 尝试读取博文信息，尽管该部分数据可能尚未加载
  const posts = resource.posts.read();
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```

Suspense 也可以用于懒加载，参考下面的代码：

```javascript

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

### 18、为什么 JSX 中的组件名要以大写字母开头？

因为 React 要知道当前渲染的是组件还是 HTML 元素。

### 19、redux 是什么？

Redux 是一个为 JavaScript 应用设计的，可预测的状态容器。

它解决了如下问题：

跨层级组件之间的数据传递变得很容易
所有对状态的改变都需要 dispatch，使得整个数据的改变可追踪，方便排查问题。
但是它也有缺点：

概念偏多，理解起来不容易
样板代码太多

### 20、react-redux 的实现原理？

通过 redux 和 react context 配合使用，并借助高阶函数，实现了 react-redux。

参考链接：[React.js 小书](http://huziketang.mangojuice.top/books/react/lesson36)

### 21、reudx 和 mobx 的区别？

得益于 Mobx 的 observable，使用 mobx 可以做到精准更新；对应的 Redux 是用 dispath 进行广播，通过Provider 和 connect 来比对前后差别控制更新粒度；

相关阅读：[Redux or MobX: An attempt to dissolve the Confusion](https://segmentfault.com/a/1190000011148981)

### 22、redux 异步中间件有什么什么作用?

假如有这样一个需求：请求数据前要向 Store dispatch 一个 loading 状态，并带上一些信息；请求结束后再向Store dispatch 一个 loaded 状态

一些同学可能会这样做：

```javascript
function App() {
  const onClick = () => {
    dispatch({ type: 'LOADING', message: 'data is loading' })
    fetch('dataurl').then(() => {
      dispatch({ type: 'LOADED' })
    });
  }

  return (<div>
    <button onClick={onClick}>click</button>
  </div>);
}
```

但是如果有非常多的地方用到这块逻辑，那应该怎么办？

聪明的同学会想到可以将 onClick 里的逻辑抽象出来复用，如下：

```javascript
function fetchData(message: string) {
  return (dispatch) => {
    dispatch({ type: 'LOADING', message })
    setTimeout(() => {
      dispatch({ type: 'LOADED' })
    }, 1000)
  }
}

function App() {
  const onClick = () => {
    fetchData('data is loading')(dispatch)
  }

  return (<div>
    <button onClick={onClick}>click</button>
  </div>);
}
```

很好，但是 fetchData('data is loading')(dispatch) 这种写法有点奇怪，会增加开发者的心智负担。

于是可以借助 rudux 相关的异步中间件，以 rudux-chunk 为例，将写法改为如下：

```javascript
function fetchData(message: string) {
  return (dispatch) => {
    dispatch({ type: 'LOADING', message })
    setTimeout(() => {
      dispatch({ type: 'LOADED' })
    }, 1000)
  }
}

function App() {
  const onClick = () => {
-   fetchData('data is loading')(dispatch)
+   dispatch(fetchData('data is loading'))
  }

  return (<div>
    <button onClick={onClick}>click</button>
  </div>);
}
```

这样就更符合认知一些了，redux 异步中间件没有什么奥秘，主要做的就是这样的事情。

相关阅读：[Why do we need middleware for async flow in Redux?](https://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux)

### 24、事件1：React 事件流程是什么 ？

- 事件注册
  - 组件装载 / 更新
  - 通过 `lastProps`、`nextProps` 判断是否新增、删除事件分别调用事件注册、卸载方法
  - 调用 `EventPluginHub` 的 `enqueuePutListener` 进行事件存储
  - 获取 `document` 对象
  - 根据事件名称（如 `onClick`、`onCaptureClick`）判断是进行冒泡还是捕获
  - 判断是否存在 `addEventListener` 方法，否则使用 `attachEvent`（兼容IE）
  - 给 `document` 注册原生事件回调为 `dispatchEvent`（统一的事件分发机制）

- 事件存储
  - `EventPluginHub`负责管理`React`合成事件的`callback`，它将`callback`存储在`listenerBank`中，另外还存储了负责合成事件的`Plugin`
  - `EventPluginHub`的`putListener`方法是向存储容器中增加一个`listener`
  - 获取绑定事件的元素的唯一标识`key`
  - 将`callback`根据事件类型，元素的唯一标识`key`存储在`listenerBank`中
  - `listenerBank`的结构是：`listenerBank[registrationName][key]`
  例如

  ```javaScript
    {
      onClick: {
          nodeid1:()=>{...}
          nodeid2:()=>{...}
      },
      onChange: {
          nodeid3:()=>{...}
          nodeid4:()=>{...}
      }
    }
  ```

- 事件触发/执行
  这里的事件执行利用了React的批处理机制：
  - 触发 `document`注册原生事件的回调`dispatchEvent`
  - 获取到触发这个事件最深一级的元素
  
  找到事件触发的最深一级元素后：
  - 遍历这个元素的所有父元素，依次对每一级元素进行处理
  - 构造合成事件
  - 将每一级的合成事件存储在`eventQueue`事件队列中
  - 遍历`eventQueue`
  - 通过`isPropagationStopped`判断当前事件是否执行了阻止冒泡方法
  - 如果阻止了冒泡，停止遍历，否则通过`executeDispatch`执行合成事件
  
  `react`在自己的合成事件中重写了`stopPropagation`方法，将`isPropagationStopped`设置为`true`，然后在遍历每一级事件的过程中根据此遍历判断是否继续执行。这就是`react`自己实现的冒泡机制。

- 合成事件
  - 调用`EventPluginHub`的`extractEvents`方法
  - 循环所有类型的`EventPlugin`（用来处理不同事件的工具方法）
  - 在每个`EventPlugin`中根据不同的事件类型，返回不同的事件池
  - 在事件池中取出合成事件，如果事件池是空的，那么创建一个新的
  - 根据元素`nodeid`(唯一标识`key`)和事件类型从`listenerBink`中取出回调函数
  - 返回带有合成事件参数的回调函数

### 25、事件2：React 为什么要手动绑定 this ？

  回调函数是**直接调用**的，并**没有指定调用的组件**，所以不进行手动绑定的情况下直接获取到的`this`是`undefined`。

  这里可以使用实验性的属性初始化语法 ，也就是**直接在组件声明箭头函数**。箭头函数不会创建自己的`this`，它只会从自己的作用域链的上一层继承`this`。因此这样我们在`React`事件中获取到的就是组件本身了。

### 26、事件3：React事件和原生事件有什么区别 ？

  - **Vue**
    - event 是原生的
    - 事件被挂在到当前元素(触发也指向当前元素), 和DOM事件是一样的
  - **React**
    - event 事件的构造函数是 SyntheticEvent, 是模拟出来 DOM 事件的所有能力
    - event.nativeEvent 是原生事件对象
    - 所有的事件都挂在到 document上, 和DOM 事件不一样, 和vue也不一样.(触发对象是和vue、DOM原生事件的对象一样)
  React自己实现了一套事件机制，自己模拟了事件冒泡和捕获的过程，采用了事件代理、批量更新等方法，并且抹平了各个浏览器的兼容性问题。

  - React 事件使用驼峰命名，原生事件命名全部小写
  - React 中的 `JSX` 事件传递的是一个函数作为事件处理程序，原生事件传递的是一个字符串
  - React 中组织默认行为必须明确调用 `preventDefault` ，原生事件 通过返回 `false` 来阻止默认行为

### 27、事件4：React事件和原生事件的执行顺序 ？

  react的所有事件都挂载在 `document` 中，当真实 `dom` 触发后冒泡到 `document` 后才会对 `react` 事件进行处理。
  所以：

- 首先原生的事件会执行
- 然后执行react合成事件
- 最后执行真正在 `document` 上挂载的事件

### 28、事件5：react事件和原生事件可以混用吗？

  `react` 事件和原生事件最好不要混用。
  原生事件中如果执行了 `stopPropagation` 方法，则会导致其他 `react` 事件失效。因为所有元素的事件将无法冒泡到 `document` 上。
  由上面的执行机制不难得出，所有的 `react` 事件都将无法被注册。

### 29、事件5：react 合成事件的浏览器兼容性问题

  事件处理程序将传递 `SyntheticEvent` 的实例，这是一个跨浏览器原生事件包装器。 它具有与浏览器原生事件相同的接口，包括 `stopPropagation()` 和 `preventDefault()` ，在所有浏览器中他们工作方式都相同。

  每个 `SyntheticEvent` 对象都具有以下属性：

  ```text
    boolean bubbles
    boolean cancelable
    DOMEventTarget currentTarget
    boolean defaultPrevented
    number eventPhase
    boolean isTrusted
    DOMEvent nativeEvent
    void preventDefault()
    boolean isDefaultPrevented()
    void stopPropagation()
    boolean isPropagationStopped()
    DOMEventTarget target
    number timeStamp
    string type
  ```

  `React`合成的`SyntheticEvent`采用了事件池，这样做可以大大节省内存，而不会频繁的创建和销毁事件对象。

  另外，不管在什么浏览器环境下，浏览器会将该事件类型统一创建为合成事件，从而达到了浏览器兼容的目的。

> [React 灵魂 23 问，你能答对几个？](https://zhuanlan.zhihu.com/p/304213203)
> [关于React事件的疑问](https://segmentfault.com/a/1190000018391074)
