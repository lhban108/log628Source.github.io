---
layout: js
title: js-手写代码
## date: 2019-04-15 22:48:29
tags: [js,手写,interview]
## keywords: 博客文章密码
## message:  输入密码，查看文章
---

## 一 JS

1 手写 new Object.create
2 获取JS类型
3 instanceOf
4 深拷贝
5 call/apply/bind
6 防抖节流
7 JSONP
8 sum(2,3) === sum(2)(3)
9 图片懒加载
10 每隔一秒打印 1,2,3,4

### 1 new/Object.create

new 实现的原理:
(1) 创建一个新对象
(2) 将构造函数的作用域赋给新对象（将新对象的 __proto__ 指向构造函数的prototype对象）
(3) 为这个新对象添加属性（将构造函数的this通过call/apply指向这个新对象，并执行了构造函数中的方法）
(4) 如果构造函数没有返回其他方法，那么 this 指向这个对象，否则 this 指向构造函数中返回的对象

```javaScript
function Person() {
	this.name = name;
	this.setName = function(name){ this.name = name };
	this.getName = function(){ return this.name };
}
// 第一步:创建一个新对象
let p1 = {};
// 第二步:将构造函数的作用域赋给新对象（
// 将新对象的 __proto__ 指向构造函数的prototype对象）
p1.__proto__ = Person.prototype;
// 第三步:为这个新对象添加属性
//（将构造函数的this通过call/apply指向这个新对象，并执行了构造函数中的方法）
Person.call(p1);


// 手写一个 new 函数
function myNew() {
	// 1、创建一个新的空对象
	let target = {};
	// 获取构造函数
	let [constructor, ...args] = [...arguments]
	// 2、将构造函数的作用域赋值给这个新对象
	target.__proto__ = constructor.prototype;
	// 3、执行构造函数，将属性或方法添加到这个空对象上
	let result = constructor.apply(target, args);
	// 4、如果构造函数执行的结构返回的是一个对象，那么返回这个对象；
	//		如果构造函数返回的不是一个对象，返回创建的新对象
	if (result && (typeof (result) == "object" 
		|| typeof (result) == "function")) {
		return result;
	}
	return target;
}

// 测试用例 ---- start
const person = function(name) {
	this.name = name;
}
person.prototype.getName = () => {
	return this.name;
}
person.prototype.setName = name => {
	this.name = name;
}

const p1 = new person('zhangsan');
p1.setName('lisi');
```

```javaScript
// Object.create方法
function create(proto) {
	function F() {};
	F.prototype = proto;
	F.prototype.constructor = F;
	
	return new F();
}

//官方版Polyfill
if (typeof Object.create !== "function") {
	Object.create = function (proto, propertiesObject) {
		if (typeof proto !== 'object' && typeof proto !== 'function') {
			throw new TypeError('Object prototype may only be an Object: ' + proto);
		} else if (proto === null) {
			throw new Error("This browser's implementation of Object.create is a shim and doesn't support 'null' as the first argument.");
		}

		if (typeof propertiesObject !== 'undefined') throw new Error("This browser's implementation of Object.create is a shim and doesn't support a second argument.");

		function F() {}
		F.prototype = proto;

		return new F();
	};
}
```

### 2 获取JS类型

```javaScript
	// 获取JavaScript类型的函数
	function getType(obj) {
		// 因为 typeof null = 'object'
		if (obj === null) {
			return obj + '';
		}
		// 判断是引用类型还是基本类型
		if (typeof obj === 'object') {
			// const objClass = Object.prototype.toString.call(obj);
			// let type = objClass.split(' ')[1].replace(']', '');
			// return type.toLowerCase();
			const typeStr = Object.prototype.toString.call(obj);
			// [object Array]
			return typeStr.substring(8, typeStr.length - 1).toLowerCase();
		} else {
			return typeof obj;
		}
	}
```

### 3 instanceOf

```javaScript
function instanceOf(left, right) {
	let proto = left.__proto__; 
	let prototype = right.prototype
	while(true) {
		if(proto == null) return false
		if(proto == prototype) return true
		proto = proto.__proto__;
	}
}
```

### 4 深拷贝/浅拷贝

深拷贝和浅拷贝是针对复杂数据类型来说的，浅拷贝只拷贝一层，而深拷贝是层层拷贝。

- **深拷贝**：深拷贝复制变量值，对于非基本类型的变量，则递归至基本类型变量后，再复制。 深拷贝后的对象与原来的对象是完全隔离的，互不影响，对一个对象的修改并不会影响另一个对象。
- **浅拷贝**：浅拷贝是会将对象的每个属性进行依次复制，但是当对象的属性值是引用类型时，实质复制的是其引用，当引用指向的值改变时也会跟着变化。

浅拷贝的实现方式：

- Object.assign()
- 函数库 lodash 的 lodash.clone 方法
- 展开运算符...
- Array.prototype.concat()
- Array.prototype.slice()

```javaScript
	// 浅拷贝实现过程
	let obj = {
		name: 'Yvette',
		age: 18,
		hobbies: ['reading', 'photography']
	}
	// 浅拷贝实现方式一: Object.assign()
	let obj2 = Object.assign({}, obj);

	// 浅拷贝实现方式二: 函数库lodash的_.clone方法
	let lodash = require('lodash');
	let obj3 = lodash.clone(obj);

	// 浅拷贝实现方式三: 展开运算符...
	let obj4 = {...obj};

	// 浅拷贝实现方式四: Array.prototype.concat()
	let arr = [1, 3, { username: 'kobe' }];
	let arr2 = arr.concat();  

	// 浅拷贝实现方式五: Array.prototype.concat()
	let arr3 = [1, 3, { username: 'kobe' }];
	let arr4 = arr3.slice();
```

可以看出，浅拷贝**只对第一层属性**进行了拷贝，当第一层的属性值是基本数据类型时，新的对象和原对象互不影响，但是如果第一层的属性值是复杂数据类型，那么*新对象和原对象的属性值其指向的是同一块内存地址*。

深拷贝的实现方式：

- JSON.parse(JSON.stringify(obj))
- 函数库 lodash 的 lodash.cloneDeep 方法
- jQuery.extend()方法
- 手写递归方法

	```javascript
	// 方法一: JSON.parse(JSON.stringify(obj))
		/*
		缺陷：
		1、对象属性是函数时，无法拷贝
		2、原型链上的属性无法拷贝
		3、不能正确处理 Date 了类型的数据
		4、不能处理 RegExp
		5、会忽略 Symbol 和 undefined
		*/

	// 方法二: 函数库 lodash 的 lodash.cloneDeep 方法
	let lodash = require('lodash');
	let obj = {
		a: 1,
		b: { f: { g: 1 } },
		c: [1, 2, 3]
	};
	let obj2 = lodash.cloneDeep(obj);

	// 方法三: .jQuery.extend()方法
	var $ = require('jquery');
	var obj3 = {
		a: 1,
		b: { f: { g: 1 } },
		c: [1, 2, 3]
	};
	var obj4 = $.extend(true, {}, obj3); //第一个参数为true,就是深拷贝

	// 方法四: 实现一个 deepClone 函数
	/*
	步骤:
	1、如果是基本数据类型，直接返回
	2、如果是 RegExp 或者 Date 类型，返回对应类型
	3、如果是复杂数据类型，递归
	4、考虑循环引用的问题
	*/
	// 深拷贝
	function deepClone(obj, hash = new WeakMap()) {
		if (obj === null || typeof obj !== 'object') {
			//如果不是复杂数据类型 或者是 Null ，直接返回
			return obj;
		}
		if (obj instanceof RegExp) return new RegExp(obj);
		if (obj instanceof Date) return new Date(obj);

		// 考虑循环引用的问题
		if (hash.has(obj)) {
			return hash.get(obj);
		}
		/**
		 * 如果obj是数组，那么 obj.constructor 是 [Function: Array]
		 * 如果obj是对象，那么 obj.constructor 是 [Function: Object]
		 */
		let cloneObj = new obj.constructor();
		hash.set(obj, cloneObj);
		for (let key in obj) {
			//递归
			if (obj.hasOwnProperty(key)) { // 是否是自身的属性
				cloneObj[key] = deepClone(obj[key], hash);
			}
		}
		return cloneObj;
	}
	```

	```javaScript
	// 满分答案
	// https://github.com/ConardLi/ConardLi.github.io/blob/master/demo/deepClone/src/clone_6.js
	const mapTag = '[object Map]';
	const setTag = '[object Set]';
	const arrayTag = '[object Array]';
	const objectTag = '[object Object]';
	const argsTag = '[object Arguments]';
	const boolTag = '[object Boolean]';
	const dateTag = '[object Date]';
	const numberTag = '[object Number]';
	const stringTag = '[object String]';
	const symbolTag = '[object Symbol]';
	const errorTag = '[object Error]';
	const regexpTag = '[object RegExp]';
	const funcTag = '[object Function]';

	const deepTag = [mapTag, setTag, arrayTag, objectTag, argsTag];

	function forEach(array, iteratee) {
		let index = -1;
		const length = array.length;
		while (++index < length) {
			iteratee(array[index], index);
		}
		return array;
	}

	function isObject(target) {
		const type = typeof target;
		return target !== null && (type === 'object' || type === 'function');
	}

	function getType(target) {
		return Object.prototype.toString.call(target);
	}

	function getInit(target) {
		const Ctor = target.constructor;
		return new Ctor();
	}

	function cloneSymbol(targe) {
		return Object(Symbol.prototype.valueOf.call(targe));
	}

	function cloneReg(targe) {
		const reFlags = /\w*$/;
		const result = new targe.constructor(targe.source, reFlags.exec(targe));
		result.lastIndex = targe.lastIndex;
		return result;
	}

	function cloneFunction(func) {
		const bodyReg = /(?<={)(.|\n)+(?=})/m;
		const paramReg = /(?<=\().+(?=\)\s+{)/;
		const funcString = func.toString();
		if (func.prototype) {
			const param = paramReg.exec(funcString);
			const body = bodyReg.exec(funcString);
			if (body) {
					if (param) {
						const paramArr = param[0].split(',');
						return new Function(...paramArr, body[0]);
					} else {
						return new Function(body[0]);
					}
			} else {
					return null;
			}
		} else {
			return eval(funcString);
		}
	}

	function cloneOtherType(targe, type) {
		const Ctor = targe.constructor;
		switch (type) {
			case boolTag:
			case numberTag:
			case stringTag:
			case errorTag:
			case dateTag:
					return new Ctor(targe);
			case regexpTag:
					return cloneReg(targe);
			case symbolTag:
					return cloneSymbol(targe);
			case funcTag:
					return cloneFunction(targe);
			default:
					return null;
		}
	}

	function clone(target, map = new WeakMap()) {

		// 克隆原始类型
		if (!isObject(target)) {
			return target;
		}

		// 初始化
		const type = getType(target);
		let cloneTarget;
		if (deepTag.includes(type)) {
			cloneTarget = getInit(target, type);
		} else {
			return cloneOtherType(target, type);
		}

		// 防止循环引用
		if (map.get(target)) {
			return map.get(target);
		}
		map.set(target, cloneTarget);

		// 克隆set
		if (type === setTag) {
			target.forEach(value => {
					cloneTarget.add(clone(value, map));
			});
			return cloneTarget;
		}

		// 克隆map
		if (type === mapTag) {
			target.forEach((value, key) => {
					cloneTarget.set(key, clone(value, map));
			});
			return cloneTarget;
		}

		// 克隆对象和数组
		const keys = type === arrayTag ? undefined : Object.keys(target);
		forEach(keys || target, (value, key) => {
			if (keys) {
					key = value;
			}
			cloneTarget[key] = clone(target[key], map);
		});

		return cloneTarget;
	}

	module.exports = {
		clone
	};
	```

