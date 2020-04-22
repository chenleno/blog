---
title: QA归档.md
updated: 2020-04-22 11:31:24
date: 2020-04-08 11:19:02
tags:
- QA
- React
categories:
- QA
---
# QA归档
归档总结常见的QA问题，问题驱动学习他不香吗~
## React fiber架构
fiber(协程)
react 执行数据更新需要递归遍历VDOM树，将需要变动的节点收集起来，然后同步更新他们，过程称为reconcilation（协调）。
在reconcilation期间，react会霸占浏览器资源，导致卡顿或掉帧。

react通过fiber,让reconcilation变得可以中断，适时的让出cpu执行权，可以让浏览器及时响应用户，体验更好.

确定是否有高优先级任务的api: requestIdleCallback
通过**超时检查**机制来确认，即确认一个超时时间，如果任务执行时间超过了，则暂停执行
为了避免任务被饿死，react定义了一个超时时间，低优先级可以慢慢等待，高优先级要立刻执行

fiber另一种解读是（纤维）,这是一种数据结构或者说执行单元，react每次执行完一个执行单元，就会看看还有没有时间，如果没有时间了，就将执行权让出去
fiber将调用栈转换为链表结构，这样即使中断了也能在下一次执行时继续
fiber的协调阶段和提交阶段
协调阶段会找到所有节点变更，可以被中断
提交阶段会将找到的变更一次执行了，必须同步执行，不能被中断(不能中断是因为要保证任务只执行一次)

