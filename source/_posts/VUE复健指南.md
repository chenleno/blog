---
title: VUE复健指南
date: 2020-04-20 10:32:12
updated: 2020-04-20 21:52:25
tags:
- VUE
categories:
- web前端
---
# VUE复健指南
搞了一年多的React，VUE有些概念模糊了，整个复健指南康康吧。
## 基础回顾
### Tips
- VUE组件名规则：**字母全小写并且包含至少一个连字符**
- VUE的`template`类似于React的`React.Fragment`
- 任何数据都不会自动地传进组件里，均需要在组件中通过props声明
- 双向数据绑定某种程度上和React的受控组件很像
- `is`属性能将html元素视为某种组件
```javascript
<dio><slot/></dio>
<li is="dio">我是dio哒</li> // === <dio>我是dio哒</dio>
```
### 数据与方法
当一个 Vue 实例被创建时，它将 data 对象中的所有的属性加入到 Vue 的响应式系统中
只有一开始就设定在data中的数据才是响应式的
### 生命周期
![VUE生命周期](https://cn.vuejs.org/images/lifecycle.png)
### 参数
v2.6新增了动态参数
```javascript
// attr可以是个变量
<a v-bind[attr]='utl'></a>
```
### 指令和修饰符
```javascript
// .default 表示默认调用e.preventDefault()
<a @click.default='handleClick'>
```
### 计算属性和侦听器
计算属性提供的函数将用作该属性的 getter 函数
#### 计算属性computed和方法methods
计算属性基于**响应式依赖**（即data中声明的值）进行缓存,只在相关响应式依赖变化时才会重新求值
方法则是每次渲染时，调用方法总会再次执行该函数
#### 计算属性computed和侦听属性watch
watch作为侦听器用来监测数据变化
### 用key管理可复用元素
vue会尽可能高效的渲染元素，通常会复用元素而不是重新渲染
添加key就是告诉vue，**这两个元素互相独立，不要复用他们**
vue 在渲染 v-for时采用**就地更新**策略，确保他们在每个索引位置被渲染（同react）
### v-show和v-if
v-if是真正的条件渲染，他会保证在切换过程中事件块内的监听器和子组件适当的被销毁和重建
v-if是**惰性**的，如果一开始为false，则什么也不做
v-show 不管条件是什么都会渲染，只会改变css样式，即display: none/normal;
v-if有较高的**切换开销**，v-show有较高的**渲染开销**
### 事件
? `$event`为特殊变量，传递原生事件对象event ????
### 组件
每个组件的`data`选项必须是一个函数，这样他的每个实例都可以维护一份被返回对象独立的拷贝
组件分为**全局注册**和**局部注册**
`VUE.component()`是全局注册,注册的组件可已被任何 通过`new VUE()`创建的根实例，也包括组件树中所有子组件的模板中
全局注册必须要在VUE实例化(`new VUE()`)之前完成
`components: {A,B}`是局部注册
#### 子组件触发父组件事件方法`$emit`
父组件一定要通过`v-on`注册事件方法，`v-bind`是不生效的
`$emit`从第二个参数开始可以往方法里传参
```javascript
// 父组件
<blog-post
  ...
  v-on:enlarge-text="postFontSize += 0.1"
></blog-post>
// 子组件
<button v-on:click="$emit('enlarge-text', ...params)">
  Enlarge text
</button>
```
#### 动态组件和异步组件
切换组件时，想要保存组件状态，避免反复创建导致的性能浪费
使用`<keep-alive></keep-alive>`包裹组件即可缓存
```javascript
<!-- 失活的组件将会被缓存！-->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```
`keep-alive`要求被包裹的组件都要有`name`属性

想要异步加载组件？
基于 webpack 的code-spliting特性,使用`import()`语法动态导入(React同)
```javascript
VUE.component('component-name', () => import('./xxx.vue'))
// 或者
new VUE({
  ...,
  components: {
    'component-name': () => import('./xxx.vue')
  }
})
```

### 自定义事件
- `.sync`修饰符 (传递对象进去时会为对象的每一个属性增加监听)
```javascript
<a :title="myTitle" @update:title="myTitle=$event">
==
<a :tilte.sync="myTitle">
```
### 插槽
2.6版本使用`v-slot`替代了`slot`和`slot-scope`
`slot`类似于React中的`props.children`，都负责分发内容
如果自定义组件中不包含插槽，则该组件使用时包裹的内容都会被抛弃
> 父级模板里的所有内容都是在父级作用域中编译的，子模板里的所有内容都是在子作用域内编译的
#### 具名插槽
一个组件可能需要预留多个插槽时，通过`<slot name='xxx'></slot>`指定name属性，外层在使用时通过对应的`<template v-slot:xxx></template>`将内容进行包裹，则template会被分发到对应的slot中
具名插槽可以缩写为`#xxx` == `v-slot:xxx`
#### 作用域插槽
父组件想要使用子组件中的数据时，通过**插槽prop**获取
```javascript
// 父组件
<div>
  <template v-slot:user="userProp">
    {{ userProp.user.age }} // 获取到了子组件的数据
  </template>
</div>
// 子组件
<a>
  <slot name='user' v-bind:user="user"/>{{user.name}}</slot> // 这里会渲染user.age
</a>
```
### 边界情况
- 使用`$root`获取根实例（类似于react, `context`）
- 在子组件用`$parent`获取父组件实例
- 在父组件通过`ref`连接子组件，然后用`this.$ref`获取子组件实例
```javascript
<child ref='childc'>
...
this.$ref.childc
```
## 可复用性&组合
### 混入mixin
同名生命周期钩子函数将合并为一个数组，都将被调用。混入组件将在本组件之前调用
同名方法，变量等，取当前对象的键值对
```javascript
const a = {
  created: function() {
    this.hello()
  }
  methods: {
    hello: function() { console.log('hello') }
  }
}
new VUE({
  mixins: [a],
  created: function() {
    console.log('本组件调用')
  }
})
// hello 
// 本组件调用
```
### 自定义指令
钩子函数：
- bind (只调用一次，指令第一次绑定到元素时调用)
- inserted(被绑定元素插入到父节点时调用)
- update (所在组件的vnode更新时调用，也可能发生在更新前)
- componentUpdated (vnode全部更新完后调用)
- unbind (只调用一次，指令与组件解绑时)
钩子函数参数：
- el （指令绑定的元素，可以直接操作dom）
- binding （一个包含多个属性的对象）
- vnode (VUE的虚拟节点)
- oldVnode （上一个虚拟节点）
```javacript
{
  ...,
  directives: {
    focus: {
      inserted: function(el) { // inserted 组件被插入到DOM中时
        el.focus()
      }
    }
  }
}
<input v-focus/>
```
### 渲染
Tips:
- 向组件中传递不带`v-slot`的指令的子节点时，这些子节点被存储在组件实例的`$slots.default`中
VUE通过`createElement()`方法建立一个vNode
## 响应式原理
```javascript
/**
 * VUE响应式原理
 * 
 * get 获取属性值
 * set 设置属性值
 * new dep() 每个属性都有自己源
 * dep.depend() 读取属性的地方，相当于订阅了该属性
 * dep.notify() 当属性值改变时，通知所有订阅了该属性的地方
 */
let data = {price: 5, num: 2}
let target, total, salePrice

class Dep {
  constructor() {
    this.subscribers = []
  }
  depend() {
    if (target && !this.subscribers.includes(target)) {
      this.subscribers.push(target)
    }
  }
  notify() {
    this.subscribers.forEach(sub => sub())
  }
}

Object.keys(data).forEach(i => {
  let dep = new Dep()
  let currentValue = data[i] // 
  Object.defineProperty(data, i, { 
    get: function() {
      dep.depend()
      return currentValue
    },
    set: function(newValue) {
      currentValue = newValue
      dep.notify()
    }
  })
})

function watcher(myFunc) {
  target = myFunc
  target()
  target = null
}

watcher(() => {
  total = data.price * data.num
})

watcher(() => {
  salePrice = data.price * 0.5
})
```
由此可见VUE无法监测对象或数组直接添加的属性
需要使用`this.$set(target, key, value)`来添加监听
## 异步更新队列
> Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。
使用`this.$nextTick(callback)`来监测数据变化
```javascript
{
  ...,
    methods: {
    updateMessage: function () {
      this.message = '已更新'
      console.log(this.$el.textContent) // => '未更新'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => '已更新'
      })
    }
  }
}
```

## 操作实例
```javascript
<div id="app">
  <p class="{active: isActive}">Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
  <p v-for="(item, index) in messageArr" key="index">{{ item.message }}</p>
  <button v-on:click="sayHello('hello', $event)">点击</button>
  <slot name="submit">submit</slot> // 没有提供插槽内容时将会默认使用该内容xs
</div>

new VUE({
  el: '#app', // el是根实例特有的属性(和VUE.component作区分)
  data: {
    message: '我有一头小毛驴',
    isActive: false,
    error: false,
    classObject: {
      active: false,
      'text-danger': false
    },
    messageArr: [
      {message: 1},
      {message: 2},
      {message: 3},
    ]
  },
  props: ['key', 'id'],
  watch: {
    message: function(newV, oldV) {
      if (newV !== oldV) {
        alert('message变化了')
      }
    }
  },
  computed: { 
    reversedMessage: { // 计算属性有get也有set方法
      get: function() {
        return this.message.split('').reverse().join('')
      },
      set: function(newVal) {
        this.message+=newVal
      }
    },
    classObject: function() {
      return {
        active: this.isActive && !this.error,
        'text-danger': this.error
      }
    }
  },
  methods: {
    sayHello: function(message, e) {
      e.preventDefault()
      alert(message)
    }
  }
})

VUE.component('test-button', {
  data: function() {
    return {
      ...
    }
  }
})

```