---
layout: React
title: React step 1 —— React初级使用
date: 2016-04-08 22:48:29
tags: [React,框架]
---

## 一、事件

```javascript
class ReserveBoard extends Component {
	constructor(props) {
      super(props);
		this.state = {
			name: '张三'
		}
		// 需要修改this的指向
		// this.click1 = this.click1.bind(this);
	}
	render () {
		return (<div>
			<span onClick={this.click1}>{this.state.name}</span>
			<span onClick={this.click2('param1', 'param2')}>{this.state.name}</span>
		</div>)
	}
	click1() {
		// 会报错，因为 this 是undefined
		this.setState({
			name: '李四'
		})
	}
	click2 = (id, name, event) => {
		// 事件传参: 前面参数是触发事件是传递的参数, 最后增加了一个参数, 是合成的 event 对象

		// 不会报错，可正常执行，this指向正确
		this.setState({
			name: '李四'
		})
		console.log(event);
		event.preventDefault(); // 阻止默认行为
		event.stopPropagation(); // 阻止冒泡
		console.log(event.target); // 指向 触发的对象 React触发的对象是 element （与原生事件一样）
		console.log(event.currentTarget); // 指向 绑定元素， 假象！！ React绑定的对象是document （与原生事件不一样）
		// event不是原始的event对象,原生的event是 MouseEvent,它的构造函数是 SyntheticEvent (组合事件)
	}
}
```

结论：

- **Vue**
  - event 是原生的
  - 事件被挂在到当前元素(触发也指向当前元素), 和DOM事件是一样的
- **React**
  - event 事件的构造函数是 SyntheticEvent, 是模拟出来 DOM 事件的所有能力
  - event.nativeEvent 是原生事件对象
  - 所有的事件都挂在到 document上, 和DOM 事件不一样, 和vue也不一样.(触发对象是和vue、DOM原生事件的对象一样)

## 二、setState

- 1、React中的state是**不可变值**

```javascript
this.state.a = 'aaa';
this.setState({});
// 这种写法是错误的
```

- 2、setState**可能是异步更新**
  - 在setTimeout中 和 原生事件（自定义的DOM事件）中，setState是同步的
  - 其他情况是异步的

```javascript
// constructor
this.state = {
	a: '123',
}
// 其他函数中
this.setState({
	a: '456',
}, () => {
	console.log(a); // 456 // （1）setState 回调函数中是可以拿到最新值
})
console.log(a); // 123  // （2）正常使用时 setState 是异步的
setTimeout(() => {
	this.setState({
		a: '789',
	})
	console.log(a); // 789  // （3）在setTimeout中 setState是同步的
}, 0)

Document.body.addEventListener('click', () => {
	this.setState({
		a: 'aaa',
	})
	console.log(a); // aaa // （4）在原生DOm事件 setState是同步的
})
```

- 3、setState**可能会被合并**

```javascript
// constructor
this.state = {
	a: 0,
}

// 其他函数中

// （1）连续多次对一个值进行操作，结果会被合并 最终 a = 1
this.setState({
	a: this.state.a + 1
})

this.setState({
	a: this.state.a + 1
})

this.setState({
	a: this.state.a + 1
})

// （2）但是在函数中，执行结果不会被合并
this.setState((prevState, props) => {
	return {
		a: prevState.a + 1
	}
})

this.setState((prevState, props) => {
	return {
		a: prevState.a + 1
	}
})

this.setState((prevState, props) => {
	return {
		a: prevState.a + 1
	}
})
```

## 三、生命周期

http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram

 常用的生命周期：
<img src="https://i.loli.net/2020/12/28/cUJPSIX6vCGab8e.png" >

包含不常用的：
<img src="https://i.loli.net/2020/12/28/cUJPSIX6vCGab8e.png" >

最适合访问初始化接口数据的生命周期是: componentDidMount

## 四、高级特性

### 1、函数组件

<img src="https://i.loli.net/2020/12/29/I2LX4ea3t5RMlUB.png" >

函数组件的特点：

- 纯函数： 输入props，输出JSX
- 没有实例，没有生命周期，因此没有state
- 不能扩展其他方法

### 2、受控组件 和 非受控组件

```javascript
// 受控组件 与 非受控组件
class demo1 extends React.Component {
	constructor(props) {
		super(props);
		this.state = {
			name: 'zhangsan',
		}
		this.fileInputRef = React.createRef(); // 创建ref
	}
	changeVal = e => {
		this.setState({
			name: e.target.value,
		})
	}
	alertFIle = () => {
		const elem = this.fileInputRef.current; // 通过ref获得DOM节点
		alert(elem.files[0].name);
	}
	render() {
		return <div>
			{/* 受控组件 */}
			<input value={this.state.name} onChange={this.changeVal} />
			{/* 非受控组件 */}
			<input type="file" ref={this.fileInputRef} />
			<button onClick={this.alertFIle}>alert File</button>

		</div>
	}
}
```

