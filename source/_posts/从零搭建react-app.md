---
title: 从零搭建react app
date: 2020-04-16 15:01:03
tags:
- react
categories:
- web前端
---
# 从零搭建react app
## 前情提要
使用react开发项目一年有余，我也从一个react小白成长为了一名react开发者（笑）。期间基本是问题驱动学习，因为要学的太多所以指哪打哪，缺乏一个对react技术栈的系统归纳与总结。所以在疫情期间趁此机会从头搭建了一个基于react全家桶的仿后台管理项目，一套下来，感触多多，行文一篇，聊表初心。
## 项目简介
使用React开发的类后台管理app。
基于create-react-app脚手架构建项目，渐进引入react-router, react-redux, redux-thunk, antd等。
并在后期进行了 **路由懒加载**， **redux懒加载** 的功能实践。后续还会基于需求驱动加入webpack优化相关的内容。
[项目github: react-app](https://github.com/chenleno/react-app)
## 功能及结构说明
项目的基本功能包含一个登陆页面，登陆后主页面包含Header,SideBar和Content部分，对应显示 人员信息，侧边栏菜单和内容部分。
页面之间的跳转基于react-router,同时在登录后初始化 redux, store可以被任意页面引用。
User模块包含一个嵌套路由，用以跳转至用户详情。
### 结构信息
    |-- index.js              // 项目主入口
    |-- App.jsx               // render App
    |-- components            // 公共组件
    |-- conf                  // 配置目录，存放API等
    |-- layout                // 布局
    |    |-- Header.jsx       // 头部导航
    |    |-- Sidebar.jsx      // 侧边导航
    |-- modules               // 业务模块
    |    |-- dashbard         // 工作台
    |    |-- login            // 登录页
    |    |-- setting          // 设置页
    |    |-- user             // 人员管理
    |    |    |-- detail      // 人员详情
    |-- reducers              // redux相关
    |-- utils                 // 工具函数
    |-- routers.js            // router集合
    |-- store.js              // redux store
## 搭建流程介绍
### 基础搭建
基础搭建部分可以参考官网实例和相关资料，如：
[通过create-react-app 从零搭建react环境](https://segmentfault.com/a/1190000015301231)
1. 安装 create-react-app 脚手架，按官网实例操作即可。
2. 尝试引入antd
```javascript
$ yarn add antd
// app.jsx
import { Button } from 'antd'
<Button />...
```
此时发现了问题，引入的antd组件样式没有生效。
解决： 
- 运行 npm run eject 命令，分离配置文件
- 安装 babel-plugin-import （一个用于按需加载组件代码和样式的 babel 插件）
- 修改package.json,添加如下代码
```javascript
{ ...,
  "babel": {
    "presets": [
      "react-app"
    ],
    "plugins": [
      ["import", { "libraryName": "antd", "libraryDirectory": "es", "style": "css" }]
    ]
  }
}
```
现在就可以看到正常样式的antd组件了
### 引入react-router 
[react-router 4.x文档](https://reacttraining.com/react-router/web/example/basic)
安装新版react-router及react-router-dom
```javascript
yarn add react-router react-router-dom
```
项目安装的react-router是最新的5.x 版本，react-router 4x版本上下使用时有较大区别，主要体现在：
- 旧版本的Router组件集成在react-router中，新版本则要导入 react-router-dom, 并且 组件还分为 `<Browserrouter/>`, `<NativeRouter/>`等.
```javascript
// 3.x
import { Router } from 'react-router'
<Router>...</Router>
// 4.x
import { BrowserRouter, Route } from 'react-router-dom'
<BrowserRouter>...</BrowserRouter>
```
- 新版本react-router 采用**组件化**的方式设置路由，路由组件<Route/>就像react组件一样包裹在外层就可以
1. 首先在App最外层外包BrowserRouter
```javascript
// index.js
import { BrowserRouter } from 'react-router-dom'
import App from './App'

ReactDOM.render(
  <BrowserRouter>
    <Route path={'/'} component={App}/>
  </BrowserRouter>
, 
document.getElementById('root'));
```
2. 然后在App中注册路由及组件
```javascript
// App.jsx
import { Route, Switch, Redirect } from 'react-router-dom'

class App extends React.Component {
  get isLogin() { // 根据isLogin参数来判断登录状态，isLogin来源于redux和localStorage是否存有登录信息
    return localStorage.getItem('profile')
  }
  render() {
    return (
      <React.Fragment>
        {this.isLogin ?
          <MainView >
            <Header class/>
            <MainContent>
              <SideBar className='sidebar'/>
              <Content>
                <Switch>
                  {/* <Redirect path={'/'} to={'/dashboard'}/> */}
                  <Route path={'/dashboard'} exact component={Dashboard}/>
                  <Route path={'/user'} exact component={User}/>
                  <Route path={'*'} component={NotFoundPage}/>
                </Switch>
              </Content>
            </MainContent>
          </MainView> 
        : 
        <React.Fragment>
          <Route path={'/login'} render={() => <Login {...this.props}/>}/>
          <Redirect to={{ pathname: '/login', from: this.props.location }} />
        </React.Fragment>
        }
      </React.Fragment>
    )
  }
}
```
3. 在使用层添加跳转逻辑即可
```javascript
// Sidebar.jsx
import { useHistory } from 'react-router-dom' 
// 新版react-router使用history替代旧版的router, 提供了hooks，可以用来获取history,params,location等
  const history = useHistory()
{
  <Menu.Item onClick={() => history.push('/dashboard')} key="1">dashboard</Menu.Item>
  <Menu.Item onClick={() => history.push('/user')} key="2">setting</Menu.Item>
}
```
### 引入react-redux
[redux文档](https://cn.redux.js.org/)
redux是一套数据管理方案，react-redux是该方案同react结合的具体实现。
1. 安装redux, react-redux
```javascript
yarn add redux react-redux
```
2. 各模块编写reducer
```javascript
// reducers/login.js
const initState = {
  profile: {},
  isLogin: false
}

const reducer = (state = initState, action) => {
  switch (action.type) {
    case 'LOGIN':
      localStorage.setItem('profile', action.data)
      return {
        ...initState,
        profile: action.data,
        isLogin: true
      }
    case 'LOGOUT':
      localStorage.clear()
      return initState
    default:
      return initState
  }
}
export default reducer
// reducers/index.js
import login from './login'
import user from './user'
import { combineReducers } from 'react-redux'
const rootReducers = combineReducers({
  login,
  user
})
export default rootReducers
```
3. 创建store文件
```javascript
import { createStore } from 'redux'
import rootReducers from './reducers/index'

const store = createStore(rootReducers)
export default store
```
4. 在App最外层外包store,这样项目便受控于redux
```javascript
// index.js 
import { Provider } from 'react-redux'
import store from './store'
<Provider store={store}>
  <BrowserRouter>
    <Route path={'/'} component={App}/>
  </BrowserRouter>
</Provider>
```
5. 使用 connect 链接组件与redux
```javascript
// login.jsx
import { connect } from 'react-redux'

const mapStateToProps = ({ profile }) => ({ profile })

const loginData = {}

const Login = props => {
  const { dispatch } = props
  const handleClick = () => {
    $axios.post('/api/xxx', { data: loginData }).then(resp => {
      dispatch({
        type: 'LOGIN',
        data: resp
      })
    })
  }
  return (
    <div>
      ...
      <Button onClick={() => handleClick()}>登录</Button>
    </div>
  )
}
export default connect(mapStateToProps)(Login)
```
实际项目中会将dispatch(action) 进行进一步的抽象
### 引入redux-thunk处理异步action
[redux-thunk使用介绍](https://zhuanlan.zhihu.com/p/85403048)
redux模型中，由事件源dispatch一个action,reducer捕获到对应的action之后进行处理，更新redux.state。此处的action必须是一个plain-object。而redux-thunk的作用就是允许我们dispatch一个function,这就给了我们很大的操作空间。
1. 安装redux-thunk
```javascript
yarn add redux-thunk
```
2. store 中使用redux-thunk
```javascript
// store.js
import thunk from 'redux-thunk'
import { applyMiddleWare } from 'react-redux'

...
const store = createStore(rootReducers, 
// redux-thunk是作为redux的中间件使用的
  applyMiddleWare(thunk)
)
...
```
3. 编写action，并使用redux-thunk的特性
```javascript
// reducers/login.js
...
export const login = data => (dispatch, getState) => {
  $axios.post('/api/xxx', { data }).then(resp => {
    dispatch({ type: 'LOGIN', data: resp })
  })
}
```
这样就可以将异步逻辑封装在action中，上层无感知
```javascript
// login.jsx
import { login } from 'reducers/login'
const loginData = {}
const Login = props => {
  const { login } = props
  ...
  const handleClick = () => {
    login(loginData)
  }
  ...
}

export default connect(mapStateToProps, {
  login
})
```

## 项目优化
### 编写 httpMiddleWare 进一步封装异步action

### react-router 懒加载模块
[react-router 4.x路由懒加载方案](https://blog.whezh.com/react-redux-code-splitting/)
react-router 懒加载在 4.x 以前只需要运用 `router.getComponent()` 方法范式即可，该范式可以在访问到当前路由时才加载路由文件。

在4.x 之后由于取消了`router.getComponent()`方法，需要借助一个`AsyncComponent`组件和 webpack的 动态 `import()` 方法来实现
1. 编写一个AsyncComponet wrap组件（也可以直接使用 [react-loadble](https://github.com/jamiebuilds/react-loadable)这个库）
```javascript
// component/AsyncComponent.js
import React, { Component } from 'react';

export default (loader) => {
  class AsyncComponent extends Component {
    state = {
      component: null,
    };

    componentWillMount() {
      if (!this.state.component) {
        loader().then(module => {
          this.setState({ component: module.default });
        });
      }
    }

    render() {
      const Comp = this.state.component;
      return Comp ? <Comp {...this.props} /> : <p>loading</p>;
    }
  }

  return AsyncComponent;
}
```
2. 在路由组件导入处包装即可
```javascript
// App.jsx
import AsyncComponent from 'components/AsyncComponent'
const Dashboard = AsyncComponent(() => import('./Dashboard.jsx'))
const Login = AsyncComponent(() => import('./Login.jsx'))

// 使用处不变
...
```
之后打开控制台查看资源加载情况就可以看见文件是随路由分开加载的了。
![文件加载情况](https://s1.ax1x.com/2020/04/16/JA0LLQ.jpg)
Tips:
新版React内部支持`React.lazy`和`Suspense`，可以完成同样的事
```javascript
import { lazy, Suspense } from 'react'

const Dashboard = <Suspense>{lazy(() => import('./Dashboard.jsx'))}</Suspense>
// 使用处不变
```

### react-redux 懒加载redux模块
