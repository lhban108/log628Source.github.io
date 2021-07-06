---
layout: webpack
title: webpack 详解
date: 2019-05-16 15:55:48
tags: [webpack,interview]
---

## 一 webpack 配置

配置项细节知识点：

### 1. loader 的执行顺序

 从后往前 (因此引入的一些全局插件,需要放到最后的配置中. 比如 postcss-loader(用于兼容所有浏览器的), 要放到所有 css loader 配置的后面)

### 2. 多入口配置方法

- 1. 入口 entry 配置多个

	```JavaScript
	entry: {
		index: path.join(srcPath, 'index.js'),
		other: path.join(srcPath, 'other.js'),
	}
	```

- 2. 输出 output 配置多个

	```JavaScript
	output: {
		// name 即多入口时 entry 中入口文件的名称
		filename: '[name].[contentHans:8].js',
		path: distPath,
	}
	```

- 3. plugin 要针对每个入口都要建一个生成 HTML 的 HtmlWebpackPlugin 配置

	```JavaScript
	plugins: [
		new HtmlWebpackPlugin({
			template: path.join(srcPath, 'index.html'),
			filename: 'index.html',
			// chunks 表示要引用哪些 chunk (设置 index.html 只引用 index.js 文件)
			chunks: ['index', 'vendor', 'common']  
			// index.js 对应 entry 中定义的入口文件名
			// vendor 代表打包的第三方文件, common 代表公共模块代码
		}),
		new HtmlWebpackPlugin({
			template: path.join(srcPath, 'other.html'),
			filename: 'other.html',
			// chunks 表示要引用哪些 chunk (设置 other.html 只引用 other.js 文件)
			chunks: ['other', 'vendor', 'common']  
			// other.js 对应 entry 中定义的入口文件名
			// vendor 代表打包的第三方文件, common 代表公共模块代码
		})
	]
	```

### 3. 文件的异步加载

 webpack 原生支持的，不需要要做配置，只需要引入的文件 用 import() 方法即可。 异步加载的文件也是生成的一个单独 chunk

```JavaScript
	// 异步加载的文件 import()方法返回一个 promise
	import('./dynamic-demo.js').then(res => {
		// 注意这里的 default, 获取对应的资源
		console.log(res.default.xxx()); 
	})

	// 非异步加载的文件
	import css from './cssFile/index.css';
```

## 二  webpack 性能优化

### 1. 优化打包构建速度 —— 开发体验和效率

#### (1) 优化 babel-loader - 开发

使用 `webpack` 缓存的方法有几种：`cache-loader`，`HardSourceWebpackPlugin` 或 `babel-loader` 的 `cacheDirectory` 标志

```JavaScript
	{
		test: /\.js$/,
		use: ['babel-loader?cacheDirectory'], // 开启缓存
		// 对于 babel-loader 解析的文件进行缓存
		includes: srcPath
		// 或者使用 cache-loader 插件
	}
```

#### (2) noParse - 开发/生产

过滤不需要解析的文件，比如打包的时候依赖了三方库（jquyer、lodash），提高打包的速度

#### (3) happyPack - 开发/生产

多进程打包或构建本地环境。(JS是单线程的).
除了 happyPack ，多进程打包还有一个插件  thread-loader
如果小项目，文件不多，无需开启多进程打包，反而会变慢，因为开启进程是需要花费时间的。

```JavaScript
	const HappyPack = require('happypack');
	module: {
		rules: [
			// js
			{
				test: /\.js$/,
				// 把 .js 文件转交给 id 为 happyScss 的 HappyPack 实例对象处理
				use: ['happyPack/loader?id=happyScss'],
				include: srcPath
			}
		]
	},
	plugins: [
		// happyPack 开启多进程打包
		new HappyPack({
			// 用唯一 id 达标当前 happyPack 是用来处理哪些文件
			id: 'happyScss',
			// 如何处理, 用法与 loader 配置中的一样
			loaders: ['babel-loader?cacheDiewctory']
		})
	]
```

#### (4) 自动刷新- 开发

modele.export = { watch: true }

#### (5) 热更新 - 开发

