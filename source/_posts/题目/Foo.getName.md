---
layout: js
title: js - Foo.getName()
## date: 2020-12-01 14:46:32
tags: [interview, 手写]
---

```javascript
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

关于 `new new foo().getName()`:  
1、执行优先级： `new foo() >  foo() > new foo `
2、拆解步骤：
	（1）`var a = new foo();`  
		=> `new a.getName()`    
	（2）`new a.getName();`


```javascript
let promise1 = Promise.resolve()
    .then(res => console.log(1))
    .then(res => console.log(2))

let promise2 = new Promise(resolve => {
    setTimeout(() => {
        console.log(6)
        resolve()
    })
}).then(res => console.log(3))

async function main() {
    console.log(4)
    console.log(await Promise.all([promise2, promise1]))
    console.log(5)
    return { obj: 5 }
}

let promise3 = Promise.resolve()
    .then(res => console.log(8))
    .then(res => console.log(9))

console.log(typeof main())
```