### 5 call/apply/bind

call 和 apply 的功能相同，都是改变 this 指向，并立即执行函数。区别在于传参方式不同：

- func.call(this, arg1, arg2, ...)
- func.apply(this, [arg1, arg2, ...])

实现步骤：

1. 如果第一个参数没有传入，那么默认指向 window / global(非严格模式)
2. 传入 call 的第一个参数是 this 指向的对象，根据隐式绑定的规则，我们知道 obj.foo(), foo() 中的 this 指向 obj; 因此我们可以这样调用函数 thisArgs.func(...args)
3. 返回执行结果

```javaScript
	// call
	Function.prototype.call2 = function() {
		let [thisArg, ...args] = [...arguments];
		if (!thisArg) {
			// context 为null 或者是 undefined
			thisArg = typeof window === 'undefined' ? global : window;
		}
		// this 就是当前函数 func(func.call)
		thisArg.func = this;
		// 执行函数
		let result = thisArg.func(...args);
		// 由于thisArg 上原本没有 func，因此执行完后需要删除
		delete thisArg.func;

		return result;
	}

	// 测试用例 --------start
	// test1
	const c = {
		name: 'c罗',
		age: '34',
		func: function(arg1, arg2) {
			console.log(this.name + ':' + this.age + '-' + arg1 + '-' + arg2);
		}
	}

	const n = {
		name: '梅西',
		age: '32',
	}
	c.func.call2(n, 't1', 't2'); // 梅西:32-t1-t2
	c.func('t3', 't4'); // c罗:34-t3-t4
	// 测试用例 --------end

	// apply 与 call 区别就是传参的不同
	// apply
	Function.prototype.apply2 = function(thisArg, rest) {
		if (!thisArg) {
			// context 为null 或者是 undefined
			thisArg = typeof window === 'undefined' ? global : window;
		}
		let result; // 函数执行结果
		thisArg.func = this;
		if (!rest) {
			result = thisArg.func();
		} else {
			result = thisArg.func(...rest);
		}
		delete thisArg.func;
		return result;
	}

	// 测试用例 --------start
	// test1
	const c = {
		name: 'c罗',
		age: '34',
		func: function(arg1, arg2) {
			console.log(this.name + ':' + this.age + '-' + arg1 + '-' + arg2);
		}
	}

	const n = {
		name: '梅西',
		age: '32',
	}
	c.func.apply(n, ['t1', 't2']); // 梅西:32-t1-t2
	c.func('t3', 't4'); // c罗:34-t3-t4
	// 测试用例 --------end
```

```javaScript
// bind:
Function.prototype.bind2 = function(content) {
	// 若没问参数类型则从这开始写
	const fn = this;
	const args = [...arguments].slice(1);
	
	const resFn = function() {
		return fn.apply(this instanceof resFn ?
			this : content, args.concat(...arguments) );
	}
	function tmp() {}
	tmp.prototype = this.prototype;
	resFn.prototype = new tmp();
	
	return resFn;
}

// 测试用例 --------start
// test 1
function fn(a, b, c) {
  return a + b + c;
}

var _fn = fn.bind2(null, 10);
console.log(_fn(20, 30, 40)); // 60

// test2
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const _Person = Person.bind(null, 'beike');
const p1 = new _Person(30);
console.log(p1); // Person {name: "hanzichi", age: 30}

// test4
var name = '梅西';
var age = 33;
var obj = {
	name: '坎特',
	age: this.age,
	myFun: function(from,to){
		console.log('name:' + this.name + ', Age:' + this.age)
	}
}
var db = {
	name: '德布劳内',
	age: 29
}
var rm = {
	name: '里克尔梅',
	age: 42
}
obj.myFun(); // name:坎特, Age:33
obj.myFun.apply2(rm); // name:里克尔梅, Age:42
obj.myFun.call2(db); // name:德布劳内, Age:29
obj.myFun.bind2(db)(); // name:德布劳内, Age:29
// 测试用例 --------end
```

### 6 防抖/节流

考点：闭包、防抖、节流

统计按钮一秒点击次数

```javaScript
	button.onclick = debounce(()=>{
		console.log('点击一次');
	},1000);

	// 统计一秒点击次数
	function debounce(fn, time) {
		let number = 0 , timer  = null
		return function(...arg) { 
			fn.apply(this, arg)
			number++
			if (timer) return
			timer = setTimeout(() => {
				console.log( '点击'+ number + '次' )
				number = 0
				timer = null
			}, time)
		}
	}

	//	- 防抖意clk延迟时间。
	// 防抖
	function debounce(fn, time = 500) {
		let timeout = null;
		// 创建一个标记用来存放定时器的返回值
		return function() { 
			clearTimeout(timeout);
			// 每当用户输入的时候把前一个 setTimeout clear 掉
			timeout = setTimeout(() => {
				// 然后又创建一个新的 setTimeout
				// 这样就能保证输入字符后的interval 
				// 间隔内如果还有字符输入的话，就不会执行 fn 函数	 
				fn.apply(this, arguments)
			}, time)
	 	}
	 }

	// - 节流: 高频事件触发，但在 n 秒内只会执行一次(第一次)，所以节流会稀释函数的执行频率。
	// 节流
	function throttle(fn, time = 500) { 
		let canRun = true //
		// 通过闭包保存一个标记
		return function() {
			if (!canRun) return
			// 在函数开头判断标记是否为 true，不为 true 则 return
			canRun = false // 立即设置为 false
			setTimeout(() => {
				fn.apply(this, arguments)
				// 最后在 setTimeout 执行完毕后再把标记设置为 true(关键)
				// 表示可以执行下一次循环了。
				// 当定时器没有执行的时候标记永远是 false，在开头被return
				canRun = true
			}, time)
		}
	}
```

防抖的应用场景:

- 搜索框输入查询，如果用户一直在输入中，没有必要不停地调用去请求服务端接口，等用户停止输入的时候，再调用，设置一个合适的时间间隔，有效减轻服务端压力。
- 表单验证
- 按钮提交事件。
- 浏览器窗口缩放，resize事件(如窗口停止改变大小之后重新计算布局)等。

节流的应用场景

- 按钮点击事件
- 拖拽事件
- onScoll
- 计算鼠标移动的距离(mousemove)

### 7 JSONP

```javaScript
const jsonp = ({ url, params, callbackName }) => {
	const generateUrl = () => {
		let dataSrc = ''
		for (let key in params) {
			if (params.hasOwnProperty(key)) {
				dataSrc += `${key}=${params[key]}&`
			}
		}
		dataSrc += `callback=${callbackName}`
		return `${url}?${dataSrc}`;
	}
	return new Promise((resolve, reject) => {
		const scriptEle = document.createElement('script')
		scriptEle.src = generateUrl();
		document.body.appendChild(scriptEle)
		window[callbackName] = data => {
			resolve(data)
			document.removeChild(scriptEle)
		}
	})
}
```

### 8 sum(2,3) === sum(2)(3)

```javaScript
console.log(sum(2,3));   // Outputs 5
console.log(sum(2)(3));  // Outputs 5

function sum() {
	var fir = arguments[0];
	if(arguments.length === 2) {
		re
	} else {
		return function(sec) {
			return fir + sec;
		}
	}
}
```

### 9 图片懒加载

```javaScript
let imgList = [...document.querySelectorAll('img')]
let length = imgList.length

// 修正错误，需要加上自执行
const imgLazyLoad = (function() {
	let count = 0;
   return function() {
		let deleteIndexList = [];
		imgList.forEach((img, index) => {
			let rect = img.getBoundingClientRect()
			if (rect.top < window.innerHeight) {
				img.src = img.dataset.src;
				deleteIndexList.push(index);
				count++;
				if (count === length) {
					document.removeEventListener('scroll', imgLazyLoad);
				}
			}
		})
		imgList = imgList.filter((img, index) => !deleteIndexList.includes(index));
   }
})()

// 这里最好加上防抖处理
document.addEventListener('scroll', imgLazyLoad)
```

### 10 闭包/作用域链

JavaScript代码的整个执行过程，分为两个阶段，代码**编译阶段**与代码**执行阶段**。

- 编译阶段由编译器完成，将代码翻译成可执行代码，这个阶段作用域规则会确定。
- 执行阶段由引擎完成，主要任务是执行可执行代码，执行上下文在这个阶段创建。

<img src="https://i.loli.net/2021/06/07/SYyLVzKbrEBIo2e.png" >

执行上下文的生命周期：

<img src="https://i.loli.net/2021/06/07/L4pMAozZuyDtwv7.png" >

作用域:
ES5 中只存在两种作用域：**全局作用域**和**函数作用域**。
在 JavaScript 中，我们将作用域定义为一套规则，这套规则用来管理引擎如何在当前作用域
以及嵌套子作用域中根据标识符名称进行变量（变量名或者函数名）查找

作用域链:
当访问一个变量，编译器在执行这段代码时，会首先从当前的作用域中查找是否有这个标识符，
如果没有找到，就会去父作用域查找，如果父作用域还没找到继续向上查找，直到全局作用域为止.
作用域链 —— 就是有当前作用域与上层作用域的一系列变量对象组成，它保证了当前执行的作用域对符合访问权限的变量和函数的有序访问。

```javaScript
// 使用闭包实现每隔一秒打印 1,2,3,4
for (var i = 1; i <= 5; i++) {
	(function (i) {
		setTimeout(() => console.log(i), 1000 * i)
	})(i)
}
```

## 二 字符串

