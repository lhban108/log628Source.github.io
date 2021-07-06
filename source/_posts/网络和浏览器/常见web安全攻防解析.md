---
layout: web安全
title: React step 1 —— React初级使用
## date: 2018-02-10 19:32:34
tags: [网络,web安全]
---


## 一、XSS

`XSS（Cross-site Scripting）`，跨站脚本攻击是指通过存在安全漏洞的web网站注册用户的浏览器内运行非法的HTML标签或JavaScript进行的一种攻击。
XSS的原理是恶意攻击者通过向Web页面插入恶意可执行的网页脚本代码，当用户浏览该页时，嵌入其Web里面的脚本代码会被执行，从而达到窃取用户信息或侵犯用户安全隐私的目的。

XSS的攻击方式大致分为非持久型XSS（反射型XSS）和持久型XSS（存储型XSS）

### (1) 反射型XSS (非持久型XSS)

反射型XSS 一般是通过给别人发送带有恶意脚本代码参数的URL，当URL地址被打开时，特有的恶意代码参数会被HTML解析并执行。

```javascript

// 攻击方式一： 通过执行URL上注入的脚本，盗取用户敏感信息：
// 比如：通过UTL  `https://xxx.com/xxx?default=<script>alert(document.cookie)</script>` 
// 注入可执行的脚本代码, 获取用户的cookie


// 攻击方式二：通过用户请求 注入在URL上的链接，将攻击者提供的脚本将在用户的浏览器中执行：

// 某网站具有搜索功能，该功能通过 URL 参数接收用户提供的搜索词：
// https: //xxx.com/search?query=123

// 服务器在对此 URL 的响应中回显提供的搜索词：
// <p>您搜索的是: 123</p>

// 如果服务器不对数据进行转义等处理，则攻击者可以构造如下链接进行攻击：
// https://xxx.com/search?query=<img src="empty.png" onerror ="alert('xss')">

// 该 URL 将导致以下响应，并运行 alert('xss')：
// <p>您搜索的是: <img src="empty.png" onerror ="alert('xss')"></p>