自动刷新: 整个网页全部刷新，速度慢，状态会丢失
热更新: 新代码生效，网页不刷新，状态不丢失

```JavaScript
	// webpack.config.js 配置文件
	const webpack = require('webpack');
	plugins: [
		new webpack.HotModuleReplacementPlugin(),
	],
	devServer: {
		hot: true,
	}

	// main.js
	// 除此之外，还需要在 main.js 中配置热更新范围
	if (module.hot) {
		module.hot.accept('./App', () => {
			ReactDOM.unmountComponentAtNode(document.getElementById('container'));
			render(App);
		});
	}
```

#### (6) DllPlugin - 开发

动态链接库插件

- webpack 已经内置了 DLLPlugin 支持
- DLLPlugin —— 打包出 dll 文件
- DllReferencePlugin —— 使用 dll 文件

步骤：

1. 打包生成 dll 文件：通过 package.json 中添加 dll 的打包命令，执行 webpack.dll.js 文件中定义的打包命令进行打包。（产出一个打包文件react.dll.js 和 一个索引文件 react.manifest.js

2. 引用 dll 文件：首先在 html 的模板文件中引用打包生成的 dll 文件（react.dll.js），然后通过 DllReferencePlugin 插件告诉 webpack 使用了哪些动态链接库 和 dll 文件索引位置（react.manifest.js）

```html
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
	 	<script src="./react.dll.js"></script>
    </head>

</html>
```

### 2. 优化产出代码 - 产品性能

#### (1) 小图片 base64 编码

```JavaScript
	module: {
        rules: [
			{
				test: /\.(png|jpg|gif)/,
				include: [path.join(__dirname, '..', 'src'), path.join(__dirname, '..', 'server')],
				use: [{
						loader: 'url-loader',
						options: { limit: 8 * 1024 },
				}],
			}
		]
	}
```

#### (2) bundle 加 hash

```JavaScript
	output: {
		path: path.join(__dirname, '..', 'dist_client'),
		filename: '[name].[contentHash:8].js',
	}
```

#### (3) 懒加载

通过 import() 实现懒加载

#### (4) splitChunks 抽离公共代码 和 第三方代码

抽离的步骤：

- 1. 在 optimization 配置要抽离的属性

	```JavaScript
	plugins: [ ],
	optimization: {
		// 压缩CSS
		minimizer: [],

		// 分割代码块
		splitChunks: {
			// chunks有三个可设置项: all initial async
			// initial(对异步导入的文件不处理) async(只处理异步文件)
			chunks: 'all',
			// 缓存分组
			cacheGrops: {
				// 第三方模块
				vendor: {
					name: 'vendor', // 打包后生成的 chunk 文件名称
					priority: 1, // 权重等级，设置为最高，优先抽离！！
					test: /node_modules/,
					minSize: 3, // 大小限制，小于 3kb 的文件不抽离
					minChunks: 1 // 最少引用次数，低于1次的不做抽离
				},
				// 公共的模块
				common: {
					name: 'common', // chunk 名称
					priority: 0, // 优先级
					miniSize: 0, // 公共模块的大小限制
					minChunks: 2 // 引用次数低于2次的不做抽离
				}		
			}
		}
	}
	```

- 2. 在 outPut 输出的文件中引用抽离的模块

	```JavaScript
	plugins: [
		new HtmlWebpackPlugin({
			template: path.join(srcPath, 'index.html'),
			filename: 'index.html',
			// chunks 表示要引用哪些 chunk 
			chunks: ['index', 'vendor', 'common']  
			// index.js 对应 entry 中定义的入口文件名
			// vendor 代表打包的第三方文件, common 代表公共模块代码
		})
	]
	```

#### (5) IgnorePlugin

忽略 moment 的本地化内容
`new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)`

#### (6) 使用 CDN 加速

```JavaScript
	// webpack.dev.js
    output: {
        filename: '[name].js',
        chunkFilename: '[name].js',
        publicPath: 'http://127.0.0.1:9000/beauty/assets/',
	 }
	 
	// webpack.prod.js (在打包的图片地址上也需要增加 cdn 地址)
    output: {
        filename: '[name]_[chunkhash].js',
		  chunkFilename: '[name]_[chunkhash].js',
		  // 修改所有静态文件的 url 的前缀
        publicPath: '//b.yzcdn.cn/beauty/assets/',
    }	
```

