# vue目录

[1. MVC和MVVM的区别 2](#_Toc29832069)

[2. Vue数据双向绑定的原理 2](#_Toc29832070)

[3. Vue组件通信 2](#_Toc29832071)

[4. Vue组件data为什么必须是函数 3](#_Toc29832072)

[5. vue-route是怎么实现的 4](#_Toc29832073)

[6. Vue中$nextTick有什么用 4](#_Toc29832074)

[7.为什么Vue的$nextTick不稳定 4](#_Toc29832075)

[8. Vue中怎么自定义指令 4](#_Toc29832076)

[9. Vue中key的作用 5](#_Toc29832077)

[10. Vue生命周期的理解 5](#_Toc29832078)

[11. Vue父组件和子组件生命周期钩子函数执行顺序 5](#_Toc29832079)

[12.父组件可以监听到子组件的生命周期吗 6](#_Toc29832080)

[13.clas与style如何动态绑定 6](#_Toc29832081)

[14.直接给一个数组项赋值，vue能监听到吗 7](#_Toc29832082)

[15.Vue是怎么实现对象和数组的监听 7](#_Toc29832083)

[16.Vue怎么用vm.$set()解决对象新增属性不能相应的问题 7](#_Toc29832084)

[17.vue3.0有哪些改变 8](#_Toc29832085)

[18.Proxy与Object.defineProperty优劣对比 8](#_Toc29832086)

[19.谈谈你对keep-alive的理解 9](#_Toc29832087)

[20.虚拟DOM的优缺点 9](#_Toc29832088)

[21.你有对vue项目进行哪些优化 9](#_Toc29832089)

[（一）代码层面的优化](#_Toc29832090) 9

[（二）](#_Toc29832091)webpack层面的优化 11

[（三）基础的](#_Toc29832092)web技术优化 13

# 1. MVC和MVVM的区别

**MVC** ：MVC是单向的，通过Collector（JS代码）处理用户与应用的交互，响应对View（Html）的操作（对事件监听），并调用Model（JS的Ajax代码）对数据进行操作，完成Model与View的同步（根据Model的改变，通过选择器对View进行操作更新）。

**MVVM** ：MVVM是双向的，它实现了Model和View的自动同步，也就是当Model的属性改变时，我们不再手动操作DOM元素来改变View的显示。

React和Vue的区别：

- **数据流** ：

- React是单向数据流（Props从父组件到子组件单向，数据到视图单向），称之为onChange/setState模式
- Vue默认支持双向绑定（数据和视图双向绑定），在vue1.0版本中props支持父子组件之间双向绑定，到vue2.x版本原生不支持了，但是提供了一个语法糖通过事件的方式修改。

- **数据监听** ：

- React是在setState后比较数据引用的方式监听，如果不优化（PureComponent/shouldComponentUpdate）可能会导致大量不必要的VDOM的从新渲染
- Vue通过ES5的数据劫持方法defineProperty监听数据变化，能精确知道数据变化，不需要特别的优化就能达到很好的性能transform

- **模板渲染** ：

- React是使用JSX，本质上在js中使用原生的语法(如if、map等方法)动态渲染模板。

好处是更加纯粹更加原生，比如React的render函数是支持闭包特性的，所以我们import的组件可以直接调用；但是在Vue中，由于模板使用的数据都必须挂在this上进行一次中转，所以我们import一个组件后还需要在components中再声明一下，这样显得很奇怪但是又不得不这样做。

- Vue则是使用指令来动态渲染模板，实现模板和JS代码分离。

- **Vuex**** 和 ****Redux** ：

- React中的每个组件都需要显示的用connect把需要的props和dispatch连接起来。原理上来说，Redux在检测数据变化的时候，是通过diff的方式比较差异的，并且Redux使用的是不可变数据，每次都是用新的state来替代旧的state

- Vue的$store直接被注入的组件实例中,因此使用非常灵活，使用dispatch和commit提交更新，通过mapState或者直接this.$store来读取数据。原来上来说，Vuex与vue原理一样，通过getter/setter来比较，如果看过源码会知道，它内部是通过直接创建一个vue实例来跟踪数据变化的

- **HOC**** 和 ****mixin** ：

- React中组合不同功能的方式通过Hoc（高阶组件），高阶组件本质就是高阶函数，React组件是一个纯粹的函数，所以高阶组件对于React来说非常简单。早期React也是使用Mixin，后来由于对组件侵入性太强会导致很多问题，后来弃用Mixin转而使用Hoc

- Vue对组合不同功能的方式使用Mixin，不能使用Hoc是因为vue是一个被包装的函数，vue在创建组件实例时隐式做了很多事情，如果我们直接把组件的声明包装一下，返回一个高阶组件，那么这个高阶组件就无法正常工作了。

# 2. Vue数据双向绑定的原理

实现MVVM的双向绑定，是通过Object.defineProperty()来劫持各个属性的getter和setter，并结合发布者-订阅者模式的方式实现。

# 3. Vue组件通信

（1）父组件向子组件传值 - Props传递数据

（2）子组件通信父组件 - $emit方法回调

（3）$emit、$on 通过Vue的实例触发事件和监听事件，实现跨级组件的通信

\&lt;!--在Vue的原型链上添加$bus属性，赋值为Vue实例--\&gt;

const EventBus = new Vue()

Object.defineProperties(Vue.prototype, {

$bus: {

get () {

return EventBus

}

}

})

window.globalVue = EventBus

\&lt;!-- 使用 --\&gt;

// 在组价B中监听固定的事件名，当A组件触发change事件后，监听到事件触发，触发事件时机不确定，一般在created或mounted中监听

// 组件A

\&lt;template\&gt;

\&lt;div @click = &quot;changeHandle&quot; \&gt;触发事件\&lt;/div\&gt;

\&lt;/template\&gt;

\&lt;script\&gt;

exportdefault {

methods: {

changeHandle () {

window.globalVue.$emit(&#39;change&#39;, 4456)

}

}

}

\&lt;/script\&gt;

// 组件B

\&lt;template\&gt;

\&lt;div\&gt;监听事件\&lt;/div\&gt;

\&lt;/template\&gt;

\&lt;script\&gt;

exportdefault {

mounted () {

window.globalVue.$on(&#39;change&#39;, str =\&gt; {

console.log(str) // 4456

})

}

}

\&lt;/script\&gt;

（3）$parent、$children 多层级传递

（4）$attrs批量向下传递属性，$listeners批量向下传递方法

（5）Provide &amp; Inject

Provide在父级中注入数据：

provide () {

return {parentMsg: &#39;wwwaa&#39;}

}

Inject在任意子组件中可以注入父级数据

inject: [&#39;parentMsg&#39;] // 会将数据挂在在当前实例上

（6）ref使用

this.$refs.grand.name // 通过$refs获取组件实例，并获取属性或方法

（7）EventBus：跨组件通知

provide () {

return {parentMsg: &#39;wwwaa&#39;}

}

Vue.prototype.$bus = new Vue()

// Son2组件和Grandson1互相通信

mounted () {

// 父组件注册

this.$bus.$on(&#39;my&#39;, data =\&gt; {

console.log(data)

})

}

mounted () {

// 侄子组件调用

this.$nextTick(() =\&gt; {

this.$bus.$emit(&#39;my&#39;, &#39;我是小红&#39;)

})

}

# 4. Vue组件data为什么必须是函数

因为组件可以被用来创建对个实例对象，如果 **data** 是一个对象，每个实例对象的 **data** 属性都指向该 **data** 的内存地址值，也就是说每个实例对象中的 **data** 都是同一个对象

如果 **data** 是一个函数的话，每创建一个组件的实例对象，就会返回一个新的 **data** ，类似于给每个组件实例创建一个私有的数据空间，让各自组件实例维护各自的数据。

# 5. vue-route是怎么实现的

vue-route是通过hash与Historyinterface两种方式实现前端路由，&quot;更新视图但不请求页面&quot;是前端路由原理的核心之一。

**hash**** 模式**：在浏览器中&quot;#&quot;以及&quot;#&quot;后面的字符称之为hash，用window.location.hash读取。特点：hash虽然在url中，但不包含在HTTP请求中，用来指导浏览器动作，对服务器安全无用，hash的值发生变化，不会重新向服务器发送请求。

**history**** 模式 **：依赖** HTML5 **的History API和服务器配置，History API提供了两个方法：pushState** ()**、replaceState()，利用history.pushState API来完成URL的跳转而无需重新加载页面，并且可以对浏览器历史记录栈进行修改，以及popState事件的监听到状态变更。（由于这种配置下我们的应用是单页客户端应用，如果后台没有正确的配置，用户请求页面将会返回404。因此需要在服务端增加一个覆盖所有情况的候选资源：如果URL匹配不到任何静态资源，则返回同一个index.html页面，这个页面就是你APP依赖的页面。）

**abstract** :支持所有JavaScript运行环境，包括NodeJS服务器端。

# 6. Vue中$nextTick有什么用

Vue.$nextTick用于延迟执行一段代码，它接收2个参数（回调函数和执行回调函数的上下文环境），如果没有提供回调函数，那么将返回 **promise** 对象。

在nextTick中定义了三个重要变量：callBacks（用来存储所有需要执行的回调函数）、pending（用来标记是否正在执行回调函数）、timeFunc（用来触发执行回调函数）。

timeFunc的触发方式有三种：

- 先判断是否原生支持promise，如果支持，则利用promise来触发执行回调函数
- 否则，如果支持MutationObserve，则实例化一个观察者对象，观察文本接点发生变化时，触发执行所有的回调函数。（MutationObserve是HTML5的新API，是个用来监听DOM变动的接口）
- 如果都不支持，则利用setTimeOut设置延迟为0来实现异步。

总结：Vue.$nextTick是一个异步函数，当DOM更新完成后执行传入的回调函数。

# 7.为什么Vue的$nextTick不稳定

因为Vue的$nextTick采用的是优雅降级：promise.then =\&gt; setImmediate =\&gt; MutationObserve =\&gt;setTimeOut， DOM是异步更新，真正更新渲染好的时间，不能完全确定，因此$nextTick并不能保证拿到最新的DOM。

# 8. Vue中怎么自定义指令

(1)自定义全局指令

// main.js

Vue.directive(&#39;mycolor&#39;, {

inserted (el) {

// 这里的el就是获取的标签元素，可以直接操作DOM

el.color.style = &#39;red&#39;

}

})

// template

\&lt;div v-mycolor \&gt;\&lt;/div\&gt;

(2)组件内部自定义指令

// template

\&lt;div v-mycolor\&gt;\&lt;/div\&gt;

// script

directives: {

mycolor: {

inserted (el) {

el.color.style = &#39;red&#39;

}

}

}

# 9. Vue中key的作用

Vue和React虚拟DOM的diff算法大致相同，其核心是基于两个简单的假设：

（1）两个相同的组件产生类似的DOM结构，不同的组件产生不同的DOM结构；

（2）同一层级的一组节点，他们可以通过唯一的id进行区分。

key是给每一个vNode的唯一id，可以依靠key更精确、更快的拿到oldVnode中对应的vNode节点。利用key的唯一性生成map对象来获取对应节点，比遍历方式更快。

# 10. Vue生命周期的理解

Vue实例从创建到销毁的过程，就是一个vue组件的生命周期。

开始创建=\&gt;初始化数据=\&gt;编译模板=\&gt;挂载DOM=\&gt;渲染、更新=\&gt;渲染、卸载

- **beforeCreate** ：组件实例被创建之初，组件的属性生效之前
- **created** ：组件实例已经完全被创建，属性已绑定，但真实的DOM还没生成，$el还不能用
- **beforeMount** ：在挂载之前被调用，相关的render函数首次被调用
- **mounted** ：el被新创建的vm.$el替换，并挂载到实例上之后调用该钩子函数
- **beforeUpdate** ：组件数据更新之前调用，发生在虚拟DOM打补丁之前
- **update** ：组件数据更新之后
- **activated** ：keep-alive专属，组件被激活时调用
- **deactivated** ：keep-alive专属，组件被销毁时调用
- **beforeDestory** ：组件销毁时调用
- **destoryed** ：组件销毁后调用

推荐在created钩子函数中调用异步请求，有以下优点：

- 能更快的获取到服务端数据，减少loading时间
- ssr不支持beforeMount、mounted钩子函数，所以放在created有助于一致性

# 11. Vue父组件和子组件生命周期钩子函数执行顺序

- 加载渲染过程：

父beforeCreate -\&gt; 父created –\&gt; 父beforeMount -\&gt; 子beforeCreate -\&gt; 子created -\&gt;子beforeMounte -\&gt; 子mounted -\&gt;父mounted

- 子组件跟新过程：

父beforeUpdate-\&gt;子beforeUpdate-\&gt;子updated-\&gt;父updated

- 父组件更新过程：

父beforeUpdate-\&gt;父updated

- 销毁过程：

父beforeDestory-\&gt;子beforeDestory-\&gt;子destoryed-\&gt;父destoryed

# 12.父组件可以监听到子组件的生命周期吗

（1）可以在子组件生命周期中通过$emit手动触发监听函数

// Parent.vue

\&lt;Child @mounted=&quot;doSomeThing&quot;\&gt;\&lt;/Child\&gt;

// Child.vue

mounted () {

this.$emit(&#39;mounted&#39;);

}

（2）通过@hook自动触发监听函数

@hook方法不仅仅可以监听mounted，其他的生命周期函数都能监听到，如createdupdated

// Parent.vue

\&lt;Child @hook:mounted=&quot;doSomeThing&quot;\&gt;\&lt;/Child\&gt;

methods: {

doSomeThing () {

console.log(&#39;父组件监听到子组件的mounted钩子函数&#39;)

}

}

// Child.vue

mounted () {

console.log(&#39;子组件触发了mounted钩子函数&#39;)

}

// 以上输出顺序为：

// 子组件触发了mounted钩子函数

// 父组件监听到子组件的mounted钩子函数

# 13.class与style如何动态绑定

Class可以通过对象语法和数组语法进行动态绑定：

// 对象语法

\&lt;div :class=&quot;{active: isActive, &#39;text-change&#39;: hasError}&quot;\&gt;\&lt;/div\&gt;

data: {

active: &#39;active1&#39;,

isActive: true,

hasError: false

}

// 数组语法

\&lt;div :class=&quot;[isActive ? activeClass : &#39;&#39;, errorClass]&quot;\&gt;\&lt;/div\&gt;

data: {

activeClass: &#39;active1&#39;,

isActive: true,

errorClass: &#39;text-danger

}

Style也可以通过对象语法和数组语法进行动态绑定

// 对象语法

\&lt;div :style=&quot;{color: activeColor, fontSize: mySize + &#39;px&#39;}&quot;\&gt;\&lt;/div\&gt;

data: {

activeColor: &#39;red&#39;,

mySize: 18

}

// 数组语法

\&lt;div : style =&quot;[styleColor, styleSize]&quot;\&gt;\&lt;/div\&gt;

data: {

styleColor:{

color: &#39;red&#39;

},

styleSize:{

fontSize: &#39;24px&#39;

}

}

# 14.直接给一个数组项赋值，vue能监听到吗

受JavaScript的限制，Vue不能检测到数组的以下变动：

- 利用索引直接给设置一个数组项，例如vm.items[indexOfItem] = newValue
- 修改数组的长度，例如 vm.items.length = newLength

为解决第一个问题，Vue提供了以下操作方法：

- Vue.set(vm.items, indexOfItem, newValue)
- vm.$set(vm.items, indexOfItem, newValue)
- vm.items.splice(indexOfItem, 1, newValue)

为解决第二个问题，可以使用 Array.property.splice：

vm.items.splice(newLength)

# 15.Vue是怎么实现对象和数组的监听

由于Object,defineProperty()只能对属性进行数据劫持，不能对整个对象进行劫持，同理无法对数组进行数据劫持。但是在Vue框架中 通过对遍历数组和递归对象，从而达到利用Object.defineProperty()也能对对象和数组（部分方法的操作）进行监听。

# 16.Vue怎么用vm.$set()解决对象新增属性不能相应的问题

受JavaScript的限制，Vue无法检测到对象属性的添加或删除。由于Vue会在初始化实例时对属性执行getter/setter转化，所以属性必须在data对象上才能让vue将它转换为响应式。

并且vue提供了Vue.set(object, propertyName, value) / vm.$set(object, propertyName, value)来实现为对象添加响应式属性。

vm.$set的实现原理是：

- 如果目标是数组，直接使用数组的splice方法触发响应式
- 如果目标是对象，会先判断属性是否存在、对象是否是响应式，最终如果要对属性进行响应式处理，则通过调用defineReactive方法进行响应式处理（defineReactive方法是vue在初始化对象时，给对象属性采用Object.defineproperty动态添加getter/setter的功能所调用的方法）

# 17.vue3.0有哪些改变

（1）检测机制的改变

Vue3.0是基于proxy的observe实现响应式，提供了全语言覆盖的反应性跟踪。避免了vue2.x中基于Object.defineProperty的许多限制：

  - 即能监测属性，也能监测对象
  - 可以监测对象属性的添加和删除
  - 监测数据索引和长度的变更
  - 支持Map、Set、WeakMap和WeakSet

新的observe还提供了以下属性：

  - **用于创建** observable **的公开** API **：** 这为中小规模场景提供了简单轻量级的跨组件状态管理解决方案。
  - **默认采用惰性观察：** 在2.x中，不管反应式数据有多大，都会在启动时 被观察到。如果数据集很大，这可能会在应用启动时带来明显的开销：在3.0中，只观察用于渲染应用程序最初可见的部分数据。
  - **更精确的通知：** 在2.x中，通过Vue.$set强制添加新属性将导致依赖于该对象的watcher收到变更通知。3.0中，只有依赖于特性属性的watcher才会收到通知。
  - **不可变的** observable **：** 我们可以创建值的&quot;不可变&quot;版本（即使是嵌套属性），除非系统在内部暂时将其&quot;解禁&quot;。这个机制可用于冻结prop传递或Vuex状态树以外的变化。
  - **更好的调试功能：** 我们可以使用新的renderTracked和renderTriggered钩子精确的跟踪组件在什么时候以及为什么重新渲染。

（2）模板

模板方面没有大的变更，只改作用域插槽。2.x中如果作用域插槽变了，父组件也会从新渲染。3.0中把作用域插槽改为函数的方式，这样只会影响子组件的重新渲染，提高了渲染性能。

（3）对象式的组件声明方式

2.x中组件是通过声明的方式传入一系列option，和TypeScript的结合需要通过一些装饰器的方式来做，虽然能实现功能，但是比较麻烦。3.0修改了组件的声明方式，改成了类式的写法，使得和TypeScript结合变得更容易。并且3.0的源码也改用TypeScript来写，使得对外暴露的API更容易结合TypeScript。当代码功能复杂之后，必须有一个静态类型系统来做一些辅助管理，静态类型系统对复杂代码的维护确实很有必要。

（4）其他方面的修改

- 支持自定义渲染器，从而使得weex可以通过自定义渲染器的方式来维护，而不是直接fork源码来改的方式。
- 支持Fragment（多个根节点）和Protal（在DOM其他部分渲染组件内容）组件，针对一些特定的场景做了处理。
- 基于treeShaking优化，提供了更多的内置功能。

# 18.Proxy与Object.defineProperty优劣对比

Proxy的优势如下：

- Proxy可以直接监听对象而非属性
- Proxy可以直接监听数组的变化
- Proxy有多达13种拦截方法，不限于apply、ownKeys、deleteProperty、has等是Object.defineProperty不具备的
- Proxy返回的是一个新对象，我们可以只操作新对象达到目的，而Object.defineProperty只能遍历对象属性直接修改
- Proxy作为新标准将受到浏览器厂商重点持续的性能优化

Object.defineProperty的优势如下：

- 兼容性好！支持 IE9，而Proxy存在浏览器兼容性问题，并且无法通过polyfill磨平。

# 19.谈谈你对keep-alive的理解

keep-alive是vue的一个内置组件，可以使被包含的组件保留状态，避免重新渲染，其有以下特性：

一般结合路由和动态组件一起使用，用于缓存组件

提供include和exclude属性，两者都支持字符串和正则表达式。include表示只有名称匹配的组件会被缓存，exclude表示任何名称匹配的组件都不会被缓存，其中exclude的优先级比include高。

对应两个钩子函数：activated和deactivated，当组件被激活时，触发钩子函数函数activated，当组件被移除时，触发钩子函数deactived。

# 20.VUE常见的修饰后缀

@click.stop = &#39;click&#39; // 阻止事件向上冒泡

@click.self = &#39;click&#39; // 根据实际操作对象执行函数

@click.prevent = &#39;click&#39; // 阻止默认动作 event.preventDefault()

@click.native = &#39;click&#39; // 为自定义组件添加原生事件

@click.once = &#39;click&#39; // 事件将只能执行一次

@input.trim = &#39;input&#39; // 去掉前后空格

# 21.虚拟DOM的优缺点

优点：

- **保证性能下限** ：框架的虚拟DOM需要适配任何上层API可能产生的操作，它的一些DOM操作的实现必须是普适的，所以它的性能并不是最优的；但是比起粗暴的DOM操作性能要好很多。因此框架的虚拟DOM至少可以保证在你不需要手动优化的情况下，依然可以提供还不错的性能，即保证性能的下限；
- **无需手动操作** DOM：我们不再需要手动去操作DOM，只需要写好View-Model的代码逻辑，框架会根据虚拟DOM和数据双向绑定，帮我们以可预期的方式更新视图，极大提高我们的开发效率。
- **跨平台** ：虚拟DOM本质上是JavaScript对象，而DOM与平台强相关，相比之下虚拟DOM可以进行更方便的跨平台操作，例如服务端渲染、Weex开发等等。

缺点：

**无法进行极致优化** ：虽然虚拟DOM + 合理的优化，足以应对绝大部分应用的性能需求，但在一些性能要求极高的应用中虚拟DOM无法进行针对性的极致优化。

# 22.你有对vue项目进行哪些优化

([https://juejin.im/post/5d548b83f265da03ab42471d#heading-1](https://juejin.im/post/5d548b83f265da03ab42471d#heading-1))

## （一）代码层面的优化

1.1 v-if和v-show区分使用场景

v-if用于运行时很少改变的文件，不需要频繁切换条件的场景；

v-show则适用于需要频繁切换条件的场景，只是简单地基于CSS的display属性进行切换。

1.2 v-for遍历必须为item添加key，且避免同时使用v-if

（1） v-for遍历必须为item添加key

（2） v-for避免与v-if同时使用

v-for比v-if优先级高，如果每一次都需要遍历整个数据，将会影响速度，尤其是当已知需要渲染很小一部分的时候，必要情况下应该替换成computed属性（将需要遍历的数组通过filter过滤掉不需要渲染的数据）

1.3 长列表性能优化

Vue是通过Object.defineProperty对数据进行劫持，来实现视图响应数据的变化，然而有些时候我们的组件就是纯粹的数据展示，不会有任何改变，我们就不需要Vue来劫持我们的数据。在大量数据展示的情况的下，这能够很明显的减少组件初始化的时间。我们可以通过Object.freeze来冻结一个对象，禁止Vue劫持我们的数据

export default {

data: () =\&gt; ({

users: {}

}),

async created() {

const users = await axios.get(&quot;/api/users&quot;);

this.users = Object.freeze(users);

}

}

1.4 事件的销毁

Vue组件销毁时，会自动清理它与其它实例的连接，解绑它的全部指定以及事件监听器，但是仅限于组件本身的事件。如果在js内使用addEventListener等方式添加的事件是不会自动销毁的，我们需要在组件销毁时手动移除这些监听事件，以免造成 内存泄漏。

created() {

addEventListener(&#39;click&#39;, this.click, false)

},

beforeDestroy() {

removeEventListener(&#39;click&#39;, this.click, false)

}

1.5 图片资源懒加载

将页面内未出现在可视区域的图片先不做加载，等到滚动到可视区域后再去加载，这样对于页面加载性能上会有很大的提升，也提高了用户体验。我们在项目中使用vue的vue-lazyload插件

1. 安装插件 – npm install vue-lazyload –-save-dev
2. 在入口文件main.js中引入并使用 import VueLazyload from &#39;vue-lazyload&#39;
3. 然后在vue中直接使用 Vue.use(VueLazyload)
4. 最后在vue文件中将img标签的src属性直接改为v-lazy，从而将图片显示方式更改为懒加载显示： \&lt;img v-lazy=&quot;/static/img/img1.png&quot;\&gt;

1.6 路由懒加载

Vue是单页面应用，可能会有很多路由引入，这样使用webpack打包后的文件很大。当进入首页时，加载的资源过多，页面就会出现白屏的情况，不利于用户体验。

因此我们结合vue的异步组件和webpack的代码分割功能，可以实现路由的懒加载。即把不同路由对应的组件分割成不同的代码块，然后当组件被访问的时候才加载对应组件。

路由懒加载：

const Foo = () =\&gt; import(&#39;./Foo.vue&#39;)

const router = new VueRouter({

routes: [

{ path: &#39;/foo&#39;, component: Foo }

]

})

把组件安组分块：

我们通过解释语法来提供chunkname，将组件安组分块.

Webpack会将任何一个异步模块与相同的模块名称组合到相同的异步块中。

const Foo = () =\&gt; import(/\* webpackChunkName: &quot;groun-foo&quot; \*/ &#39;./Foo.vue&#39; )

const Bar = () =\&gt; import(/\* webpackChunkName: &quot;groun-foo&quot; \*/ &#39;./Bar.vue&#39; )

const Baz = () =\&gt; import(/\* webpackChunkName: &quot;groun-foo&quot; \*/ &#39;./Baz.vue&#39; )

1.7 第三方插件按需引入

借助babel-plugin-component，只引入需要的组件，以达到减少项目体积的目的。

1. 安装插件 - npm install babel-plugin-component –D
2. 在.bablerc文件中增加修改

{

&quot;presets&quot;: [[&quot;es2015&quot;, { &quot;modules&quot;: false }]],

&quot;plugins&quot;: [

[

&quot;component&quot;,

{

&quot;libraryName&quot;: &quot;element-ui&quot;,

&quot;styleLibraryName&quot;: &quot;theme-chalk&quot;

}

]

]

}

1. 在main.js中引入需要的组件

import Vue from &#39;vue&#39;;

import { Button, Select } from &#39;element-ui&#39;;

Vue.use(Button)

Vue.use(Select)

1.8 优化无限列表性能

如果你的应用存在非常长或者无限滚动的列表，那么需要采用窗口化的技术来优化性能。只需要渲染少部分区域的内容，减少重新渲染组件和创建dom节点的时间。可以参考以下开源项目 vue-virtual-scroll-list和 vue-virtual-scroller来优化这种无限列表的场景。

1.9 服务端渲染SSR或预渲染

服务端渲染是指Vue在客户端将标签渲染成整个html片段的工作在服务端完成，服务端形成的html片段直接返回给客户端。

服务端渲染的优点：

- 更好的SEO：因为SPA页面的内容是通过Ajax获取，而搜索引擎爬取工具并不会等待Ajax异步完成之后再抓取页面内容，所以在SPA中是爬取不到页面通过Ajax获取到的内容；而SSR是直接由服务端返回已经渲染好的页面（数据已经包含在页面中），所以搜索引擎爬取工具可以抓取渲染好的页面。
- 更快的内容到达时间（首屏加载更快）：SPA会等待所有VUE编译后的js文件都下载完成后，才开始进行页面渲染，文件下载需要一定的时间等待，所以首屏渲染需要一定的的时间；SSR由服务端渲染好页面直接返回，无需等待下载js文件及再去渲染等，所以SSR有更快的内容到达时间。

服务端渲染的缺点：

- 更多的开发条件限制：例如服务端渲染只支持beforeCreate和created两个钩子函数，这会导致一些外部扩展库需要特殊处理，才能在服务端显然应用程序中运行；并且与可以部署在任何静态服务器上的完全静态SPA不同，服务端渲染程序需要处于Node.js server运行环境。
- 更多的服务器负载：在Node.js中渲染完整的应用程序，显然会比仅仅提供静态文件的server更加大量占用CPU资源，因此如果你预料在高流量环境下使用，请准备相应的服务器负载，并明智的采用缓存策略。

如果你的项目的 SEO和 首屏渲染 是评价项目的关键指标，那么就需要使用服务端渲染来实现最佳性能。（VueSSR踩坑之旅：[https://juejin.im/post/5cb6c36e6fb9a068af37aa35](https://juejin.im/post/5cb6c36e6fb9a068af37aa35)）

如果你的项目只需要改善少数营销页面的SEO，那么可能需要预渲染，在构建时（buildtime）简单的生成针对特定路由的静态HTML文件。优点是设置预渲染更简单 ，并且可以将你的前端作为一个完全静态的站点，具体你可以使用[prerender-spa-plugin](https://github.com/chrisvfritz/prerender-spa-plugin)（[https://github.com/chrisvfritz/prerender-spa-plugin](https://github.com/chrisvfritz/prerender-spa-plugin)）就可以轻松地添加预渲染。

## （二）webpack层面的优化

2.1 webpack对图片进行压缩

在vue项目中可以在webpack.base.conf.js中的url-loader中设置limit大小来对图片处理，小于limit的图片转化为base64格式，其余的不做处理。对于较大的图片资源，在请求资源的时候加载会很慢，可以用image-webpack-loader来压缩图片：

1. 安装image-webpack-loader : npm install image-webpack-loader --save-dev
2. 然后在webpack.base.conf.js中进行配置：

{

test: /\.(png|jpe?g|gif|svg)(\?.\*)?$/,

use:[

{

loader: &#39;url-loader&#39;,

options: {

limit: 10000,

name: utils.assetsPath(&#39;img/[name].[hash:7].[ext]&#39;)

}

},

{

loader: &#39;image-webpack-loader&#39;,

options: {

bypassOnDebug: true,

}

}

]

}

2.2减少ES6转为ES5的冗余代码

Babel插件在将ES6代码转换为ES5时会注入一些辅助函数，例如下面的ES6代码：

class HelloWebpack extends Component{...}

这段代码在转换为ES5时会注入一下两个辅助函数：

babel-runtime/helpers/createClass // 用于实现 class 语法

babel-runtime/helpers/inherits // 用于实现 extends 语法

在默认情况下，Babel会在每个输出文件中内嵌这些依赖的辅助函数代码，如果多个源码文件都依赖这些辅助函数，那么这些辅助函数的代码将会出现很多次，造成代码冗余。我们可以使用babel-plugin-transform-runtime插件来实现将这些辅助函数替换成导入语句，从而减少babel编译出来的代码文件体积。

首先安装babel-plugin-transform-runtime ：npm install babel-plugin-transform-runtime --save-dev

然后在.bablerc文件中添加配置：

&quot;plugins&quot;: [

&quot;transform-runtime&quot;

]

2.3 提取公共代码

如果项目中没有将每个页面的第三方库和公共模块提取出来，则会存在以下问题：

- 相同的资源被重复加载，浪费用户的流量和服务器成本
- 每个页面需要加载的资源太大，导致网页首屏加载缓慢，影响用户体验

所以我们要将每个页面的公共代码抽离成单独的文件，来优化以上问题。Webpack内置了专门用于提取多个Chunk中的公共部分插件CommonsChunkPlugin,项目中CommonChunkPlugin的配置如下：

// 所有在 package.json 里面依赖的包，都会被打包进 vendor.js 这个文件中。

new webpack.optimize.CommonsChunkPlugin({

name: &#39;vendor&#39;,

minChunks: function(module, count) {

return (

module.resource &amp;&amp;

/\.js$/.test(module.resource) &amp;&amp;

module.resource.indexOf(

path.join(\_\_dirname, &#39;../node\_modules&#39;)

) === 0

);

}

}),

// 抽取出代码模块的映射关系

new webpack.optimize.CommonsChunkPlugin({

name: &#39;manifest&#39;,

chunks: [&#39;vendor&#39;]

})

2.4 模板预编译

当使用DOM内模板或JavaScript内的字符串模板时，模板会在运行时被编译为渲染函数。通常情况下这个过程已经足够快了，但对性能敏感的应用还是最好避免这种用法。

预编译模板最简单的方式就是使用单文件组件——相关的构建设置会自动把预编译处理好，所以构建好的代码已经包含了编译出来的渲染函数，而不是原始的模板字符串。

如果喜欢分离JavaScript和模板文件，可以使用vue-template-loader，它可以在构建过程中把模板文件转换为JavaScript渲染函数。

2.5 提取组件的CSS

当使用单文件组件时，组件内的CSS会以style标签的方式通过JavaScript动态注入。会有一些小小的运行时开销，如果你使用服务端渲染，这会导致一段&quot;无样式内容闪烁&quot;(fous)。为了避免这个问题，可以将所有组件的CSS提取到同一个文件。

npm install extract-text-webpack-plugin --save-dev

2.6 优化SourceMap

我们在项目打包后，会将多个文件打包到一个文件，并且经过压缩、去空格、babel编译后的代码才用于生产环境，那么这样处理后的代码与源代码有很大的差别，进行Bug调试时无法定位到开发环境中的代码，因此SourceMap出现了，解决了不好调试代码的问题。

开发环境推荐：cheap-module-eval-source-map

生产环境推荐：cheap-module-source-map

2.7 构建结果输出分析

通过工具：webpack-bundle-analyzer，并且在webpack.prod.conf.js中进行配置：

if (config.build.bundleAnalyzerReport) {

var BundleAnalyzerPlugin = require(&#39;webpack-bundle-analyzer&#39;).BundleAnalyzerPlugin;

webpackConfig.plugins.push(new BundleAnalyzerPlugin());

}

再执行$ npm run build –-report后即可生成分析报告。

## （三）基础的web技术优化

3.1 开启gzip压缩

HTTP协议上的的gzip(GNUzip的缩写)编码是一种用来改进web应用性能的技术，web服务器和客户端（浏览器）必须同时支持gzip。目前主流的浏览器和服务器都支持gzip，gzip的压缩效率非常高，通常可以达到70%左右。

以熟悉的express为例：

安装：npm install compression --save

添加逻辑代码：

var compression = require(&#39;compression&#39;);

var app = express();

app.use(compression())

在responseheader中看到下面红色的字段就表明gzip开启成功

![](RackMultipart20201221-4-8pnvt0_html_954a711a5c1f437c.png)

3.2 浏览器缓存

3.3 CDN的使用

浏览器从服务器上下载CSS、JS和图片等文件时都要和服务器连接，而大部分服务器的宽带有限，如果超过限制，网页就半天反应不过来。而CDN可以通过不同的域名来加载文件，从而使下载文件的并发连接数大大增加，且CDN具有更好的可用性，更低的网络延迟和丢包率。

3.4预解析DNS

\&lt;link rel=&quot;dns-prefetch&quot; href=&quot;//host\_name\_to\_prefetch.com&quot;\&gt;放在head里面

\&lt;meta http-equiv=&quot;x-dns-prefetch-control&quot; content=&quot;on&quot;\&gt;一些高级浏览器是默认打开a标签的预解析的。如果页面是https协议开头的，浏览器基本是默认关闭dns预解析的，通过这一段标签是用来强制打开a标签的dns预解析的。

3.5 使用Chrome的Performance查找性能瓶颈

Chrome的performance面板可以录制一段时间内的js执行细节和时间。使用Chrome开发工具分析页面性能的步骤如下：

1. 打开Chrome开发工具，切换到performance面板
2. 点击Record开始录制
3. 刷新页面或展开某个节点
4. 点击stop停止录制

# 23.页面性能优化的方向

1、资源压缩合并，减少HTTP请求

2、非核心代码异步加载

3、利用浏览器缓存

4、使用CDN

5、预解析DNS

# 23.vue频繁操作DOM只更新一次

当频繁操作一个数据时，按照set-\&gt;订阅器(dep)-\&gt;订阅者(watcher)-\&gt;update 的过程，DOM应该会频繁更新，会很消耗性能，但实际上这个过程只更新了一次DOM。

这是因为vue异步执行dom更新，只要观察到数据变化，Vue将开启一个队列用于存储本次事件循环中所有watcher更新，并且同一个watcher更新只会被推入队列一次，并在本次事件循环的微任务执行结束后此更新。

DOM更新会放在下一个宏任务或当前宏任务的末尾（微任务）中进行执行。

Vue在内部尝试对异步队列使用原生的promise.then和MessageChannel,如果执行环境不支持，会采用setTimeout(fn, 0)代替。

**请说一下响应式数据的原理？**

**Vue**  **中是如何检测数组变化？**

**为什么**** Vue ****采用异步渲染？**

**谈一下**** nextTick ****的实现原理？**

**你知道**** Vue ****中**** computed ****是怎么实现的吗？**

· **28** / **28**