1 判断是不是回文字符串
2 JSON.stringify
3 解析 URL 参数为对象
4 字符串模板
5 转驼峰
6 找到出现次数最多的字符和个数
7 切花字符串大小写
8 去掉字符串中的空格

### 1 判断一个字符串是不是回文字符串

```javaScript
function isPalindrome(str) {
    str = str.replace(/\W/g, '').toLowerCase();
    return (str == str.split('').reverse().join(''));
}

```

### 2 JSON.stringify

```javaScript
function jsonStringify(obj) {
	let type = typeof obj;
	if (type !== "object" || type === null) {
		if (/string|undefined|function/.test(type)) {
			obj = '"' + obj + '"';
		}
		return String(obj);
	} else {
		let json = [];
		let arr = (obj && obj.constructor === Array);
		for (let k in obj) {
			let v = obj[k];
			let type = typeof v;
			if (/string|undefined|function/.test(type)) {
				v = '"' + v + '"';
			} else if (type === "object") {
				v = jsonStringify(v);
			}
			json.push((arr ? "" : '"' + k + '":') + String(v));
		}
		return (arr ? "[" : "{") + String(json) + (arr ? "]" : "}")
	}
}

// var json = '{"a":"1", "b":2}';
function jsonEval(json) {
	// obj 就是 json 反序列化之后得到的对象
	var rx_one = /^[\],:{}\s]*$/;
	var rx_two = /\\(?:["\\\/bfnrt]|u[0-9a-fA-F]{4})/g;
	var rx_three = /"[^"\\\n\r]*"|true|false|null|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?/g;
	var rx_four = /(?:^|:|,)(?:\s*\[)+/g;

	if (
		rx_one.test(
			json.replace(rx_two, "@")
					.replace(rx_three, "]")
					.replace(rx_four, "")
		)
	) {
		var obj = eval("(" +json + ")");
	}

	// 或者直接 return eval("(" +json + ")");
	// 但是会造成XSS攻击
}

```

```javaScript
function jsonParse(opt) {
	return eval('(' + opt + ')');
}
// 它会执行JS代码，有XSS漏洞。
// 如果你只想记这个方法，就得对参数json做校验
var rx_one = /^[],:{}s]*$/;
var rx_two = /(?:["/bfnrt]|u[0-9a-fA-F]{4})/g;
var rx_three = /"[^"nr]*"|true|false|null|-?d+(?:.d*)?(?:[eE][+-]?d+)?/g;
var rx_four = /(?:^|:|,)(?:s*[)+/g;
if (
    rx_one.test(
        json
 .replace(rx_two, "@")
 .replace(rx_three, "]")
 .replace(rx_four, "")
 )
) {
 var obj = eval("(" +json + ")");
}
```

### 3 解析 URL 参数为对象

```javaScript
function parseParam(url) {
	// 将 ? 后面的字符串取出来
	const paramsStr = /.+\?(.+)$/.exec(url)[1];
	// 将字符串以 & 分割后存到数组中
	const paramsArr = paramsStr.split('&');
	let paramsObj = {};
	// 将 params 存到对象中
	paramsArr.forEach(param => {
		if (/=/.test(param)) { // 处理有 value 的参数
		 	// 分割 key 和 value
			let [key, val] = param.split('=');
			val = decodeURIComponent(val); // 解码
			// 判断是否转为数字
			val = /^\d+$/.test(val) ? parseFloat(val) : val;
	 		// 如果对象有 key，则添加一个值
			if (paramsObj.hasOwnProperty(key)) {
					paramsObj[key] = [].concat(paramsObj[key], val);
			} else {
				// 如果对象没有这个 key，创建 key 并设置值
				paramsObj[key] = val;
			}
		} else { // 处理没有 value 的参数
			paramsObj[param] = true;
		}
	})
	
	return paramsObj;
}

parseParam('www.baidu.com?name=Ronaldo&age=45&contry')
// {
// 	age: 45
// 	contry: true
// 	name: "Ronaldo"
// }

let url = 'http://www.domain.com/?user=anonymous&id=123&id=456&city=%E5%8C%97%E4%BA%AC&enabled';
parseParam(url)
/* 结果
{ user: 'anonymous',
  id: [ 123, 456 ], // 重复出现的 key 要组装成数组，能被转成数字的就转成数字类型
  city: '北京', // 中文需解码
  enabled: true, // 未指定值得 key 约定为 true
}
*/
```

### 4 字符串模板

```javaScript
function render(template, data) {
	// 模板字符串正则
	const reg = /\{\{(\w+)\}\}/;
	// 判断模板里是否有模板字符串
	if (reg.test(template)) {
		// 查找当前模板里第一个模板字符串的字段
		const name = reg.exec(template)[1];
		// 将第一个模板字符串渲染
		template = template.replace(reg, data[name]); 
		// 递归的渲染并返回渲染后的结构
		return render(template, data); 
	}
	return template; // 如果模板没有模板字符串直接返回
}

// test
let template = '我是{{name}}，年龄{{age}}，性别{{sex}}';
let person = {
    name: '布兰',
    age: 12
}
render(template, person); // 我是布兰，年龄12，性别undefined
```

### 5 转驼峰

```javaScript
// 解法一 正则 replace
// 1-1.驼峰式转下横线：
function toLowerLine(str) {
	var resStr = str.replace(/[A-Z]/g, match => "_" + match.toLowerCase());
	//如果首字母是大写，执行replace时会多一个_ ,这里需要去掉
  	if(resStr.slice(0, 1) === '_') { // resStr.startsWith('_')
  		resStr = resStr.slice(1);
  	}
	return resStr;
};
// test:
toLowerLine("TestToLowerLine");  //test_to_lower_line
toLowerLine("testToLowerLine");  //test_to_lower_line

// 1-2.下横线转驼峰式：
function toCamel(str) {
  	return str.replace(/([^_])(?:_+([^_]))/g, function (match, $1, $2) {
		// match=t_b, $1=t, $2=b
		// match=e_c, $1=e, $2=c
		return $1 + $2.toUpperCase();
  	});
}
// test
toCamel('test_be_camel') // testBeCamel

// 解法二: reduce
// 2-1.驼峰式转下横线：
function toLowerLine(str){
	return Array.prototype.reduce.call(str, (pre, cur, index) => {
		if(/[A-Z]/.test(cur)){
			cur = cur.toLowerCase();
			return index === 0 ? pre + cur :  pre + '_' + cur
		} else {
			return pre + cur;
		}
	}, '')
}
// test
toLowerLine('TestToLowerLine'); // test_to_lower_line

// 2-2.下横线转驼峰式：
function toCamel(str) {
	return Array.prototype.reduce.call(str, (pre, cur) => {
		if (pre.endsWith('_')) {
			return pre.substring(0, pre.length - 1) + cur.toUpperCase();
		} else {
			return pre + cur;
		}
	})
}
// test
toCamel('test_to_camel'); // testToCamel

// 解法三: Array.map()
// 3-1.驼峰式转下横线：
function toLowerLine(arr){
	// return [].map.call(arr, doLowerLine).join('');
	return Array.prototype.map.call(arr, (val, index) => {
		if(/[A-Z]/.test(val)){
			return index === 0 ? val.toLowerCase() : '_' + val.toLowerCase();
		} else {
			return val;
		}
	}).join('');
}
```

### 6 找到出现次数最多的字符和个数

```javaScript
function getMore(str) {
	// str = "abcabcabcbbccccc"
	let num = 0;
	let char = '';

	// 使其按照一定的次序排列
	str = str.split('').sort().join('');
	// str = "aaabbbbbcccccccc"

	// 定义正则表达式
	let re = /(\w)\1+/g;
	str.replace(re,($0,$1) => {
		if(num < $0.length){
			num = $0.length;
			char = $1;        
		}
	});
	return {
		num,
		char,
	}
}
```

### 7 切花字符串大小写

```javaScript
function caseConvert(str){
   return str.replace(/([a-z]*)([A-Z]*)/g, (m, s1, s2)=>{
		return `${s1.toUpperCase()}${s2.toLowerCase()}`
   })
}
caseConvert('AsA33322A2aa') //aSa33322a2AA
```

### 8 去掉字符串中的空格

```javaScript
 
const POSITION = Object.freeze({
  left: Symbol(), // 左
  right: Symbol(), // 右
  both: Symbol(), // 左右
  center: Symbol(), // 中间
  all: Symbol(), // 全部
})
 
function trim(str, position = POSITION.both) {
	if (!!POSITION[position]) throw new Error('unexpected position value')
	
	switch(position) {
		case(POSITION.left):
			str = str.replace(/^\s+/, '')
			break;
		case(POSITION.right):
			str = str.replace(/\s+$/, '')
			break;
		case(POSITION.both):
			str = str.replace(/^\s+/, '').replace(/\s+$/, '')
			break;
		case(POSITION.center):
			while (str.match(/\w\s+\w/)) {
				str = str.replace(/(\w)(\s+)(\w)/, `$1$3`)
			}
			break;
		case(POSITION.all):
			str = str.replace(/\s/g, '')
			break;
		default: 
	}
	
	return str;
}

const str = '  s t  r  '
const result = trim(str)
console.log(`|${result}|`) //  |s t  r| 
```

## 三 Object

1 Symbol（for...in/of）
2 继承
3 双向绑定
4 发布订阅模式
5 观察者模式
6 Object.assign
7 EventEmitter

### 1 Symbol（for...in/of）

Symbol: 可以用来表示一个独一无二的变量防止命名冲突.

Symbol作用：

1. 还可以利用 symbol 不会被常规的方法（除了 `Object.getOwnPropertySymbols` 外）遍历到，所以可以用来模拟私有变量。
2. 用来提供遍历接口，布置了 `symbol.iterator` 的对象才可以使用 `for···of` 循环，可以统一处理数据结构。
   调用之后回返回一个遍历器对象，包含有一个 next 方法，使用 next 方法后有两个返回值 value 和 done 分别表示函数当前执行位置的值和是否遍历完毕。

`Symbol.for()` 可以在全局访问 symbol

- for……in : 遍历 key,
- for……of : 遍历 value,

ES6 规定，默认的 Iterator 接口部署在数据结构的 Symbol.iterator 属性。或者说，一个数据结构只要具有Symbol.iterator属性，就可以认为是“可遍历的”（iterable）。
Symbol.iterator 属性本身是一个函数，就是当前数据结构默认的遍历器生成函数。

因为object 没有 Symbol.iterator 属性，所以不能被 for…of 遍历。

```javaScript
	const text = {
		a: 1,
		b: 2,
		c:3
	}
	for(let i of text){
		console.log(i) // 报错：Uncaught TypeError: text is not iterable
	}
```

