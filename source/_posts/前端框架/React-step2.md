---
layout: React
title: React step 2 —— React高级特性
date: 2017-01-29 22:48:29
tags: [React,框架]
---

## 1、vDom和diff

diff:

- 只比较同一层级，不跨级比较
- tag 不相同，则直接删掉重建，不再深度比较
- tag 和 key 相同则认为是相同的节点，不再深度比较
  
vdom:

```javascript
const elem = <div className="abc" onClick={this.clickHandle} >
  <p>text</p>
  <img src={imgUrl}/>
</div>

// 通过(https://www.babeljs.cn/)babel转义后的结果是:
"use strict";
const elem = React.createElement("div", {
		className: "abc",
		onClick: (void 0).clickHandle
	},
	React.createElement("p", null, "text"),
	React.createElement("img", { src: imgUrl })
);

// 即vNode的格式为
React.createElement('div', null, [child1, child2, child3])
React.createElement('div', {...}, child1, child2, child3)
React.createElement(List, null, child1, child2, 'text字符串')
```

## 2、JSX的本质

- JSX 即 createElement 函数（即h函数），执行生成vnode
- 第一个参数 可能是组件，也可能是 html 的tag
- 对于组件名，首字母必须大写（React的规定）
- 最后通过执行 patch(elem, vnode) 和 patch(vnode, newVnode)

## 3、合成事件 SyntheticEvent

### （1）DOM事件的机制现象

1. React的 event 是 SyntheticEvent(合成事件)，是模拟出来 DOM 事件所有能力
2. event.nativeEvent 是原生事件对象
3. 绑定对象: React所有的事件都被挂在到 document 上 —— event.currentTarget => 指向绑定元素，React绑定的对象是document（DOM原生事件和Vue的绑定对象是 element）
4. 触发对象: React 与原生对象的触发对象是一样的 —— event.target => 指向触发的对象，React触发的对象是 element （与原生事件一样）

### （2）绑定和触发的顺序

![SyntheticEvent.png](https://i.loli.net/2020/12/31/pYmSe3xXwfCA2yB.png)

1、DOM事件触发后，由于冒泡机制，最终会冒泡的顶层的 document 上
2、由于 React 所有的事件都是绑定在 document 上，于是 React 会统一生成一个 SyntheticEvent 对象（实例化成统一的react Event）
3、然后 SyntheticEvent 对象会对事件进行 dispatchEvent (派发事件)
4、所谓的 dispatchEvent (派发事件)，就是根据 event.target 找到触发的组件对象，在通过组件对象的绑定关系，触发执行绑定函数(再把合成事件的event对象传递进来，通过参数接收)

### （3）React 为何要使用合成事件机制？

1. 更好的兼容性和跨平台
  - 自己实现的一套事件机制，尽可能的摆脱了 Document 的事件逻辑，可以更好的实现兼容性和跨平台
  - 如果需要迁移到其他平台，只需要修改与 DOM 之间的关系就可以了，降低了对平台的依赖性
2. 挂在到 Document 上，减少内存消耗，避免频繁解绑，减少了事件对 DOM 的依赖
3. 方便事件的统一管理（如事务机制）
 - 将事件统一挂在到一个对象上，有一个统一入口，方便管理

## 4、 setState 和 batchUpdate

- setState 主流程
- batchUpdate 机制
- transaction （事务）机制

主流程图：

<img src="https://i.loli.net/2021/01/01/unmj1FWvMrRVTw3.png" height="80%" width="80%">

当 isBatchingUpdates = false 时，会把组件保存到 dirtyComponents 中，因此当我们需要更新时，需要更新dirtyComonents 组件，执行组件的更新和渲染

主流程分析图：
<img src="https://i.loli.net/2021/01/01/6g3hDPmwix5kyAl.png" > 

React在钩子函数和执行的事件中，首先会初始化一个变量 isBatchingUpdates = true， 到结束时 isBatchingUpdates = false。
而 setTimeout 由于宏任务 后执行，因此在执行时 isBatchingUpdates = false，因此执行setState后可以同步拿到最新的state的值。

原生DOM事件中的 setState：
原生的DOM事件在注册时，React并未用 isBatchingUpdates 变量进行管理

<img src="https://i.loli.net/2021/01/01/tuiyK9vIDY6UNhe.png" >

1. setState 是**异步还是同步**？
 - setState无所谓异步还是同步
 - 看是否能命中 batchUpdate 机制
 - 通过判断 isBatchingUpdates 看是否命中

2.  哪些能命中 batchUpdate 机制
 - React的生命周期函数（和它调用的函数）
 - React中注册的事件（和它调用的函数）
 - React可以”管理“的入口

3. 哪些不能命中 batchUpdate 机制
 - setTimeout setInterval 等（和它调用的函数）
 - 自定义的DOM事件（和它调用的函数）
 - React ”管不到“的入口

## 5、fiber

组件渲染流程：

1. 解析 props ，自定义state
2. 执行 render() 函数，通过解析JSX结构生成 vnode
3. 通过 patch(elem, vnode) vnode

组件更新流程：

1. 通过 setState(newState) 生成 dirtyComponents （可能有子组件）
2. 遍历dirtyComponents所有的组件， render() 生成newVnode
3. 调用patch(vnode, newVnode)渲染

对以上的patch 拆分为两个阶段：

1. reconciliation 阶段： 执行diff算法， 纯JS机选
2. commit 阶段： 将diff结果渲染DOM

可能会有新能问题：

- JS 是单线程， 且和DOM渲染共用一个线程
- 当组件足够复杂，组件更新时计算和渲染都压力大
- 同时再有DOM操作需求（动画、鼠标拖拽等）， 将卡顿

解决方案—— fiber：

- 将 reconciliation 阶段进行任务拆分（commit阶段无法拆分），拆分为多个任务片段
- DOM需要渲染时暂停 reconciliation 拆分的任务，空闲时间恢复计算
- 判断浏览器DOM需要更新渲染的API： window.requestIdleCallback (有些浏览器不支持，因此fiber不是支持所有浏览器)

## 6、React事务机制

。。。
