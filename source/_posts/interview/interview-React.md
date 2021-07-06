---
layout: React
title: React百问百答
## date: 2017-05-08 20:48:29
tags: [React,interview]
keywords: 博客文章密码
## password: 123123
## message:  输入密码，查看文章
---

## 一、生命周期相关

### 1、React V16生命周期发生了什么变化？

React16废弃的三个生命周期函数

- ~~componentWillMount~~
- ~~componentWillReceiveProps~~
- ~~componentWillUpdate~~
  
取而代之的是两个新的生命中后期函数:

- static getDerivedStateFromProps
- getSnapshotBeforeUpdate
  
>注：目前在16版本中componentWillMount、componentWillReceiveProps、componentWillUpdate并未完全删除这三个生命周期函数，而且新增了UNSAFE_componentWillMount、UNSAFE_componentWillReceiveProps、UNSAFE_componentWillUpdate三个函数，官方计划在17版本完全删除这三个函数，只保留UNSAVE_前缀的三个函数，目的是为了向下兼容，但是对于开发者而言应该尽量避免使用他们，而是使用新增的生命周期函数替代它们  

### 2、setState 异步设计原理

异步的作用是**提高性能，降低冗余。简单说，因为state具有更新队列，将所有更新都累计到最后进行批量合，并再去渲染可以极大提高应用的性能，像源生JS那样修改后就进行DOM的重渲染，会造成巨大的性能消耗。**同样这与react优化后的diff算法也有关系。

**结论：**
**1. setState只在合成事件和钩子函数中是"异步"的，在原生事件和setTimeout中都是同步的。**
**2. setState的"异步"并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形成了所谓的"异步"，当然可以通过第二个参数setState(partialState, callback)中的callback拿到更新后的结果。**
**3. setState的批量更新优化也是建立在"异步"（合成事件、钩子函数）之上的。在原生事件和setTimeout中不会批量更新，在"异步"中如果对同一个值进行多次setState，setState的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时setState多个不同的值，在更新时会对其进行合并批量更新。**

- setState 中的批量更新
在setState的时候React内部会创建一个updateQueue，通过firstUpdate、lastUpdate、lastUpdate.next 去维护一个更新的队列，在最终的perforWork中，相同的key会被覆盖，只会对最后一次的setState进行更新。

- 为什么原生事件中 setState 可以同步拿到更新后的state的值？
原生事件自带的事件监听addEventListener，或者用原生js、jq直接document.querySelector().onclick这种绑定事件的实行都属于原生事件。
由于原生事件没有走合成事件的那一大堆逻辑，直接触发click事件，到requestWork，在requestWork里由于expirationTime===Sync的原因，直接走了performSyncWork去更新，并不想合成事件或者钩子函数中被return，所以原生事件中的setState能同步拿到更新后的state的值。

- 为什么 setTimeout 的 setSate 可以同步拿到更新后的 state 的值？
由于interactiveUpdates文件中，setTimeout( () => { this.setState() }, 0) 在try代码块中，当你try代码执行到setTimeout的时候，把它丢到了队列里，并没有去执行，而是限制性finally代码块，等finally执行完了，isBatchingUpdates又变为了false，导致最红去执行队列的setState时候，requestWork走的是和原生事件一样的 expirationTime === Sync，直接走了performSyncWork去更新，表现就会和原生事件一样了。

### 3、在哪些钩子函数里调用setState不会引起组件重新渲染

1. componentWillMount
  因为这个钩子函数是在组件挂载到DOM前调用，且只会被调用一次。

2. componentWillReceiveProps
  因为`componentWillReceiveProps`中判断`props`是否发生变，若变化了，`this.setState`将引起`state`变化，从而引起`render`，此时就没必要再做第二次因重传`props`引起的`render`了，不然重复做一样的渲染了。

## 二、输出结果

### 1、输出结果是什么，如何改进

```javascript
state = {
    num: 0
}

ComponentDidMount() {
    this.setState({
        num: this.state.num + 1
    }, ()=>{
        console.log(this.state.num) // 1
    })

    this.setState({
        num:this.state.num + 1
    }, () => {
        console.log(this.state.num) // 1
    })
}
```

两次都打印1的原因是：回调函数仅仅可以获取到当前修改后的state，再次执行setState方法**获取到的state值依然为挂载后初始state值**。

解决办法：

```javascript
state = {
    num: 0
}

ComponentDidMount() {
    this.setState(
        preState => {
            num: preState.num + 1
        }, () => {
            console.log(this.state.num) // 1
        }
    )

    this.setState(
        preState => {
            num: preState.num + 1
        }, ()=>{
            console.log(this.state.num) // 2
        }
    )
}
```

实现原理 是setState本身可以接受函数作为参数，而在这里我们使用的参数就是上一次的state。

## 2、输出结果是什么

```javascript
class App extends Component {
  state = {
    count: 0
  };
​
  componentDidMount() {
    // 生命周期中调用
    this.setState({ count: this.state.count + 1 });
    console.log("lifecycle: " + this.state.count);
    setTimeout(() => {
      // setTimeout中调用
      this.setState({ count: this.state.count + 1 });
      console.log("setTimeout: " + this.state.count);
    }, 0);
    document.getElementById("div2").addEventListener("click", this.increment2);
  }
​
  increment = () => {
    // 合成事件中调用
    this.setState({ count: this.state.count + 1 });
    console.log("react event: " + this.state.count);
  };
​
  increment2 = () => {
    // 原生事件中调用
    this.setState({ count: this.state.count + 1 });
    console.log("dom event: " + this.state.count);
  };
​
  render() {
    return (
      <div className="App">
        <h2>couont: {this.state.count}</h2>
        <div id="div1" onClick={this.increment}>
          click me and count+1
        </div>
        <div id="div2">click me and count+1</div>
      </div>
    );
  }
}
```



>  参考  
[setState异步、同步与进阶](https://zhuanlan.zhihu.com/p/50335551)  
[React的setState你真的用对了吗？](https://zhuanlan.zhihu.com/p/61847529)  
[你真的理解setState吗？](https://juejin.cn/post/6844903636749778958#comment)  

