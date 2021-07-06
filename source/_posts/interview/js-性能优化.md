---
layout: js
title: js-性能优化
## date: 2019-04-15 22:48:29
tags: [js,interview]
## keywords: 博客文章密码
## message:  输入密码，查看文章
---

## 一 网络层面

### 1 减少 HTTP 请求

通过 `webpack` 打包的方式，对文件压缩和图片合并。

一个完整的 HTTP 请求需要经历 **DNS 查找 -> TCP 握手 -> 浏览器发出 HTTP 请求 -> 服务器接收请求 -> 服务器处理请求并发回响应 -> 浏览器接收响应**等过程。接下来看一个具体的例子帮助理解 HTTP

### 2 使用 HTTP2

- **二进制分帧** —— 不再需要像 `HTTP1.1` 那样不断地读入字节，直到遇到分隔符 `CRLF` 为止。`HTTP2` 是基于帧的协议，每个帧都有表示帧长度的字段。
- **多路复用** —— `HTTP1.1` 如果要同时发起多个请求，就得建立多个 TCP 连接，因为一个 TCP 连接同时只能处理一个 `HTTP1.1` 的请求。在 `HTTP2` 上，多个请求可以共用一个 TCP 连接，同一个请求和响应用一个流来表示，并有唯一的流 ID 来标识。并且可以乱序发送，到达目的地后再通过流 ID 重新组建。
- **首部压缩** —— `HTTP/2` 在客户端和服务器端使用“首部表”来跟踪和存储之前发送的键－值对，对于相同的数据，不再通过每次请求和响应发送。当客户端发送请求时，它会根据首部值创建一张表，如果服务器收到了请求，它会照样创建一张表。当客户端发送下一个请求的时候，如果首部相同，客户端只需要发送表的索引，服务端按照表的索引查找并还原成完整首部。
- **优先级** —— `HTTP2` 可以对比较紧急的请求设置一个较高的优先级，服务器在收到这样的请求后，可以优先处理。
- **服务器推送** —— 服务器根据所客户端需要的资源，主动向客户端推送资源，而无需客户端明确地请求。

### 3 使用服务端渲染

- 优点：首屏渲染快，SEO 好
- 缺点：配置麻烦，增加了服务器的计算压力

### 4 静态资源使用 CDN

在多个位置部署 CDN 服务器，让用户离服务器更近，从而缩短请求时间。

### 5 善用缓存，不重复加载相同的资源

对于不经常修改的文件使用强缓存: `Expires:12 Mar 2021 14:36:50 GMT` 或者 `Cache-Control: max-age=300`
可能会修改变动的文件使用协商缓存: `Last-Modified / If-Modified-Since` 或者 `ETag / If-None-Match`

## 二 打包层面

### 1 减少 ES6 转为 ES5 的冗余代码

`@babel/runtime` 包声明了所有需要用到的帮助函数，而 `@babel/plugin-transform-runtime` 的作用就是将所有需要 `helper` 函数的文件，从 `@babel/runtime` 包 引进来.

```javaScript
// 在 .babelrc 文件中
"plugins": [
	"@babel/plugin-transform-runtime"
]
```

### 2 压缩文件

webpack 可以使用如下插件进行压缩：

- `JavaScript：UglifyPlugin`
- `CSS ：MiniCssExtractPlugin`
- `HTML：HtmlWebpackPlugin`

或者使用 gzip 压缩。可以通过向 HTTP 请求头中的 `Accept-Encoding` 头添加 gzip 标识来开启这一功能，但是服务器也得支持这一功能。

```javaScript
// webpack 配置
const CompressionPlugin = require('compression-webpack-plugin');
module.exports = {
  plugins: [new CompressionPlugin()],
}

// node 层配置
const compression = require('compression')
// 在其他中间件前使用
app.use(compression())
```

### 3 根据文件内容生成文件名，结合 import 动态引入组件实现按需加载

通过配置 output 的 filename 属性可以实现这个需求。filename 属性的值选项中有一个 [contenthash]，它将根据文件内容创建出唯一 hash。当文件内容发生变化时，[contenthash] 也会发生变化。

### 4 提取第三方库

打包时提取公共代码和第三方库
`optimization - splitChunks - cacheGroups - { vendor, common}`

### 5 图片压缩

打包时降低图片质量。
将图片切成 JPG 格式，并且将它压缩到 60% 的质量，基本上看不出来区别。
压缩方法有两种，一是通过 webpack 插件 `image-webpack-loader` ，二是通过在线网站进行压缩。

## 三 代码层面

### 1 图片优化

#### (1) 图片延迟加载

先不给图片设置路径，只有当图片出现在浏览器的可视区域时，才去加载真正的图片.