// 如果有用户请求攻击者的 URL ，则攻击者提供的脚本将在用户的浏览器中执行。
```

不过大部分的浏览器如Chrome内置了一些XSS过滤器，可以防止大部分的反射型XSS攻击。

非持久性XSS有以下几个特征：

- XSS 脚本来自当前 HTTP 请求，不经过服务器存储，直接通过一次HTTP的请求完成攻击
- 攻击者需要诱骗用户点击链接才能发起攻击

### (2)持久型XSS（存储型XSS）

持久型XSS（存储型XSS）一般存在于form表单提交等交互功能，如文章留言、评论、提交文本信息等，攻击者将带有可执行的HTML代码或script代码通过表单内容经正常功能提交进入数据库持久保存，当前端页面通过后端从数据库读出注入的代码时，浏览器恰好将其渲染执行。
持久型XSS（存储型XSS）不需要诱骗点击，只需要在提交表单的地方完成注入即可，但是这种XSS攻击的成本相对还是很高。

攻击成功需要满足以下几点：

- POST请求提交表单后端没有做转义直接入库
- 后端从数据库取出数据后没做转义直接输出给前端
- 前端拿到后端的数据没做转义直接渲染成 DOM

### (3) 如何防御 XSS

- 服务端防御措施：

	1）CSP

	CSP的本质上是建立白名单，开发者明确告诉浏览器哪些外部资源可以加载和执行。我们只需要配合规则，如何拦截是由浏览器自己实现的。

	通常可以通过两种方式开启CSP：

	1.设置 HTTP的Header中Content-Security-Policy
	2.设置meta标签的方式

	比如设置HTTP Header：

	```text
	// 只允许加载本站资源
	Content-Security-Policy: default-src 'self'
	// 只允许加载HTTPS协议图片
	Content-Security-Policy: img-src https://* 
	// 允许加载任何来源框架
	Content-Security-Policy: child-src 'none'  
	```

	对于这种方式来说，只要开发者配置了正确的规则，那么即使网站存在漏洞，攻击者也不能执行他的攻击代码，并且CSP的兼容性也很好。

	2）转义字符

	用户的输入永远不可信任，最普遍的做法是转义输入输出内容，对引号、尖括号、斜杠进行转义。

	```javascript
	str.replace(/>/g, '&gt;')
	str.replace(/</g), '$lt;')
	// 但是对于显示富文本来说，不能通过上面的方法转义所有字符，因为这样会把需要的格式也过滤掉。
	// 这是需要通过白名单过滤的方法（黑名单也可以，但是需要过滤的的标签和标签属性太多，推荐使用白名单）

	// 实例用js-xss来实现
	const jsXSs = require('xss')
	let html = jsXss('<h1 id='title'>XSS Code</h1><script>alert('xss')</script>')
	```

	3）httpOnly cookie
	这是预防xss攻击最有效的防御手段，可以避免网页的cookie被JavaScript代码恶意窃取。

- 非持久型XSS防御措施：

  - Web 页面渲染的所有内容和和渲染的所有数据必须来自服务端
  - 尽量不要从URL、document.referrer（上一个页面的URL）、document.forms等这种DOM API中获取数据直接渲染
  - 尽量不要使用eval、new Function()、document.write()、innerHTML、window.setTimeout、document.createElement()等可执行字符串的方法
  - 如果做不到以上几点，也必须对涉及DOM渲染的方法传入的字符串参数做escape转义
  - 前端渲染的时候对任何的字段都需要做escape转义编码

## 二、CSRF

`CSRF（Cross Site Request Forgery）` 即跨站伪造请求，是一种常见的web攻击。它利用用户已登录的身份，在用户毫不知情的情况下，以用户的名义完成非法操作。

完成CSRF需要满足三个条件：

- 1)用户已经登录了站点A，并在本地记录了cookie
- 2)用户在没有登出站点A的情况下（也就是Cookie生效的情况下），从该网站访问了恶意攻击者提供的引诱危险站点B（B站点要求访问站点A）
- 3)站点A没有做任何CSRF攻击

防范CSRF攻击可以遵循以下几种原则：

- Get请求不对数据进行修改
- 不让第三方网站访问用户cookie
- 阻止第三方网站请求接口
- 请求时附带验证信息，比如验证码或者token

如何防御：

- 1）SameSite
可以对Cookie设置SameSite属性。该属性表示Cookie不能随着跨域请求发送，可以很大程度上减少CSRF攻击，但是该属性不是所有浏览器都兼容。

- 2）Referer Check
HTTP Referer 是header的一部分，当浏览器向web服务器发送请求时，一般会带上Referer信息告诉服务器是从哪个页面链接过来的，用来获取请求者的信息。
服务器可以通过检查请求者的来源来防御CSRF攻击。通过检查HTTP包中的头Referer的值是不是这个页面，来判断是不是CSRF攻击。
但是某些情况下，如从https跳转到http，浏览器处于安全考虑，不会发送referer，此时服务器就无法check了，因此不能完全依赖Referer Check作为防御CSRF的主要手段。

- 3）Anti CSRF Token
Token可以在用户登陆后产生并存在session或cookie中，然后每次	请求时，服务器验证token的有效性和真实性。由于token的存在，攻击者无法再构造出一个完整的 URL实施CSRF攻击。

- 4）验证码
应用程序与用户交互过程中，特别是账户交易这种核心步骤，强制用户输入验证码，才能完成请求。验证码可以很好的遏制CSRF攻击，但是验证码降低了用户体验，只能在关键业务点设置验证码。

## 三、SQL注入

一次SQL注入包括以下几个过程：

- 获取用户请求参数
- 拼接到代码中
- SQL语句按照我们构造参数的语义执行成功

SQL注入必备的条件：1、可以控制输入的数据；2、服务器要执行的代码拼接了控制的数据。
SQL注入的本质是：数据和代码未分离，即数据当作代码来执行。

如何防御：

- 1）严格限制Web应用的数据库的操作权限；
- 2）后台代码检查输入的数据是否符合预期；
- 3）对进入数据库的特殊字符（如*<>\&|等）进行转义处理，或编码转换；
- 4）所有的检查语句建议使用数据库提供的参数化查询接口，即不要直接拼接SQL语句

## 四、OS命令注入攻击

OS命令注入与 SQL注入差不多，只不过OS命令注入是针对操作系统的。
OS命令注入攻击是指通过web应用，执行非法的操作系统命令达到攻击的目的，只要能调用shell函数的地方就存在被攻击的风险。
比如：黑客构造命令提交到web应用程序，web应用程序提取命令并且拼接到被执行的命令中，因黑客构造的命令打破了原有命令结构，导致web应用执行了黑客构造的命令。

如何防御：

- 1）后端对前端提交内容进行规则检查（比如正则表达式）
- 2）在调用系统命令前对所有传入参数进行转义过滤
- 3）不要直接拼接命令语句，借助一些工具做拼接、转义处理，例如NodeJS的 shell-escape npm包

## 五、点击劫持

点击劫持是一种视觉欺骗的攻击手段，攻击者将需要攻击的网站通过iframe嵌套的方式嵌入自己的网页中，并将iframe设置为透明，在页面中透出一个按钮诱惑用户点击。  

特点：

- 隐蔽性高，骗取用户操作
- UI-覆盖攻击
- 利用iframe或其他标签的属性

如何防御：

- 1）X-Frame-Options
X-Frame-Options是一个Http相应头，这个相应头就是为了防御iframe嵌套的点击劫持攻击，在现在浏览器中有一个很好的支持。
X-Frame-Options有3个值可选：
  - Deny：表示页面不允许通过iframe的方式展示
  - SameOrion：表示页面允许通过相同的域名下通过iframe的方式展示
  - AllowFrom：表示页面可以在指定来源的iframe中展示

- 2）JavaScript防御
对于不支持X-Frame-Options的远古浏览器来说，只能通过JS的方式来防御点击劫持了。

## 六、URL跳转漏洞

借助未验证的URL跳转，将应用程序引导到不安全的第三方区域，从而导致安全问题。
其原理是黑客构造恶意链接（链接需要进行伪装，尽可能迷惑），发布到QQ群或者贴吧、论坛中，当用户点击后经过服务器或者浏览器解析后，跳转到恶意的网站中。
恶意链接一般是在熟悉的链接后面加上一个恶意的网站，这样才能迷惑用户。
一般通过Header头跳转、JavaScript跳转、META标签跳转等方式实现。

例如：
`http://www.baidu.com?act=go&url=http://t.cn/RVTatrd`
`http://www.qq.com?flag=1& jumpTo=http://t.cn/RVTatrd`
`http://www.taobao.com?jumpUrl= http://t.cn/RVTatrd`

