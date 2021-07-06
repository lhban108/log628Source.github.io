---
layout: js
title: project 项目
date: 2019-04-15 22:48:29
tags: [js,interview,project]
## keywords: 博客文章密码
## password: 123123
## message:  输入密码，查看文章
---

## 一 性能监控和错误监控

### 1、性能监控

前端性能监控流程：
数据采集 --> 数据上报 --> 服务端处理 --> 数据库存储 --> 数据监控可视化平台

#### (1) 接口请求时间监控

在 Node 层的中间件中判断请求的是 接口(reqInfo.type == 'api') 还是 页面

```JavaScript
export default async (ctx, next) => {
   const { reqInfo } = ctx;
	if (reqInfo.type == 'api') {
		// 表示请求的是 接口
		// 请求接口时，定义一个时间戳
	} else {
		// 否则请求的是 页面
	}
	// 在 await next() 时表示收到响应了，中间的时间差 就是接口请求时间
}
```

#### (2) 页面监控

[前端性能和错误监控](https://github.com/woai3c/Front-end-articles/blob/master/monitor.md)

页面性能数据采集的api: **Web API-> window.performance**

`Performance` 接口可以获取到当前页面中与性能相关的信息，它是 `High Resolution Time API` 的一部分，同时也融合了 `Performance Timeline API`、`Navigation Timing API`、 `User Timing API` 和 `Resource Timing API`。

通过 `window.performance.timing` 属性，可以获取页面加载的各个有效数据。

<img src="https://i.loli.net/2021/02/24/sxKmXLWdOAjJM2F.png" >

<img src="https://i.loli.net/2021/02/24/cFLPZYtdofmp1MD.jpg" >

通过以上数据，我们可以得到几个有用的时间:

```javaScript
    // 重定向耗时 (最后一个HTTP重定向完成时的时间戳 - 第一个HTTP重定向开始时的时间戳)
    redirect: timing.redirectEnd - timing.redirectStart,

    // DOM 渲染耗时
    // domComplete:当前文档解析完成，即Document.readyState 变为 'complete' 
    //      且相对应的readystatechange 被触发时的时间戳
    // domLoading: 当前网页DOM结构开始解析时
    //      （即Document.readyState 属性变为 “loading” 、
    //       相应的 readystatechange 事件触发时）的时间戳。
    dom: timing.domComplete - timing.domLoading,

    // 页面加载耗时
    // loadEventEnd: 当load事件结束，即加载事件完成时的时间戳
    // navigationStart: 同一个浏览器上一个页面卸载(unload)结束时的时间戳。
    //      如果没有上一个页面，这个值会和fetchStart相同
    // fetch: 浏览器准备好使用HTTP请求来获取(fetch)文档的时间戳。这个时间点会在检查任何应用缓存之前
    load: timing.loadEventEnd - timing.navigationStart,

    // 页面卸载耗时
    unload: timing.unloadEventEnd - timing.unloadEventStart,

    // 请求耗时
    // responseEnd: 返回浏览器从服务器收到（或从本地缓存读取，或从本地资源读取）最后一个字节时
    //      （如果在此之前HTTP连接已经关闭，则返回关闭时）的Unix毫秒时间戳
    // requestStart: 返回浏览器从服务器收到（或从本地缓存读取）第一个字节时的时间戳。
    //      如果传输层在开始请求之后失败并且连接被重开,
    //      则该属性将会被数制成新的请求的相对应的发起时间。
    request: timing.responseEnd - timing.requestStart,

    // 获取性能信息时当前时间
    time: new Date().getTime(),
```

除此之外，还有一个比较重要的时间就是**白屏时间**，它指从输入网址，到页面开始显示内容的时间。

```html
<!-- 将以下脚本放在 </head> 前面就能获取白屏时间。 -->
<script>
    whiteScreen = new Date() - performance.timing.navigationStart
</script>
```

获取资源加载时间API: `window.performance.getEntriesByType('resource')`

<img src="https://i.loli.net/2021/02/24/sfEplXLtPn19NSe.png" >

它一般包括以下几个类型: script、link、img、css、fetch、other、xmlhttprequest
数据中比较有用的信息是:

```javaScript
    // 资源的名称
    name: item.name,
    // 资源加载耗时
    duration: item.duration.toFixed(2),
    // 资源大小
    size: item.transferSize,
    // 资源所用协议
    protocol: item.nextHopProtocol,
```

**用户信息收集**：

1. 使用 `window.navigator` 可以收集到用户的设备信息，操作系统，浏览器信息
   用户设备信息：`window.navigator.userAgent` 中可以获取用户设备信息
    设备网络信息：`window.navigator.connection`
2. `UV(Unique visitor)` —— 是指通过互联网访问、浏览这个网页的自然人。00:00-24:00内相同的客户端只被计算一次，多次访问只计算一次UV。
   在用户访问网站时，可以生成一个随机字符串+时间日期，保存在本地。在网页发生请求时（如果超过当天24:00，则重新生成），把这些参数传到后端，后端利用这些信息生成 UV 统计报告。
   统计方法：统计 PV 时采集 userId 去重即可
   SPA页面，可以监听hashChange
3. `PV（Page View）`—— 即页面浏览量或点击量，用户每1次对网站中的每个网页访问均被记录1个PV。用户对同一页面的多次访问，访问量累计，用以衡量网站用户访问的网页数量。
   统计方法：（以 Vue 应用为例）
    - SPA 应用：仅单入口，在入口文件全局定义 Router.beforeEach 方便可行（或者可以监听hashChange）
    - MPA 应用：多入口，在每个入口文件定义 Router.beforeEach？可封装公用逻辑（伪装单入口），免去重复构造 entry 的成本
    - SSR 应用：调用 TemplateView 则为渲染页面（不同于前后端分离项目，服务端无法获知接口调用与页面渲染的对应关系），统计其调用次数及 TemplateName 可知页面 PV。
4. 页面停留时间 ——
   **传统网站** 用户在进入 A 页面时，通过后台请求把用户进入页面的时间捎上。过了 10 分钟，用户进入 B 页面，这时后台可以通过接口捎带的参数可以判断出用户在 A 页面停留了 10 分钟。
   **SPA** 可以利用 router 来获取用户停留时间，拿 Vue 举例，通过 `router.beforeEach destroyed` 这两个钩子函数来获取用户停留该路由组件的时间。
5. 浏览深度 —— 通过 `document.documentElement.scrollTop` 属性以及屏幕高度，可以判断用户是否浏览完网站内容。
6. 页面跳转来源 —— 通过 `document.referrer` 属性，可以知道用户是从哪个网站跳转而来。

**用户信息收集**：

1. `FP（First Paint）`首次绘制：首次绘制,是时间线上的第一个时间点，它代表网页的第一个像素渲染到屏幕上所用时间，也就是页面在屏幕上首次发生视觉变化的时间

    ```javaScript
    function getFPTime(){
        const timings = performance.getEntriesByType('paint')[0];
        return timings ? Math.round(timings.startTime) : null
    }
    ```

    FP与FCP这两个指标之间的主要区别是：

    - FP是当浏览器开始绘制内容到屏幕上的时候，只要在视觉上开始发生变化，无论是什么内容触发的视觉变化，在这一刻，这个时间点，叫做FP。
    - FCP指的是浏览器首次绘制来自DOM的内容。例如：文本，图片，SVG，canvas元素等，这个时间点叫FCP。
    FP和FCP可能是相同的时间，也可能是先FP后FCP。

2. `FCP（ First Contentful Paint）`首次内容绘制

    首次内容绘制，顾名思义，它代表浏览器第一次向屏幕绘内容。
    通过performance.getEntriesByType('paint’)，取第二个pain的时间，或者通过Mutation Observer观察到首次节点变动的时间。

    ```javaScript
    function getFPTime(){
        const timings = performance.getEntriesByType('paint');
        if(timings.length > 1)return timings[1]
        return timings ? Math.round(timings.startTime) : null
        //伪代码,算 DOM 变化时的最小那个时间，即节点首次变动的时间
        return Math.round(domEntries.length ? Math.min(...domEntries.map(entry => entry.time)) : 0);
    }
    const domEntries = []
    const observer = new MutationObserver((mutationsList)=>{
        for(var mutation of mutationsList) {
            if (mutation.type == 'childList') {
                console.log('A child node has been added or removed.');
            }
            if (mutation.type == 'addedNodes') {
                //TODO新增了节点，做处理，计算此时的可见性/位置/出现时间等信息，然后 push 进数组
                ...
                domEntries.push(mutation)
            }
        }
    })
    ```

3. `FMP（First Meaningful Paint)`首次有意义的绘制
首次有意义的绘制，是页面主要内容出现在屏幕上的时间, 这是用户感知加载体验的主要指标。目前尚无标准化的定义, 因为很难以通用的方式去确定各种类型页面的关键内容。
目前没有统一逻辑，阿里有个标准为最高可见增量元素，采用深度优先遍历方法\
其次，可以自定义，比如通 Mutation Observer 观察页面加载的一段时间(如前20s)内页面节点的变化, 即元素新增/移除/隐藏等变化记录下来，这样可以得到页面元素的可见时间点及元素与可视区域的交叉信息等。
然后，自定义一个计算公式，比如根据元素的类型进行权重取值(div 权重1，img 权重2等), 然后取元素与可视区域的交叉区域面积、可见度、 权重值之间的乘积为元素评分。
根据上面得到的信息, 以时间点为X轴, 该时间点可见元素的评分总和为Y轴, 取最高点对应的最小时间为页面主要内容出现在屏幕上的时间。

4. `FID（ First Input Delay）`首次输入延迟
   首次输入延迟，是测量用户首次与站点交互时的时间（即当他们单击链接/点击按钮/使用自定义的JavaScript驱动控件时）到浏览器实际能够回应这种互动的时间。

    方式一，通过performanceObserver(目前支持性为88.78%)观察类型为first-input的entry，获得其startTime/duration等数即可

    方式二，初始化时为特定事件类型(click/touch/keydown)绑定通用统计逻辑事件,开始调用时从event.timeStamp取开始处理的时间(这个时间就是首次输入延迟时间)，在事件处理中注册requestIdleCallback事件回调onIdleCallback，当onIdleCallback被执行时，当前时间减开始的event.timeStamp即为duration时间

    ```javaScript
    // 方式一
    function getFIDTime(){
        const timings = performance.getEntriesByType('first-input')[0];
        return timings ? timings : null
    }
    // 方式二，以下代码仅代表思路
    ['click','touch','keydown'].forEach(eventType => {
        window.addEventListener(eventType, eventHandle);
    });
    function eventHandle(e) {
        const eventTime = e.timeStamp;
        window.requestIdleCallback(onIdleCallback.bind(this, eventTime, e));
    }
    function onIdleCallback(eventTime, e) {
        const now = window.performance.now();
        const duration = now - eventTime;

        return {
            duration: Math.round(duration),
            timestamp: Math.round(eventTime)
        }

        ['click','touch','keydown'].forEach(eventType => {
            window.removeEventListener(eventType, eventHandle);
        });
    }
    ```

5. `TTI（ Time To Interactive）`可交互时间

    可交互时间，指的是应用在视觉上都已渲染出了，完全可以响应用户的输入了。是衡量应用加载所需时间并能够快速响应用户交互的指标。

    与 FMP 相同，很难规范化适用于所有网页的 TTI 指标定义。

    统计方式一：谷歌实验室写的npm包，tti-polyfill

    ```javaScript
    import ttiPolyfill from 'tti-polyfill';

    ttiPolyfill.getFirstConsistentlyInteractive().then((tti) => {
    ga('send', 'event', {
        eventCategory:'Performance Metrics',
        eventAction:'TTI',
        eventValue: tti,
        nonInteraction: true,
    });
    });
    ```

6. `FCI（First CPU Idle）`首次CPU空闲时间
   首次CPU空闲时间代表着一个网页已经满足了最小程度的与用户发生交互行为的时刻。当我们打开一个网页，我们并不需要等到一个网页完全加载好了，每一个元素都已经完成了渲染，然后再去与网页进行交互行为。网页满足了我们基本的交互的时间点是衡量网页性能的一个重要指标。

    FCI为在FMP之后，首次在一定窗口时间内没有长任务发生的那一时刻，并且如果这个时间点早于DOMContentLoaded时间，那么FCI的时间为DOMContentLoaded时间，窗口时间的计算函数可以根据Lighthouse提供的计算公式 `N = f(t) = 4 * e^(-0.045 * t) + 1` 进行自定义设计

7. `FPS（ Frames Per Second）`为每秒帧率
    表示的是每秒钟画面更新次数，当今大多数设备的屏幕刷新率都是60次/秒。

    参考标准：单位(帧率)
        50 ～ 60 FPS:动画将会相当流畅，让人倍感舒适；
        30 ～ 50 FPS 之间的动画，因各人敏感程度不同，舒适度因人而异；
        30 FPS 以下的动画，让人感觉到明显的卡顿和不适感；
        帧率波动很大的动画，亦会使人感觉到卡顿。

    统计逻辑
    利用requestAnimationFrame,循环调用，当now大于lastTime+1S时，计算FPS。若小于某个阀值则可以认为当前帧率较差，若连续小于多个阀值，则停止统计，当前页面处于卡顿状态，进入卡顿处理逻辑

**总结**:收集这些数据的代码：

```javaScript
    function monitorInit() {
        const monitor = {
            // 数据上传地址
            url: '',
            // 性能信息
            performance: {},
            // 资源信息
            resources: {},
            // 错误信息
            errors: [],
            // 用户信息
            user: {
                // 屏幕宽度
                screen: screen.width,
                // 屏幕高度
                height: screen.height,
                // 浏览器平台
                platform: navigator.platform,
                // 浏览器的用户代理信息
                userAgent: navigator.userAgent,
                // 浏览器用户界面的语言
                language: navigator.language,
            },
            // 重置 monitor 对象
            reset() {
                window.performance && window.performance.clearResourceTimings()
                monitor.performance = getPerformance()
                monitor.resources = getResources()
                monitor.errors = []
            },
            // 清空 error 信息
            clearError() {
                monitor.errors = []
            },
            // 上传监控数据
            upload() {
                // 自定义上传
                // axios.post({
                //     url: monitor.url,
                //     data: {
                //         performance,
                //         resources,
                //         errors,
                //         user,
                //     }
                // })
            },
            // 设置数据上传地址
            setURL(url) {
                monitor.url = url
            },
        }

        // 获取性能信息
        const getPerformance = () => {
            if (!window.performance) return
            const timing = window.performance.timing
            const performance = {
                // 重定向耗时
                redirect: timing.redirectEnd - timing.redirectStart,
                // 白屏时间
                whiteScreen: whiteScreen,
                // DOM 渲染耗时
                dom: timing.domComplete - timing.domLoading,
                // 页面加载耗时
                load: timing.loadEventEnd - timing.navigationStart,
                // 页面卸载耗时
                unload: timing.unloadEventEnd - timing.unloadEventStart,
                // 请求耗时
                request: timing.responseEnd - timing.requestStart,
                // 获取性能信息时当前时间
                time: new Date().getTime(),
            }

            return performance
        }

        // 获取资源信息
        const getResources = () => {
            if (!window.performance) return
            const data = window.performance.getEntriesByType('resource')
            const resource = {
                xmlhttprequest: [],
                css: [],
                other: [],
                script: [],
                img: [],
                link: [],
                fetch: [],
                // 获取资源信息时当前时间
                time: new Date().getTime(),
            }

            data.forEach(item => {
                const arry = resource[item.initiatorType]
                arry && arry.push({
                    // 资源的名称
                    name: item.name,
                    // 资源加载耗时
                    duration: item.duration.toFixed(2),
                    // 资源大小
                    size: item.transferSize,
                    // 资源所用协议
                    protocol: item.nextHopProtocol,
                })
            })

            return resource
        }

        window.onload = () => {
            // 在浏览器空闲时间获取性能及资源信息
            // https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback
            if (window.requestIdleCallback) {
                window.requestIdleCallback(() => {
                    monitor.performance = getPerformance()
                    monitor.resources = getResources()
                    console.log('页面性能信息')
                    console.log(monitor.performance)
                    console.log('页面资源信息')
                    console.log(monitor.resources)
                })
            } else {
                setTimeout(() => {
                    monitor.performance = getPerformance()
                    monitor.resources = getResources()
                    console.log('页面性能信息')
                    console.log(monitor.performance)
                    console.log('页面资源信息')
                    console.log(monitor.resources)
                }, 0)
            }
        }

        // 捕获资源加载失败错误 js css img...
        addEventListener('error', e => {
            const target = e.target
            if (target != window) {
                monitor.errors.push({
                    type: target.localName,
                    url: target.src || target.href,
                    msg: (target.src || target.href) + ' is load error',
                    // 错误发生的时间
                    time: new Date().getTime(),
                })

                console.log('所有的错误信息')
                console.log(monitor.errors)
            }
        }, true)

        // 监听 js 错误
        window.onerror = function(msg, url, row, col, error) {
            monitor.errors.push({
                type: 'javascript', // 错误类型
                row: row, // 发生错误时的代码行数
                col: col, // 发生错误时的代码列数
                msg: error && error.stack? error.stack : msg, // 错误信息
                url: url, // 错误文件
                time: new Date().getTime(), // 错误发生的时间
            })

            console.log('所有的错误信息')
            console.log(monitor.errors)
        }

        // 监听 promise 错误 缺点是获取不到行数数据
        addEventListener('unhandledrejection', e => {
            monitor.errors.push({
                type: 'promise',
                msg: (e.reason && e.reason.msg) || e.reason || '',
                // 错误发生的时间
                time: new Date().getTime(),
            })

            console.log('所有的错误信息')
            console.log(monitor.errors)
        })

        return monitor
    }

    const monitor = monitorInit()
```

性能数据可以在页面加载完之后上报，尽量不要对页面性能造成影响
`window.requestIdleCallback()`方法将在浏览器的空闲时段内调用的函数排队.

```javaScript
    // 信息收集
    window.onload = () => {
        // 在浏览器空闲时间获取性能及资源信息
        // https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback
        if (window.requestIdleCallback) {
            window.requestIdleCallback(() => {
                monitor.performance = getPerformance()
                monitor.resources = getResources()
            })
        } else {
            setTimeout(() => {
                monitor.performance = getPerformance()
                monitor.resources = getResources()
            }, 0)
        }
    }
```

`window.performance API` 是有缺点: 在 SPA 切换路由时，`window.performance.timing` 的数据不会更新.
解决办法之一: 拿Vue举例，在路由的全局前置守卫 `beforeEach` 里获取开始时间，
在组件的 `mounted` 钩子里执行 `vm.$nextTick` 函数来获取组件的渲染完毕时间。

```JavaScript
router.beforeEach((to, from, next) => {
	store.commit('setPageLoadedStartTime', new Date())
})
mounted() {
	this.$nextTick(() => {
		this.$store.commit('setPageLoadedTime', new Date() - this.$store.state.pageLoadedStartTime)
	})
}
```

### 2、错误监控

前端错误监控流程：
错误日志采集 --> 上报  --> 服务端处理 --> 数据库存储  --> 日志收敛 --> 数据监控可视化平台

- 可疑区域增加 `try...catch`
- 全程监控JS异常`window.onerror`
- 静态资源异常`window.addEventListener`
- `Promise.catch` 异常 `window.addEventListener("unhandledrejection", function(){})`
- `Vue errorHandler` 和 `React componentDidCatch`
- 跨域异常通过`crossOrigin`解决
- 监控网页崩溃：`window`对象的`load`和`beforeunload` 或`navigator.serviceWorker.controller.postMessage API`
 
#### (1) sentry引入

1、 在 src/main.js 入口文件引入 sentry 并初始化
	在初始化时，需要传入配置信息

```JavaScript
// main.js
import InitSentry from 'common/sentry';
// 初始化sentry
InitSentry();

// common/sentry.js
import moment from 'moment';
import { EnvEnum } from 'common/const/common/env'; 
// 配置环境信息: 开发环境、预发环境、线上环境 等等
import { init, captureException, withScope, Severity } from '@sentry/browser';
import { beforeSendHandler } from './handler';
export const initSentry = () => {
    try {
        const _global = window._global;
        const { NODE_ENV_DOCKER } = _global.env;

			// 预发和线上环境才监控
		  if (NODE_ENV_DOCKER !== EnvEnum.ONLINE 
				  && NODE_ENV_DOCKER !== EnvEnum.PROD 
				  && NODE_ENV_DOCKER !== EnvEnum.PRE) {
            return;
        }

        const { BUILD_TIME = Date.now() } = process.env;

        const dsn = 'https://4a7e72a34bdd438ebc1a9c649de1dc72@crash.youzan.com/31';
        /** 统一线上环境变量值 */
        const environment = NODE_ENV_DOCKER === EnvEnum.ONLINE ? EnvEnum.PROD : NODE_ENV_DOCKER;
        /** 使用打包时的时间作为 release 版本控制 */
        const release = moment(BUILD_TIME).format('YYYY-MM-DD HH:mm:ss');

        /** 初始化 传入配置 */
        init({
            dsn, // 识别
            release, // 发布版本
            environment, // 环境
            beforeSend: beforeSendHandler, // 上报的钩子 附加信息
        });
    } catch (error) {
        console.error('sentry 初始化失败');
    }
};

export const catchReqException = err => {
    withScope(scope => {
        scope.setLevel(Severity.Warning);
        captureException(new Error(JSON.stringify(err)));
    });
};
/** 自定义上传sentry */
export const catchDebugException = err => {
    withScope(scope => {
        scope.setLevel(Severity.Debug);
        captureException(new Error(JSON.stringify(err)));
    });
};
export default initSentry;


// handler.js
import { Event } from '@sentry/types';
import { merge } from 'lodash';
import { ATTENTION_URLS } from '../config';
// 以下url中发生的错误都会标记 alert 的标识 
// const ATTENTION_URLS: string[] = ['#/bill', '#/cashier', '#/dashboard', '#/reserve', '#/verification'];
const { _global } = window;
const DeptInfo = {
    kdtId: _global.env.team_info.kdt_id,
    shopName: _global.env.team_info.team_name,
    userId: _global.env.user_info.user_id,
    userAccount: _global.env.user_info.account,
    roleName: _global.env.user_info.roleName,
};
export const composeTag = (
    event: Event,
    targetTag: {
        [key: string]: any;
    }
) => {
    if (!event.tags) {
        event.tags = {};
    }
    return merge(event.tags, targetTag);
};
export const composeUser = (
    event: Event,
    targetUrl: {
        [key: string]: any;
    }
) => {
    if (!event.user) {
        event.user = {};
    }
    return merge(event.user, targetUrl);
};
export const setUserContext = (event: Event): Event => {
    /**
     * 设置基础用户上下文
     * 提交 event 时会带上
     */
    composeUser(event, DeptInfo);

    return event;
};
/** 重置 tags.url 为完整路径 */
export const setTagUrl = (event: Event): Event => {
    composeTag(event, {
        url: window.location.href,
    });
    return event;
};

/** 设置店铺信息tag方便sentry搜索 */
export const setTagDept = (event: Event): Event => {
    composeTag(event, DeptInfo);
    return event;
};
/** 根据 url 判断是否需要增加tags.alert: send */
export const setTagsAlert = (event: Event): Event => {
    const requestUrl = event.request.url;
    const isAttentioned = !!ATTENTION_URLS.find((url: string) => !!requestUrl.match(url));
    if (isAttentioned) {
        composeTag(event, {
            alert: 'send',
        });
    }
    return event;
};
/** 设置event 的 tags.route 可以根据 route 进行分类 */
export const setTagsRoute = (event: Event) => {
    const requestUrl = event.request.url;
    const hash = requestUrl.split('#').slice(1)[0];
    if (!hash) {
        return event;
    }
    const route = hash.split('?').slice(0, 1)[0];
    if (!route) {
        return event;
    }
    composeTag(event, {
        route,
    });
    return event;
};
export const beforeSendHandler = (event: Event): Event => {
    try {
        setTagUrl(event);
        setTagDept(event);
        setTagsAlert(event);
        setTagsRoute(event);
        setUserContext(event);
    } catch (error) {
        console.error(error);
    } finally {
        return event; // eslint-disable-line
    }
};

```

2、 接口：接口请求和响应 发生错误时的收集

```JavaScript
	// 在 ajax.js 中引入 sentry 的 catchReqException方法
	import { catchReqException } from 'common/sentry';

	// 2、在接口请求和响应 发生错误时 调用 catchReqException(err) 方法，收集 错误信息
	catchReqException(err);
```

3、增加 config/sentry.properties 配置文件

```text
	auth.token=35a38b7ba2fb4fb5b4946f94d572f89c997f00a136074df2a68c70eefac0e05f
	defaults.url=https://sentry.xxx.com/
	defaults.org=sentry
	defaults.project=project-web
	http.verify_ssl=false
```

#### (2) 错误日志收敛

#### (3) 错误收集(接口错误和页面错误)

(Koa框架) 在 Node 层的中间件中，采用 try...catch -> await next() 方式抓取错误

```javaScript
/**
 * 统一错误处理
 */
import _ from 'lodash';
export default async (ctx, next) => {
    const { reqInfo } = ctx;
    try {
        await next();
        // 如果能正常收到响应，但是响应体是 404时，需要做对应的错误处理
        if (ctx.status == 404) {
            if (reqInfo.type == 'api') {
                // 如果是接口报错，则在响应体重增加 响应错误的信息
                ctx.body = { response: { code: 99999, desc: '接口不存在' } };
            } else {
                // 如果是页面请求报错，则重定向到 错误页面的路由（错误页面已经准备好）
                ctx.redirect('/dashboard#/notFound');
            }
        }
    } catch (e) {
        // 如果无法收到服务端的响应，被 catch 了
        if (reqInfo.type == 'api') {
            // 接口报错
            ctx.body = {
                response: {
                    code: e.code || 99999,
                    desc: e.desc || '接口调用出错',
                    message: e.message || '',
                },
            };
        } else {
            // 页面请求报错，则通过 ctx.render 将错误信息渲染到模板上
            await ctx.render('error', { status: 500, msg: '系统错误' });
        }
        // 并且将错误信息上报到自定义的 错误收集平台
        nodejsLogger.warn('系统错误', e, {
            url: ctx.url,
            userId: _.get(ctx.state, 'userInfo.id'),
            kdtId: _.get(ctx.state, 'shopInfo.kdtId'),
        });
    }
};
```

## 二、上报与展示

## 三 Node.js环境性能监控

[Node.js环境性能监控](https://juejin.cn/post/6844903781889474567#heading-1)

### 1、指标

- CPU
- 内存
- 磁盘
- I/O
- 网络

在大多数场景下，CPU、内存以及网络就可以说是 Node 的主要性能瓶颈。

### 2、CPU指标

- CPU负载和CPU使用率
  - CPU负载： 进程角度
  - CPU使用率： CPU时间分配
- 量化CPU指标

### 3、内存指标

内存是一个非常容易量化的指标。 内存占用率是评判一个系统的内存瓶颈的常见指标。 对于Node来说，内部内存堆栈的使用状态也是一个可以量化的指标。

```JavaScript
		// /app/lib/memory.js
		const os = require('os');
		// 获取当前Node内存堆栈情况
		const { rss, heapUsed, heapTotal } = process.memoryUsage();
		// 获取系统空闲内存
		const sysFree = os.freemem();
		// 获取系统总内存
		const sysTotal = os.totalmem();

		module.exports = {
		memory: () => {
			return {
				sys: 1 - sysFree / sysTotal,  // 系统内存占用率
				heap: heapUsed / headTotal,   // Node堆内存占用率
				node: rss / sysTotal,         // Node占用系统内存的比例
			}
		}
		}
```

对于process.memoryUsage()拿到的值有一些需要关注的地方：

- rss：表示node进程占用的内存总量
- heapTotal：表示堆内存的总量
- heapUsed：实际堆内存的使用量
- external：外部程序的内存使用量，包含Node核心的C++程序的内存使用量

### 4、QPS

### 5、环境压测

## 四 项目

### 1 发现性能瓶颈并优化的经验

分享一下我近期的经验，之前项目也碰到过用起来很卡的情况，就是用element ui的tab切换组件时，点击tab切换非常卡，非常耗时，在排除了网络请求和js代码执行时间过长等原因后，跑了一次perfermance，结果发现大部分时间都花费在了 DOM GC上了，分析了下原因可能时dom结构太多导致每次tab切换渲染太耗时了。由于我每个tab里面的html结构都一样，都是一个table，只是每次tab切换时请求的数据不一样，我就把table抽离出来了，放到tab组件外面，然后tab里面就空了，就没有那么多dom了，tab切换就不卡了，很流畅。（ps：tab有20-30个切换选项，本人语文水平不行，描述的不清楚，望轻喷。）

### 2 项目

登录 - 本域登录、第三方接入登录（小红书、微博、爱逛等等）
店铺装修 - 素材、模板、员工（列表、排班、考勤、业绩提成）
商品 - 服务、卡项、产品，耗材、入库、出库
订单 - 开单、采购单、退款、欠款、核销、
预约
客户
数据
资产
会员
营销活动
