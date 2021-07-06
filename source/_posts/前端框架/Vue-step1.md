---
layout: Vue
title: Vue step 1 —— Vue初级使用
date: 2017-04-08 22:48:29
tags: [Vue,框架]
---

## 一、基本特性

### 1、 computed 和 watch

- computed 有缓存，data 不变则不会重新计算
- watch 如何深度监听
- watch 监听时引用类型， 拿不到 oldVal

### 2、 v-if 和 v-for 不能同时使用

v-for 比 v-if 优先级更高

v-if 和 v-show 的区别：

- v-show 是通过 CSS 的display:none 的方式控制
- v-if 是通过 vue 本身的机制控制组件是否显示或销毁

当组件频繁显示、隐藏的时候，推荐使用 v-show

### 3、事件event

- event 参数，自定义参数 -> 不传参数时，触发的第一个参数默认是event对象，传参时 用 $event 占位，表示 event 对象(mouseEvent对象)
- 事件修饰符修饰符, 按键修饰符, 表单修饰符
  - 事件修饰符
    - v-on:click.stop="doThis" 阻止单击事件继续传播
    - v-on:submit.prevent="onSubmit" 提交事件不再重载页面
    - v-on:click.stop.prevent="doThis" 修饰符可以串联
    - v-on:submit.prevent 只有修饰符
    - v-on:click.capture 添加事件监听时使用事件捕捉模式,即内部元素触发的事件先在此处理, 然后才交由内部元素进行处理
    - v-on:clikc.self 只当在event.target 是当前元素自身时触发处理函数,即事件不是从内部元素触发的
  - 按键修饰符
    - @click.ctrl="onClick" 即使 Alt 或 Shift 被一同按下时也会触发
    - @click.ctrl.exact="onCtrlClick" 有且只有 Ctrl 被按下时才触发
    - @click.exact="onClick" 没有任何修饰符被按下时才触发
  - 表单修饰符
    - v-modle.lazy
    - v-modle.trim
    - v-modle.number
- 【观察】事件被绑定到哪里
  - event.target 事件绑定的对象, 事件被挂在到当前元素,是与原生DOM事件是一样
  - event.currentTarget 事件触发对象, 是与原生DOM事件是一样

### 4、自定义事件

使用场景： 当两个组件没有关系，或者嵌套层级比较深（兄弟组件之间通讯）

- 绑定事件: event.$on(eventName, eventTarget)
- 触发事件: event.$emit(eventName, params)
(event是Vue实例, eventName是自定义和触发的自定义事件名称, eventTarget是绑定自定义事件最终执行的函数, params是触发事件时传递给自定义事件的参数)

```html
<!-- index.vue -->
<template>
    <div>
        <Input @add="addHandler"/>
        <List :list="list" @delete="deleteHandler"/>
    </div>
</template>
<script>
import Input from './Input'
import List from './List'
export default {
    components: {
        Input,
        List
    },
    data() {
        return {
            list: [
                {
                    id: 'id-1',
                    title: '标题1'
                }
            ]
        }
    },
    methods: {
        addHandler(title) {
            this.list.push({
                id: `id-${Date.now()}`,
                title
            })
        },
        deleteHandler(id) {
            this.list = this.list.filter(item => item.id !== id)
        }
    }
}
</script>
```

```html
<!-- Input.vue -->
<template>
    <div>
        <input type="text" v-model="title"/>
        <button @click="addTitle">add</button>
    </div>
</template>
<script>
import event from './event'
export default {
    data() {
        return {
            title: ''
        }
    },
    methods: {
        addTitle() {
            this.$emit('add', this.title)

            // 调用自定义事件
            event.$emit('onAddTitle', this.title)
            this.title = ''
        }
    }
}
</script>
```

```html
<!-- List.vue -->
<template>
    <div>
        <ul>
            <li v-for="item in list" :key="item.id">
                {{item.title}}
                <button @click="deleteItem(item.id)">删除</button>
            </li>
        </ul>
    </div>
</template>

<script>
import event from './event'
export default {
    props: {
        list: {
            type: Array,
            default() {
                return []
            }
        }
    },
    data() {
        return { }
    },
    mounted() {
        // 绑定自定义事件
        event.$on('onAddTitle', this.addTitleHandler)
    },
    methods: {
        deleteItem(id) {
            this.$emit('delete', id)
        },
        addTitleHandler(title) {
            console.log('on add title', title)
        }
    },
    beforeDestroy() {
        // 及时销毁，否则可能造成内存泄露
        event.$off('onAddTitle', this.addTitleHandler)
    }
}
</script>

```

```javascript
// event.js
import Vue from 'vue'
export default new Vue()
```

### 5、 生命周期

- 挂载阶段
  父Created -> 子Created -> 子Mounted -> 父Mounted
- 更新阶段
  父BeforeUpdate -> 子BeforeUpdate -> 子Updated -> 父Updated