```javaScript
	// 通过设置 data-src 来保存真实的 src 图片地址
	<img data-src="https://cdn.xxx...." />
	// 等页面可见时，使用 JS 加载图片：
	const img = document.querySelector('img')
	img.src = img.dataset.src

	// 判断图片是否显示
	let coords = el.getBoundingClientRect();
   return (coords.left >= 0 && coords.left >= 0 && coords.top)
		<= (document.documentElement.clientHeight || window.innerHeight)
		 + parseInt(offset);
	// getBoundingClientRect: 元素的上下左右分别相对浏览器视窗的位置
	// document.querySelector('div').clientWidth  可视区域的宽度（包括padding，不包括滚动条）
	// document.querySelector('div').offsetWidth  可视区域的宽度（包括padding、border、滚动条）
	// document.querySelector('div').scrollWidth  实际内容的宽度（可视区域宽度+被隐藏区域宽度）
```

#### (2) 响应式图片

可以通过 `picture` 标签 或者 媒体查询(`@media`) 实现

```javaScript
// picture 标签
<picture>
    <source srcset="banner_w1000.jpg" media="(min-width: 801px)">
    <source srcset="banner_w800.jpg" media="(max-width: 800px)">
    <img src="banner_w800.jpg" alt="">
</picture>
```

#### (3) 使用 webp 格式的图片

如果能通过服务器端判断浏览器支持WebP就用WebP或SVG格式(颜色数多用JPG格式，而很少使用PNG格式)。
Chrome浏览器完全兼容，但是 IE、Firefox、Safari完全不支持。
WebP 具有更优的图像数据压缩算法，能带来更小的图片体积，而且拥有肉眼识别无差异的图像质量；同时具备了无损和有损的压缩模式、Alpha 透明以及动画的特性，在 JPEG 和 PNG 上的转化效果都相当优秀、稳定和统一。

PNG/JPG：无损 webp 体积减少 40% 左右，有损 webp 图片体积减少 75% 左右
GIF：无损 webp 体积减少 88% 左右

#### (4) 小图片转 base64

在 `url-loader` 中增加 options 配置，指定 limit，图片转 base64 的图片大小

#### (5) 使用 iconfont 代替图片图标

icon-font 字体图标是矢量图，不会失真，还有一个优点是生成的文件特别小。
压缩字体文件：使用 fontmin-webpack 插件对字体文件进行压缩。

#### (6) 为图片标明高度和宽度

如果浏览器没有找到这两个参数，它需要一边下载图片一边计算大小。如果图片很多，浏览器需要不断地调整页面。这不但影响速度，而且影响浏览体验。当浏览器知道高度和宽度参数后，即使图片暂时无法显示，页面上也会腾出图片的空位，然后继续加载后面的内容，从而优化加载时间，提升浏览体验。

#### (7) CSS Sprite(图精灵、雪碧图)

### 2 Js/CSS 代码优化

#### (1) 减少重绘重排

浏览器渲染过程：
1 解析HTML生成DOM树
2 解析CSS生成CSSOM规则树
3 将DOM树与CSSOM规则树合并在一起生成渲染树
4 遍历渲染树开始布局，计算每个节点的位置大小信息
5 将渲染树每个节点绘制到屏幕

重排: 当改变 DOM 元素位置或大小时，会导致浏览器重新生成渲染树.
重绘: 当重新生成渲染树后，就要将渲染树每个节点绘制到屏幕，这个过程叫重绘。

重排和重绘这两个操作都是非常昂贵的，因为 JavaScript 引擎线程与 GUI 渲染线程是互斥，它们同时只能一个在工作。

什么操作会导致重排: 添加或删除可见的 DOM 元素、元素位置改变、元素尺寸改变、内容改变、浏览器窗口尺寸改变
什么操作会导致重绘: 改变字体颜色、改变背景色 等等

如何减少重排重绘：
1 避免逐项更改样式。最好一次性更改style属性，或者将样式列表定义为class并一次性更改class属性
2 避免循环操作DOM。 可以在一个display:none的元素上进行操作，最终把它显示出来。因为`display:none`上的DOM操作不会引发重排和重绘。
3 避免循环读取`offsetLeft`等属性。在循环之前把它们存起来。读取`offsetWidth`等属性值，会会强制浏览器刷新队列，先回流重排，在计算结果。
4 绝对定位具有复杂动画的元素。绝对定位使它脱离文档流，否则会引起父元素及后续元素大量的重排。
5 尽量使用 `CSS3` 提供的功能实现动画效果，比如 `transform、opacity、filters`，CSS 使用了硬件加速(GPU 加速)。

CSS最终的表现分为四步：`Recalculate([ri:'kælkjuleit]) Style` -> `Layout` -> `Paint Setup and Paint` -> `Composite Layers`
按照中文的意思大致是：查找并计算样式 -> 排列 -> 绘制 -> 组合层
`transform`是位于`Composite Layers`（组合层），而`width、left、margin`等是位于`layout`（排列层），所以所以`transform`实现动画肯定比`left`这些更流畅。

#### (2) 使用事件委托

好处：
1 子元素增加或者删除时，可以避免重复操作绑定解绑等操作，因此可以减少重复与DOM的交互次数，提高性能，减少重绘与重排的次数
2 减少内存，只需要在父元素绑定一次事件

