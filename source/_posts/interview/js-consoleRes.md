---
layout: js
title: 打印结果
tags: [js,interview,consoleResult]
---

### 1. 打印结果 (Object再次赋值)

```javaScript
    function changeObjProperty(o) {
        o.siteUrl = "http://www.baidu.com";
        o = new Object();
        o.siteUrl = "http://www.google.com";
    }
    let webSite = new Object();
    changeObjProperty(webSite);
    console.log(webSite.siteUrl);
```

解答: `webSite` 属于复合数据类型，函数参数中以地址传递，修改值会影响到原始值， 但如果将其完全替换成另一个值，则原来的值不会受到影响.
因为当执行到 `o = new Object();` 时，相当于执行 `var o = new Object();`，即从新定义了一个对象，函数参数传递的对象不受影响。 因此打印结果是`"http://www.baidu.com"`;

### 2. 执行结果

```javaScript
    const a = { b: 3 };
    function foo(obj) {
        obj.b = 5;
        return obj;
    }
    const aa = foo(a);

    console.log(a.b);
    console.log(aa.b);
    console.log(a === aa);
```

解答:  函数参数中以地址传递，修改值会影响到原始值, 因此 a 与 aa 都指向同一个对象

### 3. 执行结果

```javaScript
    function Ofo() {}
    function Bick() {
        this.name = 'mybick';
    }

    var m1 = new Ofo();
    Ofo.prototype = new Bick();
    var m2 = new Bick();
    var m3 = new Ofo();

    console.log(m1.name);
    console.log(m2.name);
    console.log(m3.name);
```

解答:
m1 = new Ofo() => m1.construetor = function Ofo() {}, 因此 m1.name = undefined
m2 = new Bick() => m1.construetor = function Bick() { this.name = 'mybick'}, 因此 m2.name = 'mybick'
m3 = new Ofo() => 由于 Ofo.prototype = new Bick(), 因此 m3.__proto__ = Bink {name: 'mybick'}, m3.name = 'mybick'

### 4. 执行结果

```javaScript
'use strict'
var x = 0;
class C {
	constructor() {
		this.x = 3;
	}
	method() { console.log(this.x); }
}
class D {
	constructor() {
		this.x = 4;
	}
	method = () => { console.log(this.x); }
}
const a = { 
	x: 1,
	method() { console.log(this.x); }
}
const b = {
	x: 2,
	method() { console.log(this.x); }
}
const c = new C();
const d = new D();
const e = {
	x: 5,
	method() { console.log(x); }
}

a.method();
b.method = a.method;
b.method();

const fc = c.method.bind(a);
fc();

const fd = d.method.bind(a);
fd();

const fe = e.method.bind(a);
fe();
```

解答： 1 2 1 4 0

### 5. 执行结果

```javaScript
fun();
var fun;
function fun () {
  console.log(1);
}
var fun = function () {
  console.log(2);
}
fun();
```

解答： 1 2

### 6. 执行结果

```javaScript
    function Page(){
        return this.hosts;
    }

    Page.hosts = ['h1'];
    Page.prototype.hosts = ['h2'];

    var p1 = new Page();
    var p2 = Page();

    console.log(p1.hosts);
    console.log(p2.hosts);
```

p1: new 的时候首先判断构造函数是否有返回值
  如果返回值是 Object Array Function，则实例对象就是这个返回对象.
  因此 var p1 = new Page() 时，Page构造函数返回的是 this.hosts
  构造函数没有 hosts，沿着原型链找到了 ['h2'], 因此 p1 = ['h2']
  结论是：p1 = ['h2'], p1.hosts = undefined
p2: p2 = Page() = this.hosts, 此时this = window
  因此 p2 = p2 = Page() = this.hosts = undefined
  结论是：p2 = undefined, p2.hosts = Uncaught TypeError: Cannot read property 'hosts' of undefined

### 7. 执行结果

```javaScript
function foo () {
    getName = function () {
        console.log(1);
    }
    return this;
}
foo.getName = function () {
    console.log(2);
}
foo.prototype.getName = function () {
    console.log(3);
}
var getName = function () {
    console.log(4);
}
function getName () {
    console.log(5);
}

foo.getName();
getName();
foo().getName();
getName();
new foo().getName();
new new foo().getName();
```

题解：

1. foo.getName(): 给 function foo 赋值了一个属性,该属性的key是 getName, value是一个function(){console.log(2)}
   因此 执行结果是 2
2. getName(): 这里由于遍历提升, var getName = function() { console.log(4)} 比 function getName () {console.log(5) } 后提升
   因此最后执行的是 var getName = function() { console.log(4)}, 执行结果是 4
3. foo().getName(): 执行了foo(), 该函数中的getName 没有用 var/let/const 定义，因此是覆盖了全局的 getName 方法
   此时全局的 getName = function(){console.log(1)}, 因此执行结果是 1
4. getName(): 经过foo().getName()之后，全局的 getName 已经被覆盖，因此执行结果是 1
5. new foo().getName(): new foo() 返回一个空的实例对象, 执行该对象的getName() 需要沿着原型链查找
   因此执行了 foo.prototype.getName = function () {console.log(3)}, 执行结果是 3
6. new new foo().getName(): 这一部比较复杂，需要拆分为3步
   1. 执行 new foo() —— 返回一个foo构造函数的实例对象(空对象)，并且该实例对象的原型链上有个属性 foo.prototype.getName = function () {console.log(3)}
   2. 执行 new foo().getName —— 返回该实例对象原型链上的getName属性: function () {console.log(3)}
   3. 执行 new new foo().getName() —— 等同于 new function () {console.log(3)}, 返回该构造函数的实例对象(一个空对象)，但是却执行了 console.log(3), **因此输出 3**

### 8. 执行结果

```javaScript
console.log('1')

async function async1() {
	console.log('2');
	await async2()
	console.log('3');
}
async function async2() {
	console.log('4');
}
async1()

setTimeout(function() {
	console.log('5 - setTimeOut')
}, 0)

requestAnimationFrame(() => console.log('10'));
requestIdleCallback(() => console.log('11'));

new Promise(resolve => {
	console.log('6')
	resolve()
}).then(function() {
	console.log('7')
})
.then(function() {
	console.log('8')
})

setTimeout(function() {
	console.log('12')
	setTimeout(function() {
		console.log('13')
	}, 0)
	Promise.resolve().then(() => console.log('14'))
}, 0)

console.log('9')

// 1 2 4 6 9 3 7 8 undefined 10 5 12 14 11 13
```
