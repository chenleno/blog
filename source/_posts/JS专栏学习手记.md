---
title: JS专栏学习手记
date: 2020-04-03 18:40:11
tags:
- JS基础
categories:
- web前端
---
# JS基础复习专栏
无意间看到了冴羽大大的JS专栏系列，仿佛置身宝库，知识点的讲解非常好，特将学习笔记和理解记录如下。
附宝库链接~~
[冴羽's blog](https://github.com/mqyqingfeng/Blog)
## [原型到原型链](https://github.com/mqyqingfeng/Blog/issues/2)
都除了null, null表示“此处没有对象”，即不应该有值
只有函数才有prototype
原型是一个对象(每个javascript 对象在创建时都会关联另一个对象，这个对象就叫原型，每个对象都会从原型继承属性)
函数的 prototype 指向 => 这个构造函数所构造实例的原型对象
![原型和实例的关系](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype1.png)
__proto__
每一个javascript对象都会有一个__proto__属性,这个属性指向该对象的原型 (函数也是对象！！！)
![__propo__关系](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype2.png)
constructor
每个原型都有一个属性constructor 指向该原型的构造函数
![constructor关系](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype3.png)

实例和原型
读取实例属性时，如果找不到，会去找实例原型的属性，如果找不到，会去找原型的原型...
![原型和Object和null](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype5.png)
原型链是**对象**的指向链 实例.__proto__ => xxx.prototype .__proto__ => Object.prototype .__proto__ => null
__proto__ 实际上来源于 Object.prototype 
相当于一个getter/setter,即只要访问 obj.__proto__ 都会 返回 Object.getPrototypeOf(obj)
实例的constructor是从原型上继承过来的
## [词法(静态)作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3)
js是词法作用域
作用域就是定义变量的区域,规定了对变量的访问权限
词法作用域：函数定义时确定的
动态作用域：函数调用时确定的
(一个是书写位置，一个是调用位置)
**作用域链是在函数定义时确定的,函数的作用域基于函数创建时的位置**
## [执行上下文栈（ECS)](https://github.com/mqyqingfeng/Blog/issues/4)
js引擎是一段一段解析代码的
段包括三种： 全局代码，函数代码，eval代码
执行栈[]最底层永远是个globalContext
js解析代码时，遇到函数调用，就会压入执行栈[].push(FuncContext)
[].pop()
函数**执行**的时候，创建执行上下文，压入执行上下文栈，函数**执行完毕**的时候，该执行上下文从栈中弹出
## [变量对象(VO)](https://github.com/mqyqingfeng/Blog/issues/5)
对于每个执行上下文，都有三个重要属性 **变量对象，作用域链，this**
变量对象是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明
全局上下文中的变量对象就是全局对象
函数上下文中用活动对象（AO）表示变量对象
**VO和AO是同一对象处于执行上下文的不同周期**
活动对象是在进入执行上下文中时创建的,他通过函数的arguments属性初始化，arguments的属性值是Arguments对象
- 全局上下文的变量对象初始化就是全局对象
- 函数上下文的变量对象初始化只包括Arguments对象
- 在进入执行上下文时会给变量对象添加 **形参，函数声明，变量声明**等初始的属性值
- 在执行阶段，会再次修改变量对象的属性值
Tip: 何谓Arguments对象
> 调用函数时，会为其创建一个Arguments对象，并自动初始化局部变量arguments,指代该Arguments对象。所有作为参数传入的值都会作为Arguments对象的数组元素
## [作用域链](https://github.com/mqyqingfeng/Blog/issues/6)
查找变量时，会从当前上下文的变量对象中查找,如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。**这样由多个执行上下文的变量对象构成的链表就叫做作用域链**
## [从EcmaScript规范角度解读this](https://github.com/mqyqingfeng/Blog/issues/7)
Reference
> 这里的 Reference 是一个 Specification Type，也就是 “只存在于规范里的抽象类型”。它们是为了更好地描述语言的底层行为逻辑才存在的，但并不存在于实际的 js 代码中。
GetValue
**调用getValue,返回的是一个具体的值，而不再是reference**
此章节需要结合EcmaScript文档学习
## [闭包](https://github.com/mqyqingfeng/Blog/issues/9)
> 闭包即能访问自由变量的函数
> 自由变量即既不是函数的参数，也不是函数局部变量的变量
> 闭包 = 函数 + 函数能够访问的自由变量
理论上，所有的js函数都是闭包，因为都被包裹在全局上下文中
实践上：
- 即使创建函数的上下文已经销毁，他依然存在（例如内部函数被return出来）
- 代码中引用了自由变量
## [参数按值传递](https://github.com/mqyqingfeng/Blog/issues/10)
ECMAScript规定函数传参都是**按值传递**的
但是会有传递对象进去却修改了原对象的现象发生
其实还有一种方式是**按共享传递**
即**传递对象的引用的副本** （区别于按引用传递，按引用传递传递的是对象的引用）
总结：
参数为基本类型时，按值传递
参数为引用类型时，按共享传递
### 个人思考
实际上是信息存储在栈和堆上的问题，实际上传递的是栈上的内容
存储对象时将值存储在堆里，将堆地址存在栈上
```javascript
// 函数接收形参，会在栈上给形参开辟一个新的存储空间,记作n
function a(o) {
  /**
   * 接收对象类型的实参，n实际上接收了栈上的堆地址值的副本，（即对象引用的副本）
   * 通过该引用可以找到堆上的对象，进行了修改
   */
  o.value = 1
}
function a(o) {
  /**
   * 这里n接收了地址值副本后，进行了重新赋值，把引用的副本覆盖了，故原对象不会被修改
   */
  o = 1
}
a(obj)
```
## [call和apply的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)
> call方法在使用一个指定的this值和若干个参数的前提下调用某个函数或方法
见实际代码
## [bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)
> bind方法会创造一个新函数，当这个新函数被调用时，使用传入的第一个参数作为this指向，其后传入的一序列参数会在传递的实参前传入作为他的参数
**call,apply,bind**都会改变指定的对象，对象会具有调用函数中和this相关的特性，相当于把这个对象在函数里过了一遍
## [new的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)
new 实现的过程
1. 创建一个空对象 obj = new Object()
2. 找到构造函数
3. 将空对象的obj.__proto__指向构造函数的prototype
4. 过一遍 构造函数.apply(obj, arguments)
5. return 出 obj (如果构造函数有返回值，还需要判断返回值的类型) return ret instanceof Object ? ret : obj
## [类数组对象与arguments](https://github.com/mqyqingfeng/Blog/issues/14)
> 拥有一个length属性和若干索引属性的对象
类数组调用数组方法需要借助Function.call
类数组转数组：slice,splice,map,cancat.apply,Array.from
Arguments对象只定义在函数体中，包含了函数的参数和其他属性。在函数体重，arguments指代该函数的Arguments对象
arguments的length只和实参个数有关
## [创建对象的多种方式](https://github.com/mqyqingfeng/Blog/issues/15)
- 工厂模式
- 构造函数模式
- 原型模式
- 动态原型模式
- 混合模式
- 寄生构造模式（就是工厂模式使用 new 创建对象）
- 稳妥模式 （不适用new 构建，函数内不使用this）
## [继承的多种方式及优缺点](https://github.com/mqyqingfeng/Blog/issues/16)
- 原型链继承
  + 引用类型的属性被所有实例共享
- 借用构造函数（经典继承）
- 组合式继承（最常用的模式）
- 原型式继承
模拟实现Object.create()
```javascript
function create(o) {
  function F() {}
  F.prototype = o
  return new F()
}
```
- 寄生式继承
- 寄生组合式继承 （最理想的范式）
```javascript
function object(o) {
  function F(){}
  F.prototype = o
  return F()
}
function prototype(child, parent) {
  var prototype = object(parent.prototype)
  prototype.constructor = child
  child.prototype = prototype
}
prototype(child, parent)
// 只调用了一次parent构造函数，不丢失原型链
```
## [浮点数精度问题](https://github.com/mqyqingfeng/Blog/issues/155)
ECMAScript采用64位双精度浮点数
## [类型转换](https://github.com/mqyqingfeng/Blog/issues/159)