如何防御：

- 1）Referer限制
保证跳转来源地址的可靠性和有效性，避免恶意用户自己生成跳转链接

- 2）加入有效的token
通过在URL中加入用户不可控的token对生成的链接进行校验，从而避免恶意用户生成自己的链接而被利用。缺点：如果功能本身要求比较开放，可能导致有一定的限制。

## 七、React 如何防止 XSS 攻击

无论使用哪种攻击方式，其本质就是将恶意代码注入到应用中，浏览器去默认执行。
React 官方中提到了 React DOM 在渲染所有输入内容之前，默认会进行转义。
它可以确保在你的应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转换成了字符串，因此恶意代码无法成功注入，从而有效地防止了 XSS 攻击。

### 1、React 的防止措施

#### (1) 自动转义

React 在渲染 HTML 内容和渲染 DOM 属性时都会将 "'&<> 这几个字符进行转义，转义部分源码如下：

```javascript
for (index = match.index; index < str.length; index++) {
  switch (str.charCodeAt(index)) {
    case 34: // "
      escape = '&quot;';
      break;
    case 38: // &
      escape = '&amp;';
      break;
    case 39: // '
      escape = '&#x27;';
      break;
    case 60: // <
      escape = '&lt;';
      break;
    case 62: // >
      escape = '&gt;';
      break;
    default:
      continue;
    }
  }
```

这段代码是 React 在渲染到浏览器前进行的转义，可以看到对浏览器有特殊含义的字符都被转义了，恶意代码在渲染到 HTML 前都被转成了字符串，如下：

```html
一段恶意代码:
<img src="empty.png" onerror ="alert('xss')"> 

转义后输出到 html 中:
&lt;img src=&quot;empty.png&quot; onerror =&quot;alert(&#x27;xss&#x27;)&quot;&gt; 
```

这样就有效的防止了 XSS 攻击。

#### (2) JSX 语法

`JSX` 实际上是一种语法糖，`Babel` 会把 `JSX` 编译成 `React.createElement()` 的函数调用，最终返回一个 `ReactElement`，以下为这几个步骤对应的代码：

```javaScript
// JSX
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
// 通过 babel 编译后的代码
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
// React.createElement() 方法返回的 ReactElement
const element = {
  $$typeof: Symbol('react.element'),
  type: 'h1',
  key: null,
  props: {
    children: 'Hello, world!',
      className: 'greeting'   
  }
  ...
}
```

我们可以看到，最终渲染的内容是在 Children 属性中，那了解了 JSX 的原理后，我们来试试能否通过构造特殊的 Children 进行 XSS 注入，来看下面一段代码：

```javaScript
const storedData = `{
  "ref":null,
  "type":"body",
  "props":{
  "dangerouslySetInnerHTML":{
  "__html":"<img src=\"empty.png\" onerror =\"alert('xss')\"/>"
      }
  }
}`;
// 转成 JSON
const parsedData = JSON.parse(storedData);
// 将数据渲染到页面
render () {
  return <span> {parsedData} </span>; 
}
```

