---
layout: Vue
title: Vue step 2 —— Vue高级特性
date: 2017-11-15 19:01:57
tags: [Vue,框架]
---

- **组件化和 MVVM**
- **响应式原理**
- **vdom 和 diff 算法**
- **模板编译**
- **组件渲染过程**
- **前端路由**

### 1、组件化和 MVVM

- 很早就有组件化, 在Vue中通过数据驱动图的模式，对组件化进行了发展
- 数据驱动视图 - Vue MVVM
- 数据驱动视图 - React setState

<img src="https://i.loli.net/2021/01/03/zk7arT2w8KdfGi6.png" >

MVVM是Model-View-ViewModel的简写,它本质上就是 MVC 的改进版。MVVM 就是将其中的View 的状态和行为抽象化，让我们将视图 UI 和业务逻辑分开.

### 2、响应式原理

- 核心API: Object.defineProperty
- Object.defineProperty 的缺点
  - **深度监听需要递归到底，一次性计算量大**
  - **无法监听 新增属性/删除属性 （Vue.set  Vue.delete）**
- Vue3.0启用Proxy，Proxy 兼容性不好, 且无法 polyfill

```javascript
// Object.defineProperty 的基本用法

function observer(target) {
	if (typeof target !== 'object' || target === null) {
		return target;
	}
	// 重新定义各个属性 for...in 可以监听数组 也可以监听对象
	for(let key in target) {
		defineReactive(target, key, target[key]);
	}
}

function defineReactive(target, key, val) {
	// 深度监听 ——缺点： 深度监听需要递归到底，一次性计算量大
	observer(val);

	// 核心API
	Object.defineProperty(target, key, {
		get: function() {
			return val;
		},
		set: function(newVal) {
			if (val !== newVal) {
				// 深度监听 ——缺点： 深度监听需要递归到底，一次性计算量大
				observer(newVal);

				// 更新值 （注意: val 一直在闭包中,此处设置完之后,再 get 时 拿到的也是最新的值）
				val = newVal;
				// 通知视图更新
				updateView();
			}
		}
	})
}

function updateView() {
	// ... 更新视图
}

// 测试
const data = {
	name: 'zhangsan',
	age: 20
}

observer(data);

data.name = 'lisi';
data.age = 21;

```

1. 如何监听对象、如何监听数组 ?
2. 如何实现复杂对象的深度监听?
   缺点： 深度监听需要递归到底，一次性计算量大

### 3、虚拟DOM（vdom）和 diff

1、用 JS 模拟 DOM 结构

```html
<div id="id1" class="container">
	<p>v-dom</p>
	<ul style="font-size: 20px">
		<li>a</li>
	</ul>
</div>
```

```javascript
vodom = {
	tag: 'div',
	props: {
		className: 'container',
		id: 'id1'
	},
	children: [
		{
			tag: 'p',
			children: 'v-dom'

		},
		{
			tag: 'ul',
			props: {
				style: 'font-size: 20px'
			},
			children: [
				{
					tag: 'li',
					children: 'a'
				}
			]
		}
	]
}
```

2、diff 算法

- diff 即对比，是一个广泛的概念，如 Linux diff命令、git diff等
- 两个 js 对象也可以做diff，如： https://github.com/cujojs/jiff
- 树 diff 的时间复杂度是 O(n^3)
- 优化时间复杂度到 O(n)
  - 只比较同一层级，不跨级比较
  - tag 不相同，则直接删掉重建，不再深度比较
  - tag 和 key 都相同，则认为是相同的节点，不再深度比较

Vue2.x 的 diff 算法:

Vue3.x 的 diff 算法:

### 4、模板编译

- 前置知识: with 语法
- vue template complier 将模板编译为 render 函数
- 执行 render 函数生成 vnode

1. js with 语法

   - 改变 `{}` 内自由变量的查找规则，当做 obj 属性来查找
   - 如果匹配不到 obj 属性，就会报错
   - with 要慎用，它打破了作用域规则，易读性变差

   <img src="https://i.loli.net/2021/01/23/O3iAEV57wX6ZcFh.png" >

2. 编译模板
   - 模板编译为 render 函数，执行 render 函数会返回 vnode
   - 基于 vnode 再执行 patch 和 diff
   - 使用 webpack vue-loader，会在开发环境下编译模板

3. render 函数
   vue 组件可以用 render 代替 template

### 5、组件渲染过程

初次渲染的过程：

1. 解析模板为 render 函数（或在开发环境已完成，vue-loader）
2. 触发响应式，监听 data 属性 getter setter
3. 执行 render 函数，生成 vnode，执行patch(elem, vnode)

更新过程：

1. 修改 data，触发 setter （此前在 getter 中已被监听）
2. 重新执行 render 函数，生成 newVnode
3. patch(oldVnode, newVnode)

### 6、前端路由

1. hash 模式
   - hash 变化会触发页面跳转，即浏览器前进、后退
   - hash 变化不会刷新页面，SPA 必需的特点
   - hash 永远不会提交到 server 端

2. history 模式
   - 用 url 规范的路由，但是跳转时不刷新页面
   - history.pushState : 将路由信息保存到 history 记录中
   - window.onpopstate : 用来监听浏览器前进后退, 找到路由对应的组件, 实现页面的跳转
  history 模式需要后端配合，即 无论访问域名下的什么页面，都返回 index.html，将页面跳转的控制权交给前端

选择：

1. 一般 to B 用 hash 模式，对路由不敏感，成本低，不需要后端支持
2. 一般 to C 用 history 模式
