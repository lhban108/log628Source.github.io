---
layout: js
title: js Q&A
## date: 2019-04-15 22:48:29
tags: [js,interview]
## keywords: 博客文章密码
## password: 123123
## message:  输入密码，查看文章
---

## 一.如何正确判断 this 的指向

1. 全局环境中的 this

  	&nbsp;&nbsp;&nbsp;&nbsp;-- 浏览器环境：无论是否在严格模式下，在全局执行环境中（在任何函数体外部）this 都指向全局对象 `window`
	&nbsp;&nbsp;&nbsp;&nbsp;-- node 环境：无论是否在严格模式下，在全局执行环境中（在任何函数体外部），this 都是空对象 `{}`
2. 是否是 new 的绑定

	如果是 new 绑定，并且构造函数中没有返回 function 或者是 object，那么 this 指向这个新对象。如下：

	```javascript
		// 构造函数返回值不是 function 或 object
		function Super(name) {
			this.name = name;
		}
		let instance = new Super('zhangsan');
		console.log(instance.name); // zhangsan
	```

	```javascript
		// 构造函数返回值是 function 或 object
		// 这种情况下 this 指向的是返回的对象
		function Super(name) {
			this.name = name;
			let obj = {
				a: '123'
			}
			return obj;
		}
		let instance = new Super('hello');
		console.log(instance.name); // undefined	
	```

3. 函数通过 call、apply 调用，或者通过 bind 绑定
	this 绑定的就是指向的对象
	**特殊情况**：如果 call,apply 或者 bind 传入的第一个参数值是 undefined 或者 null，严格模式下 this 的值为传入的值 null /undefined。非严格模式下，实际应用的默认绑定规则，this 指向全局对象(node环境为global，浏览器环境为window)

4. 箭头函数
	箭头函数没有自己的this，继承外层上下文绑定的this。

	```javascript
		let obj = {
			age: 20,
			info: function() {
				return () => {
						console.log(this.age); 
						//this继承的是外层上下文绑定的this
				}
			}
		}

		let person = {age: 28};
		let info = obj.info();
		info(); //20

		let info2 = obj.info.call(person);
		info2(); //28
	```

5. 隐式绑定——函数的调用是在某个对象上触发,即调用位置上存在上下文对象。典型的隐式调用为: `obj1.fn()`

	```javascript
		function info(){
		console.log(this.age);
		}
		var person = {
			age: 20,
			info
		}
		var age = 28;
		person.info(); //20;执行的是隐式绑定
	```

## 二.剩余参数 和 `arguments` 对象的区别

剩余参数 —— 剩余参数语法允许我们将不定数量的参数表示为一个数组：

```javascript
	function sum(...args) {
	// ...
	}
	function sum2(agr1, arg2, ...rest) {
	// ...
	}
```

区别：

1. 剩余参数只包含那些没有对应形参的实参（可以是参数的 部分）
而 arguments 对象包含了传给函数的所有实参（是参数的全部）
2. arguments 对象**不是一个真实的数组**
剩余参数**是真实的 Array 实例**。也就是说，能够在它上面直接使用所有的数组方法，比如 sort、map、forEach、pop
3. Arguments Objects 是函数上下文里的激活对象AO中的内部对象，arguments 对象还有一些附加的属性(如 callee 属性)
  arguments.callee 是一个指针，指向拥有这个arguments对象的函数
4. 如果想在 arguments 对象上使用数组方法，首先要将它转换为真实的数组，比如使用
   - [].slice.call(arguments)
   - Array.from(arguments)
   - [... arguments]
   - Array.protoType.slice.call(arguments)

## 三.`['1', '2', '3'].map(parseInt)`

```javaScript
	// Array.prototype.map() 语法:
	var new_array = arr.map(function callback(currentValue[, index[, array]]) {
		// Return element for new_array 
	}[, thisArg])

	/* 参数
	-- callback 生成新数组元素的函数，使用三个参数
		-- currentValue callback 数组中正在处理的当前元素
		-- index (可选) 数组中正在处理的当前元素的索引
		-- array (可选) 方法调用的数组
	-- thisArg (可选) 执行 callback 函数时值被用作this
	*/
```