#### (7) 使用 production 环境

- 作用1：自动压缩
	在 `webpack 4.0` 中只需要添加 `mode: 'production'`， 则自动开启代码压缩(开发环境 `mode: 'development'`)。
	如果`webpack` 自己的压缩比较慢，可以通过 `ParalleUgifyPlugin`(`webpack-parallel-uglify-plugin`) 手动开启多进程压缩。
	`webpack 4.0` 默认内置了 `terser-webpack-plugin`

- 作用2：`Vue/React` 会自动删掉调试代码（如开发环境的 `warning`），体积更小

- 作用3：启动 `Teee-Shaking`
	过滤掉未引用 或者 未使用的函数、文件。
	前提：`ES6 Module` 才能让 `Tree-Shaking` 生效；`commonJs` 就不行。
	`ES6 Module` 和 `commonJS` 的区别：
	— `ES6 Module` 是静态引入，编译时引入(`import a from './a.js'`)
	— `commonJS` 是动态态引入，执行时引入(`a = require('./a.js')`)

#### (8) Scope Hosting

好处：

- 代码体积更小
- 创建函数作用域更少
- 代码可读性更好

```JavaScript
const ModuleConcatenationPlugin = require('webpack/lib/optimize/ModuleConcatenationPlugin');
module.export = {
	resolve: {
		// 针对 npm 中的第三方模块优先采用 jsnext:main 中
		// 指向的ES6 模块化语法的文件
		plugins: [
			// 开启Scope Hosting
			new ModuleConcatenationPlugin(),
		]
	}
}
```

#### (9) CSS 文件抽离压缩

打包时针对 CSS 文件进行抽离并压缩。抽离并压缩的目的：开发环境用的 style-loader 是将css 代码打包到 js 文件的，需要执行 js 文件才能将 css 解析出来并塞到 html 中，对性能不友好。抽离的步骤：

- 1. 打包解析中 mini-css-extract-plugin 插件代替 style-loader

	```JavaScript
	{
		test: /\.css$/,
		loader: [
			MiniCssExtractPlugin.loader, // 注意，这里 替代了 style-loader
			'css-loader', // 解析预发
			'postcss-loader' // 通过添加前缀增加兼容性
		]
	}
	```

- 2. 在 plugins 中增加抽离 css 文件的配置

	```JavaScript
	plugins: [
		new MiniCssExtractPlugin({
			filename: 'css/main.[contentHash:8].css'
		})
	]
	```

- 3. 压缩 CSS 文件

	```JavaScript
	plugins: [ ],
	optimization: {
		// 压缩CSS
		minimizer: [
			new TerserWebpackPlugin({}), 
			new OptimizeCssAssetsWebpackPlugin({})
		]
	}
	```

#### (10) 多进程压缩 ParalleUgifyPlugin

前提：开启`mode: 'production'`会自动进行压缩，当webpack默认的压缩功能速度慢时，可以通过此插件进行开启 多进程压缩。

打包时对输出的 js 代码进行多进程压缩。
UglifyJS 是单进程压缩，ParalleUgifyPlugin 是webpack 4 的多进程压缩

```JavaScript
	const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');
	plugins: [
		// happyPack 开启多进程打包
		new HappyPack({
			// 用唯一 id 达标当前 happyPack 是用来处理哪些文件
			id: 'happyScss',
			// 如何处理, 用法与 loader 配置中的一样
			loaders: ['babel-loader?cacheDiewctory']
		})
		new ParallelUglifyPlugin({
			uglifyJS: {
				output: {
					beautify: false, // 最紧凑的输出
					comments: false, // 删除注释
				},
				compress: {
					drop_console: true, // 删除 console 语句
					drop_debugger: true, // 删除 debugger
					// 内嵌虽然已经定义了，但是只用到一次的变量
					collapse_vars: true, 
					// 提取出现了多次但是没有定义成变量去引用的静态值
					reduce_vars: true,
				}
			}
		})
	]
```