使用场景：

- 必须手动操作DOM元素，setState实现不了
- 文件上传 `<input type=file>`
- 某些富文本编辑器，需要传入DOM元素

如何选择：

- 优先使用受控组件，符合React设计原则 （数据驱动视图）
- 必须操作DOM的时候，再使用非受控组件

### 3、Portals

问题：组件默认会按照既定层次嵌套渲染，如何让组件渲染到父组件以外？

```javascript
// 父组件 index.js
import React from 'react'
import PortalsDemo from './PortalsDemo'
class AdvancedUse extends React.Component {
	constructor(props) {
		super(props)
	}
	render() {
		return <div>
         <PortalsDemo> Modal 内容 </PortalsDemo>
		</div>
	}
}

export default AdvancedUse

// 子组件
import React from 'react'
import ReactDOM from 'react-dom' // 需要用到 ReactDOM.createPortal API
import './style.css'

class App extends React.Component {
	constructor(props) {
		super(props)
		this.state = {
		}
	}
	render() {
		// // 正常渲染
		// return <div className="modal">
		//     {this.props.children}  {/* vue slot */}
		// </div>

		// 使用 Portals 渲染到 body 上。 // fixed 元素要放在 body 上，有更好的浏览器兼容性。
		return ReactDOM.createPortal(
			<div className="modal">{this.props.children}</div>,
			document.body // 需要挂在到的 DOM 节点
		)
	}
}

export default App
```

Portals 使用场景

- overflow: hiddem
- 父组件 z-index 值太小
- fixed 需要放在body的第一层

### 4、content

问题: 公共信息（语言、主题等）如何传递给每个组件
用 props 太繁琐， 用redux 小题大做

```JS
import React from 'react'

// 创建 Context 填入默认值（任何一个 js 变量）
const ThemeContext = React.createContext('light')

// 底层组件 - 函数是组件
function ThemeLink (props) {
    // const theme = this.context // 会报错。函数式组件没有实例，即没有 this

    // 函数式组件可以使用 Consumer
    return <ThemeContext.Consumer>
        { value => <p>link's theme is {value}</p> }
    </ThemeContext.Consumer>
}

// 底层组件 - class 组件
class ThemedButton extends React.Component {
    // 指定 contextType 读取当前的 theme context。
    // static contextType = ThemeContext
	 // 也可以用 ThemedButton.contextType = ThemeContext
    render() {
		 	// React 会往上找到最近的 theme Provider，然后使用它的值。
        const theme = this.context 
        return <div>
            <p>button's theme is {theme}</p>
        </div>
    }
}
// 指定 contextType 读取当前的 theme context。
ThemedButton.contextType = ThemeContext

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar(props) {
    return (
        <div>
            <ThemedButton />
            <ThemeLink />
        </div>
    )
}

// 根组件
class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            theme: 'light'
        }
    }
    render() {
        return <ThemeContext.Provider value={this.state.theme}>
            <Toolbar />
            <hr/>
            <button onClick={this.changeTheme}>change theme</button>
        </ThemeContext.Provider>
    }
    changeTheme = () => {
        this.setState({
            theme: this.state.theme === 'light' ? 'dark' : 'light'
        })
    }
}

export default App

```

### 5、异步组件

- import()
- React.lazy
- React.Suspense

```javascript
import React from 'react'
// 通过React.lazy()方法 对import引入的组件进行懒加载
const ContextDemo = React.lazy(() => import('./ContextDemo'))

class App extends React.Component {
    constructor(props) {
        super(props)
    }
    render() {
        return <div>
            <p>引入一个动态组件</p>
            <hr />
				{/* React.Suspense 使用懒加载引入的组件 */}
            <React.Suspense fallback={<div>Loading...</div>}>
                <ContextDemo/>
            </React.Suspense>
        </div>
    }
}

export default App

```

## 五、React性能优化

### 1、shouldComponentUpdate

```javascript
shouldComponentUpdate(nextProps, nextState) {
	if (nextState.count !== this.state.count) {
		return true; // 可以渲染 // 默认返回true
	}
	return false; // 不重复渲染
}
```

为什么React不在源码中直接进行 state 对比，如果没有改变就不渲染，而是要给开发者这个操作的权利？

- React 默认： 父组件更新，子组件则无条件也更新！！ 因此性能优化对React更加重要
- shouldComponentUpdate 一定要配合**不可变值**的前提来使用。（如果违反了不可变值的规则，shouldComponentUpdate中对比this.state与nextState中的值时，this.state === nextState，因此无法更新页面）
- 可不考虑使用shouldComponentUpdate， 性能优化时再考虑

### 2、 PureComponent 和 memo

- PureComponent(纯组件) shouldComponentUpdate 中实现了浅比较
- memo 函数组件中的 PureComponent
- 浅比较已经适用大部分场景（尽量不要做深度比较）
  
