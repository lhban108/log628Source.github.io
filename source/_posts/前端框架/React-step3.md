---
layout: React
title: React step 3 —— React面试题
date: 2019-05-16 15:55:48
tags: [React,框架]
---

### 1、组件之间如何通讯

- 父子组件 props
- 自定义事件
- Redux 和 Context

### 2、JSX本质是什么

- JSX解析出来后是一个 createElement 函数
- 函数执行后会返回 vnode
- vnode 通过 patch 渲染到页面

### 3、Context 是什么，如何使用

- 父组件 向其下所有子孙组件传递信息
- 比如一些简单的公共信息：如 主题色、语言等
- 复杂的公共信息 请用 Redux

### 4、shouldComponentUpdate 用途

- 性能优化
- 配合state的”不可变值“ 一起使用，否则会出错

### 5、Redux 单项数据流

<img src="https://i.loli.net/2020/12/30/1XfzRaJ8dYcnT6x.png" >
  
<img src="https://i.loli.net/2020/12/30/luqEbTaCHQL3Iwf.png" >

### 5、setState 场景题

<img src="https://i.loli.net/2021/01/01/pHPVRyw5iodLrJD.png" >

### 6、什么是纯函数

纯函数： 执行函数后 返回一个新值, 没有副作用（不会修改函数以外的值）

### 7、React组件生命周期

- 单个组件生命周期
- 父子组件生命周期、调用顺序（创建、更新、销毁）
- shouldComponentUpdate 函数的作用

### 8、React发起ajax应该在哪个生命周期

- 同Vue（mounted）
- componentDidMount
  
### 9、渲染列表，为什么使用key

- 同Vue。必须使用key， 且不能是 index 和 random
- diff算法中通过 tag 和 key 来判断 是否是 sameNode
- 目的是 减少渲染次数，提升渲染性能

### 10、函数组件和 class 组件的区别

- 纯函数， 输入 props， 输出JSX
- 没有实例，没有生命周期，没有state
- 不能扩展其他方法

### 11、 什么是受控组件 和 非受控组件

受控组件：

- 表单的值 受 state 控制
- 需要自行监听 onChange， 更新 state
  
非受控组件：表单的值 不受 state 控制

非受控组件使用场景：

- 必须手动操作DOM元素，setState实现不了
- 比如： 文件上传 `<input type=file>`
- 某些富文本编辑器，需要传入DOM元素

### 12、 何时使用异步组件

- 同vue
- 加载大组件
- 路由懒加载

API：

- import()
- React.lazy
- React.Suspense

### 13、 多个组件有公共逻辑，如何抽离

- 高阶组件 HOC
- Render Props

### 14、 Redux 如何进行异步请求

- 使用异步 action
- 如 redux-thunk

### 15、 react-router 如何配置路由懒加载

<img src="https://i.loli.net/2021/01/01/dq1AOPsMivhaHRe.png" >

### 16、 component 和 PureComponent 有何区别

- PureComponent 实现了浅比较的 shouldComponentUpdate （diff了state第一层的值）
- 优化性能
- 但是要配合 state 的不可变值使用

### 17、 React 事件 和 DOM 事件的区别

- 所有事件挂在到 document 上
- event 不是原生的，是 SyntheticEvent 合成事件对象
- 引用了 dispatchEvent 机制 (派发事件)

### 18、 React 性能优化

1. 渲染事件是增加合适的key
2. 自定义事件、DOM 事件 及时销毁
3. 合理使用异步组件 ——异步加载大组件
4. 减少函数 bind this 的次数
5. 合理使用 shouldComponentUpdate PureComponent 和 memo
6. 合理使用 Immutable.js ——是否彻底拥抱不可变值
7. webpack 层面的优化
8. 前端通用的性能优化，如图片懒加载
9. 使用 SSR

### 19、 React 和 Vue 的区别

相同点：

1. 都支持组件化
2. 都支持数据驱动视图
3. 都是用 vdom 操作 DOM

区别：

1. React 使用 JSX 拥抱 JS， Vue 使用模板拥抱 HTML
2. React 函数式编程（setState传入一个state，返回一个JSX或者视图）， Vue 声明时编程（声明state的值）
3. React 需要更多的自力更生（比如shouldComponentUpdate，没有watch，没有computed，自己通过js实现）， Vue 把想要的都给你
