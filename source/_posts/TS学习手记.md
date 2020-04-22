---
title: TS学习手记
date: 2020-04-04 15:18:47
tags:
- TypeScript
categories:
- web前端
---
# TypeScript
TypeScript作为今年的重点学习对象，在此记录学习过程和总结。
参考资料：
[从JS到TS](https://github.com/xcatliu/typescript-tutorial)
## 基础类型
- string
- number
- boolean
- underfined
- null
- any
- symbol
## 接口 interface
interface 常用来描述一个对象有哪些类型的属性
```javascript
interface Ifoo {
  name: string
  age?: number
  sayHello(props: string | number): void
}
function foo: Ifoo() {}
```
## 泛型 <T>
> 在定义函数或者接口或者类时，不预先定义具体的类型，而是在使用时再定义的一种特性
比如一个函数接收的参数即可能是字符串数组也可能是数字数组时
```javascript
interface Ibar<T> {
  props: T[]
}
let bar: Ibar<string | number> = props => console.log(props)
```
## 类型断言
> 可以用来手动指定一个类型的值
常用在，当ts不知道某个值的类型，而却要使用这种类型的值才有的方法的时候
```javascript
function (props: string | []) {
  console.log(props as [].map(i => console.log(i)))
}
```
两种写法：
- 值 as 类型 (tsx中只能用这种)
- <类型>值
## 声明文件x.d.ts
想要在别的项目上开发时，也拥有方法，校验提示，就需要在项目内编写声明文件
```javascript
// xxx.d.ts
declare namespace <object> {
  a: string
  b: string
}
```
使用webpack打包时，package.json 中要指明 types 属性的入口，该属性用于引入声明文件
```javascript
// package.json
{...
  types: 'dist/index.d.ts'
...}
```
## type 定义类型别名和字符串字面量约束
可以用type 来定义类型别名
```javascript
type Tstring = string
const a: Tstring = 'hhh'
```
也可以定义字符串约束
```javascript
type Tstring = 'click' | 'dbclick' | 'move'
const b: Tstring = 'hhh' // 错，b只能是约束中的一种
```


