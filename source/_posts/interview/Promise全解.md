---
layout: Promise
title: Promise全解
## date: 2018-02-08 21:48:29
tags: [Promise,interview]
keywords: 博客文章密码
## password: 123123
message:  输入密码，查看文章
---

## 一、Promise规范

Promise/A+兼容扩展了Promise/A而来，ES6里的Promise遵守了Promise/A+规范，也就是当今的标准规范。
  
1. 一个 Promise 必然处于以下几种状态之一：
　　· **pending**（待定/等待）: 初始状态，既没有被兑现，也没有被拒绝。
　　· **fulfilled**（已兑现/已完成）: 意味着操作成功完成。
　　· **rejected**（已拒绝）: 意味着操作失败。
2. 一个Promise对象的状态只能从`pending -> fulfilled`, 或者`pending -> rejected`状态，不能逆向转换，也不能互相转换。
3. 一个Promise必须实现then方法（then是Promise的核心），而then必须返回一个Promise，同一个Promise的then可以调用多次，执行顺序跟它们的定义顺序一致。

4. then方法接收两个参数: resolve和reject，第一个参数是成功的回调，第二个是失败的回调。同时，then可以接收另一个Promise传入，也接收一个“类then”的对象或方法，即thenable对象。

> [MDN-Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

## 二、手写一个Promise

- 实现**Promise类**类

```javascript
/**
 * 1、 定义status状态
 * 2、 定义fn1 fn2 数组
 * 3、 定义 resolve reject 方法
 * 4、 executor执行
 */
function Promise(executor) {
  let self = this;

  // 初始化 status 为等待状态
  self.status = 'pending';

  self.fn1Callback = [];
  self.fn2Callback = [];

  // resolve 做的事情：
  // 1. 修改 this 实例的状态
  // 2. 修改 this 实例的data
  // 3. 遍历执行this fn1Callback 上挂载的方法
  function resolve(value) {
    if(value instanceof Promise) {
	  return value.then(resolve, reject);
	}
	// 异步执行所有的回调函数
	setTimeout(() => {
	  if(self.status === 'pending') {
		self.status = 'resolved';
		self.data = value;
		for(let i = 0; i < self.fn1Callback.length; i++) {
			self.fn1Callback[i](value);
		}
	  }
	})
  }
	
  // reject 做的事情
  function reject(reason) {
	// 异步执行所有的回调函数
	  if (self.status === 'pending') {
		self.status = 'rejected';
		self.data = reason;
		for(let i = 0; i < self.fn2Callback.length; i++) {
		  self.fn2Callback[i](reason);
		}
	  }
	})
  }

  // 如果 executor 执行报错, 直接执行reject
  try {
    executor(resolve, reject);
  } catch (err) {
    reject(err);
  }
}
```

- 实现**then**方法

  - Promise对象有一个then方法，用来注册在这个Promise状态确定后的回调;
  - 当Promise状态发生了转变，不论成功或者失败都会调用then方法;
  - then方法时在Promise实例上调用，因此then方法的实现是在Promise的prototype上;
  - then方法会返回一个Promise，而且是返回一个新的Promise对象

![PromiseThen.png](https://i.loli.net/2020/12/13/vBFdAnoX2OrNb14.png)

```javascript
Promise.prototype.then = function(fn1, fn2) {
  var self = this;
  var promose2;

  // 首先对fn1 fn2 做判断
  fn1 = typeof fn1 === 'function' ? fn1 : function(v) { return v; };
  fn2 = typeof fn2 === 'function' ? fn2 : function(r) { throw r; };

  // 执行到 then, 并不确定 Promise的状态
  if (self.status === 'pending') {
    return promise2 = new Promise((resolve, reject) => {
	  // 先定义一个方法，把方法挂载到 onResolvedCallback 数组上
	  // 方法里面就是调用传入的 fn1
	  self.onResolvedCallback.push(value => {
		try {
		  let x = fn1(value);
		  resolvePromise(promise2, x, resolve, reject);
		} catch(r) {
		  reject(r);
		}
	  })

	  self.onRejectedCallback.push(reason => {
	    try {
		  let x = fn2(reason);
		  resolvePromise(promise2, x, resolve, reject)
		} catch(r) {
		  reject(r);
		}
	  })
	})
  }

  if (self.status === 'resolved') {
	// then 执行后， 返回一个Promise
	return promise2 = new Promise((resolve, reject) => {
	  // 异步执行onResolved
	  setTimeout(() => {
		try {
		  // 执行 fn1(), 拿到结果
		  // fn1 是用户传入的, 所以fn1的返回值 就可能有很多种,因此封装到 resolvePromise处理
		  let x = fn1(self.data);
		  resolvePromise(promise2, x, resolve, reject);
		} catch (err) {
			reject(reason);
		}
	  })
	})
  }

  if (self.status === 'rejected') {
    return promise2 = new Promise((resolve, reject) => {
      setTimeout(() => { // 异步执行onRejected
        try {
          let x = fn2(self.data);
          resolvePromise(promise2, x, resolve, reject);
        } catch (reason) {
          reject(reason);
        }
      })
    })
  }
}

// 1. 普通值
// 2. promise 值
// 3. thenable 的值，执行 then
function resolvePromise(promise2, x, resolve, reject) {
  // 为了防止循环引用
  if (promise2 === x) {
	return reject(new TypeError('Chaining cycle detected for promise!'));
  }
  // 如果 x 是 promise
  if (x instanceof Promise) {
	x.then(function (data) {
	  resolve(data)
	}, function (e) {
	  reject(e)
	});
	return;
  }
  // 如果 x 是 object 类型或者是 function
  if ((x !== null) && ((typeof x === 'object') || (typeof x === 'function'))) {
	// 拿x.then可能会报错
	try {
	   // 先拿到 x.then
	    var then = x.then
	    var called;
	    if (typeof then === 'function') {
		    // 这里的写法，是 then.call(this, fn1, fn2)
		    then.call(x, (y) => {
				// called 是干什么用的呢？
				// 有一些 promise 实现的不是很规范，瞎搞的，比如说，fn1, fn2 本应执行一个，
				// 但是有些then实现里面，fn1, fn2都会执行
				// 为了 fn1 和 fn2 只能调用一个, 设置一个 called 标志位
				if (called) {
				return;
				}
				called = true;
				return resolvePromise(promise2, y, resolve, reject);
			}, (r) => {
				if (called) {
					return;
				}
				called = true;
				return reject(r);
			});
		} else {
			resolve(x);
		}
	} catch (err) {
	  if (called) {
        return;
	  }
	  return reject(e);
	}
  } else {
	  resolve(x);
  }
}
```

> [可能是目前最易理解的手写promise](https://mp.weixin.qq.com/s/oURuka-Qgbbj8JKtlYNMaw)

## 三、手写Promise.all / Promise.retry

**`Promise.all`**

```javascript
// 只要有一个 promise 失败即返回失败的结果
Promise.all = function (arr) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(arr)) {
	  throw new Error('argument must be a array');
	}
	let dataArr = [];
	let num = 0;
	for(let i = 0; i < arr.length; i++) {
	  let p = arr[i];
	  p.then(data => {
	    dataArr.push(data);
	    num ++;
	    if (num === arr.length) {
	      return resolve(dataArr);
	    }
	  }).catch(e => reject(e))
	}
  })
}
```

**`Promise.retry`**

```javascript
// retry 是报错会尝试，尝试超过一定次数才真正的 reject
Promise.retry = function(getData, times, delay) {
  return new Promise((resolve, reject) => {
	function attemp() {
	  getData().then(data => {
		resolve(data);
	  }).catch(err => {
		if (times === 0) {
		  reject(err);
		} else {
		  times --;
		  setTimeout(attemp, delay);
		}
	  })
	}
	attemp();
  })
}
```

## 四、promise 题

1

```javascript
	new Promise(resolve => {
		resolve(a)
	}).then(result => {
		console.log(`成功：${result}`)
		return result * 10
	}).then(result => {
		console.log(`成功：${result}`)
	},reason => {
		console.log(`失败：${reason}`)
	}).catch(e => {
		console.log(`catch：${e}`)
	})
```

2

```javascript
console.log('1');

async function async1() {
	await async2();
	console.log('2');
}
async function async2() {
	console.log('3');
}

async1();

setTimeout(function() {
	console.log('4');
}, 0)

new Promise(resolve => {
	console.log('5');
	resolve();
}).then(function() {
	console.log('6');
}).then(function() {
	console.log('7')
})

console.log('8')

// script start -> Promise -> script end -> async2 end -> promise1 -> async1 end -> promise2 -> setTimeout
```