fiber 的实现是非常复杂的东西。
## 宏任务和微任务
参考资料：
[阮一峰的宏任务微任务介绍](http://www.ruanyifeng.com/blog/2018/02/node-event-loop.html)

![js同步异步任务图](https://user-gold-cdn.xitu.io/2018/7/14/164974fb89da87c5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
宏任务和微任务都属于异步任务，它们都属于一个队列，主要区别于它们的执行顺序
![宏任务微任务图](https://user-gold-cdn.xitu.io/2018/7/14/164974fa4b42e4af?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
执行描述
1. 主线程执行栈执行完了
2. event loop 去查看微任务队列，如果有，就把它们全**执行完**
3. event loop 去查看宏任务队列，如果有，就拿出一个放到主线程执行栈执行
4. 重复123...

总结:
- 所有会进入的异步都是事件回调中的那部分代码（如：new Promise()在实例化的过程中是同步的，之后的then中注册的回调是异步的）
- 所有同步任务执行完后才会去执行当前已注册的微任务，所有微任务都执行完后才会去执行一个新的宏任务

### 如何界定宏任务和微任务
宏任务：
- I/O
- setTimeout, setInterval, setImmediate
- requestAnimationFrame
微任务：
- process.nextTick, promise.then, catch finally
**process.nextTick追加在其他微任务之前**
### node中的 event loop和宏微任务之间的关系
node的event loop依赖libuv调度
node 事件循环的六个阶段
- timers (处理 setTimeout 和 setInterval的回调，是否执行根据当前时间是否满足回调条件)
- I/O callbacks (除了以下操作的回调，其他的都在这里执行)
  + setTimeout和setInterval
  + setImmediate
  + 用于关闭请求的回调（如：socket.on('close', ...）
  + **Tip:执行I/O回调时可能会遇到定时器到时了，但也必须执行完当前回调后才会去执行定时器的，这也是为什么有时候定时器回调执行时间会晚于定时时间的原因**
- idle, prepare (libuv内部调用，可以忽略)
- poll (轮询时间，用于等待还未返回的I/O事件或定时器)
- check (执行setImmediate回调)
- close callbacks （执行关闭请求的回调，如：socket.on('close', ...)）
每个阶段都有一个先进先出的回调函数队列，只有一个阶段的回调函数队列清空了，该执行的回调函数都执行了，事件循环才会进入下一个阶段
```javascript
// 思考题
fs.readFile('test.js', () => {
  setTimeout(() => console.log('timeout'))
  setImmediate(() => console.log('immediate'))
})
// 该代码一定会先输出 immediate, 后输出timeout
// 解析：
// 第一轮loop, 解析到I/O操作，在poll状态等待I/O执行完毕，I/O执行完毕，进行第二轮loop
// 第二轮loop, 在 I/O阶段执行回调，检测到setTimeout和setImmediate, 在check阶段执行 setImmediate的回调，输出"immediate",进入第三轮
// 第三轮loop, 在timer处处理setTimeout的回调，输出"timeout"

```
## 代码不写;会导致的问题
```javascript
/**
 * 乱序方法
*/
function shuffle(arr) {
  for(let i=arr.length;i;i--) {
    let j = Math.floor(Math.random() * i)
    [arr[i-1], arr[j]] = [arr[j], arr[i-1]]
  }
  return
}
shuffle([1,2,3,4,5])
// Uncaught ReferenceError: Cannot access 'j' before initialization
```
代码执行会报错，变量j进入了临时死区报错
经分析，问题出在这里
```javascript
  let j = Math.floor(Math.random() * i) 
  [arr[i-1], arr[j]] = [arr[j], arr[i-1]] // 新行以 ( [ / + - * % , . 开始,会与上一行未加;的代码拼在一起解析
```
因此需要在
```javascript
  let j = Math.floor(Math.random() * i); // 需要在这里加个;
```
真坑爹，原本以为js加不加;只是代码风格问题，没想到这里也有坑，真是学到了学到了...
## session, cookie和token
session和cookie都是为了弥补http协议的无状态性
### session
客户端访问服务端时，服务端在内存上开辟一块空间，这个对象便是session对象
1. 服务端第一次收到请求，创建session对象，并生成一个sessionId
2. 通过响应头的set-cookie: JSESSIONID=xxxx,向客户端发送要求设置cookie的响应
3. 客户端收到响应后，在本机客户端设置了一个JSESSIONID=xxx的cookie信息，过期时间为会话结束
4. 接下来客户端向同一个站点发送请求，请求头上都会带上该cookie信息，服务端通过读取cookie信息得到本次请求的sessionId
![set-cookie要求](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0405%2F19c183f7p00q8b9mk001md200u000aog015w00ew.png&thumbnail=690x2147483647&quality=75&type=png)
session的缺点：
假如只有a服务器存储了session,负载均衡转发到b服务器，b服务器无法访问到a的sessionId,会导致session失效
### cookie
他是服务器发到浏览器的一小块数据，浏览器进行存储，并与下一个请求一起发送到服务器，cookie通常用来判断两个请求是否来源于同一个浏览器
cookie有两种类型，分别是**session cookie**和**persistant cookie**
- session cookie
  + 不会写入磁盘，没有过期时间，会在会话结束后删除
- persistant cookie
  + 会写入磁盘，有expires或者max-age过期时间，并在到期后删除
cookie的http-only属性指定该cookie是否可以通过客户端脚本访问,为true时可能导致cookie被窃取
cookie的domain和path属性指定该cookie应该发送给哪些url
cookie的SameSite属性可以让cookie在跨站请求时不被发送，从而阻止跨站请求伪造攻击（CSRF）
SameSite的三个值
- strict (仅允许一方请求携带 Cookie，即浏览器将只发送相同站点请求的 Cookie，即当前网页 URL 与请求目标 URL 完全一致。)
- lax (允许部分第三方请求携带 Cookie)
- none (无论是否跨站都会携带cookie)

### token
token 和session cookie就是用来保证用户在不同页面间切换，保存登录信息的机制
token 由三部分组成
- uid 用户唯一身份标识
- time 时间戳
- sign + [固定参数]
token认证流程
1. 用户登录，成功后服务器返回token给客户端
2. 客户端收到数据后保存在客户端
3. 客户端再次访问服务器，token放在header中
4. 服务器校验，校验成功返回数据，不成功返回错误码

token可以抵抗csrf,session+cookie不行
客户端登录将身份信息发给服务端，服务端把信息加密（token）传给客户端，客户端将token存放于localStorage中，客户端每次访问都传递token给服务端，服务端解密token获取用户身份，这种方法叫做jwt(JSON web token)
### 总结
- session存放于服务端，可以理解为一个状态列表，拥有一个唯一识别符合sessionId，通常存放于cookie中。服务端收到cookie后解析出sessionId,再去session列表中查找，找到相应session。依赖cookie
- cookie类似一个令牌，拥有sessionId,存放于客户端，浏览器通常会自动添加
- token也类似一个令牌，无状态，用户信息被加密到token中，服务端收到后解密就可以知道哪个用户，需要手动添加
- jwt只是一个跨域认证方案
## 网络层问题
### TCP 三次握手四次挥手
- TCP三次握手（建立连接前，客户端和服务端需要握手来确认对方）
  + 客户端发送syn(同步序列序号)请求，进入syn_send状态，等待确认
  + 服务端接收并确认syn后，发送syn+ack包，进入syn_recv状态
  + 客户端收到syn+ack包后，发送ack包，双方进入established状态
- TCP四次挥手
  + 客户端 -- FIN --> 服务端，FIN-WAIT
  + 服务端 -- ACK --> 客户端，CLOSE-WAIT
  + 服务端 -- ACK,FIN --> 客户端，LAST-ACK
  + 客户端 -- ACK --> 服务端，CLOSE
## hybrid App webView和native交互总结
hybrid app 就是在原生app的某个页面中放置了一个webView,用来展示web页面。web页面可以通过某种方式和原生app交互。webView相当于一个微型浏览器，内部有window等全局对象。web端和app端约定好事件名和数据格式，通过挂载在window上的方法来进行通信。
### 理论上的
前端开发web页面生成静态文件上传到server端并打成zip包，包有hash值。客户端打开app时检查包是否有更新并下载解压。在WebView访问时直接访问本地file文件。

首先确定scheme协议：'custom://xxx'
> scheme 协议能让app端定位到具体跳转页面，功能等，依赖app端约定
```javascript
// web端
const scheme = 'custom://xxx/details?id=111&callback=__callback__'
<a href=`${scheme}`></a>
(
  function _invoke(action, data, callback) {
    let scheme = 'custom://xxx' + action + '?'
    for(let i in data) {
      if (data.hasOwnProperty(i)) {
        scheme += i + '=' + data[i] + '&'
      }
    }
    let callbackName = ''
    if (callback && typeof callback === 'function') {
      callbackName = action + +new Date()
      window[callbackName] = callback
    }
    scheme += `callback=${callbackName}`
  }
  window.evoke = {
    share: function(data, callback) {
      _invoke('share', data, callback)
    }
  }
)()
```
### 项目中的
项目中看起来并没有打包zip文件，都是app访问对应的web页面。由app端和web端一起约定事件，scheme协议没有对外暴露。
首先确定交互事件api
- ios: `window.webkit.messageHandlers.NativeApi.postMessage({ method: actionName, ...(params || {}) })`
- android: `window.Native[actionName](JSON.stringfy(params))`
```javascript
class callNativeApi {
  constructor() {
    const ua = navigator.userAgent
    this.callbackBucket = {}
    if （/android/i.test(ua)）{
      this.platform = 'android'
    }...
    window.callBackHandler = this.callBackHandler.bind(this)
  }
  callNative(actionName, params) {
    if(params.callback && typeof params.callback === 'funciton') {
      const actionId = +new Date()
      this.callbackBucket[actionId] = params.callback
      params.actionId = actionId
      delete params.callback
    }

    if (this.platform === 'ios') {
      window.webkit.messageHandler.NativeApi.postMessage({ method: actionName, ...(params || {}) })
    } else if (this.platform === 'android') {
      window.Native[actioName](JSON.stringify(params))
    }
  }
  callBackHandler(arg) {
    const { actionId, data } = arg
    this.callbackBucket[actionId](data)
    delete this.callbackBucket[actionId]
  }
  jumpToUrl(url) {
    this.callNative('jumpToUrl', { url })
  }
}
export default new callNativeApi()
// 用例
...
import NativeApiCallable from './nativeApi'
NativeApiCallable.jumpToUrl('xxx')
```
## vdom相关问题
vdom就是用js模拟dom,dom变化的对比，放到js层来做
[snabbdom](https://github.com/snabbdom/snabbdom)
### 核心api
- h(tag, props, children) 生成vnode节点
- dispatch()
  + dispatch(container, vnode) 将vnode塞进真实dom节点
  + dispatch(vnode, newVnode) 比对新旧vnode
### diff算法
不考虑边界情况
- createElement(vnode)
```javascript
function createElement(vnode) {
  const tag = vnode.tage
  const attrs = vnode.attrs
  const children = vnode.children
  const elem = document.createElement(tag)
  for(let i in attrs) {
    if (attrs.hasOwnProperty(i)) {
      elem.setAttribute(i, attrs[i])
    }
  }
  children.forEach(child => {
    elem.appendChild(createElement(child))
  })
  return elem
}
```
- updateChildren(vnode, newVnode)
```javascript
function updateChildren(vnode, newVnode) {
  const children = vnode.children
  const newChildren = newVnode.children
  children.forEach((child, index) => {
    const newChlid = newChildren[index]
    if (newChlid === null ) return
    if (child.tag === newChild.tag) {
      updateChildren(child, newChild)
    } else {
      replace(child, newChild)
    }
  })
}
```
## 理解MVVM