- 销毁阶段
  父BeforeDestroy -> 子BeforeDestroy -> 子Destroy -> 父Destroy

## 二、高级特性

- 自定义 v-model
- $nextTick
- slot
- 动态、异步组件
- keep-alive
- mixin

### 1、 自定义 v-model

```html
<!-- 父组件 -->
<template>
    <div>
        <p>vue 高级特性</p>
        <!-- 自定义 v-model -->
        <p>{{name}}</p>
        <CustomVModel v-model="name"/>
    </div>
</template>

<script>
import CustomVModel from './CustomVModel'

export default {
  components: {
    CustomVModel
  },
  data() {
    return { 
      name: '123321',
    }
  }
}
</script>
```

```html
<!-- 子组件 -->
<template>
  <!-- 例如：vue 颜色选择 -->
  <input type="text"
    :value="text1"
    @input="$emit('change1', $event.target.value)"
  >
  <!--
    1. 上面的 input 使用了 :value 而不是 v-model
    2. 上面的 change1 和 model.event1 要对应起来
    3. text1 属性对应起来
  -->
</template>

<script>
export default {
  model: {
    prop: 'text1', // 对应 props text1
    event: 'change1'
  },
  props: {
    text1: String,
    default() {
      return ''
    }
  }
}
</script>
```

### 2、$nextTick

- Vue 是异步渲染
- data 改变之后， DOM 不会立刻渲染
- $nextTick 会在 DOM 渲染之后被触发, 以获取最新的 DOM 节点

```javascript
this.$nextTick(() => {
  // 可以获取最新的 data 值和 DOM 元素
})
```

### 3、slot

作用：父组件向子组件插入一段内容

解释：插槽就是在子组件中提供给父组件使用的一个占位符，用 `<slot></slot>` 表示，父组件可以在这个占位符中填充任何模板代码，如 HTML、组件等，填充的内容会替换子组件的 `<slot></slot>` 标签。

具名插槽：具名插槽其实就是给插槽取个名字。一个子组件可以放多个插槽，而且可以放在不同的地方，而父组件填充内容时，可以根据这个名字把内容填充到对应插槽中。
<img src="https://i.loli.net/2021/01/03/2y6t5E8V3SgGads.png" >

作用域插槽：作用域插槽其实就是带数据的插槽，即带参数的插槽，简单的来说就是子组件提供给父组件的参数，该参数仅限于插槽中使用，父组件可根据子组件传过来的插槽数据来进行不同的方式展现和填充插槽内容。

### 4、动态组件

- 用法 `:is= "componentName"`
- 需要根据数据，动态渲染的场景。即组件类型不确定

### 5、异步组件

- import() 函数
- 按需加载，异步加载大组件

```html
<template>
  <div>
    <!-- 异步组件 -->
    <FormDemo v-if="showFormDemo"/>
    <button @click="showFormDemo = true">show form demo</button>
  </div>
</template>

<script>
export default {
  components: {
    // 异步组件, 在需要使用的时候才加载
    FormDemo: () => import('../BaseUse/FormDemo')
  },
  data() {
    return {
      showFormDemo: false
    }
  }
}
</script>
```

### 6、keep-alive

比如，当用v-if控制组件时，组件不会重复Destroy 和 Mounted
适用场景: Tab切换的场景...

### 7、mixin

- 多个组件有相同的逻辑, 抽离出来
- mixin 并不是完美解决方案, 会有一些问题
  - 变量来源不明确，阅读代码增加困难
  - 多个 mixin 会造成命名冲突
  - mixin 和组件可能出现多对多的关系，复杂度较高
- Vue 3 提出的 Composition API 旨在解决这个问题

```html
<!-- 会把引入的 mixin 中data、mounted、method与 原来组件进行融合到一起-->
<!-- mixin中的 mounted 优先于 原来组件的 mounted 执行 -->
<template>
    <div>
        <p>{{name}} {{city}}</p>
        <button @click="showName">显示姓名</button>
    </div>
</template>
<script>
import myMixin from './mixin'
export default {
    mixins: [myMixin], // 可以添加多个，会自动合并起来
    data() {
        return {
            name: '123',
        }
    },
    methods: {
    },
    mounted() {
        console.log('component mounted', this.name)
    }
}
</script>
```

```javascript
// mixin.js
export default {
    data() {
        return {
            city: '北京'
        }
    },
    methods: {
        showName() {
            console.log(this.name)
        }
    },
    mounted() {
        console.log('mixin mounted', this.name)
    }
}
```

## 三、vuex 和 vue-router

### 1、Vuex

- state
- getters
- action
- mutation

<img src="https://i.loli.net/2021/01/03/N7OtYVuJECdBazI.png" >

Actions 是异步操作，可以调用 Backend API

### 2、vue-router

- 路由模式: hash、H5 history
- 路由配置: 动态路由、懒加载