## 三  babel

```JavaScript
// .babelrc 文件 或者 babel.config.json 文件
{
	// babel 插件的集合
	"presets": [
		"@babel/preset-env",
		"@babel/preset-react",
		"@babel/preset-typescript",
	],
	"plugins": []
}
```

- `@babel/polyfill`: 补丁，根据浏览器的兼容性打补丁
- `polyfill` = `core-js` + `regenerator`
  - `Babel7.4` 之后弃用 `babel-polyfill`
  - 推荐直接使用 `core-js` 和 `regenerator`

### 1. corejs

问题： `polyfill` 文件体积很大，项目中只是用了 `polyfill` 的部分模块，如何按需引入？

```JavaScript
// .babelrc 文件 或者 babel.config.json 文件
{
	// babel 插件的集合
	"presets": [
		"@babel/preset-env",
		{	
			// 这样就不需要再引入 @babel/polyfill
			"useBuiltIns": "usage",
			"corejs": 3, // 声明corejs版本
		}
	],
	"plugins": []
}
```

### 2. babel-runtime

问题：由于 polyfill 是定义在全局 window 上，会污染全局环境，如何处理？

通过 babel-runtime 实现

`package.json` 中 `devDependencies` 安装 `@babel/plugin-transform-runtime`， `dependenvies` 中安装 `@babel/runtime`

```JavaScript
// .babelrc 文件 或者 babel.config.json 文件
{
	// babel 插件的集合
	"presets": [
		"@babel/preset-env",
		{	
			"useBuiltIns": "usage",
			"corejs": 3, // 声明corejs版本
		}
	],
	"plugins": [
		[
			// 引用插件 "@babel/plugin-transform-runtime
			"@babel/plugin-transform-runtime",
			{
				// 添加配置
				"corejs": 3, // 声明corejs版本
				"absoluteRuntime": false,
				"helper": true,
				"regenerator": true,
				"useESModule": false,
			}
		]
	]
}
```

## 四 题

### 1 前端代码为何要进行打包和构建？

代码相关：

- 体积更小（ Tree-Shaking、压缩、合并），加载更快
- 编译高级语言和语法（TS、ES6+、模块化、SCSS 等）
- 兼容性和错误检查（Polyfill、postCSS、eslint）

工程化相关：

- 统一、高效的开发环境
- 统一的构建流程和产出标准
- 集成公司构建规范（提测、上线等）

### 2 module、chunk、bundle 分别是什么意思？有何区别？

- module: 各个源码文件，webpack 中一切皆模块
- chunk: 多个模块合成的， 如 entry splitChunk import()
- bundle: 最终的输出文件。一个 chunk 对应一个 bundle，可能有多个

### 3 loader 和 plugin 的区别？

- loader 是模块转换器， 如 less -> css
  - style-loader、css-loader、postcss-loader、url-loader、file-loader
- plugin 是扩展插件，如 HtmlWebpackPlugin
  - HotModuleReplacementPlugin、DllPlugin、UglifyJSPlugin、HappyPack、SourceMapDevToolPlugin、SourceMapUploadPlugin

### 4 babel 和 webpack 的区别？

- babel：JS 新语法编译工具，不关心模块化
- webpack：打包构建工具，是多个 loader 和 plugin 的集合

babel 原理

babel 的转译过程分为三个阶段：parsing、transforming、generating

以 ES6 代码转译为 ES5 代码为例，babel 转译的具体过程如下：

- ES6 代码输入
- babylon 进行解析得到 AST
- plugin 用 babel-traverse 对 AST 树进行遍历转译,得到新的 AST 树
- 用 babel-generator 通过 AST 树生成 ES5 代码

### 5 如何产出一个 lib

参考 webpack.dll.js， output.library

```JavaScript
output: {
	// lib 的中文名
	filename: 'lodash.js',
	// 输出 lib 到 dist 目录下
	path: distPath,
	// lib 的全局变量名
	library: 'lodash',
}
```

### 6 babel-runtime 和 babel-polyfill 的区别？

