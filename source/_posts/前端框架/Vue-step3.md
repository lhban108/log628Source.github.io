---
layout: Vue
title: Vue step 3 —— Vue面试题
date: 2017-11-21 17:52:57
tags: [Vue,框架]
---
v-if 和 v-show 的区别
key 的作用，为什么不能用index 和 random
父子组件的生命周期

## 1. 组件的通讯过程

## 2. 双向数据绑定 v-model 的实现原理

- input 元素的 value = this.name
- 绑定 input 事件的 this.name = $event.target.value
- data 更新触发 re-render

## 3. computed 有何特点

- 缓存， data 不变不会重新计算
- 提高性能

## 4. 为何组件 data 必须是一个函数

因为每个组件实际是一个 class, 每次使用组件都是对 class 的一个实例化, 因此每个组件都有自己实例单独持有的 data 数据

## 5. ajax 请求应该放在哪个生命周期

mounted， JS 是单线程的， ajax 异步获取数据，放在 mounted 之前没有太多的用处，只会让逻辑更加混乱

## 6. 如何将组件所有 props 传递给子组件？

- 通过`$props`
- `<User v-bind="$props" />`

## 7. 如何自己实现 v-model

<img src="https://i.loli.net/2021/01/23/lPKnNQ5JAoaHbw3.png" >

## 8. 多个组件有相同的逻辑，如何抽离

- `mixin`
- `mixin` 有一些缺点：

## 9. 何时要使用异步组件

- 加载大组件
- 路由异步加载

## 10. 何时使用 `keep-alive`

- 缓存组件，不需要重复渲染
- 如多个静态 tab 页的雀环
- 优化性能

## 11. 何时需要使用 beforeDestroy

- 解绑自定义事件 event.$off
- 清除定时器
- 解绑自定义的 DOM 事件，如 window scroll 等

## 12. Vuex 中 action 和 mutation 有何区别

- action 中处理异步(经常用来请求接口，并在请求结果后调用mutation去修改state的值), mutation不可以(经常用来更新state的值)
- mutation 做原子操作
- action 可以整合多个 mutation

## 13. 请用 vnode 描述一个DOM结构

<img src="https://i.loli.net/2021/01/23/9FYLJP6jOlERS7I.png" >

## 14. Vue 如何监听数据变化

- Object.defineProperty 不能监听数据变化
- 重新定义原型，重写 push pop 等方法，实现监听
- Proxy 可以原生支持监听数据变化

## 14. diff 算法的时间复杂度

- `O(n)`
- 在 `O(n^3)` 基础上做的一些调整

## 15. 简述 diff 算法过程

- patch(elem, vnode) 和 patch(vnode, newVnode)
- patchVnode 和 addVnodes 和 removeVnodes
- updateChildren(key的重要性)

## 16. Vue 为何是异步渲染，$nextTick 的作用

- 异步渲染（以及合并 data 修改），以提高渲染性能
- $nextTick 在DOM 更新完之后，可以拿到更新后的DOM

## 17. Vue 常见性能优化

- 合理使用 v-show 和 v-if
- 合理使用 computed
- v-for 时使用正确的key，以及避免和 v-if 同时使用(v-for 比 v-if 具有更高的优先级)
- 自定义事件、DOM 事件及时销毁
- 合理使用异步组件
- 合理使用 keep-alive
- data 层级不要太深，尽量扁平一些
- 使用 vue-loader 在开发环境做模板编译（预编译）
- 使用 SSR
