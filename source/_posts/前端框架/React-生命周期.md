---
layout: React
title: React 生命周期
date: 2018-12-08 20:48:29
tags: [React,框架]
keywords: 博客文章密码
## password: 123123
message:  输入密码，查看文章
---

## 一、React v16前的生命周期

![React生命周期.png](https://i.loli.net/2020/12/14/NSUGMIZ1jLE6WOy.png)

### 1、第一阶段(`initialization`): 组件初始化阶段

该状态主要用于将父组件的`props`注入给子组件，并初始化`state`，对应`React V16`后的`constructor()`阶段。

```javascript
// React v16前
class MyComponent extends React.Component {
  state = {
    stateA: 'aaa',
    stateB: 'bbb',
  };
  render() {
    // return ...
  };
}

// React v16后
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      stateA: 'aaa',
      stateB: 'bbb',
    }
  };
  render() {
    // return ...
  };
}
```

### 2、第二阶段(`Mounting`): 组件挂载阶段

这个阶段分为三个子阶段： **`componentWillMount => render => componentDidMount`** ：

- `componentWillMount`:
  在组件挂载到DOM前调用，且只会被调用一次，在这里调用`this.setState`不会引起组件重新渲染，也可以把写在这边的内容提前到`constructor()`中，所以项目中很少用。  
  此时可以进行开启定时器、向服务器发送请求等操作。

- `render`
  根据组件的`props`和`state`（两者的重传递和重赋值，无论值是否有变化，都可以引起组件重新`render`），`return` 一个`React`元素（描述组件，即UI），不负责组件实际渲染工作。
  之后由`React`自身根据此元素去渲染出页面DOM。`render`是纯函数，不能在里面执行`this.setState`，否则会有改变组件状态的副作用。

- `componentDidMount`
 组件挂载到DOM后调用，全程只执行一次，此时我们可以获取到DOM节点并操作，比如对canvas、svg的操作、服务器请求、订阅都可以写在这个里面，但是记得在componentWillUnmount中取消订阅。
 需要注意的是本组件的render执行完，并不会调用componentDidMount()，而是全部子组件的render调用完后才会调用。

### 3、第三阶段(`update`): 组件更新阶段

`setState`引起的`state`更新或父组件重新`render`引起的`props`更新（更新后的`state`和`props`相对之前无论是否有变化），都将引起子组件的重新`render`。

造成组件更新有两类（三种）情况：

1. 父组件重新`render`
父组件重新`render`引起子组件重新`render`的情况有两种：  
　　（a）每当父组件重新`render`导致的重传`props`，子组件将直接跟着重新渲染(无论`props`是否有变化)。可通过`shouldComponentUpdate`方法优化。  

```javascript
class Child extends ParentComponent {
   shouldComponentUpdate(nextProps){
      // 应该使用这个方法，否则无论state是否有变化都将会导致组件重新渲染
      if(nextProps.someThings === this.props.someThings){
        return false
      }
    }
    render() {
      return <div>{this.props.someThings}</div>
    }
}
```

　　（b）在`componentWillReceiveProps`方法中，将`props`转换成自己的`state`。

```javascript
class Child extends ParentComponent {
  constructor(props) {
    super(props);
    this.state = {
        someThings: props.someThings
    };
  }
  componentWillReceiveProps(nextProps) { // 父组件重传props时就会调用这个方法
    this.setState({someThings: nextProps.someThings});
  }
  render() {
    return <div>{this.state.someThings}</div>
  }
}
```

> 在函数`componentWillReceiveProps`中调用 `this.setState()` 将不会引起第二次渲染。  
> (是因为`componentWillReceiveProps`中判断`props`是否发生变，若变化了，`this.setState`将引起`state`变化，从而引起`render`，此时就没必要再做第二次因重传`props`引起的`render`了，不然重复做一样的渲染了。)

1. 组件本身调用`setState`(无论`state`有没有变化)。可通过`shouldComponentUpdate`方法优化。

```javascript
class Child extends Component {
  constructor(props) {
    super(props);
    this.state = {
      someThings:1
    }
  }
  shouldComponentUpdate(nextStates){ 
    // 应该使用这个方法，否则无论state是否有变化都将会导致组件重新渲染
    if(nextStates.someThings === this.state.someThings){
      return false
    }
  }

  handleClick = () => { 
    const preSomeThings = this.state.someThings
    // 虽然调用了setState ，但state并无变化
    this.setState({
      someThings: preSomeThings
    })
  }

  render() {
    return <div onClick = {this.handleClick}>{this.state.someThings}</div>
  }
}
```

在调用`this.setState`方法到渲染完成阶段分为: 

- **-> `componentWillReceiveProps`**
- **->`shouldComponentUpdate`**
- **-> `componentWillUpdate`**
- **-> `render`**
- **-> `componentDidUpdate`**:
  - `componentWillReceiveProps(nextProps)`  
此方法只调用于`props`引起的组件更新过程中，响应 `Props` 变化之后进行更新的唯一方式，参数`nextProps`是父组件传给当前组件的新`props`。但父组件`render`方法的调用不能保证重传给当前组件的`props`是有变化的，所以在此方法中根据`nextProps`和`this.props`来查明重传的`props`是否改变，以及如果改变了要执行啥，比如根据新的`props`调用`this.setState`出发当前组件的重新`render`.
  - `shouldComponentUpdate(nextProps, nextState)`  
此方法通过比较`nextProps`、`nextState`及当前组件的`this.props`、`this.state`，返回true时当前组件将继续执行更新过程，返回`false`则当前组件更新停止，以此可用来减少组件的不必要渲染，优化组件性能。
ps：这边也可以看出，就算`componentWillReceiveProps()`中执行了`this.setState`，更新了`state`，但在`render`前（如`shouldComponentUpdate`，`componentWillUpdate`），`this.state`依然指向更新前的`state`，不然`nextState`及当前组件的`this.state`的对比就一直是`true`了。
如果`shouldComponentUpdate` 返回`false`，那就一定不用`rerender`(重新渲染)这个组件了，组件的`React elements`(React 元素) 也不用去比对。 但是如果`shouldComponentUpdate`返回`true`，会进行组件的`React elements`比对，如果相同，则不用`rerender`这个组件，如果不同，会调用`render`函数进行`rerender`。

  - `componentWillUpdate(nextProps, nextState)`
此方法在调用`render`方法前执行，在这边可执行一些组件更新发生前的工作，一般较少用。

  - `render`

  - `componentDidUpdate`
此方法在组件更新后被调用，可以操作组件更新的DOM，`prevProps`和`prevState`这两个参数指的是组件更新前的`props`和`state`

### 4、第四阶段(`componentWillUnmount`): 组件卸载阶段

此阶段只有一个生命周期方法：`componentWillUnmount`：
此方法在组件被卸载前调用，可以在这里执行一些清理工作，比如清楚组件中使用的定时器，清除`componentDidMount`中手动创建的DOM元素等，以避免引起内存泄漏。

## 二、React v16+ 的生命周期

`React V16`版本后生命周期的分界点在`React V16.3`

**`React V16.3`的生命周期为**
![React V16.3生命周期](https://i.loli.net/2020/12/09/5R7AXftT6szW1nq.png)

**`React V16.4+`的生命周期为**
![React V16.4+生命周期](https://i.loli.net/2020/12/09/gEyt6U785HqOIVd.png)

`React 16`废弃了之前的三个生命周期函数:

- ~~componentWillMount~~
- ~~componentWillReceiveProps~~
- ~~componentWillUpdate~~
  
取而代之的是两个新的生命中后期函数:

- static getDerivedStateFromProps
- getSnapshotBeforeUpdate
  
> 注：目前在16版本中componentWillMount，componentWillReceiveProps，componentWillUpdate并未完全删除这三个生命周期函数，而且新增了UNSAFE_componentWillMount，UNSAFE_componentWillReceiveProps，UNSAFE_componentWillUpdate三个函数，官方计划在17版本完全删除这三个函数，只保留UNSAVE_前缀的三个函数，目的是为了向下兼容，但是对于开发者而言应该尽量避免使用他们，而是使用新增的生命周期函数替代它们

### 1、第一阶段(`Mounting`): 组件挂载阶段

这个阶段分为四个子阶段：

- constructor
- getDerivedStateFromProps
- ~~componentWillMount/UNSAVE_componentWillMount~~
- render
- componentDidMount
  
**constructor**
  
这个阶段一般会做两件事:  
  - 初始化state对象
  - 给自定义方法绑定this
  
```javascript
constructor(props) {
    super(props)
    
    this.state = {
      stateA: 'aaa',
      stateB: 'bbb'
    }

    this.handleChange1 = this.handleChange1.bind(this)
}
```
  
**getDerivedStateFromProps**
  
> static getDerivedStateFromProps(nextProps, prevState)
  
一个静态方法，所以不能在这个函数里面使用`this`，这个函数有两个参数`props`和`state`，分别指接收到的新参数和当前的`state`对象，这个函数会返回一个对象用来更新当前的`state`对象，如果不需要更新可以返回`null`
该函数会在挂载时、接收到新的props、调用了setState和forceUpdate时被调用。（在React v16.3时只有在挂载时和接收到新的props被调用，据说这是官方的失误，后来修复了）

![V16.3](https://i.loli.net/2020/12/09/AdGQNkLVj1FZn5z.png)

```javascript
class ExampleComponent extends React.Component {
  state = {
    isScrollingDown: false,
    lastRow: null
  }
  static getDerivedStateFromProps(nextProps, prevState) {
    if (nextProps.currentRow !== prevState.lastRow) {
        return {
            isScrollingDown: nextProps.currentRow > prevState.lastRow,
            lastRow: nextProps.currentRow
        }
    }
    return null
  }
}
```
  
**Render**

React最核心的方法，一个组件中必须要有这个方法  
返回的类型有以下几种： 
   
- 原生的DOM，如div
- React组件
- Fragment（片段） -（React16新增）
- Portals（插槽） -（React16新增）
- 字符串和数字，被渲染成text节点
- Boolean和null，不会渲染任何东西
  
**componentDidMount**
  
组件挂载到DOM后调用，全程只执行一次，此时我们可以获取到DOM节点并操作，比如对canvas、svg的操作、服务器请求、订阅都可以写在这个里面，但是记得在componentWillUnmount中取消订阅。
需要注意的是本组件的render执行完，并不会调用componentDidMount()，而是全部子组件的render调用完后才会调用。

### 2、第二阶段(`Updating`): 更新阶段

更新阶段，当组件的`props`发生改变、组件内部调用了`setState` 或者`forceUpdate`发生，会发生多次.  
  
- ~~`componentWillReceiveProps/UNSAFE_componentWillReceiveProps`~~
- `getDerivedStateFromProps`
- `shouldComponentUpdate`
- ~~`componentWillUpdate/UNSAFE_componentWillUpdate`~~
- `render`
- `getSnapshotBeforeUpdate`
- `componentDidUpdate`

**getDerivedStateFromProps**

与挂载阶段的 `getDerivedStateFromProps(nextProps, prevState)`一样
> static getDerivedStateFromProps(nextProps, prevState)

**shouldComponentUpdate**

```javascript
shouldComponentUpdate(nextProps, nextState)
```

参数nextProps和nextState分别表示新的属性和变化之后的state，返回一个布尔值。
true表示会触发重新渲染，false表示不会触发重新渲染，默认返回true。
> 注意当我们调用forceUpdate并不会触发此方法**
因为默认是返回true，也就是只要接收到新的属性和调用了setState都会触发重新的渲染，这会带来一定的性能问题，所以我们需要将this.props与nextProps以及this.state与nextState进行比较来决定是否返回false，来减少重新渲染.(在未来的版本，shouldComponentUpdate返回false，仍然可能导致组件重新的渲染，这是官方自己说的)
  
**render** 
  
与挂载阶段的 `render`一样

**getSnapshotBeforeUpdate**
  
```javascript```
getSnapshotBeforeUpdate(prevProps, prevState)
```

这个方法在`render`之后、`componentDidUpdate`之前调用，有两个参数`prevProps`和`prevState`，表示之前的props和之前的`state`。
这个函数有一个返回值，会作为第三个参数传给`componentDidUpdate`，如果你不想要返回值，请返回`null`，不写的话控制台会有警告.
还有这个方法一定要和`componentDidUpdate`一起使用，否则控制台也会有警告。
前面说过这个方法时用来代替 `componentWillUpdate/UNSAFE_componentWillUpdate`.

**componentDidUpdate**

```javascript
componentDidUpdate(prevProps, prevState, snapshot)
```

该方法在`getSnapshotBeforeUpdate`方法之后被调用，有三个参数`prevProps`、`prevState`、`snapshot`，表示之前的`props`、之前的`state`，和`snapshot`。第三个参数是`getSnapshotBeforeUpdate`返回的。

在这个函数里我们可以操作DOM，和发起服务器请求，还可以`setState`，但是注意一定要用`if`语句控制，否则会导致无限循环。

### 3、第三阶段(`UnMounting`): 卸载阶段

这个阶段的生命周期函数只有一个：

- componentWillUnmount

当我们的组件被卸载或者销毁了就会调用，我们可以在这个函数里去清除一些定时器，取消网络请求，清理无效的DOM元素等垃圾清理工作。
  
注意不要在这个函数里去调用setState，因为组件不会重新渲染了。

>参考资料：  
> [我对 React v16.4 生命周期的理解](https://juejin.cn/post/6844903655372488712#heading-12)  
> [详解React生命周期(包括react16最新版)](https://www.jianshu.com/p/514fe21b9914)  
> [生命周期图谱](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)
> [State & 生命周期](https://zh-hans.reactjs.org/docs/state-and-lifecycle.html)