- babel-polyfill 会污染全局
- babel-runtime 不会污染全局
- 产出第三方 lib 需要用 babel-runtime

### 7 webpack 如何实现懒加载？

- import() 语法
- 结合 Vue React 异步组件
- 结合 vue-loader 和 react-router 异步加载路由

### 8 terser 是什么

`webpack4` 默认内置使用 `terser-webpack-plugin` 插件压缩优化代码，而该插件使用 `terser` 来缩小 `JavaScript`
(废弃了 `webpack3` 的 `UglifyJsPlugin`(单线程压缩) 和 `ParallelUglifyPlugin`(多线程压缩))

所谓 `terser`，官方给出的定义是：**用于 ES6+ 的 JavaScript 解析器、mangler/compressor（压缩器）工具包**

为什么 `webpack` 选择 `terser`？

- 不再维护 `uglify-es` ，并且 `uglify-js` 不支持 `ES6 +`。
- `terser` 是 `uglify-es` 的一个分支，主要保留了与 `uglify-es` 和 `uglify-js@3` 的 API 和 CLI 兼容性。
- `terser` 使用多进程并行运行来提高构建速度。并发运行的默认数量为 os.cpus().length - 1

### 9 如何开启多线程

- 多进程打包 —— happyPack（或者thread-loader）
- 多进程压缩 —— (web3)webpack-parallel-uglify-plugin（对应单进程压缩web3 UglifyJsPlugin）
- 多进程构建和压缩 —— (web4)terser-webpack-plugin

### 10 webpack构建流程

- 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
- 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
- 确定入口：根据配置中的 entry 找出所有的入口文件；
- 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
- 完成模块编译：在经过第 4 步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
- 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
- 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。
- 在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。


### 10 热更新

Hot Module Replacement

流程：

1. Webpack Compile: watch 打包本地文件 写入内存
2. Boundle Server: 启一个本地服务，提供文件在浏览器端进行访问
3. HMR Server: 将热更新的文件输出给 HMR(Hot Module Replacement) Runtime
4. HMR Runtime: 生成的文件，注入至浏览器内存
5. Bundle: 构建输出文件

开启热更新的方法有两种：

1. 直接通过运行 webpack-dev-server 命令时 加入 --hot参数 直接开启 HMR
2. 写入配置文件

```javaScript
// ./webpack.config.js
const webpack = require('webpack')
module.exports = {
	// ...
	devServer: {
		// 开启 HMR 特性 如果不支持 MMR 则会 fallback 到 live reload
		hot: true,
	},
	plugins: [
		// ...
		// HMR 依赖的插件
		new webpack.HotModuleReplacementPlugin()
	]
}
```

在 webpack-dev-server 源码中，通过 sockJs(github.com/sockjs/sockjs-client) 提供的服务端与浏览器端之间的桥梁.
在 devServer 启动的同时，建立了一个 webSocket 长链接，用于通知浏览器在 webpack 编译和打包下的各个状态，
同时监听 compile 下的 done 事件，当 compile 完成以后，通过 sendStats 方法, 将重新编译打包好的新模块 hash 值发送给浏览器。

浏览器 刷新策略选择：
 webpack-dev-server/client 中首先会根据 hot 配置决定是采用哪种更新策略【刷新浏览器或者代码进行热更新（HMR）】，
 如果配置了 HMR，就调用 webpack/hot/emitter 将最新 hash 值发送给 webpack，
 如果没有配置模块热更新，就直接调用 applyReload下的location.reload 方法刷新页面。

webpack 根据 hash 请求最新模块代码：
 这个过程是三个模块相互配合：webpack/hot/dev-serve、webpack-dev-server/client、webpack/lib/HotModuleReplacement.runtime
 1.首先是 webpack/hot/dev-server（简称 dev-server） 监听 webpack-dev-server/client 发送的 webpackHotUpdate 消息
 2.调用 webpack/lib/HotModuleReplacement.runtime（简称 HMR runtime）中的 check 方法，检测是否有新的更新
 3.check 过程中利用 webpack/lib/JsonpMainTemplate.runtime（简称 jsonp runtime）中的两个方法检查和更新文件：
 	(1) hotDownloadManifest ——通过调用 AJAX 向服务端请求是否有更新的文件，如果有则会将新的文件返回给浏览器;
	(2) hotDownloadUpdateChunk ——通过 jsonp 请求最新的模块代码，然后将代码返回给 HMR runtime，HMR runtime 会根据返回的新模块代码做进一步处理，可能是刷新页面，也可能是对模块进行热更新。