```javaScript
	// parseInt() 语法:
	parseInt(string, radix);

	/* 参数
	-- string 要解析的值。
		如果参数不是一个字符串，则将其转换为字符串(使用 ToString 抽象操作)。
		字符串开头的空白符将会被忽略
	-- radix (可选) 一个介于2到36之间的整数，代表字符串的基数(数学数字系统中的基)
		小心-这并不是默认为10
	*/

	// 返回值: 从给定的字符串中解析出的一个整数 或者 NaN
	// 	返回NaN 的情况:
	//		 (1)radix 小于 2 或大于 36 (2)第一个非空格字符不能转换为数字
```

执行过程：

1. parseInt('1', 0)
	-> radix 为 0 时，且 string 参数不以“0x”和“0”开头时，按照 10 为基数处理。这个时候返回 1
2. parseInt('2', 1)
	-> radix 基数为 1（1 进制),而 radix 要求是小于 2 或大于 36，否则返回 NaN
3. parseInt('3', 2)
	-> radix 基数为 2 (2 进制),而 二进制表示的数中最大值(为1)小于 3, 所以无法解析，返回 NaN

因此执行结果是  [1, NaN, NaN]

## 四.让 (a == 1 && a == 2 && a == 3) 的值为true

方法一: 利用隐式类型转换

将 a 转换为复杂数据类型(即 Object)。当 复杂数据类型(Object) 与 简单数据类型(Number)进行对比时, Object 转换为原始类型.
Object 转换为原始类型会调用什么方法呢？
(1) 如果部署了 [Symbol.toPrimitive] 接口，那么调用此接口，若返回的不是基本数据类型，抛出错误
(2) 如果没有部署 [Symbol.toPrimitive] 接口，那么根据要转换的类型，先调用 valueOf / toString
	- 非Date类型对象，hint 是 default 时，调用顺序为：valueOf >>> toString，即valueOf 返回的不是基本数据类型，才会继续调用 valueOf，如果toString 返回的还不是基本数据类型，那么抛出错误。
	- 如果 hint 是 string(Date对象的hint默认是string) ，调用顺序为：toString >>> valueOf，即toString 返回的不是基本数据类型，才会继续调用 valueOf，如果valueOf 返回的还不是基本数据类型，那么抛出错误。
	- 如果 hint 是 number，调用顺序为： valueOf >>> toString

```javascript
	// 部署 [Symbol.toPrimitive] / valueOf/ toString 皆可
	// 一次返回1，2，3 即可。
	let a = {
		[Symbol.toPrimitive]: (function(hint) {
					let i = 1;
					//闭包的特性之一：i 不会被回收
					return function() {
						return i++;
					}
		})()
	}
```

方法二: 利用数据劫持(Proxy/Object.definedProperty)

```javascript
	let a = new Proxy({}, {
		i: 1,
		get: function () {
			return () => this.i++;
		}
	});
```

方法三: 数组的 toString 接口默认调用数组的 join 方法，重新 join 方法

```javascript
	let a = [1, 2, 3];
	a.join = a.shift;
```

