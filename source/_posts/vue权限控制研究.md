---
title: vue权限控制研究
date: 2020-04-03 16:23:27
tags: 
- vue
- 权限控制
categories:
- web前端
---
# VUE系统权限控制

## 参考资料
[基于Vue实现后台系统权限控制](https://refined-x.com/2017/08/29/%E5%9F%BA%E4%BA%8EVue%E5%AE%9E%E7%8E%B0%E5%90%8E%E5%8F%B0%E7%B3%BB%E7%BB%9F%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/)

[用addRoutes实现动态路由](https://refined-x.com/2017/09/01/%E7%94%A8addRoutes%E5%AE%9E%E7%8E%B0%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1/)

[Vue2.0用户权限控制解决方案](https://refined-x.com/2017/11/28/Vue2.0%E7%94%A8%E6%88%B7%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)

[git 项目地址](https://github.com/tower1229/Vue-Access-Control)

## 思路

目的：（请求拦截，视图控制，路由控制）

方法：

![Image text](https://raw.githubusercontent.com/chenleno/picture/master/design.png)

## 流程分析
![](https://raw.githubusercontent.com/chenleno/picture/master/%E6%B5%81%E7%A8%8B%E5%88%86%E6%9E%90.png)

## 请求拦截

在请求发起前集中拦截，这时可以直接根据请求方法和请求地址来校验权限

示例：
后端返回的用户权限
```javascript
[
                {
                    "id": "cba62955646c4a489e2ca1442c786270",
                    "name": "获取基础数据",
                    "summary": null,
                    "url": "/base",           //重要，对应api
                    "method": "GET"           //重要
                },
                {
                    "id": "daweqeqeqwrerettewrewrew",
                    "name": "获取top10数据",
                    "summary": null,
                    "url": "/goods/**",
                    "method": "GET"
                }...
            ]
```
本地维护的接口
```javascript
const getBaseData = {
    p: [`get,${preUrlPath}/base`],     // p 属性用于视图控制元素显隐
    r: params => {                     // r 属性用户实际请求，实际上是对axios请求的封装
        return instance.get(`${preUrlPath}/base`, {params})
    }
};
//获取top10
const getTopData = {
    p: [`get,${preUrlPath}/goods/**`],
    r: (id , params) => {
        return instance.get(`${preUrlPath}/goods/${id}`,{params})
    }
};
```
验证核心逻辑：

主要是将 后端提供的用户权限的method 和 url 处理成 
```javascript
"get,/base":true || "get,/goods/**":true 
```
格式的键值对，然后放进resourcePermission 对象中 （处理方式见 getPermission 方法）

```javascript
resourcePermission = {
  "get,/base":true , 
  "get,/goods/**":true
  }
```

在请求时将请求实例对象中的 r 对应的 url 转化成对应格式

然后与 resourcePermission 对象中的每一个键名作比较，不匹配则拦截（转化方式见 setInterceptor 方法）


## 视图控制

将权限跟请求同时维护，验证方法接收请求对象数组为参数，返回是否具有权限的布尔值
```javascript
v-if='' 方法 , 为true时显示，为fasle时隐藏
```
思路

```javascript
let permissions = {
  "get,/resources":true,
  "delete,/resources":true,
  "post,/resources":true,
  "put,/resources":true,
  ...
}

let has = function(permission){
  if(!permissions[permission]){
    return false;
  }
  return true;
}

<div v-if="has('get,/sources') && something">
    一个需要同时具备'get,/sources'权限和somthing为真值才显示的div
</div>

```
实现
```javascript
//请求对象格式
//获取账户列表
const getBaseData = {
    p: [`get,${preUrlPath}/base`],     // p 属性用于视图控制元素显隐
    r: params => {                     // r 属性用户实际请求，实际上是对axios请求的封装
        return instance.get(`${preUrlPath}/base`, {params})
    }
};
//调用格式
v-if="$_has([getBaseData])"

//$_has 方法
Vue.prototype.$_has = function (rArray) {

    let resources = [];
    let permission = true;
//提取权限数组
    if (Array.isArray(rArray)) {
        rArray.forEach(function (e) {
//对api表中的param参数进行处理，去掉前缀
            let newE = e.p[0].replace('/api/udcp-base', '')
            resources = resources.concat(newE);
        });
    } else {
         resources = resources.concat(rArray.p);
      }
      //校验权限
      resources.forEach(function (p) {
          if (!resourcePermission[p]) {
              return permission = false;
          }
      });
      return permission;
}
```
拓展———— v-has 指令，防止频繁检查变量

## RESTful 接口要求
验证的核心是将请求api按照一定的规则转化成规定格式，因此api在定义时应遵守一定的规范

接口定义时应注意安全性和幂等性
- 安全性 ：不会改变资源状态，可以理解为只读的；

- 幂等性 ：执行1次和执行N次，对资源状态改变的效果是等价的。

![安全性幂等性](https://raw.githubusercontent.com/chenleno/picture/master/%E5%AE%89%E5%85%A8%E5%B9%82%E7%AD%89.png)


参考文档：

[阮一峰 RESTful设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api)

[一些RESTful架构的理解](https://blog.csdn.net/chasel_s/article/details/78075762)

[关于RESTful的一些注意事项](https://blog.csdn.net/u013731455/article/details/56278168)

接口格式
```javascript
静态资源请求需要 "/" 在结尾封闭
如：/base/sc/cs/

对于带子资源的资源
1. 格式：/resources/:id
   示例：/resources/1
   url: /resources/**
   解释：一个名词后跟一个参数，参数通常表示名词的id
   
2. 格式：/store/:id/member
   示例：/store/1/member
   url：/store/*/member
   解释：两个名词之间夹带一个参数，参数通常表示第一个名词的id

```
接口处理方法
```javascript

例： //config.url = "/api/udcp-base/base"
    //config.url = "/api/udcp-base/goods/offline"
    //config.url = "/api/udcp-base/goods/1/cup"

  let perName = config.url.replace(config.baseURL, '').replace('/api/udcp-base', '').split('?')[0];

                    //preName = /base

                    //权限格式1 /path/${param}
                    let reg1 = perName.match(/^(\/[^\/]+\/)[^\/]+$/);
                    if (reg1) {
                        perName = reg1[1] + '**';
                        // preName = /goods/**
                    }
                    //权限格式2 /path/${param}/path
                    let reg2 = perName.match(/^\/[^\/]+\/([^\/]+)\/[^\/]+$/);
                    if (reg2) {
                        perName = perName.replace(reg2[1], '*');
                        // preName = /goods/*/cup
                    }
```
后端api要求示例 
```javascript
旧：
用户(平台使用者)登录
POST: /api/udcp-base/session/create             //create为动词，不符合规范

新：
POST: /api/udcp-base/session                    //对session资源进行post操作

旧：
修改密码
POST: /api/udcp-base/session/change             //change为动词，不符合规范

新：
PUT: /api/udcp-base/session                     //对session资源进行put操作

旧：
查询用户群中的用户
POST: /api/udcp-base/users/{pageNo}/{pageSize}  //查询应使用get,pageNo和pageSize为过滤条件，不是资源，应作为参数

新：
GET: /api/udcp-base/users?pageNo=&pageSize=

旧：
畅销商品列表
GET: /api/udcp-base/goods/sales/{deviceType}/{timeSlot}     //goods后存在无关修饰，timeSlot为过滤条件，不是资源，应作为参数
新：
GET: /api/udcp-base/goods/{deviceType}?timeSlot=

```

根据修改后的api,后端给出的用户权限数据示例
```javascript
{
                    "id": "cba62955646c4a489e2ca1442c786270",
                    "name": "登录",
                    "summary": null,
                    "url": "/session",          
                    "method": "POST"           
                },
                {
                    "id": "daweqeqeqwrerettewrewrew",
                    "name": "修改密码",
                    "summary": null,
                    "url": "/session",
                    "method": "PUT"
                },
                {
                    "id": "ytjyhshfgfgd",
                    "name": "查询用户群中的用户",
                    "summary": null,
                    "url": "/users",
                    "method": "GET"
                },
                {
                  "id": "ptorejfklvxnksldfk",
                    "name": "畅销商品列表",
                    "summary": null,
                    "url": "/goods/**",
                    "method": "GET"
                }
```

局限性：

无法解析 /path/path1/${param} 这样的格式，会解析为 /path/*/${param}

无法解析 /path/${param1}/path1/${param2}这样的格式，会解析为 /path/*/path1/${param2} 

解决方法：
1. 采用 ? 后接参数形式
2. 新正则




## 路由控制 
> 动态注册路由

事先在本地存一份完整的路由数据，然后根据用户权限对完整路由进行筛选
需先将后台返回的路由数据处理成
```javascript
let hashMenus = {
   "/route1":true,
   "/route1/route1-1":true,
   "/route1/route1-2":true,
   "/route2":true,
   ...
}
```


> 动态生成菜单


> 配置目录结构
```javascript
src/
  |-- api/                  //接口文件
  |     |-- index.js             //输出通用axios实例
  |     |-- account.js           //按业务模块组织的接口文件，所有接口都引用./index提供的axios实例
  |-- router/
  |     |-- fullpath.js         //完整路由数据，用于匹配用户的路由权限得到实际路由
  |     |-- index.js            //输出基础路由实例
```
> 数据格式约定
- 路由权限数据（前端提供，从后端请求）
```javascript
[
    {
      "id": "1",
      "name": "菜单1",
      "parent_id": null,
      "route": "route1"
    },
    {
      "id": "2",
      "name": "菜单1-1",
      "parent_id": "1",
      "route": "route2"
    }
  ]
```
- 资源权限数据（后端提供，格式固定）
```javascript
 [
    {
      "id": "2c9180895e172348015e1740805d000d",
      "name": "账号-获取",
      "url": "/accounts",
      "method": "GET"
    },
    {
      "id": "2c9180895e172348015e1740c30f000e",
      "name": "账号-删除",
      "url": "/account/**",
      "method": "DELETE"
    }
]
```