HMR Runtime 对模块进行热更新：
 webpack/lib/HotModuleReplacement.runtime 主要是对模块进行热更新，核心方法是 hotApply
 大概流程分为三步：
 1.找出 outdatedModules（过期模块） 和 outdatedDependencies（过期模块对应的依赖）
 2.删除过期的模块以及对应依赖
 3.把新模块添加至 modules 中
 整个流程结束后，已经可以获取最新的模块代码了，下一步是业务代码如何 知晓模块已经发生了变化

HotModuleReplaceMentPlugin —— module.hot.accept：

```javaScript
// module.hot.accept 其实等价于 module.hot._acceptedDependencies('./child) = render
// 业务逻辑实现
module.hot.accept('./child', () => {
	console.log('child 模块更新啦～')
})
```

 accept方法的第一个参数是 需要监听模块的 path(相对路径)，第二个参数就是当模块更新以后如何处理
 当模块改变时，对模块需要做的变更，搜集到 _acceptedDependencies 中，
 同时当被监听的模块内容发生了改变以后，父模块可以通过_acceptedDependencies 知道哪些内容发生了变化。

## 五 webpack 优化

优化本地构建速度：

- webpack 缓存：cache-loader,HardSourceWebpackPlugin, babel-loader 的 cacheDirectory
- noParse: 过滤不需要解析的文件，比如依赖的第三方库 lodash/jQ
- happyPack: 开启多进程
- 热更新 devServer: { hot: true }）
- DllPlugin （动态链接库插件）

速度分析用：speed-measure-webpack-plugin 插件
优化打包速度：

- noParse: 过滤不需要解析的文件，比如依赖的第三方库 lodash/jQ
- happyPack: 开启多进程
- 多进程压缩: web4:terser-webpack-plugin，web3:ParallelUglifyPlugin

体积分析用 webpack-bundle-analyzer 插件
优化产出代码：

- 小图片 base64 编码
- bundle 加 hash
- 懒加载 （ import() ）
- 代码压缩 （web4:terser-webpack-plugin，web3:UglifyJsPlugin/ParallelUglifyPlugin ）
- 提取公共代码 （splitChunks）
- 使用 CDN 加速 （publicPath：'//b.yzcdn.cn/beauty/assets/'）
- IgnorePlugin (忽略 moment 的本地化内容 new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/))
- 使用 production (web4:自动压缩、vue/React删掉调试代码、Teee-Shaking)
- Scope Hosting （代码体积小、作用域更少、可读性好 Module-Concatenation-Plugin）
- CSS 文件抽离/压缩 （抽离mini-css-extract-plugin 代替 style-loader，压缩TerserWebpackPlugin/OptimizeCssAssetsWebpackPlugin ）

## 六 webpack 5