```javascript
// 普通组件 extends React.Component
class Input extends React.Component { }

// 纯组件 extends React.PureComponent
class List extends React.PureComponent {
	shouldComponentUpdate() {
		// React.PureComponent 默认做了一个浅比较的操作
	}
}

// 函数组件的使用方式： 先定义函数组件，再用React.memo()包装一下
function MyComponent(props) {
	/* 使用 props 渲染 */
}
function areEquals(prevProps, nextProps) {
	// 类似于 shouldComponentUpdate 的函数
	/* 如果把 nextProps 传入 render 方法的返回结果与 将 prevProps 传入 render 方法的返回结果一致 则返回true， 否则返回false */
}
export default React.memo(MyComponent, areEquals);
```

### 3、 immutable.js

- 彻底拥抱“**不可变值**”
- 基于共享数据（不是深拷贝），速度好
- 缺点是 有一定的学习和迁移成本，按需使用

## 六、高阶组件 HOC 和 Render Props

### 1、 HOC

- mixin 已被React弃用
- 使用了高阶组件 HOC
- Render Props
- 高阶组件不是一种功能，而是一种模式
- redux connect 是高阶组件

<img src="https://i.loli.net/2020/12/29/I2LX4ea3t5RMlUB.png"  width="100%" height="100%">

```javascript


import React from 'react'

// 高阶组件
const withMouse = (Component) => {
    class withMouseComponent extends React.Component {
        constructor(props) {
            super(props)
            this.state = { x: 0, y: 0 }
        }
  
        handleMouseMove = (event) => {
            this.setState({
                x: event.clientX,
                y: event.clientY
            })
        }
  
        render() {
            return (
                <div style={{ height: '500px' }} onMouseMove={this.handleMouseMove}>
                    {/* 1.透传所有 props; 2.增加 mouse 属性 */}
                    <Component {...this.props} mouse={this.state}/>
                </div>
            )
        }
    }
    return withMouseComponent
}

const App = (props) => {
    const a = props.a
    const { x, y } = props.mouse // 接收 mouse 属性
    return (
        <div style={{ height: '500px' }}>
            <h1>The mouse position is ({x}, {y})</h1>
            <p>{a}</p>
        </div>
    )
}

export default withMouse(App) // 返回高阶函数
```

### 2、Render Props

<img src="https://i.loli.net/2020/12/30/iLBsy15Y6ZTP9Gu.png" >
  
**HOC与 Render Props的对比**

- HOC： 模式简单，但是会增加组件层级
- Render Props： 代码简洁，学习成本较高

## 七、 Redux

### 1、基本概念

- store state
- action
- 单向数据流

```javascript
// https://www.redux.org.cn/
import { createStore } from 'redux';

/**
 * 这是一个 reducer，形式为 (state, action) => state 的纯函数。
 * 描述了 action 如何把 state 转变成下一个 state。
 *
 * state 的形式取决于你，可以是基本类型、数组、对象、
 * 甚至是 Immutable.js 生成的数据结构。惟一的要点是
 * 当 state 变化时需要返回全新的对象，而不是修改传入的参数。
 *
 * 下面例子使用 `switch` 语句和字符串来做判断，但你可以写帮助类(helper)
 * 根据不同的约定（如方法映射）来判断，只要适用你的项目即可。
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1;
  case 'DECREMENT':
    return state - 1;
  default:
    return state;
  }
}

// 创建 Redux store 来存放应用的状态。
// => => => => 【 API 是 { subscribe, dispatch, getState } 】！！！
let store = createStore(counter);

// 可以手动订阅更新，也可以事件绑定到视图层。
store.subscribe(() =>
  console.log(store.getState())
);

// 改变内部 state 惟一方法是 dispatch(派遣) 一个 action。
// action 可以被序列化，用日记记录和储存下来，后期还可以以回放的方式执行
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```

单向数据流的概述：

1. dispatch(action): 通过 dispatch 一个 action 改变 state 的值
2. reducer -> newState: 通过 reducer 生成一个新的 state
3. subscribe 触发通知: 通过 subscribe 触发通知，进行视图的更新

### 2、异步action

第一步: 引用中间件 redux-thunk （或者redux-promise、redux-saga）

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';

// 创建 store 时 , 作为中间件引入 redux-thunk
const store = createStore(rootReducer, applyMiddleware(thunk));
```

第二步: 在同步返回的 action 对象 替换为 异步返回函数, 其中有 dispatch 参数

<img src="https://i.loli.net/2020/12/30/aAXwmZgs2I6jfFu.png" >

### 3、redux 数据流

<img src="https://i.loli.net/2020/12/30/1XfzRaJ8dYcnT6x.png" >
  
中间件的调用顺序：
<img src="https://i.loli.net/2020/12/30/luqEbTaCHQL3Iwf.png" >