所以，object想要被 for…of遍历 ，必须在原来的基础上加上 Symbol.iterator 接口属性。

```javaScript
	const text = {
		a: 1,
		b: 2,
		c: 3,
	};
	// 给对象加上 iterator 接口,使之能被 for…of 遍历
	text[Symbol.iterator] = function (){
		const _this = this;
		return {
			index: -1,
			next() {
				const arr = Object.keys(_this);
				if(this.index < arr.length){
					this.index++;
					return {
						value: _this[arr[this.index]],
						done: false,
					}
				} else {
					return {
						value: undefined,
						done: true,
					}
				}
			}
		}
	}

	for(let i of text){
		console.log(i) // 1 2 3 undefined
	}
```

### 2 继承

```javaScript
// 1 原型链继承
// 缺点：
// 问题1：原型中包含的引用类型属性将被所有实例共享；
// 问题2：子类在实例化的时候不能给父类构造函数传参；
function Animal() {
    this.colors = ['black', 'white']
}
Animal.prototype.getColor = function() {
	return this.colors
}
function Dog() {}
Dog.prototype =  new Animal()

let dog1 = new Dog()
dog1.colors.push('brown')
let dog2 = new Dog()
console.log(dog2.colors)  // ['black', 'white', 'brown']

// 2 借用构造函数实现继承
// 借用构造函数实现继承解决了原型链继承的 2 个问题:
// 引用类型共享问题以及传参问题。
// 但是由于方法必须定义在构造函数中，所以会导致每次创建子类实例都会创建一遍方法。
function Animal(name) {
	this.name = name
	this.getName = function() {
		return this.name
	}
}
function Dog(name) {
	Animal.call(this, name)
}
Dog.prototype =  new Animal()

// 3 组合继承
// 组合继承结合了原型链和盗用构造函数，将两者的优点集中了起来。
// 基本的思路是使用原型链继承原型上的属性和方法，而通过盗用构造函数继承实例属性。
// 这样既可以把方法定义在原型上以实现重用，又可以让每个实例都有自己的属性。
function Animal(name) {
	this.name = name
	this.colors = ['black', 'white']
}
Animal.prototype.getName = function() {
	return this.name
}
function Dog(name, age) {
	Animal.call(this, name)
	this.age = age
}
Dog.prototype = new Animal()
Dog.prototype.constructor = Dog

let dog1 = new Dog('奶昔', 2)
dog1.colors.push('brown')
let dog2 = new Dog('哈赤', 1)
console.log(dog2) 
// { name: "哈赤", colors: ["black", "white"], age: 1 }

// 4 寄生式组合继承
// 组合继承已经相对完善了，但还是存在问题:
// 它的问题就是调用了 2 次父类构造函数，第一次是在 new Animal()，第二次是在 Animal.call() 这里。
// 所以解决方案就是不直接调用父类构造函数给子类原型赋值，而是通过创建空函数 F 获取父类原型的副本。
function Animal(name) {
	this.name = name
	this.colors = ['black', 'white']
}
Animal.prototype.getName = function() {
	return this.name
}
function Dog(name, age) {
	Animal.call(this, name)
	this.age = age
}

Dog.prototype =  Object.create(Animal.prototype)
Dog.prototype.constructor = Dog

let dog1 = new Dog('奶昔', 2)
dog1.colors.push('brown')
let dog2 = new Dog('哈赤', 1)
console.log(dog2) 
// { name: "哈赤", colors: ["black", "white"], age: 1 }
```

### 3 双向绑定

```javaScript
<body>
	<input type="text" id="model" />
	<div id="modelText"></div>
</body>

window.onload = function () {
	var user = {
		name: 'aaaa'
	};
	var input = document.querySelector("#model");
	var text = document.querySelector("#modelText");
	input.value = user.name;
	text.textContent = user.name;
	// 数据到视图 model => view
	Object.defineProperty(user, "name", {
		get:function(){
			console.log('获取user')
		},
		set:function(val){
			console.log('修改user')
			input.value = val;
			text.textContent = val;
		}
	})
	// 视图到数据 view => model (可以监听 input 和 keyup)
	input.addEventListener('input', function(val){
		user.name = input.value;
	})
	// 输入框发生的事件流程依次为
	// focus -> keydown -> input -> keyup -> change -> blur
}
```