> [构建效率大幅提升，webpack5 在企鹅辅导的升级实践](https://mp.weixin.qq.com/s/vBBfUdy4pHIPChs7TUkFLQ)

webpack5 的发布带来了很多新的特性，例如**优化持久缓存、优化长期缓存、Node Polyfill 脚本的移除、更优的 tree-shaking 以及 Module Federation**等。

### 1 优化持久缓存

webpack4 及之前的版本本身是没有持久化缓存的能力的，只能借助其他的插件或 loader 来实现：
使用 `webpack4` 缓存的方法有几种：`cache-loader`，`HardSourceWebpackPlugin` 或 `babel-loader` 的 `cacheDirectory` 标志

webpack5 缓存方案
webpack5 统一了持久化缓存的方案，有效降低了配置的复杂性。另外由于 webpack 提供了构建的 runtime，所有被 webpack 处理的模块都能得到有效的缓存，大大提高了缓存的覆盖率，因此 webpack5 的持久化缓存方案将会比其他第三方插件缓存性能要好很多。
webpack5 默认将构建的缓存结果放在 node_modules/.cache 目录下,可以通过配置更改目录.

cache 的属性 type 会在开发模式下被默认设置成 memory，而且在生产模式中被禁用，所以如果想要在生产打包时使用缓存需要显式的设置。
为了防止缓存过于固定，导致更改构建配置无感知，依然使用旧的缓存，默认情况下，每次修改构建配置文件都会导致重新开始缓存。当然也可以自己主动设置 version 来控制缓存的更新。

```javaScript
module.exports = {
    cache: {
      // 将缓存类型设置为文件系统
      type: "filesystem", 
      buildDependencies: {
        /* 将你的 config 添加为 buildDependency，
           以便在改变 config 时获得缓存无效*/
        config: [__filename],
        /* 如果有其他的东西被构建依赖，
           你可以在这里添加它们*/
        /* 注意，webpack.config，
           加载器和所有从你的配置中引用的模块都会被自动添加*/
      },
      // 指定缓存的版本
      version: '1.0' 
    }
}
```

### 2 长效缓存

仅仅改了其中一个文件，结果构建出来的所有 js 文件的 hash 值都变了，不利于浏览器进行长效缓存。
长效缓存指的是能充分利用浏览器缓存，尽量减少由于模块变更导致的构建文件 hash 值的改变，从而导致文件缓存失效。

webpack 4 之前的解决办法是使用 HashedModuleIdsPlugin 固定 moduleId，它会使用模块路径生成的 hash 作为 moduleId；使用 NamedChunksPlugin 来固定 chunkId。

其中 webpack4 中可以根据如下配置来解决此问题：

```javaScript
optimization.moduleIds = 'hashed'
optimization.chunkIds = 'named'
```

webpack5 长效缓存方案

webpack5 增加了确定的 moduleId，chunkId 的支持，如下配置：

```javaScript
optimization.moduleIds = 'deterministic'
optimization.chunkIds = 'deterministic'
```

此配置在生产模式下是默认开启的，它的作用是以确定的方式为 module 和 chunk 分配 3-5 位数字 id，相比于 v4 版本的选项 hashed，它会导致更小的文件 bundles。
由于 moduleId 和 chunkId 确定了，构建的文件的 hash 值也会确定，有利于浏览器长效缓存。同时此配置有利于减少文件打包大小。
在开发模式下，建议使用:

```javaScript
optimization.moduleIds = 'named'
optimization.chunkIds = 'named'
```

此选项生产对调试更友好的可读的 id

### 3 Node Polyfill 脚本被移除

webpack4 版本中附带了大多数 Node.js 核心模块的 polyfill，一旦前端使用了任何核心模块，这些模块就会自动应用，但是其实有些是不必要的。
webpack5 将不会自动为 Node.js 模块添加 polyfill，而是更专注的投入到前端模块的兼容中。因此需要开发者手动添加合适的 polyfill。

### 4 更优的 tree-shaking

`webpack5` 对 `tree-shaking` 进行了优化，分析模块的 `export` 和 `import` 的依赖关系，去掉未被使用的模块

```javaScript
// const.js
export const a = 'hello';
export const b = 'world';

// module.js
export * as module from './const';

// index.js
import * as main from './module';
console.log(main.module.a)

// 以上3个文件，在 webpack4 中会被打包为：
!function(){"use strict"; console.log("hello")}();
// 分析了依赖关系，去掉未被使用的模块
```

### 5 Module Federation

Module Federation 使得使 JavaScript 应用得以从另一个 JavaScript 应用中动态地加载代码 —— 同时共享依赖。相当于 webpack 提供了线上 runtime 的环境，多个应用利用 CDN 共享组件或应用，不需要本地安装 npm 包再构建了，这就有点云组件的概念了。

依赖共享主要是由插件 ModuleFederationPlugin 来提供 `const { ModuleFederationPlugin } = require("webpack").container;`

例如可以将我们项目中常用的依赖包 react 全家桶等打成一个包，做成一个 runtime,开发环境和生产环境依赖一个 runtime，这样可以大大减少项目的大小，提高编译速度。
