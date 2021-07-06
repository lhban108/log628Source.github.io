---
layout: 链表
title: 浏览器与JS运行机制
## date: 2020-12-01 14:46:32
tags: [网络,浏览器]
---

## 一、JavaScript预解析

JavaScript代码运行分为两个阶段：

- (1) 预解析

所有函数定义提前，函数体提升(当然不包括如var box = function() {} )
形参声明并赋值
变量声明(不赋值)

- (2) 执行

按照js运行机制从，从上到下执行

## 二、进程与线程

- **进程是cpu资源分配的最小单位**（是能够拥有资源和独立运行的最小单位）
- **线程是cpu调度的最小单位**（线程是建立在进程的基础上的一次程序运行单位，一个进程可以有多个线程

举例：此处有多个工厂，每个工厂有1个或多个工人。此时**工厂就好比进程**，有单独专属自己的工厂资源；**工人就好比是线程**，多个工人在工厂中写作工作。工厂的空间是工人们共享的，这象征一个进程的内存空间是共享的，每个线程都可以共享内存。并且每个工厂之间相互独立存在。
![进程和线程.png](https://i.loli.net/2020/12/02/fvhtrWzeMkpCR15.png)

- 应用程序必须运行在某个进程的某个线程上
- 一个进程至少有一个运行的线程：主线程，进程启动后自动创建

## 三、浏览器进程

浏览器内核是指支持浏览器运行的最核心的部分，分为渲染引擎和JS引擎。现在JS引擎比较独立，内核更加倾向于说渲染引擎

**(1)浏览器内核分类**

- Chrome、Safari： Webkit (Bink)
- Firefox：Gecko
- IE：Trident
- 360、搜狗等国内浏览器：Trident+Webkit
- ...

**(2)浏览器进程**

+ 浏览器是多进程的
+ 浏览器之所以能运行，是因为系统给它的进程分配了资源（cpu、内存）
+ 简单来说，每打一个Tab页，就相当于创建了一个独立的浏览器进程

浏览器进程的组成：

- **Browser进程**
浏览器的主进程，负责协调、主控，只有一个。
负责内容：浏览器页面显示；与用户交互（前进、后退等）；网络资源的管理、下载；各个页面的管理，创建和销毁其他进程等
- **第三方插件进程**
   每种类型的插件对应一个进程，仅当插件使用时才创建
- **GPU进程**
 最多一个，用于3D绘制等
- **浏览器渲染进程**（浏览器内核，Renderer进程，内部是多线程的）
 默认 每个Tab页面一个进程，互不影响
 负责内容：页面渲染；脚本执行；事件处理

浏览器是多线程的优势：避免单个Tab页崩溃或单个插件崩溃影响其他整个浏览器，可以充分多核优势，方便使用沙盒模型隔离插件等进程，提高浏览器的稳定性。缺点是，内存和cpu消耗会更大，有点空间换时间的意思。

Borwser进程与浏览器内核（Renderer进程）的通信过程：

- Browser进程收到用户请求，首先需要获取页面内容（譬如通过网络下载资源），随后将该任务通过RendererHost接口传递给Render进程
  - 渲染线程接收请求，加载网页并渲染网页，这其中可能需要Browser进程获取资源和GPU进程来帮助渲染
  - 当然可能会有JS线程操作DOM（可能会造成回流并重绘）
  - 最后Renderer进程将结果传递给Browser进程
- Renderer进程的Renderer接口收到消息，简单解释后，交给渲染线程，然后开始渲染
- Browser进程收到结果并将结果绘制出来

## 四、浏览器渲染进程

对于前端操作来说 ，最重要的是渲染进程，并且**渲染进程也是多线程的**。
渲染进程包含哪些线程？

- **GUI渲染线程**
  - 负责渲染浏览器页面，解析HTML、CSS，构建DOM树和RenderObject树，布局和绘制等
  - 负责重绘（Repaint）和回流（Reflow）
  - GUI渲染线程和JS引擎线程是互斥的，当JS引擎执行时GUI线程会被挂起，GUI线程会保存在一个队列里等 js引擎空闲时执行。
- **JS引擎线程**
  - 负责处理JavaScript脚本，执行代码
- **事件触发线程**
  - 主要负责将准备好的事件交给JS引擎线程执行
比如setTimeout定时器计数结束、ajax等异步请求成功并触发回调函数、用户触发点击事件等，该线程会将整装待发的事件加入到任务队列的队尾，等待JS引擎线程的执行。
- **定时器触发线程**
  - 主要负责异步定时器一类的函数处理，如setTimeout、setInterval
主线程依次执行代码时，遇到定时器，会将定时器交给该线程处理。当计数完毕后，事件触发线程会将计数完毕的事件加入到任务队列的尾部，等待JS引擎线程执行。
- **异步HTTP请求线程**
  - 负责执行异步请求一类的函数，如：ajax、axios、promise等
主线程依次执行代码是，遇到异步请求，会将异步请求函数交给该线程处理。当监听到状态码变更，如果有回调函数，事件触发线程会将回调函数加入到任务队列的尾部，等待 JS引擎线程执行。

## 五、事件循环

### **1 浏览器中的事件循环**

JavaScript语言是单线程的，意思是同一时间只能做一件事。后来为了有效利用多核CPU的计算能力，HTML5提出Web Server标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，并且子线程不能操作DOM。所以新标准并没有改变JavaScript单线程的本质。

简单描述JS的执行机制：

1. 首先判断JS是同步任务还是异步任务，同步任务就进入主线程执行，异步任务进入event table
2. 异步任务在event table中注册函数，异步函数又分为宏任务(macro-task)和微任务(micro-task)，当满足触发条件后，宏任务被推入宏任务队列(macro-task queue)，微任务被推入微任务队列(micro-task queue)
3. 同步任务在主线程中一直执行，直到同步任务执行完毕，主线程空闲空闲时，才去微任务队列(micro-task queue)中查看是否有可执行的异步任务，如果有就推入主线程中执行
4. 直到全部微任务依次执行完毕后，主线程空闲，再去宏任务队列(macro-task queue) 查看是否有可执行的异步任务，如果有就推入主线程中执行
以上四步循环执行，就是event loop。  

一个完整的Event Loop过程：

① 所有的同步任务都在主线程上执行，形成一个执行栈(exection context stack)，我们可以认为执行栈是一个函数调用的栈结构，遵循先进后出的原则。除了主线程的执行栈，还存在一个任务队列(task queue)，任务队列分为宏任务队列(macro-task queue)和微任务队列(micro-task queue)。  
一开始执行栈为空，宏任务队列(macro-task queue)里只有一个script代码(整体代码)，微任务队列(micro-task queue)队列为空。  
② 宏任务队列(macro-task queue)中的全局上下文(script标签)会被推入执行栈，同步代码执行。在执行的过程中会判断是同步任务还是异步任务，同步任务依次执行，异步任务会通过对一些接口的调用而产生新的macro-task和micro-task(只要异步任务有了运行结果，就会在对应的任务队列中放置一个事件，等待调用)。同步代码执行完了，script脚本会行和出队的过程。  
③ 上一步出队的是一个macro-task，这一步要处理的是micro-task。需要注意的是，当macro-task出队时，任务是一个一个执行的，而micro-task出队时，任务是一队一队执行的。因此，我们处理micro-task这一步，会逐个执行队列中的任务并把它出队，直到队列被清空。  
④ 执行渲染操作，更新页面  
⑤ 检查是否存在Web worker任务，如果有，则对其进行进行处理  
⑥ 上述过程重复循环，直到两个队列都清空  
![event-loop.jpg](https://i.loli.net/2020/12/02/ZTb26DuxecrvmJt.jpg)
  
宏任务队列可以有多个，而微任务队列只有一个：  

- 常见的macro-task：setTimeout、setInterval、script(整套代码)、I/O操作、UI渲染等；  
- 常见的micro-task：new Promise().then(回调)、process.nextTick、MutationObserver(HTML5新特性)等

### **2 Node中的事件循环**

Node中的事件循环与浏览器的是完全不同不同的东西。Node采用V8作为js的解析引擎，而I/O处理方面使用自己设计的libuv。  
libuv是一个基于事件驱动的跨平台抽象层，封装了不同操作系统的一些底层特性，对外提供API，事件循环也是在它里面实现：  
![node-eventLoop.png](https://i.loli.net/2020/12/02/2dRg1pCtJw5894s.png)

NodeJS运行机制如下：

- V8引擎解析JavaScript脚本
- 解析后的代码调用Node API
- libuv库负责Node API的执行。它将不同的任务分配给不同的线程，形成Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎
- V8引擎再将结果返回给用户
  
libuv引擎的事件循环分为6个阶段：

1. timers阶段：执行timers（setTimeout和setInterval）的回调
2. I/O callbacks阶段：处理上一轮循环少数未执行的的I/O回调
3. idel、prepare阶段：仅Node内部使用
4. poll阶段：获取新的I/O事件，执行I/O回调
5. check阶段：执行setImmediate()回调
6. close callbacks阶段：执行socket的close事件回调  
绝大部分的异步任务都在timers、poll、check这个3个阶段处理
  
NodeJS执行环境下的特殊情况：  
1）**setTimeout和setImmediate**  
二者非常相似，区别主要在于调用时机不同：

- setImmediate设计在poll阶段完成时执行，即check阶段
- setTimeout设计在poll阶段为空闲时，且设定阶段到达后执行，但它在timers阶段执行

```javascript
setTimeout(function timeout () {
  console.log('timeout');
},0);

setImmediate(function immediate () {
  console.log('immediate');
});
```
  
对于以上代码，setTimeout可能执行在前，也可能执行在后；
取决于setImmediate的准备时间；因为当setTimeout指定时间小于4ms，则增加到4ms（4ms是H5de新标准，2010年以前的浏览器是10ms）
  
但是如果二者在I/O callback内部回调时，总是先执行setImmediate，后执行setTimeout:

```javascript
const fs = require('fs')
fs.readFile(__filename, () => {
    setTimeout(() => {
        console.log('timeout');
    }, 0)
    setImmediate(() => {
        console.log('immediate')
    })
})
// immediate
// timeout
// 因为这两个代码都写在I/O回调中，I/O回调是在poll阶段执行，当回调执行完毕后队列清空，发现SetImmediate回调，所以立即跳转到check阶段执行回调。
});
```

2）**process.nextTick**

process.nextTick是独立于Event Loop之外的，它有一个自己的队列，会优先于其他micro-task队列执行：

```javascript
setTimeout(() => {
    console.log('timer1')
    Promise.resolve().then(function() {
  		console.log('promise1')
	})
}, 0)

process.nextTick(() => {
	console.log('nextTick')
	process.nextTick(() => {
		console.log('nextTick')
		process.nextTick(() => {
			console.log('nextTick')
			process.nextTick(() => {
				console.log('nextTick')
			})
   		})
 	})
})
// nextTick=>nextTick=>nextTick=>nextTick=>timer1=>promise1

```

### **3 浏览器与Node的Event Loop差异**

**浏览器环境下，micro-task的任务队列是每个macro-task执行之后执行；**
**Node环境下，在node10及其以前版本，micro-task会在事件循环的各个阶段之间执行，也就是一个阶段执行完毕，就会执行micro-task队列的任务**
**Node在node11版本开始，Event Loop的运行原来发生了变化，一旦一个阶段里的宏任务执行完，就会立即执行微任务队列，这一点与浏览器一直。**

![diff-eventLoop.png](https://i.loli.net/2020/12/02/KAmfl3ceQICY64i.png)

### **4 Web worker**

由于JS是单线程，当遇到计算密集型或高延迟的任务，用户界面可能会短暂“冻结”，不能做其他操作。
于是HTML5提出Web Worker，它允许JavaScript创造多线程环境，允许主线程创建Worker线程，将一些任务分配给后者。主线程运行的同时，Worker线程在后台运行，两者互不干扰，等到Worker完成计算任务，在把结果返回给主线程。
Web Worker的优点是可以承担一些密集型或高延迟任务，使主线程流畅，不被阻塞或拖慢。
缺点：

- 不能跨域加载JS
- Worker内部代码不能访问DOM
- 不是所有浏览器都支持这个新特性

Web Worker使用方法：

- 主线程调用Worker线程：

  1. 主线程通过new Worker()调用Worker构造函数，新建一个Worker线程
  2. 主线程调用worker.postMessage()方法，向Worker发消息
  3. 主线程通过worker.onmessage指定监听函数，接收子线程发回来的消息

```javascript
// 主线程：
var input = document.getElementById('number')
document.getElementById('btn').onclick = function () {
	var number = input.value
	//1、创建一个Worker对象
	var worker = new Worker('worker.js')
	// 3、绑定接收消息的监听
	worker.onmessage = function (event) {
		console.log('主线程接收分线程返回的数据: '+event.data)
		alert(event.data)
	}
	// 2、向分线程发送消息
	worker.postMessage(number)
	console.log('主线程向分线程发送数据: '+number)
}
console.log(this) // window
```

Worker线程响应：

1. Worker内部通过onmseeage()监听事件
2. 通过postMessage(data)方法向主线程发送数据

```javascript
//worker.js文件
function fibonacci(n) {
	return n<=2 ? 1 : fibonacci(n-1) + fibonacci(n-2)  //递归调用
}
console.log(this)//[object DedicatedWorkerGlobalScope]
this.onmessage = function (event) {
	var number = event.data
	console.log('分线程接收到主线程发送的数据: '+number)
	//计算
	var result = fibonacci(number)
	postMessage(result)
	console.log('分线程向主线程返回数据: '+result)
	// alert(result)  alert是window的方法, 在分线程不能调用
	// 分线程中的全局对象不再是window, 所以在分线程中不可能更新界面
}
```

>参考资料：  
>https://github.com/ljianshu/Blog/issues/54  
>https://juejin.im/post/5bb05494e51d450e7428da59  
>[深入浅出JavaScript运行机制](https://juejin.im/post/5bee1e1ff265da61590b3fff)  
>[10分钟理解JS引擎的执行机制](https://segmentfault.com/a/1190000012806637)  
>[浏览器组成](https://www.jianshu.com/p/5e90253b5907)  
>[全面梳理JS引擎的运行机制](https://mp.weixin.qq.com/s/bWGjZlBhlIfdSwRDK8XDHQ)
