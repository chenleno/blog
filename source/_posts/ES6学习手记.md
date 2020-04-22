---
title: ES6学习手记
date: 2020-04-03 18:49:22
tags:
- JS基础
- ES6
categories:
- web前端
---
# ES6专栏
同样源于冴羽大大的ES6解读，发现了很多之前忽略的点（如临时死区等概念）。同样仍在学习中，blog将分阶段更新。
每次都要附上宝库链接：
[冴羽's blog](https://github.com/mqyqingfeng/Blog)
## [let和const](https://github.com/mqyqingfeng/Blog/issues/82)
块级声明，用于声明在指定块的作用域之外无法访问的变量
- 不会提升
- 重复声明报错
- 不绑定全局作用域
### let,const区别
const 用于声明常量，其值一旦设定不能修改，否则会报错
const 不允许修改绑定，但允许修改值，这意味着const声明的对象依然可以被添加属性和修改属性的值
### 临时死区(TDZ)
>  JavaScript 引擎在扫描代码发现变量声明时，要么将它们提升到作用域顶部(遇到 var 声明)，要么将声明放在 TDZ 中**(遇到 let 和 const 声明)**。访问 TDZ 中的变量会触发运行时错误。只有执行过变量声明语句后，变量才会从 TDZ 中移出，然后方可访问。
```javascript
var value = "global";
// 例子1
(function() {
  console.log(value); // 报错 Uncaught ReferenceError: Cannot access 'value' before initialization
  let value = 'local';
}());

// 例子2
{
  console.log(value); // 报错
  const value = 'local';
};
```
let 声明在循环内的行为是专门定义的,不一定与let不提升有关
**每次迭代循环时都创建一个新变量，并以之前迭代中同名变量的值将其初始化**
for in 和 for of 可以用const 声明变量，因为每次都会创建一个新的绑定
## [模板字符串](https://github.com/mqyqingfeng/Blog/issues/84)
写法： `${}`
模板字符串可以嵌套
## [箭头函数](https://github.com/mqyqingfeng/Blog/issues/85)
- 没有this,需要查找作用域链确定this
这就意味着，如果箭头函数被普通函数包含，他的this就是离他最近的普通函数的this
如果没有普通函数，会一直向上找到全局中去
因为没有this，所以也不能用call, apply, bind 之类的方法改变this指向
- 箭头函数没有arguments对象，他也访问的是外层的arguments
- 箭头函数不能通过 new 调用,箭头函数内部没有[[Construct]]方法
JavaScript函数内部有两个方法,[[Call]]和[[Construct]]
当通过new 调用时，执行[[Construct]]方法，创建一个实例对象，然后再执行函数体，将this绑定到实例对象上
当作为函数调用时，执行[[Call]]方法
- 没有 new.target,由外层非箭头函数决定 (new.target,返回new命令作用于的那个构造函数，用于判断实例是否是通过new 构造函数生成的)
[new.target](https://es6.ruanyifeng.com/#docs/class#new-target-%E5%B1%9E%E6%80%A7)
- 没有原型prototype
- 没有super,也是由外层非箭头函数决定
## [Symbol类型与模拟实现](https://github.com/mqyqingfeng/Blog/issues/87)
Symbol表示独一无二的值
- 通过Symbol()创建,不能用new创建，var a = Symbol()
- a instanceof Symbol // false
- Symbol可以接收字符串作为参数，接收对象时，会调用对象的toString()方法，才生成一个Symbol值
```javascript
Symbol({a:1}) // Symbol([object Object])
```
- Symbol函数的参数只是对当前Symbol值的描述，通过同样方式创建的两个Symbol值不相等（!= && !==）
- **Symbol的作用**，用于对象的属性名，可以保证不会出现重复属性
Symbol作为对象属性名时不会被常规遍历方式遍历到（for in, Object.keys等），只有一个Object.getOwnPerprotySymbols可以获取到
- 可以使用Symbol.for()使用同一个Symbol值
```javascript
Symbol.for('111') === Symbol.for('111')
```
- Symbol.keyFor() 返回一个已登记的Symbol值的key
```javascript
Symbol.keyFor(Symbol('aaa')) // 'aaa'
```
经典应用场景： 单例模式，判断函数是否已创建，已创建就返回，没有就创建
```javascript
var Person = function(name, age) {
  this.name = name
  this.age = age
}
var key = Symbol.for(func)
if(!global[key]) {
  global[key] = new Person()
}
export default global[key]
```
## [迭代器与for of](https://github.com/mqyqingfeng/Blog/issues/90)
迭代器：就是一个具有next()方法的对象，每次调用next()都会返回一个结果对象，该结果对象有两个属性，value表示当前值，done表示遍历是否结束

什么是可遍历的？
一种数据结构只要部署了 iterator 接口，就称这种数据结构为可比遍历的
ES6规定，iterator接口部署在Symbol.iterator属性
**所以for of 遍历的实际是对象的Symbol.iterator属性**
### 默认部署Symbol.iterator的对象
- Array
- Set
- Map
- 类数组对象，如arguments
- Generator 对象
- 字符串
### 内建迭代器
ES6为Array,Map,Set内置了三种迭代器
- entries() 返回一个遍历器对象，用来返回 [键名，键值]组成的数组，对于数组，键名就是索引
- keys() 返回一个遍历器对象，用来遍历所有的键名
- values() 返回一个遍历器对象，用来遍历所有的键值
Tips:
Set数据结构的 keys() 和 values()返回的是相同的迭代器，也就是说Set数据结构中键名和键值相同
Set默认迭代器是values(), Map默认是entries()

Tips: for循环的实质
**for(initialize;test;increment) statement**
相当于 
```javascript
initialize
while(test) {
  statement
  increment
}
```
TODO: 暂时先跳过
## [模拟实现Set](https://github.com/mqyqingfeng/Blog/issues/91)
## [WeakMap](https://github.com/mqyqingfeng/Blog/issues/92)

## [聊聊Promise](https://github.com/mqyqingfeng/Blog/issues/98)
TODO: Promise A+ 规范的理解
回调地狱的问题
- 难以复用
- 堆栈信息被断开，出错难以追溯

Tips:
Promise.race()处理错误
```javascript
/**
 * func1 为业务函数
 * func2 为错误处理函数
 */
func2() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(new Error('time out'))
    }, 5000)
  })
}
// 这样如果func1未按预期执行，就会func2会先触发，进入catch流程
Promise.race([func1, func2]).then(...).catch(...)
```
Promise 内部的错误不会影响到外部的代码, “吃掉错误”
## [Generator的自动执行](https://github.com/mqyqingfeng/Blog/issues/99)
TODO: 暂跳过
## [babel编译class上](https://github.com/mqyqingfeng/Blog/issues/105)
## [babel编译class下](https://github.com/mqyqingfeng/Blog/issues/106)
类内部定义的所有方法都是不可枚举的
```javascript
class Person {
  constructor(name) [
    this.name = name
  ]
  sayHello() {
    console.log('hh' + this.name)
  }
}
// 等价于
function Person(name) {
  this.name = name
}
Person.prototype.sayHello = function() { console.log('hh' + this.name) }
```
static 关键字标识的方法为静态方法，只能通过类调用，类的实例不能调用
### 继承extends
```javascript
class child extends parent {
  constructor(props) {
    super(props)
  }
}
```
super 表示父类的构造函数，相当于es5的 parent.call(this)
子类没有自己的this对象，需要继承父类的this对象。
父类的静态方法可以被子类继承是因为，class作为构造函数的语法糖，同时拥有prototype和__proto__属性
```javascript
child.__proto__ === parent
child.prototype.__proto__ === parent.prototype
```
![class继承图](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/class/class-prototype.png)

## [聊聊Async](https://github.com/mqyqingfeng/Blog/issues/100)
Async实际上是Generator的语法糖形式
```javascript
async function(args) {
  ...
}
function fn(args) {
  return spawn(function* () {
    ...
  })
}
```
### 继发和并发
```javascript
// 继发形式
async function() {
  await fn1()
  await fn2()
  await fn3()
  return 'all done'
}
// 并发形式
async function() {
  var res = Promise.all([fn1, fn2, fn3])
  return 'all done'
}
```
## [defineProperty和proxy](https://github.com/mqyqingfeng/Blog/issues/107)
都可用于处理数据绑定
### Object.defineProperty 
可以在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象
> Object.defineProperty(obj, prop, descriptor) 
obj: 修改的对象，prop: 修改的属性，descriptor: 修改属性的描述符(**必须**)
```javascript
var obj = {}
Object.defineProperty(obj, num, {
  value: 1,
  writable: true,
  enumerable: true,
  configurable: true
})
```
descriptor 有两种形式，数据描述符和存取描述符,只能是其中一种，不能两者都是
两者均有 configurable 和 enumerable
数据描述符有 value 和 writable
存取描述符有 get 和 set
> get: 一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。该方法返回值被用作属性值。默认为 undefined。
> set: 一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。该方法将接受唯一参数，并将该参数的新值分配给该属性。默认为 undefined。
### proxy
defineProperty只能重定义set和get, proxy可以重定义更多的行为
>var proxy = new Proxy(target, handler)
- target 拦截的目标对象
- handler 拦截的行为

### defineProperty和proxy异同
- 都可以起到 **监听数据** 的作用
- defineProperty只能用到get和set属性,proxy可以代理多达13种属性（has, apply...等）
- 使用defineProperty只要修改源对象就会触发监听数据，修改的是源对象
var proxyInstance = new Proxy(obj, handler)
- proxy只有修改 生成的实例 proxyInstance 才会触发源对象 obj 上的数据监听
## [模块加载规范](https://github.com/mqyqingfeng/Blog/issues/108)
- AMD (Require.js在推广过程中对模块化定义的规范产出) 讲究 **依赖前置**,即提前依赖好,提前执行
- CMD (Sea.js在推广过程中对模块化定义的规范产出) 讲究 **依赖就近**,即随用随依赖,延迟执行
AMD是把模块加载完再执行代码
CMD是在require的时候才去加载模块文件，加载完接着执行代码
AMD和CMD都是浏览器端的模块规范
- CommonJS
CommonJS是服务器node端的模块规范
也是在require的时候才加载模块，加载完接着执行
- CommonJS和AMD
> CommonJS加载模块是同步的，也就是说，只有模块加载完了才会执行后面的操作
> AMD规范是非同步加载模块，允许指定回调函数
> 由于 Node.js 主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以 CommonJS 规范比较适用。
> 但是浏览器端如果要从服务器端加载模块，就必须采用非同步模式，因此浏览器端一般用AMD规范
- ES6模块
浏览器加载ES6模块用 `<script src='...' type='module'>`
ES6模块是将模块加载完再执行代码
- ES6和CommonJS
> CommonJS输出的是一个值的拷贝,ES6输出的是对值的引用
> CommonjS模块是运行时家长，ES6模块是在编译时输出接口
> CommonJS加载的是一个对象（即module.exports属性）,该对象只有在代码执行完后才会生成。而ES6模块不是对象，他的对外接口只是一种静态定义，在代码静态解析阶段就会生成
> CommonJS输出的是一个值的拷贝，也就是说，一旦输出一个值，模块内部的变化影响不到这个值
> ES6是动态引用，不会缓存值，模块里面的变量绑定其所在的模块
浏览器不支持CommonJS语法是因为浏览器端没有 exports, require, module等环境变量
Babel直接转ES6代码会把它转换成CommonJS规范代码，是不能直接跑在浏览器的，需要webpack打包,webpack实际上模拟了这些变量的行为