#### (3) if-else 对比 switch

当判断条件数量越来越多时，越倾向于使用 switch 而不是 if-else。
因为 if-else 语句要进行 多次 判断，switch 只需要进行一次判断。
或者将需要查找的遍历存放在一个数组、Object对象、Map对象中，可以直接通过key值索引到对应的value。

#### (4) 不要覆盖原生方法

无论你的 JavaScript 代码如何优化，都比不上原生方法。因为原生方法是用低级语言写的（C/C++），并且被编译成机器码，成为浏览器的一部分。当原生方法可用时，尽量使用它们，特别是数学运算和 DOM 操作。比如 js Math方法等

#### (5) 降低 CSS 选择器的复杂性

- 原则是从选择器的右边到左边读取 —— 因此当可以确定能准确查找到对象即可，不要过于复杂
- CSS 选择器优先级 —— 优先级：内联 > ID选择器 > 类选择器 > 标签选择器

因此选择器越短越好，尽量使用高优先级的选择器，避免使用通配符 *
其实，CSS 选择器没有优化的必要，因为最慢和慢快的选择器性能差别非常小

#### (6) 使用 requestAnimationFrame 来实现视觉变化

目前大多数设备的屏幕刷新率为 60 次/秒（60fps）,最好保持浏览器渲染动画或页面的每一帧的速率也需要跟设备屏幕的刷新率保持一致。
其中每个帧的预算时间仅比 16 毫秒多一点 (1 秒/ 60 = 16.66 毫秒)。但实际上，浏览器有整理工作要做，因此您的所有工作需要在 10 毫秒内完成。如果无法符合此预算，帧率将下降，并且内容会在屏幕上抖动，此现象通常称为卡顿。
假设某个动画其中的一帧花了 50 ms，那么此时的帧率为 1s / 50ms = 20fps，页面看起来就像卡顿了一样。
保证 JavaScript 在帧开始时运行的唯一方式是使用 requestAnimationFrame。

#### (7) 将 CSS 放在文件头部，JavaScript 文件放在底部

所有放在 head 标签里的 CSS 和 JS 文件都会堵塞渲染。如果这些 CSS 和 JS 需要加载和解析很久的话，那么页面就空白了。所以 JS 文件要放在底部，等 HTML 解析完了再加载 JS 文件。
如果先加载 HTML 再加载 CSS，会让用户第一时间看到的页面是没有样式的、“丑陋”的，为了避免这种情况发生，就要将 CSS 文件放在头部了。
JS 文件也不是不可以放在头部，只要给 script 标签加上 defer 属性就可以了，异步下载，延迟执行。

#### (8) 使用 Web Worker

Web Worker 使用其他工作线程从而独立于主线程之外，它可以执行任务而不干扰用户界面。一个 worker 可以将消息发送到创建它的 JavaScript 代码, 通过将消息发送到该代码指定的事件处理程序（反之亦然）.

Web Worker 适用于那些处理纯数据，或者与浏览器 UI 无关的长时间运行脚本。

不过在worker内，不能直接操作DOM节点，也不能使用window对象的默认方法和属性。然而你可以使用大量window对象之下的东西，包括WebSockets，IndexedDB以及FireFox OS专用的Data Store API等数据存储机制。

#### (9) 不滥用 Float

Float在渲染时计算量比较大，尽量少使用

## 四 移动端性能如何优化

- 尽量使用CSS3动画，开启硬件加速。
- 适当使用 touch事件代替 click事件。
- 避免使用CSS3渐变阴影效果。
- 可以用 transform:translateZ（0）来开启硬件加速。
- 不滥用 Float, Float在渲染时计算量比较大，尽量少使用。
- 不滥用 Web 字体，Web字体需要下载、解析、重绘当前页面，尽量少使用。
- 合理使用requestAnimation Frame动画代替 setTimeout。
- 合理使用CSS3中的属性（CSS3 transitions、CSS3 3D transforms、 Opacity、 Canvas、 WebGL、Video）触发GPU渲染。过度使用会使手机耗电量増加。

H5秒开方案：

- 降低请求量：合并资源，减少 HTTP 请求数，minify / gzip 压缩，webP，lazyLoad。
- 加快请求速度：预解析DNS，减少域名数，并行加载，CDN 分发。
- 缓存：HTTP 协议缓存请求，离线缓存 manifest，离线数据缓存 localStorage。
- 渲染：JS/CSS优化，加载顺序，服务端渲染模板直出。

## 五 优化首屏加载速度

- 降低请求量：合并资源，减少 HTTP 请求数，minify / gzip 压缩，webP，lazyLoad。
- 加快请求速度：预解析DNS，减少域名数，并行加载，CDN 分发。
- 缓存：HTTP 协议缓存请求，离线缓存 manifest，离线数据缓存 localStorage。
- 渲染：JS/CSS优化，加载顺序，服务端渲染模板直出。
  