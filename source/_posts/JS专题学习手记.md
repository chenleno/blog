---
title: JS专题学习手记
date: 2020-04-03 18:44:45
tags:
- JS基础
- 算法
categories:
- web前端
---
# JS专题学习手记
同样源自冴羽大大的blog系列，JS专题相对增加了更多对算法和方法实现上的思考，值得细细品味，目前专题还在持续学习中，具体的练习代码就不放在这里了。blog将分阶段更新。
同样是宝库链接：
[冴羽's blog](https://github.com/mqyqingfeng/Blog)
## [防抖debounce](https://github.com/mqyqingfeng/Blog/issues/22)
防抖和节流都是为了解决频繁事件触发
例如 lodash.debounce()
> 防抖即你随便触发事件，我只在你不再触发事件的n时间后执行事件
Tip: setTimeout 和 setInterval的返回值都是一个正整数，用来表示该计时器的编号，
使用 clearTimeout和clearInterval 方法不会清空返回值

## [节流throttle](https://github.com/mqyqingfeng/Blog/issues/26)
> 如果你持续触发事件,每隔一段时间，只执行一次事件
两种主流实现方式，一种是时间戳，一种是定时器
时间戳：第一次触发时立即执行，停止触发后不会再执行
定时器：不会马上执行，最后一次停止触发后wait后会再执行一次
Tip: 定时器的延时如果是一个负值时，他会忽略并使用最小延时,HTML5规格4ms（有浏览器差异）
TODO: 双剑合璧和options方式还需要理解
## [数组去重](https://github.com/mqyqingfeng/Blog/issues/27)
**sort()方法改变原数组**
- 双层去重
- 排序后去重（先将数组sort排序，排序后相同的值就会被挤在一起，然后只用判断当前值与前一个值是否相等即可）
- 对象去重 （obj.getOwnProperty 方法判断属性）
## [类型判断上](https://github.com/mqyqingfeng/Blog/issues/28)
- Object.prototype.toString()
调用 Object.prototype.toString 会返回一个由 "[object " 和 class 和 "]" 组成的字符串，而 class 是要判断的对象的**内部属性**
Tips: 判断数组是否是数组的四种方法
```javascript
Array.isArray([]) // true
[] instanceof Array // true
[].constructor === Array // true
Object.prototype.toString.call([]) // [object Array]
```
## [类型判断下](https://github.com/mqyqingfeng/Blog/issues/30)
- plainObject
> 纯粹的对象，有内部的键值对，和null,[],document等同样判断是对象的东西作区分
- window 对象
window对象是javascript客户端的全局对象，有一个window属性指向自身。
**window === window**，全等于，客户端只有一个window的引用地址
Tips: for...in... 会遍历原型上的属性， Object.keys不会
## [深浅拷贝](https://github.com/mqyqingfeng/Blog/issues/32)
数组浅拷贝
[].concat(), [].slice()都可以实现浅拷贝
问题是如果数组内都是基本类型，则拷贝值，数组内存在引用类型，只会拷贝引用
**即只拷贝栈上的东西**
深拷贝
一个简单粗暴的方法，不仅能拷贝数组还能拷贝对象 (但是不能拷贝函数)
Tips: 不安全的JSON值
> 为了简单起见， 我们来看看什么是 不安全的 JSON 值 。 undefined 、 function 、 symbol （ES6+）和包含循环引用（对象之间相互引用，形成一个无限循环）的 对象 都不符合 JSON 结构标准，支持 JSON 的语言无法处理它们。
```javascript
function(item) {
  return JSON.parse(JSON.stringfy(item)) // 有问题的，JSON.stringfy()调用底层也会调用toString,结果有时会不符合预期
}
// Tips: 还有ES6的...操作符
{...item}, [...item]
// 不安全的操作：
JSON.parse(JSON.stringify([1, undefined, symbol, 2])) // [1, null, null, 2]
```
## [JQuery extend](https://github.com/mqyqingfeng/Blog/issues/33)
第一遍看不动了，回头再看
## [求数组最大和最小值](https://github.com/mqyqingfeng/Blog/issues/35)
Math.min() Infinity
Math.max() -Infinity
Math.min() > Math.max() // true
Tips: [1,2,3,4]+'' 会隐式类型转换成 1,2,3,4
## [数组扁平化](https://github.com/mqyqingfeng/Blog/issues/36)
Tips: Array.concat(item) concat操作实际上添加的都是值，如果item是数组，则会添加数组中的值而不是数组
- 基础循环
- reduce方法
- while循环和...展开方法
## [实现findIndex方法](https://github.com/mqyqingfeng/Blog/issues/37)
es6新增的Array.findIndex方法，他会返回符合回调函数条件的第一个元素的索引，没有则返回-1
Array.findeIndex(function(item, i, array))
技能点：通过传参的方式，在一个循环中实现正序和倒序查找
二分查找: high, mid, low 二分区间
Tips: splice方法，
**splice(start, deleteCount, val1, val2...),从指定位置start起删除deleteCount项，并插入val1,val2...**
**splice会改变原数组，返回值是被删除的元素组成的数组**
## [柯里化](https://github.com/mqyqingfeng/Blog/issues/42)
一句话解释柯里化：
> 用闭包把参数保存起来，等到参数攒够了，再执行
```javascript
// 易懂的柯里化
function curry(fn, args) {
  var length = fn.length
  var args = args || []
  return function() {
    var _args = args.slice(0)
    for(var i=0;i<arguments.length;i++) {
      _args.push(arguments[i])
    }
    if(_args.length < fn.length) {
      return curry.call(this, fn, _args)
    }else {
      return fn.apply(this, _args)
    }
  }
}
var newCurry = curry(function(a,b,c) {
  console.log([a,b,c])
})
newCurry(1)(2)(3) // [1,2,3]
```
## [偏函数](https://github.com/mqyqingfeng/Blog/issues/43)

## [递归](https://github.com/mqyqingfeng/Blog/issues/49)
递归条件：
一个递归需具备返回条件，递归前进段和递归返回段
递归函数会不断的创造执行上下文栈，优化方式就是**尾调用**
尾调用不会创建多个执行上下文
> 尾调用：即函数的最后一个执行结果为函数调用，并且该调用的返回值返回给该函数
> 注意：只有不再用到外层函数的内部变量，内层函数的执行上下文栈才会替代外层函数的执行上下文栈，否则不会产生“尾调用优化”
函数调用自身，称为递归
尾调用自身，称为尾递归
优化方式：
把所有用到的内部变量改写成函数的**参数**
```javascript
// 尾调用
function f(x) {
  return g(x)
}
// 执行栈：
// globalContext.push(fContext)
// globalContext.pop()
// globalContext.push(gContext)
// globalContext.pop()
// 非尾调用
function f(x) {
  return g(x) + 1
}
// 执行栈
// globalContext.push(fContext)
// globalContext.push(gContext) 多创建了一个执行栈
// globalContext.pop()
// globalContext.pop()
```
## [v8排序解析](https://github.com/mqyqingfeng/Blog/issues/52)
v8对数组的排序是纯js实现的
数组小于10用插排，大于10用快排（这种说法不太严谨）
### 插入排序
![插入排序图](https://github.com/mqyqingfeng/Blog/raw/master/Images/sort/insertion.gif)
![原地快排](https://github.com/mqyqingfeng/Blog/raw/master/Images/sort/quicksort.gif)