> [手写bind_手写JavaScript几个方法](https://blog.csdn.net/weixin_39516865/article/details/111388183)
> [github-blog](https://github.com/YvetteLau/Blog/issues/35)

### 4 发布订阅模式

```javaScript
class EventEmitter {
    constructor() {
        this.cache = {};
    }
    on(name, fn) {
        if (this.cache[name]) {
            this.cache[name].push(fn);
        } else {
            this.cache[name] = [fn];
        }
    }
    off(name, fn) {
        let tasks = this.cache[name];
        if (tasks) {
            const index = tasks.findIndex(f => f === fn || f.callback === fn)
            if (index >= 0) {
                tasks.splice(index, 1)
            }
        }
    }
    emit(name, once = false, ...args) {
        if (this.cache[name]) {
            // 创建副本，如果回调函数内继续注册相同事件，会造成死循环
            let tasks = this.cache[name].slice()
            for (let fn of tasks) {
               fn(...args)
            }
            if (once) {
               delete this.cache[name]
            }
        }
    }
}

// 测试
let eventBus = new EventEmitter()
let fn1 = function(name, age) {
	console.log(`${name} ${age}`)
}
let fn2 = function(name, age) {
	console.log(`hello, ${name} ${age}`)
}
eventBus.on('aaa', fn1)
eventBus.on('aaa', fn2)
eventBus.emit('aaa', false, '布兰', 12)
// '布兰 12'
// 'hello, 布兰 12'

```

### 5 观察者模式

```javaScript
class Subject{
	constructor(name){
		this.name = name
		this.observers = []
		this.state = 'XXXX'
	}
	// 被观察者要提供一个接受观察者的方法
	attach(observer){
		this.observers.push(observer)
	}

	// 改变被观察着的状态
	setState(newState){
		this.state = newState
		this.observers.forEach(o=>{
			o.update(newState)
		})
	}
}

class Observer{
	constructor(name){
		this.name = name
	}

	update(newState){
		console.log(`${this.name}say:${newState}`)
	}
}

// 被观察者 灯
let sub = new Subject('灯')
let mm = new Observer('小明')
let jj = new Observer('小健')
 
// 订阅 观察者
sub.attach(mm)
sub.attach(jj)
 
sub.setState('灯亮了来电了')
```

### 6 Object.assign

```javaScript
Object.assign2 = function(target, ...source) {
	if (target == null) {
		throw new TypeError('Cannot convert undefined or null to object')
	}
	let ret = Object(target);
	source.forEach(obj => {
		if (obj != null) {
			for (let key in obj) {
				if (obj.hasOwnProperty(key)) {
					ret[key] = obj[key]
				}
			}
		}
	})
	return ret;
}
```

### 7 EventEmitter

```javaScript
// 发布订阅模式
class EventEmitter {
	constructor() {
		// 事件对象，存放订阅的名字和事件  如:  { click: [ handle1, handle2 ]  }
		this.events = {};
	}
	// 订阅事件的方法
	on(eventName, callback) {
		if (!this.events[eventName]) {
			// 一个名字可以订阅多个事件函数
			this.events[eventName] = [callback];
		} else {
			// 存在则push到指定数组的尾部保存
			this.events[eventName].push(callback);
		}
	}
	// 触发事件的方法
	emit(eventName, ...rest) {
		// 遍历执行所有订阅的事件
		this.events[eventName] &&
			this.events[eventName].forEach(f => f.apply(this, rest));
	}
	// 移除订阅事件
	remove(eventName, callback) {
		if (this.events[eventName]) {
			this.events[eventName] = this.events[eventName].filter(f => f != callback);
		}
	}
	// 只执行一次订阅的事件，然后移除
	once(eventName, callback) {
		// 绑定的时fn, 执行的时候会触发fn函数
		const fn = (...rest) => {
			callback.apply(this, rest) // fn函数中调用原有的callback
			this.remove(eventName, fn) // 删除fn, 再次执行的时候之后执行一次
		}
		this.on(eventName, fn)
	}
}


// test1
const event = new EventEmitter()

const handle = (...pyload) => console.log(pyload)

event.on('click', handle)
event.emit('click', 100, 200, 300, 100)
event.remove('click', handle)
event.once('dbclick', function() {
	console.log('click')
})

event.emit('dbclick', 100);

// test 2
let event = new EventEmitter();

event.on('say',function(str) {
  console.log(str);
});

event.once('say', function(str) {
  console.log('这是once:' + str)
})

event.emit('say','visa');
event.emit('say','visa222');
event.emit('say','visa333');
```

## 四 Promise

1 手写promise
2 实现promise串行
3 实现Promise并发调度
4 JavaScript并发控制
5 手写generator
6 用 setTimeout 模拟 setInterval

### 1 Promise

```javaScript
const STATUS = {
	PENDING: 'pending',
	FULFILLED: 'fulfilled',
	REJECTED: 'rejected'
}
class MyPromise {
	constructor(executor) {
		this._status = STATUS.PENDING; // Promise初始状态
		this._value = undefined; // then回调的值
		this._resolveQueue = []; // resolve时触发的成功队列
		this._rejectQueue = []; // reject时触发的失败队列
	
		// 使用箭头函数固定this（resolve函数在executor中触发，不然找不到this）
		const resolve = value => {
			const run = () => {
				// Promise/A+ 规范规定的Promise状态只能从pending触发，变成fulfilled
				if (this._status === STATUS.PENDING) {
					this._status = STATUS.FULFILLED; // 更改状态
					this._value = value; // 储存当前值，用于then回调
		
					// 执行resolve回调
					while (this._resolveQueue.length) {
						const callback = this._resolveQueue.shift();
						callback(value);
					}
				}
			}
			// 把resolve执行回调的操作封装成一个函数,放进setTimeout里
			// 以实现promise异步调用的特性（规范上是微任务，这里是宏任务）
			setTimeout(run);
		}
	
		// 同 resolve
		const reject = value => {
			const run = () => {
				if (this._status === STATUS.PENDING) {
					this._status = STATUS.REJECTED;
					this._value = value;
		
					while (this._rejectQueue.length) {
						const callback = this._rejectQueue.shift();
						callback(value);
					}
				}
			}
			setTimeout(run);
		}
	
		// new Promise()时立即执行executor,并传入resolve和reject
		executor(resolve, reject);
	}
 
	// then方法,接收一个成功的回调和一个失败的回调
	then(onFulfilled, onRejected) {
		// 根据规范，如果then的参数不是function，则忽略它, 让值继续往下传递，链式调用继续往下执行
		typeof onFulfilled !== 'function' ? onFulfilled = value => value : null;
		typeof onRejected !== 'function' ? onRejected = error => error : null;
	
		// then 返回一个新的promise
		return new MyPromise((resolve, reject) => {
			const resolveFn = value => {
				try {
					const res = onFulfilled(value);
					// 分类讨论返回值,如果是Promise,那么等待Promise状态变更,否则直接resolve
					res instanceof MyPromise ? res.then(resolve, reject) : resolve(res);
				} catch (error) {
					reject(error);
				}
			}
	
			const rejectFn = error => {
				try {
					const res = onRejected(error);
					res instanceof MyPromise ? res.then(resolve, reject) : resolve(res);
				} catch (error) {
					reject(error);
				}
			}
	
			switch (this._status) {
				case STATUS.PENDING:
					this._resolveQueue.push(resolveFn);
					this._rejectQueue.push(rejectFn);
					break;
				case STATUS.FULFILLED:
					resolveFn(this._value);
					break;
				case STATUS.REJECTED:
					rejectFn(this._value);
					break;
			}
		})
	}
 
	catch(rejectFn) {
		return this.then(undefined, rejectFn);
	}
 
	finally(callback) {
	  return this.then(value => MyPromise.resolve(callback()).then(() => value), error => {
			MyPromise.resolve(callback()).then(() => error);
	  })
	}
 
	// 静态resolve方法
	static resolve(value) {
		return value instanceof MyPromise ? value : new MyPromise(resolve => resolve(value));
	}
 
	// 静态reject方法
	static reject(error) {
		return new MyPromise((resolve, reject) => reject(error));
	}
 
	// 静态all方法
	static all(promiseArr) {
		let count = 0;
		let result = [];
		return new MyPromise((resolve, reject) => {
			if (!promiseArr.length) {
				return resolve(result);
			}
			promiseArr.forEach((p, i) => {
				MyPromise.resolve(p).then(value => {
					count++;
					result[i] = value;
					if (count === promiseArr.length) {
						resolve(result);
					}
				}, error => {
					reject(error);
				})
			})
		})
	}
 
	// 静态race方法
	static race(promiseArr) {
		return new MyPromise((resolve, reject) => {
			promiseArr.forEach(p => {
				MyPromise.resolve(p).then(value => {
					resolve(value);
				}, error => {
					reject(error);
				})
			})
		})
	}
}
```

```javaScript
Promise.all = function (iterator) {
	let count = 0;
	let res = [];
	return new Promise((resolve, reject) => {
		for(let i in iterator){
			// 先转化为Promise对象
			Promise.resolve(iterator[i]).then((data) => {
				res[i] = data;
				if(++count === iterator.length){
					resolve(res);
				}
			})
			.catch(e => {
				reject(e);
			})
		}
	})
}

Promise.all = function (iterators) {
	return new Promise((resolve, reject) => {
		if (!iterators || iterators.length === 0) {
			resolve([]);
		} else {
			let count = 0; // 计数器，用于判断所有任务是否执行完成
			let result = []; // 结果数组
			for (let i = 0; i < iterators.length; i++) {
				// 考虑到iterators[i]可能是普通对象，则统一包装为Promise对象
				Promise.resolve(iterators[i]).then(data => {
						result[i] = data; // 按顺序保存对应的结果
						// 当所有任务都执行完成后，再统一返回结果
						if (++count === iterators.length) {
							resolve(result);
						}
					}, err => {
						reject(err); // 任何一个Promise对象执行失败，则调用reject()方法
						return;
					}
				);
			}
		}
	});
};

Promise.race = function (iterators) {  
	return new Promise((resolve,reject) => {
		for (const p of iterators) {
			Promise.resolve(p).then((res) => {
				resolve(res)
			}).catch(e => {
				reject(e)
			})
		}
	})
}

// 1 空数组或者所有 Promise 都是 rejected
// 	则返回状态是 rejected 的新 Promsie，且值为 AggregateError 的错误；
// 2 只要有一个是 fulfilled 状态的，则返回第一个是 fulfilled 的新实例；
// 3 其他情况都会返回一个 pending 的新实例
Promise.any = function(promiseArr) {
    let index = 0
    return new Promise((resolve, reject) => {
        if (promiseArr.length === 0) return 
        promiseArr.forEach((p, i) => {
            Promise.resolve(p).then(val => {
                resolve(val)
                
            }, err => {
                index++
                if (index === promiseArr.length) {
                  reject(new AggregateError('All promises were rejected'))
                }
            })
        })
    })
}

// 1 所有 Promise 的状态都变化了，那么新返回一个状态是 fulfilled 的 Promise
//   且它的值是一个数组，数组的每项由所有 Promise 的值和状态组成的对象；
// 2 如果有一个是 pending 的 Promise，则返回一个状态是 pending 的新实例；
Promise.allSettled = function(promiseArr) {
	let result = []
	return new Promise((resolve, reject) => {
		promiseArr.forEach((p, i) => {
			Promise.resolve(p).then(val => {
					result.push({
						status: 'fulfilled',
						value: val
					})
					if (result.length === promiseArr.length) {
						resolve(result) 
					}
			}, err => {
					result.push({
						status: 'rejected',
						reason: err
					})
					if (result.length === promiseArr.length) {
						resolve(result) 
					}
			})
		})  
	})   
}

// promise 串行
// create Promise
function createPromise(time) {
    return (resolve, reject)=>
    new Promise((resolve, reject)=>{
        setTimeout(()=>{
            console.log('time in' + time)
            resolve();
        }, time * 1000)
    })
}

function iteratorPromise(arr) {
	arr.reduce((pre, next, index, carr) => {
		return pre.then(next)
	}, Promise.resolve())
}

var arr = [createPromise(1), createPromise(3), createPromise(2), createPromise(4)]
iteratorPromise(arr);
```

### 2 实现一个串行promise

```javaScript
function createPromise(time) {
    return (resolve, reject)=>
    new Promise((resolve, reject)=>{
        setTimeout(()=>{
            console.log('time in' + time)
            resolve();
        }, time * 1000)
    })
}

function iteratorPromise(arr) {
	arr.reduce((pre, next, index, carr) => {
		return pre.then(next)
	}, Promise.resolve())
}

var arr = [createPromise(1), createPromise(3), createPromise(2), createPromise(4)]
iteratorPromise(arr);
```

### 3 promise 并发调度

```javaScript
// 实现一个并发请求的函数
var jsonArr = ['api1.json', 'api2.json', 'api3.json', 'api4.json', 'api5.json', 'api6.json']

fetchAll(jsonArr, 2).then(([res1, res2, res3, res4, res5, res6]) => {

})

// 其中 jsonArr 为包含需要请求接口名的数组，limit 为最大并发请求数
function fetchAll(jsonArr, limit) {
	return new Promise((resolve) => {
		// 结果
		let resList = [];
		// 当前正在执行的并发任务队列
		const currPList = [];

		let i = jsonArr.length - 1;
		// 调度任务队列
		const fetcher = async () => {
			while(jsonArr.length > 0) {
				let j = i = i - 1;
				resList[j] = await fetch(jsonArr.shift());
			}
		}

		// 线程池
		while(limit) {
			--limit;
			// 控制当前并发调度队列只有2个
			currPList.push(fetcher());
		}
	
		// 让2个并发队列同时执行
		await Promise.all(currPList);
		resolve(resList);
	});
}
```

### 4 JavaScript 并发调度

该函数接收 3 个参数：

- poolLimit（数字类型）：表示限制的并发数；
- array（数组类型）：表示任务数组；
- iteratorFn（函数类型）：表示迭代函数，用于实现对每个任务项进行处理，该函数会返回一个 Promise 对象或异步函数。

```javaScript
// ES6 版本实现
function asyncPool(poolLimit, array, iteratorFn) {
	let i = 0;
	const ret = []; // 存储所有的异步任务
	const executing = []; // 存储正在执行的异步任务
	const enqueue = function () {
		if (i === array.length) {
			return Promise.resolve();
		}
		const item = array[i++]; // 获取新的任务项
		const p = Promise.resolve().then(() => iteratorFn(item, array));
		ret.push(p);

		let r = Promise.resolve();

		// 当poolLimit值小于或等于总任务个数时，进行并发控制
		if (poolLimit <= array.length) {
			// 当任务完成后，从正在执行的任务数组中移除已完成的任务
			const e = p.then(() => executing.splice(executing.indexOf(e), 1));
			executing.push(e);
			if (executing.length >= poolLimit) {
				r = Promise.race(executing); 
			}
		}
	
		// 正在执行任务列表 中较快的任务执行完成之后，才会从array数组中获取新的待办任务
		return r.then(() => enqueue());
	};
	return enqueue().then(() => Promise.all(ret));
}

// ES7 版本实现
async function asyncPool(poolLimit, array, iteratorFn) {
	const ret = []; // 存储所有的异步任务
	const executing = []; // 存储正在执行的异步任务
	for (const item of array) {
		// 调用iteratorFn函数创建异步任务
		const p = Promise.resolve().then(() => iteratorFn(item, array));
		ret.push(p); // 保存新的异步任务

		// 当poolLimit值小于或等于总任务个数时，进行并发控制
		if (poolLimit <= array.length) {
			// 当任务完成后，从正在执行的任务数组中移除已完成的任务
			const e = p.then(() => executing.splice(executing.indexOf(e), 1));
			executing.push(e); // 保存正在执行的异步任务
			if (executing.length >= poolLimit) {
				await Promise.race(executing); // 等待较快的任务执行完成
			}
		}
	}
	return Promise.all(ret);
}
```

### 5 简易版generator

```javaScript
function myGenerator(list) {
	var index = 0;
	var len = list.length;
	return {
		next: function() {
			var done = index >= len;
			var value = !done ? list[index++] : undefined; 
			return {
				done: done,
				value: value
			}
		}
	}
}

var gen = myGenerator(['大罗', '小罗', 'C罗']);
gen.next(); // {done: false, value: "大罗"}
gen.next(); // {done: false, value: "小罗"}
gen.next(); // {done: false, value: "C罗"}
gen.next(); // {done: true, value: undefined}
```

### 6 用 setTimeout 模拟 setInterval

题目：用 setTimeout 函数 模拟 setInterval

前置条件：

1. 假设 setTimeout 里执行的目标函数返回的是一个 Promise
2. 假设 interval 的值为 3000, 单位为 ms

要求：

1. 不能使用 setInterval 函数
2. 调用 mockInterval 函数时，确保能够立即执行一次目标函数，无需等待 3000ms
3. 当 Promise 执行的时间小于 3000ms 时，需要确保过了 3000ms 之后才能执行下一次 callback
4. 当 Promise 执行的时间超出 3000ms 时，需要等待 Promise 执行完成，才能走回想下一次 callBack 

```javaScript
async function mockInterval(callBack, time=3000) {
	// 为了防止传入的 callBack 是一个异步函数
	await callBack();
	setIntervalFn(callBack, time);
}

function setIntervalFn(callBack, time) {
	setTimeout(() => {
		return new Promise(async function fn(resolve){
			// 为了防止传入的 callBack 是一个异步函数
			await callBack();
			setIntervalFn(callBack, time);
		})
	}, time)
}

// test
mockInterval(() => {
	console.log('call Back fn');
}, 3000)
```

## 五 函数/数组

1 函数柯里化
2 偏函数
3 compose 函数
4 LRU 函数
5 loadsh -> get
6 sleep函数
7 数组随机打乱
8 数组扁平化
9 Array.prototype.fill
10 Array.prototype.map
11 Array.prototype.reduce
12 Array.prototype.forEach
13 Array.prototype.filter
14 Array.prototype.some
15 两数相加
16 两数相减
17 两数相乘
18 两数相除
19 Math.pow()

### 1 函数柯里化

函数柯里化是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

```javaScript
function add(a, b, c) {
    return a + b + c
}
add(1, 2, 3)
let addCurry = curry(add)
addCurry(1)(2)(3)

// 现在就是要实现 curry 这个函数
// 使函数从一次调用传入多个参数变成多次调用每次传一个参数。
function curry(fn) {
	let judge = (...args) => {
		if (args.length == fn.length) return fn(...args)
		return (...arg) => judge(...args, ...arg)
	}
	return judge
}

// 函数柯里化
const curry = (fn, ...args) =>
	args.length < fn.length
		//参数长度不足时，重新柯里化该函数，等待接受新参数
		? (...arguments) => curry(fn, ...args, ...arguments)
		//参数长度满足时，执行函数
		: fn(...args);

function sumFn(a, b, c) {
	return a + b + c;
}
var sum = curry(sumFn);
console.log(sum(2)(3)(5));//10
console.log(sum(2, 3, 5));//10
console.log(sum(2)(3, 5));//10
console.log(sum(2, 3)(5));//10
```

### 2 偏函数

```javaScript
// 偏函数就是将一个 n 参的函数转换成固定 x 参的函数
// 剩余参数（n - x）将在下次调用全部传入
function add(a, b, c) {
    return a + b + c
}
function partial(fn, ...args) {
	return (...arg) => {
		return fn(...args, ...arg)
	}
}
let partialAdd = partial(add, 1)
partialAdd(2, 3)
```

### 3 手写 compose 函数

compose就是执行一系列的任务（函数），比如有以下任务队列，

`let tasks = [step1, step2, step3, step4]`
每一个step都是一个步骤，按照步骤一步一步的执行到结尾，这就是一个compose
compose在函数式编程中是一个很重要的工具函数，在这里实现的compose有三点说明

1 第一个函数是多元的（接受多个参数），后面的函数都是单元的（接受一个参数）
2 执行顺序的自右向左的
3 所有函数的执行都是同步的（异步的后面文章会讲到）

例如
let init = (...args) => args.reduce((ele1, ele2) => ele1 + ele2, 0)
let step2 = (val) => val + 2
let step3 = (val) => val + 3
let step4 = (val) => val + 4

pipe 函数与 compose函数的共同点是都返回“组合函数”，区别则是执行的顺序不同，前者是从左向右执行，后者则是从右向左执行。 实现 pipe函数非常简单，只需要对 compose 函数的包裹顺序进行调整一下即可。

```javaScript
// 解法一 面向过程
const compose = function(...args) {
	let length = args.length
	let count = length - 1
	let result
	return function f1 (...arg1) {
		result = args[count].apply(this, arg1)
		if (count <= 0) {
			count = length - 1
			return result
		}
		count--
		return f1.call(null, result)
	}
}

// 解法二
function compose(...funcs) {
	if (funcs.length === 0) {
		return arg => arg;
	}
	if (funcs.length === 1) {
		return funcs[0];
	}
	return funcs.reduce((a, b) => (...args) => a(b(...args)));
}

const compose = (...args) => x => args.reduceRight((res, cb) => cb(res), x);
const pipe = (...args) => x => args.reduce((res, cb) => cb(res), x)


// 解法三 promise
const compose = function(...args) {
	let init = args.pop()
	return function(...arg) {
		return args.reverse().reduce(function(sequence, func) {
			return sequence.then(function(result) {
				return func.call(null, result)
			})
		}, Promise.resolve(init.apply(null, arg)))
	}
}

// 解法三 Generator

function* iterateSteps(steps) {
	let n
	for (let i = 0; i < steps.length; i++) {
		if (n) {
			n = yield steps[i].call(null, n)
		} else {
			n = yield
		}
	}
}
const compose = function(...steps) {
	let g = iterateSteps(steps)
	return function(...args) {
		let val = steps.pop().apply(null, args)
		// 这里是第一个值
		console.log(val)
		// 因为无法传参数 所以无所谓执行 就是空耗一个yield
		g.next()
		return steps.reverse.reduce((val, val1) => g.next(val).value, val)
  }
}

// 解法四
function compose() {
	var fns = [].slice.call(arguments)
	return function (initialArg) {
		var res = initialArg
		for (var i = fns.length - 1; i > -1; i--) {
			res = fns[i](res);
		}
		return res
	}
}

function pipe() {
    var fns = [].slice.call(arguments)
    return function (initialAgr) {
        var res = initialAgr
        for (var i = 0; i < fns.length; i++) {
            res = fns[i](res)
        }
        return res
    }
}

var greet = function (name) { return 'hi:' + name }
var exclaim = function (statement) { return statement.toUpperCase() + '!' }
var transform = function (str) { return str.replace(/[dD]/, 'DDDDD') }
var welcome1 = compose(greet, exclaim, transform)
var welcome2 = pipe(greet, exclaim, transform)
console.log(welcome1('dot'))//hi:DDDDDOT!
console.log(welcome2('dolb'))//HI:DDDDDOLB!

```

### 4 手写 LRU 函数

运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制 。
实现 LRUCache 类：

1 LRUCache(int capacity) 以正整数作为容量 capacity 初始化 LRU 缓存
2 int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
3 void put(int key, int value) 如果关键字已经存在，则变更其数据值；
	如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，
	它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

进阶：你是否可以在 O(1) 时间复杂度内完成这两种操作？

```text
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```

```javaScript
/**
 * @param {number} capacity
 */
class LRUCache {
	constructor(capacity) {
		this.capacity = capacity
		this.map = new Map();
	}

	get(key) {
		let val = this.map.get(key);
		if (val === undefined) return -1;

		this.map.delete(key); // 因为被用过一次，原有位置删除
		this.map.set(key, val); // 放入最下面表示最新使用
		return val;
	}

	put(key, val) {
		if (this.map.has(key)) this.map.delete(key); // 如果有，删除

		this.map.set(key, val); // 放到最下面表示最新使用

		if (this.map.size > this.capacity) {
			// 这里有个知识点
			// map的entries方法，还有keys方法(可以看mdn))，会返回一个迭代器
			// 迭代器调用next也是顺序返回，所以返回第一个的值就是最老的，找到并删除即可
			this.map.delete(this.map.entries().next().value[0])
		}
	}
}
```

### 5 loadsh -> get

```javaScript
// 题目描述
	var obj = {
		selector: { 
			to: { 
				toutiao: 'FE coder' 
			} 
		},
		target: [1, 2, { name: 'byted' }] 
	};
	// 运行代码
	var res = getValsByKeys(obj, 'selector.to.toutiao', 'selector.feishu.toutiao', 'target[0]', 'target[2].name', 'select.to.douyin')
	console.log(res);
	//  输出结果
	// ['FE coder', '',  1, 'byted', '']

// 解答：
function getValsByKeys() {
	if (arguments.length < 2) {
		return new Error('arguments error');
	}
	const [obj, ...args] = [...arguments];
	const res = [];
	args.forEach(key => {
		key = key.replaceAll('[', '.').replaceAll(']', '');
		res.push(getValByKey(obj, key));
	});
	return res;
}
function getValByKey(obj, key) {
	const arr = key.split('.');
	if (arr.length > 1) {
		return obj[arr[0]] ? getValByKey(obj[arr[0]], arr.slice(1).join('.')) : '';
	} else {
		return obj[key] ? obj[key] : '';
	}
}
```

### 6 sleep函数

```javaScript
function sleep(delay) {
	var start = (new Date()).getTime();
	while ((new Date()).getTime() - start < delay) {
		continue;
	}
}

function test() {
  console.log('111');
  sleep(2000);
  console.log('222');
}

test()
```

### 7 数组随机打乱

```javaScript
function randArr(arr){
	if(!Array.isArray(arr)) {
		throw new Error('type error');
	}
	for(let i = 0; i < arr.length; i++) {
		const randIndex = Math.floor(arr.length * Math.random());
		if (i !== randIndex) {
			[arr[i], arr[randIndex]] = [arr[randIndex], arr[i]];
		}
	}
	return arr;
}
```

### 8 数组扁平化

ES6 为数组实例新增了 flat 方法 用于将数组扁平化。
`var newArray = arr.flat(Infinity)`

```javaScript
	// 手动实现 一个 flat函数：
	// 方式一: ES6 递归 —— reduce
	function flatArray(arr) {
		return arr.reduce((acc, val) => 
			Array.isArray(val) ? 
				acc.concat(flatArray(val)) : acc.concat(val), [])
	}

	// 方式二: 利用栈(stack) 
	function flatArray(arr) {
		const stack = [...arr];
		const newArr = [];
		while (stack.length) {
			const item = stack.pop();
			if (Array.isArray(item)) {
				stack.push(...item);
			} else {
				newArr.unshift(item);
			}
		}
		return newArr;
	}

	// 方法三: ES5 递归
	function flatArray(arr) {
		var result = [];
		for (var i = 0, len = arr.length; i < len; i++) {
			if (Array.isArray(arr[i])) {
					result = result.concat(flatArray(arr[i]))
					// result = [...result, ...flatArray(arr[i])];
			} else {
					result.push(arr[i])
			}
		}
		return result;
	}
```

### 9 Array.fill

```javaScript
// arr.fill(value [,start [,end]])
Array.prototype.myFill = function(val, start = 0, end = this.length){
	if (start < end) {
		this[start] = val;
		this.myFill(val, start + 1, end)
	}
}

Array.prototype.myFill2 = function(val, start = 0, end = this.length) {
	while (start < end) {
		this[start] = val;
		start++;
	}
}
```

### 10 Array.map

核心要点:
1.回调函数的参数有哪些，返回值如何处理。
2.不修改原来的数组。

```javaScript
/*
var new_array = arr.map(function callback(currentValue[, index[, array]]) {
	Return element for new_array 
}[, thisArg])
callback - 生成新数组元素的函数，使用三个参数 currentValue, index(可选), array(可选)
thisArg - 执行 callback 函数时值被用作this
*/
Array.prototype.MyMap = function(fn, thisArg){
	var arr = Array.prototype.slice.call(this);//由于是ES5所以就不用...展开符了
	var mappedArr = [];
	for (var i = 0; i < arr.length; i++ ){
		mappedArr.push(fn.call(thisArg, arr[i], i, this));
	}
	return mappedArr;
}
```

### 11 Array.reduce

核心要点:
1、初始值不传怎么处理
2、回调函数的参数有哪些，返回值如何处理。

```javaScript
// arr.reduce(callback(accumulator, currentValue [,index [,array]])  [,initialValue])

// 简单版本
Array.prototype.myReduce = function(fn, initialValue) {
	// 判断调用对象是否为数组
	if (Object.prototype.toString.call(this) !== '[object Array]') {
		throw new TypeError('not a array');
	}
	// 判断传入的第一个参数是否为函数
	if (typeof fn !== 'function') {
		throw new TypeError(`${fn} is not a function`);
	}
	// 判断调用数组是否为空数组
	if (this.length === 0) {
		if (initialValue) {
			return initialValue;
		} else {
			throw new TypeError('empty array');
		}
	}
	const arr = Array.prototype.slice.call(this);
	const startIndex = initialValue ? 0 : 1;
	let res = initialValue ? initialValue : arr[0];
	for(let i = startIndex; i < arr.length; i++) {
		res = fn.call(null, res, arr[i], i, this);
	}
	return res;
}

Array.prototype.myReduceRight = function(fn, initialValue) {
	const arr = Array.prototype.slice.call(this);
	let startIndex = initialValue ? arr.length - 1 : arr.length - 2;
	let res = initialValue ? initialValue : arr[arr.length - 1];
	while(startIndex > -1) {
		res = fn.call(null, res, arr[startIndex], startIndex, this);
		startIndex--;
	}
	return res;
}

// 完美版本

// 思路
// 1 常规判断 这里我们主要要判断的是调用数组、传入参数
// 2 初始化各个变量，为第一次执行函数做准备
// 	这里我们需要准备的变量有需要传入回调函数的4个参数
// 	callback的4个参数: accumulator, currentValue, currentIndex, sourceArray
// 		accumulator: 累计器累计回调的返回值; 它是上一次调用回调时返回的累积值，或initialValue
// 		currentValue: 数组中正在处理的元素
//			currentIndex(可选): 数组中正在处理的当前元素的索引.如果提供了initialValue，则起始索引号为0，否则从索引1起始
//			sourceArray(可选): 调用reduce()的数组
//		initialValue(可选): 作为第一次调用 callback函数时的第一个参数的值.如果没有提供初始值，则将使用数组中的第一个元素.在没有初始值的空数组上调用 reduce 将报错
// 3 开始循环 同时记得更新几个参数和返回结果
// 4 返回结果
Array.prototype.myreduce = function (fn, initialValue) {
	// 判断调用对象是否为数组
	if (Object.prototype.toString.call(this) !== '[object Array]') {
		throw new TypeError('not a array');
	}
	// 判断调用数组是否为空数组
	const sourceArray = this;
	if (sourceArray.length === 0) {
		throw new TypeError('empty array');
	}
	// 判断传入的第一个参数是否为函数
	if (typeof fn !== 'function') {
		throw new TypeError(`${fn} is not a function`);
	}

	// 第二步 回调函数参数初始化
	let accumulator;
	let currentValue;
	let currentIndex;
	if (initialValue) {
		accumulator = initialValue;
		currentIndex = 0;
	} else {
		accumulator = arr[0];
		currentIndex = 1;
	}

	// 第三步 开始循环
	while (currentIndex < sourceArray.length) {
		if (Object.prototype.hasOwnProperty.call(sourceArray, currentIndex)) {
			currentValue = sourceArray[currentIndex];
			accumulator = fn(accumulator, currentValue, currentIndex, sourceArray);
		}
		currentIndex++;
	}

	// 第四步 返回结果
	return accumulator;
}

// test
const rReduce = ['1', null, undefined, , 3, 4].reduce((a, b) => a + b, 3)
const mReduce = ['1', null, undefined, , 3, 4].myreduce((a, b) => a + b, 3)

console.log(rReduce); // 31nullundefined34
console.log(mReduce); // 31nullundefined34
```

### 12 Array.forEach

```javaScript
// arr.forEach(callback(currentValue [,index [,array]])  [,thisArg])
Array.prototype.forEach2 = function(callback, thisArg) {
	if (this == null) {
		throw new TypeError('this is null or not defined')
	}
	if (typeof callback !== "function") {
		throw new TypeError(callback + ' is not a function')
	}
	const arr = Object(this)  // this 就是当前的数组
	const len = arr.length >>> 0  // 后面有解释(非number 转成 number 类型)
	let index = 0
	while (index < len) {
		if (index in arr) {
			callback.call(thisArg, arr[index], index, arr);
		}
		index++;
	}
}
// O.length >>> 0 是什么操作？就是无符号右移 0 位
/// 就是为了保证转换后的值为正整数。其实底层做了 2 层转换
// 第一是非 number 转成 number 类型
// 第二是将 number 转成 Uint32 类型。
// >>> 1 : 无符号右移 1 位，意思是 除二取整
```

### 13 Array.filter

```javaScript
// var newArray = arr.filter(callback(element [,index [,array]])  [,thisArg])
Array.prototype.filter2 = function(callback, thisArg) {
	if (this == null) {
		throw new TypeError('this is null or not defined')
	}
	if (typeof callback !== "function") {
		throw new TypeError(callback + ' is not a function')
	}
	const arr = Object(this)
	const len = arr.length >>> 0;
	let index = 0;
	const res = []
	while (index < len) {
		if (index in arr) {
			if (callback.call(thisArg, arr[index], index, arr)) {
				res.push(arr[index])                
			}
		}
		index++;
	}
   return res;
}

```

### 14 Array.some

```javaScript
// arr.some(callback(element [,index [,array]]) [,thisArg])
Array.prototype.some2 = function(callback, thisArg) {
	if (this == null) {
		throw new TypeError('this is null or not defined')
	}
	if (typeof callback !== "function") {
		throw new TypeError(callback + ' is not a function')
	}
	const O = Object(this)
	const len = O.length >>> 0
	let k = 0
	while (k < len) {
		if (k in O) {
			if (callback.call(thisArg, O[k], k, O)) {
				return true
			}
		}
		k++;
	}
   return false
}
```

### 15 两数相加

```javaScript
// 考虑 正负数的情况
function preciseAdd(str1, str2) {
	if (arguments.length !== 2) {
		throw new Error('arguments is error');
	}
	let res = '';
	let s1 = str1.split('');
	let s2 = str2.split('');
	if ((str1.startsWith('-') && !str2.startsWith('-')) || (!str1.startsWith('-') && str2.startsWith('-'))) {
		// 一正一负
		const str1LessZero = str1.startsWith('-'); // 表示前面数字是负数，后面是正数
		if (str1LessZero) {
			s1.shift();
		} else {
			s2.shift();
		}
		// 判断两个数字大小
		let s1LagerS2 = s1.length > s2.length;
		if (s1.length === s2.length) {
			for(var i = 0; i < s1.length; i++) {
				if(s1[i] === s2[i]) continue;
				s1LagerS2 = s1[i] > s2[i];
				break;
			}
		}
		// 如果 s1 比 s2 小，则互换
		if (!s1LagerS2) {
			// s1 = [s2, s2 = s1][0] // 快速互换
			const tem = [...s1];
			s1 = [...s2];
			s2 = [...tem];
		}
		while(s1.length) {
			const val = ~~s1.pop() - ~~s2.pop();
			if (val >= 0) {
				res = val + res;
			} else {
				res = val + 10 + res;
				// 不够减，需要把前一个值减一
				s1[s1.length - 1]--;
			}
		}
		res = res.replace(/^0*/g, ''); // 去掉前面的无效0
		// (前正后负，前比后小) || (前负后正，前比后大)
		res = (str1LessZero && s1LagerS2) || (!str1LessZero && !s1LagerS2) ? '-' + res : res;
		// 如果 两数相等，则 res = ''或者 res = '-'
		if(res === '-' || !res) {
			res = '0';
		}
	} else {
		// 两正数 或者 两负数
		// 两个负数
		const twoF = str1.startsWith('-') && str2.startsWith('-');
		if (twoF) {
			s1.shift();
			s2.shift();
		}
		let carry = 0;
		while (s1.length || s2.length || carry) {
			const val = ~~s1.pop() + ~~s2.pop() + carry;
			res = val % 10 + res;
			carry = Math.floor(val / 10);

		}
		res = twoF ? '-' + res : res;
	}
	return res;
}

// 简易版 只考虑正数
function preciseAdd(str1, str2) {
	if (arguments.length !== 2) {
		throw new Error('arguments is error');
	}
	const arr1 = str1.split('');
	const arr2 = str2.split('');
	let res = '';
	let carry = 0;
	while(arr1.length || arr2.length || carry) {
		const val = ~~arr1.pop() + ~~arr2.pop() + carry;
		res = val % 10 + res;
		carry = Math.floor(val / 10);
	}
	return res;
}
```

### 16 两数相减

```javaScript
function minus(a, b) {
	let res = '';

	let arr1 = a.split('');
	let arr2 = b.split('');

	// 判断 a 和 b 的大小
	let aLagerB = arr1.length > arr2.length;
	if (arr1.length === arr2.length) {
		for(let i = 0; i < arr1.length; i++) {
			if(arr1[i] === arr2[i]) {
				continue;
			}
			aLagerB = arr1[i] > arr2[i];
			break;
		}
	}

	// 如果 a 比 b 小，则互换位置
	if (!aLagerB) {
		// arr1 = [arr2, arr2 = arr1][0]
		const tem = arr1;
		arr1 = arr2;
		arr2 = tem;
	}
	while (arr1.length) {
		const val = ~~arr1.pop() - ~~arr2.pop();
		if (val >= 0) {
			res = val + res;
		} else {
			res = val + 10 + res;
			arr1[arr1.length - 1]--;
		}
	}
	res = res.replace(/^0*/g, '');
	res = !aLagerB ? '-'  + res : res;
	// 如果出现两数相等，则 res = '-'
	res = res === '-' ? '0' : res;
	return res;
}
```

### 17 两数相乘

```javaScript
// 普通乘法
function cheng(str1, str2) {
	let res = 0;
	const s1 = ~~str1;
	const s2 = ~~str2;
	for(let i = 0; i < s2; i++) {
		res += s1;
	}
	return res;
}

// 大数相乘 a b 都是字符串
var multiply = function(num1, num2) {
	if (num1 === '0' || num2 === '0') {
		return '0';
	}
	const arr = new Array(num1.length + num2.length).fill(0);
	// 得到乘积的数组
	for(let i = 0; i < num1.length; i++) {
		for(let j = 0; j < num2.length; j++) {
			arr[i + j + 1] += ~~num1[i] * ~~num2[j];
		}
	}
	// 处理进位
	let i = arr.length - 1;
	while(i >= 0) {
		const carry = Math.trunc(arr[i] / 10);
		if (carry) {
			arr[i - 1] += carry;
		}
		arr[i] = arr[i] % 10;
		i--;
	}
	// 去掉前面多余的0
	while(arr[0] == 0) {
		arr.shift();
	}
	return arr.join('');
};
```

### 18 两数相除

给定两个整数，被除数 dividend和除数divisor。将两数相除，要求不使用乘法、除法和 mod 运算符。
返回被除数 dividend 除以除数divisor 得到的商。

```javaScript
var divide = function (dividend, divisor) {
	let result = 0;
	const flag = (dividend > 0 && divisor > 0) || (dividend < 0 && divisor < 0);
	let mul = 1;

	let s1 = Math.abs(dividend);
	let s2 = Math.abs(divisor);
	let s3 = s2;

	while (s1 >= s3) {
		if (s1 > (s3 + s3)) {
			s3 += s3;
			mul += mul;
		}
		s1 -= s3;
		result += mul;
	}
	while (s1 >= s2) {
		s1 -= s2;
		result += 1;
	}

	if (flag && result > (Math.pow(2, 31) - 1)) {
		return Math.pow(2, 31) - 1;
	} else if (!flag && result < -Math.pow(2, 31)) {
		return -Math.pow(2, 31);
	}
	return !flag ? -result : result;
};
```

### 19 Math.pow()

```javaScript
// 解法一: 迭代
// res 变量的值由 奇次or偶次幂 决定，如果是奇次幂，res 值为 num，反之，为1
// res 最后乘上累乘后的 num，返回
var myPow = function (num, power) {
	if (power < 0) return 1 / num * myPow(1 / num, -(power + 1));
	if (power === 0) return 1;
	if (power === 1) return num;
	// 以上分别为power小于0 等于0 等于1 的情况
	let res = 1
	while (power > 1) { // power大于1
		if (power % 2 === 1) {
			res = res * num;
			power--;
		}
		num = num * num;
		power = power / 2;
	}
	return res * num;
};

// 解法二: 递归
// 递归版似乎更好理解一些，奇次幂的话，幂次-1，转成偶次幂
// 只需要写好偶次幂下的调用就好：myPow(num * num, power / 2)
var myPow = (num, power) => {
	if (power < 0) return 1 / (num * myPow(num, -(power + 1)));
	if (power === 0) return 1;
	if (power === 1) return num;
	return power % 2 === 1 ?
		num * myPow(num, power - 1) :
		myPow(num * num, power / 2);
}
```

## 六 开放题

1 实现一个简单路由
2 手写时钟
3 手写红绿灯

### 1 实现一个简单路由

```javaScript
// hash路由
class Route{
	constructor(){
		// 路由存储对象
		this.routes = {};
		// 当前hash
		this.currentHash = '';
		// 绑定this，避免监听时this指向改变
		this.freshRoute = this.freshRoute.bind(this);
		// 监听
		window.addEventListener('load', this.freshRoute, false);
		window.addEventListener('hashchange', this.freshRoute, false);
	}
	// 存储
	storeRoute (path, cb) {
		this.routes[path] = cb || function () {};
	}
	// 更新
	freshRoute () {
		this.currentHash = location.hash.slice(1) || '/';
		this.routes[this.currentHash]();
	}
}

```

### 2 手写时钟

```html
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8">
		<title>Title</title>
		<style>
			*{
				padding: 0;
				margin: 0;
			}
			html, body {
				height: 100%;
			}
			.warp{
				width: 230px;
				height: 230px;
				margin: 50px auto;
			}
			.clock{
				width: 200px;
				height: 200px;
				border-radius: 115px;
				border: 15px solid #ccc;
				background: #fff;
				position: relative;
			}
			.number div{
				position: absolute;
				width: 20px;
				height: 200px;
				left: 90px;
			}
			.number span{
				display: block;
				width: 20px;
				height: 20px;
				text-align: center;
			}
			.pointer{
				position: absolute;
				bottom: 90px;
				transform-origin: 50% 90%;
				-webkit-transform-origin: 50% 90%;
			}
			.houre{
				width: 4px;
				height: 60px;
				left: 100px;
				background: #000;
			}
			.minute{
				width: 2px;
				height: 80px;
				left: 100px;
				background: #777;
			}
			.second{
				width: 1px;
				height: 90px;
				left: 100px;
				background: #ff0000;
			}
		</style>
		<script>
			window.onload = () => {
				var domNumber = document.getElementById("number");
				var domDiv = domNumber.getElementsByTagName("div");
				var domSpan = domNumber.getElementsByTagName("span");
		
				// 布局 => 让数字旋转到相应的位置并调整方向
				for(var i = 0; i < domDiv.length; i++){
					domDiv[i].style.WebkitTransform="rotate(" + (i + 1) * 30 + "deg)";
				}

				for(var j = 0; j < domSpan.length; j++){
					domSpan[j].style.WebkitTransform="rotate("+ (j + 1) * -30 + "deg)";
				}
	
				function clockRun(){
					var domHour = document.getElementById("houre");
					var domMinute = document.getElementById("minute");
					var domSecond = document.getElementById("second");

					var nowTime = new Date();
					var nowHour = nowTime.getHours();
					var nowMinute = nowTime.getMinutes();
					var nowSecond = nowTime.getSeconds();

					// 计算指针的角度，其中最重要的是在不满一小时或不满一分钟时，时针或分针应该转多少度
					var houreDeg = (nowMinute/60) * 30; // 每小时是30度(1h = 60min, 1h =>  360°/12h = 30 °/h)
					var minuteDeg = (nowSecond/60) * 6; // 每分钟是6度(1min = 60sec, 1min => 360°/60min = 6 °/h)

					domHour.style.WebkitTransform = "rotate(" + (nowHour * 30 + houreDeg) + "deg)";
					domMinute.style.WebkitTransform = "rotate(" + (nowMinute * 6 + minuteDeg) + "deg)";
					domSecond.style.WebkitTransform = "rotate(" + (nowSecond * 6) + "deg)";
				}
				clockRun();
				setInterval(clockRun, 1000);
			}
		</script>
	</head>
	<body>
		<div class="warp" >
			<div class="clock" >
				<div id="number" class="number">
					<div><span>1</span></div>
					<div><span>2</span></div>
					<div><span>3</span></div>
					<div><span>4</span></div>
					<div><span>5</span></div>
					<div><span>6</span></div>
					<div><span>7</span></div>
					<div><span>8</span></div>
					<div><span>9</span></div>
					<div><span>10</span></div>
					<div><span>11</span></div>
					<div><span>12</span></div>
				</div>
				<div id="houre" class="pointer houre" ></div>
				<div id="minute" class="pointer minute" ></div>
				<div id="second" class="pointer second" ></div>
			</div>
	</div>
	</body>
</html>
```

### 3 手写红绿灯

```html
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8">
		<title>Title</title>
		<style>
			.light-content {
				width: 400px;
				height: 150px;
				border: 1px solid #666;
				border-radius: 20px;
				background: #f7f7f7;
				display: flex;
				justify-content: space-around;
				align-items: center;
			}
			#red-light, #yellow-light, #green-light {
				width: 100px;
				height: 100px;
				border-radius: 50px;
				display: flex;
				align-items: center;
				justify-content: center;
				font-size: 35px;
			}
		</style>
		<script>
			window.onload = () => {
				let red = document.getElementById('red-light');
				let yellow= document.getElementById('yellow-light');
				let green = document.getElementById('green-light');

				let colorsChange = (color, duration) => {
					if(color === 'red') {
						// 设置灯的颜色变化
						red.style.background = color;
						green.style.background = '#E2DCDC';
						yellow.style.background = '#E2DCDC';
						// 设置倒计时
						green.innerText = 0;
						yellow.innerText = 0;
						let timeOut = duration;
						red.innerText = timeOut / 1000;
						let times = setInterval(() => {
							red.innerText = (timeOut - 1000) / 1000;
							timeOut -= 1000;
							if (timeOut === 0) {
								clearInterval(times);
							}
						}, 1000);
					}
					if(color === 'yellow') {
						// 设置灯的颜色变化
						yellow.style.background = color;
						green.style.background = '#E2DCDC';
						red.style.background = '#E2DCDC';
						// 设置倒计时
						red.innerText = 0;
						green.innerText = 0;
						let timeOut = duration;
						yellow.innerText = timeOut / 1000;
						let times = setInterval(() => {
							yellow.innerText = (timeOut - 1000) / 1000;
							timeOut -= 1000;
							if (timeOut === 0) {
								clearInterval(times);
							}
						}, 1000);
					}
					if(color === 'green') {
						// 设置灯的颜色变化
						green.style.background = color;
						red.style.background = '#E2DCDC';
						yellow.style.background = '#E2DCDC';
						// 设置倒计时
						yellow.innerText = 0;
						red.innerText = 0;
						let timeOut = duration;
						green.innerText = timeOut / 1000;
						let times = setInterval(() => {
							green.innerText = (timeOut - 1000) / 1000;
							timeOut -= 1000;
							if (timeOut === 0) {
								clearInterval(times);
							}
						}, 1000);
					}
				};

				let setColor = (color, duration) => {
					return new Promise((res,rej) => {
						colorsChange(color, duration);
						setTimeout(res, duration);
					})
				}

				async function setLight() {
					await setColor('red', 7000);
					await setColor('yellow', 3000);
					await setColor('green', 5000);
					await setLight();
				}
				setLight();
			}
		</script>
	</head>
	<body>
		<div class="light-content">
			<div id="red-light"></div>
			<div id="yellow-light"></div>
			<div id="green-light"></div>
		</div>
	</body>
</html>
```
