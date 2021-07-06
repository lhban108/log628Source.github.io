---
layout: React
title: React Hooks
date: 2018-05-29 22:48:29
tags: [React,框架]
---

### 1. useState 在 组件初始化（Mount）阶段

- 1.获取当前 Hook 节点，同时将当前 Hook 添加到 Hook 链表中
- 2.初始化 Hook 的状态，即读取初始 state 值
- 3.创建一个新的链表作为更新队列，用来存放更新操作（setXxx()）
- 4.创建一个 dispatch 方法（即 useState 返回的数组的第二个参数：setXxx()），该方法的用途是用来修改 state，并将此更新操作添加到更新队列中，另外还会将该更新和当前正在渲染的 fiber 绑定起来
- 5.返回当前 state 和 修改 state 的方法（dispatch）

### 2. Hook 数据结构

一个函数组件中的所有 Hook 是以 链表 的形式存储的。链表中的每个节点就是一个 HooK

```JavaScript
	type Hook = {
		memoizedState: any, // Hook 自身维护的状态
		baseState: any,
		baseUpdate: Update<any, any> | null,
		queue: UpdateQueue<any, any> | null, // Hook 自身维护的更新队列
		next: Hook | null, // next 指向下一个 Hook
	};
```

链表保存在哪里：假如原来还没有 Hook 链表，那么就会将新建的 Hook 节点作为 Hook 链表的头结点，然后把 Hook 链表的头结点保存在 currentlyRenderingFiber.memoizedState 中，也就是当前 FiberNode 的 memoizedState 属性

### 3. useState 如何处理 state 更新

当我们调用跟新 state 的方法（setXxx）的时候，即时调用了 dispatchAction()，这个函数的工作是：新建一个新的 update 对象，添加到更新队列中（queue 链表）上，而且这实际是一个 循环链表。

何时触发组件更新（重新渲染）使组件到达Update 阶段？

在将 update 对象添加到 Hook 的更新队列链表后，dispatchAction() 还会去判断当前调用 setXxx(action) 传入的值（action）和上一次渲染的 state（此时正显示在屏幕上的 state）作对比，看看有没有变化，如果有变化，则调用 scheduleWork() 安排 fiberNode 的更新工作（组件重新渲染），如果没变化，则直接跳过，不安排更新（组件重新渲染）

Hook 当 Update 阶段：

- 1.获取正在执行的处于更新阶段 Hook 节点；
- 2.获取该 Hook 节点的更新队列链表；
- 3.从该更新队列的最早的 update 对象节点开始遍历，一直遍历到最近添加的（最新的）update 对象节点，遍历到每个节点的时候执行该节点的更新操作，将该次更新的 state 值存到 newState 中；
- 4.当遍历完最近的一个 update 对象节点后，此时 newState 里存放的就是最新的 state 值，最后返回 newState，于是用户就拿到了最新的 state；

### 4. useState 运行流程

当第一次执行 useState，也就是 mount 阶段，所以执行的是 mountState。

- 1 在 Hook 链表上添加该 useState 的 Hook 节点
- 2 初始化 state 的值
- 3 返回此次渲染的 state 和 修改 state 的方法

当调用 setXxx/dispatchAction 时

- 1 创建 update 对象，并将 update 对象添加到该 Hook 节点的更新队列链表；
- 2 判断传入的值（action）和当前正在屏幕上渲染的 state 值是否相同，若相同则略过；若不相同，则调用 scheduleWork 安排组件的重新渲染；
- 3 当前所有 setXxx 都逐一执行完后，假如其中能满足（2）的条件，即有调用 scheduleWork 的话，则触发更新（组件重新渲染），进入 Update 阶段；

组件重新渲染（更新）时 ，进入 Update 阶段，即第 2 、第 3 、... n 次执行 useState：

- 1 获取该 useState Hook 的更新队列链表；
- 2 遍历这个更新队列链表，从最早的那一个 update 对象进行遍历，直至遍历到最近的添加那一个 update 对象，最后得到最新的 state 并返回，作为组件此次渲染的 state；
- 3 返回此次渲染的 state 和 修改 state 的方法

### 5. useEffect 运行流程

一个使用 useEffect Hook 的函数组件，在运行的时候的运行流程如下：
组件初次渲染（挂载）：

- 1 执行 useEffect 时，将 useEffect Hook 添加到 Hook 链表中，然后创建 fiberNode 的 updateQueue，并把本次 effect 添加到 updateQueue 中；
- 2 渲染组件的 UI；
- 3 完成 UI 渲染后，执行本次 effect；

组件重新渲染（更新）：

- 1 执行 useEffect 时，将 useEffect Hook 添加到 Hook 链表中，判断依赖：

  - （1）假如没有传入依赖（useEffect 没有传入第二个参数），那么直接给这个 effect 打上 “需要执行” 的 tag（HookHasEffect）；
  - （2）假如有传入依赖 deps 并且当前依赖和上次渲染时的依赖对比有发生改变，那么就给这个 effect 打上 “需要执行” 的 tag（HookHasEffect）；
  - （3）假如有传入依赖 deps，但是依赖没有发生改变，则 不会 给这个 effect “需要执行” 的 tag；
  - （4）假如有传入依赖 deps，但是传入的是一个空数组 []，那么也 不会 给这个 effect “需要执行” 的 tag；

- 2 渲染组件的 UI；
- 3 假如有清除函数（effect 中的 return 内容），则执行上一次渲染的清除函数；如果依赖是 []，则先不用执行清除函数，而是等到组件销毁时才执行；
- 4 判断本次 effect 是否有“需要执行” 的 tag（HookHasEffect），如果有，就执行本次 effect；如果没有，就直接跳过，不执行 本次 effect；

组件销毁时：

在组件销毁之前，先执行完组件上次渲染时的清除函数

### 7. useEffect 和 useMemo 、 useCallBack 的区别

React.memo 为高阶组件，默认对 props 做一次浅比较，如果 props 没有变化，则子组件不会重新执行。
它与 React.PureComponent 非常相似，但它适用于函数组件，但不适用于 class 组件。

### 6. Hooks 闭包

一个 Fiber节点就对应的是一个组件，Fiber 对象上有一个 memoizedState 用于存放组件的 state。
hooks 所针对的 FunctionComponnet ，一个对象都只能有一个 state 属性或者 memoizedState 属性。
那么怎么满足 一个 FunctionComponnet 可以存放多个 state 的需求呢？
所以，react用了链表这种数据结构来存储 FunctionComponent 里面的 hooks。
memoizedState属性就是用来存储组件上一次更新后的 state,next毫无疑问是指向下一个hook对象，
在组件更新的过程中，hooks函数执行的顺序是不变的，就可以根据这个链表拿到当前hooks对应的Hook对象，函数式组件就是这样拥有了state的能力。

`useEffect` 会捕获 `props` 和 `state`，形成闭包。所以即便在回调函数里，你拿到的还是初始的 `props` 和 `state`。
如果想得到“最新”的值，可以使用 `ref`（`useRef`）。

在组件每一次渲染的过程中。 比如 ref = useRef() 所返回的都是同一个对象，每次组件更新所生成的ref指向的都是**同一片内存空间**， 那么当然能够每次都拿到最新鲜的值了。