这段代码中， 运行后会报以下错误，提示不是有效的 ReactChild

```javaScript
Uncaught (in promise) Error: Objects are not valid as a React child (found: object with keys {ref, type, props}). 
If you meant to render a collection of children, use an array instead.
```

那究竟是哪里出问题了？我们看一下 ReactElement 的源码：

```javaScript
const symbolFor = Symbol.for;
REACT_ELEMENT_TYPE = symbolFor('react.element');
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 这个 tag 唯一标识了此为 ReactElement
    $$typeof: REACT_ELEMENT_TYPE,
    // 元素的内置属性
    type: type,
    key: key,
    ref: ref,
    props: props,
    // 记录创建此元素的组件
    _owner: owner,
  };
  ...
  return element;
}
```

注意到其中有个属性是 `$$typeof`，它是用来标记此对象是一个 `ReactElement`，`React` 在进行渲染前会通过此属性进行校验，校验不通过将会抛出上面的错误。
`React` 利用这个属性来防止通过构造特殊的 `Children` 来进行的 `XSS` 攻击，原因是 `$$typeof` 是个 `Symbol` 类型，进行 `JSON` 转换后会 `Symbol` 值会丢失，无法在前后端进行传输。
如果用户提交了特殊的 `Children`，也无法进行渲染，利用此特性，可以防止存储型的 `XSS` 攻击。

### 2、React 中可能引起漏洞的一些写法

#### (1) 使用 dangerouslySetInnerHTML

dangerouslySetInnerHTML 是 React 为浏览器 DOM 提供 innerHTML 的替换方案。通常来讲，使用代码直接设置 HTML 存在风险，因为很容易使用户暴露在 XSS 攻击下，因为当使用 dangerouslySetInnerHTML 时，React 将不会对输入进行任何处理并直接渲染到 HTML 中，如果攻击者在 dangerouslySetInnerHTML 传入了恶意代码，那么浏览器将会运行恶意代码。看下源码：

```javaScript
function getNonChildrenInnerMarkup(props) {
	// 有dangerouslySetInnerHTML属性，会不经转义就渲染__html的内容
  const innerHTML = props.dangerouslySetInnerHTML; 
  if (innerHTML != null) {
    if (innerHTML.__html != null) {
      return innerHTML.__html;
    }
  } else {
    const content = props.children;
    if (typeof content === 'string' || typeof content === 'number') {
      return escapeTextForBrowser(content);
    }
  }
  return null;
}
```

所以平时开发时最好避免使用 dangerouslySetInnerHTML，如果不得不使用的话，前端或服务端必须对输入进行相关验证，例如对特殊输入进行过滤、转义等处理。
前端这边处理的话，推荐使用白名单过滤 `(https://jsxss.com/zh/index.html)`，通过白名单控制允许的 HTML 标签及各标签的属性。

#### (2) 通过用户提供的对象来创建 React 组件

举个例子：

```javaScript
// 用户的输入
const userProvidePropsString = `{"dangerouslySetInnerHTML":{"__html":"<img onerror='alert(\"xss\");' src='empty.png' />"}}"`;
// 经过 JSON 转换
const userProvideProps = JSON.parse(userProvidePropsString);
// userProvideProps = {
//   dangerouslySetInnerHTML: {
//     "__html": `<img onerror='alert("xss");' src='empty.png' />`
//      }
// };
render() {
     // 出于某种原因解析用户提供的 JSON 并将对象作为 props 传递
    return <div {...userProvideProps} /> 
}
```

这段代码将用户提供的数据进行 JSON 转换后直接当做 div 的属性，当用户构造了类似例子中的特殊字符串时，页面就会被注入恶意代码，所以要注意平时在开发中不要直接使用用户的输入作为属性。

#### (3) 使用用户输入的值来渲染 a 标签的 href 属性，或类似 img 标签的 src 属性等

```javaScript
const userWebsite = "javascript:alert('xss');";
<a href={userWebsite}></a>
```

如果没有对该 URL 进行过滤以防止通过 javascript: 或 data: 来执行 JavaScript，则攻击者可以构造 XSS 攻击，此处会有潜在的安全问题。用户提供的 URL 需要在前端或者服务端在入库之前进行验证并过滤。

> [常见六大Web安全攻防解析](https://github.com/ljianshu/Blog/issues/56)
> [浅谈 React 中的 XSS 攻击](https://mp.weixin.qq.com/s/yf0jhXiCXw8oTbjoUI3tYw)