> [github-blog](https://github.com/YvetteLau/Blog/issues/35)

## 五.异步加载JS脚本的方式

### 1.async 和 defer

`<script>` 标签中增加 `async(html5)` 或者 `defer(html4)` 属性,脚本就会异步加载。

<img src="https://i.loli.net/2021/02/23/qGLYkT7COZFlhz4.png" >

(绿色代表HTML解析，蓝色代表JavaScript加载，红色代表JavaScript执行)

defer 和 async 的区别在于：

- defer 要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），在window.onload 之前执行；
- async 一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染。
- defer是顺序加载，async是无序加载

### 2.动态创建 script 标签

动态创建的 script ，设置 src 并不会开始下载，而是要添加到文档中，JS文件才会开始下载。

```javascript
	let script = document.createElement('script');
	script.src = 'XXX.js';
	// 添加到html文件中才会开始下载
	document.body.append(script);
```

### 3.XHR 异步加载JS

```javascript
	let xhr = new XMLHttpRequest();
	xhr.open("get", "js/xxx.js",true);
	xhr.send();
	xhr.onreadystatechange = function() {
		if (xhr.readyState == 4 && xhr.status == 200) {
			eval(xhr.responseText);
		}
	}
```

## 六.隐藏页面中的某个元素

影藏元素的类型有：

- 完全影藏： 元素从DOM树中消失，不占空间
- 视觉上的影藏：屏幕上不可见，占据空间
- 语义上的影藏：读屏软件不可读，但政策占据空间

### 1. 完全影藏

- display: none
- `<div hidden></div>` : HTML5 新增的属性，相当于 display:none

### 2.视觉上的影藏

- 利用 `position` 和 盒模型 将元素移出可视区范围
  - 设置元素`posoition:absolute` 或 `position:fixed`,再通过设置 top、left 等值，将其移出可视区域

	```css
		div {
			position:absolute;
			left: -99999px;
		}
	```

  - 设置元素`posoition:relative`，通过设置 `top、left` 等值，将其移出可视区域

	```css
		div {
			position: relative;
			left: -99999px;
			height: 0
		}
	```

  - 设置 margin 值，将其移出可视区域范围（可视区域占位）

	```css
		div {
			margin-left: -99999px;
			height: 0;
		}
	```

- 利用 transform
  - 缩放 scale

	```css
		div {
			transform: scale(0);
			height: 0;
		}
	```

  - 移动 translateX, translateY

	```css
		div {
			transform: translateX(-99999px);
			height: 0;
		}
	```

  - 旋转 rotate

	```css
		div {
			transform: rotateY(90deg);
		}
	```

- 设置其大小为0
  - 宽高为0，字体大小为0

	```css
		div {
			height: 0;
			width: 0;
			font-size: 0;
		}
	```

  - 宽高为0，超出影藏

	```css
		div {
			height: 0;
			width: 0;
			overflow: hidden;
		}
	```

- 设置透明度为0 `opacity: 0;`
- `visibility: hidden`
- 层级覆盖, z-index 属性

	```css
		div {
			position: relative;
			z-index: -999;
		}
	```

- clip-path:polygon(0 0, 0 0, 0 0, 0 0); 裁剪

### 3.语义上的隐藏

aria-hidden 属性
读屏软件不可读，占据空间，可见。

```html
	<div aria-hidden="true"></div>
```

## 七.执行上下文栈和作用域链

### 1. 执行上下文

执行上下文就是当前 JavaScript 代码被解析和执行时所在环境的抽象概念， JavaScript 中运行任何的代码都是在执行上下文中运行。

执行上下文创建过程中，需要做以下几件事:

- 创建变量对象：首先初始化函数的参数arguments，提升函数声明和变量声明。
- 创建作用域链（Scope Chain）：在执行期上下文的创建阶段，作用域链是在变量对象之后创建的。
- 确定this的值，即 ResolveThisBinding。

### 2.执行上下文栈 （执行栈）

执行栈，也叫做调用栈，具有 LIFO (后进先出) 结构，用于存储在代码执行期间创建的所有执行上下文。

### 3.作用域

作用域负责收集和维护由所有声明的标识符（变量）组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限

### 4.作用域链

作用域链就是从当前作用域开始一层一层向上寻找某个变量，直到找到全局作用域还是没找到，就宣布放弃。这种一层一层的关系，就是作用域链。

## 八 可迭代对象有哪些特点

（呼应 js-手写代码 的 7-给对象加上 iterator 接口,使之能被 for…of 遍历）

ES6 规定，默认的 Iterator 接口部署在数据结构的 Symbol.iterator 属性，换个角度，也可以认为，一个数据结构只要具有 Symbol.iterator 属性(Symbol.iterator 方法对应的是遍历器生成函数，返回的是一个遍历器对象)，那么就可以其认为是可迭代的。

可迭代对象的特点:

- 具有 Symbol.iterator 属性，Symbol.iterator() 返回的是一个遍历器对象
- 可以使用 for ... of 进行循环
- 通过被 Array.from 转换为数组

原生具有 Iterator 接口的数据结构：Array、Map、Set、String、TypedArray、函数的arguments对象、NodeList对象.

> [这儿有20道大厂面试题等你查收](https://github.com/YvetteLau/Blog/issues/35)

## 九 diff 时间复杂度从 O(n^3) 优化到 O(n)

oldNode 每一个节点都去遍历 newNode 的节点，直到找到新树对应的节点。那么这个流程就是 O(n^2).
再紧接着找到不同之后，再计算最短修改距离然后修改节点，这里是 O(n^3).

vue的更新策略就是：深度优先、同层比较。就是只比较同层级，也就是 O(n)
