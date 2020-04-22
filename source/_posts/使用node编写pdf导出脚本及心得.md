---
title: 使用node编写pdf导出脚本及心得
date: 2020-04-03 16:45:33
tags:
- node
categories:
- node
---
# 使用Node编写pdf导出脚本项目心得
## 背景
这个项目是我入职后负责的第一个项目，需求是需要对学校孩子的测评报告进行批量导出，当时的我对于node编程还处于小白的状态，通过这个项目让我对于非前端的内容有了更深的认识，拓宽了视野
## 项目介绍
使用node编写命令行脚本，通过输入语句获取基本信息，核心是通过无头浏览器puppeteer自动访问网页，导出为pdf，重命名并保存到本地
## 核心模块
- [readline](http://nodejs.cn/api/readline.html)
- [puppeteer](https://github.com/GoogleChrome/puppeteer)
## 流程分析
```javascript
主入口
||
readline 模块，控制台获取基本信息（环境，token，学生id，模板id等）
||
puppeteer 启动无头浏览器，通过基本信息访问 对应页面
||
pdf 模块，pdf模块负责对页面进行打印操作，生成pdf文件，并存放至pdf文件夹中
||
结束
```
此流程通过对业务步骤进行分层，将其区分为独立的功能模块，将来如有需求变动（如截图等），可以通过更换功能模块的方式进行复用
## 项目结构
```javascript
|-- config
    |-- env.js // 环境判断
|-- module
    |-- pdf.js  // pdf 生成模块
    |-- puppeteer.js // 无头浏览器模块
    |-- readline.js // 命令行交互工具
|-- utils
    |-- fsUtil.js // 目录生成工具
|-- index.js // 主入口
```
## 代码实现
### 入口index
主入口只作为脚本执行的入口文件，无多余业务逻辑
```javascript
const openRL = require('./utils/readline')
if(require.main === module) {
    openRL()
}
```
### readline
readline 模块通过命令行交互为后续逻辑提供数据，需要在此处定义数据格式和相关交互提示语
```javascript
const readline = require('readline')
const env = require('../config/env') // 环境判断处理
const openBrowser = require('./puppeteer')

let userConfig = {
    token: '', // 用户token
    env: 'development', // 浏览器访问环境，正式或测试
    reportId: '', // 所需生成报告的类型
    childId: 'all' // 学生id, 不输入默认为all，否则为id数组
}
```
启动readline
```javascript
const openRL = async function () {
    const rl = readline.createInterface({ // 创建readline实例
        input: process.stdin, // 监听的可读流
        output: process.stdout, // 写入readline的可写流
        prompt: '敲回车开始>'
    })
    // 调用
    rl.prompt() // 控制台显示： '敲回车开始>'

    rl.on('line' , async line => { // line事件，用户输入后按下回车时触发该事件
        userConfig.env = await getArgs('请选择环境 (development/production),不输入默认为 development > ') || 'development'
        console.log('----')
        userConfig.token = await getArgs('请输入token > ')
        console.log('----')
        userConfig.reportId = await getArgs('请输入报告ID > ')
        console.log('----')
        userConfig.childId = await getArgs('请输入childId,使用“,”隔开，不输入默认为all > ')  || 'all'
        console.log('----')
        rl.close() // 输入完成，rl实例结束
    })

    // getArgs 方法通过Promise将用户输入的值作为rl.question的返回值
    const getArgs = question => new Promise(resolve => {
        rl.question(question, resolve)
    })

    rl.on('close', async () => {
        try{
            const env_config = await env(userConfig) // env函数加工用户输入的数据
            console.log('即将开始打印任务')
            await openBrowser(env_config)
        }catch (err){
            throw `处理 envConfig 出错 ${err}`
        }
        process.exit(0)
    })
}
```
env 函数将用户输入的值结合对应环境生成实际数据传递给puppeteer
```javascript
const envConfig = function (USER_CONFIG) {
    return new Promise( resolve => {
        const isStable = USER_CONFIG.env === 'production'
        let envConfig = USER_CONFIG
        envConfig.childId = USER_CONFIG.childId === 'all' ? 'all' : USER_CONFIG.childId.split(',').concat() // 处理childId为数组
        envConfig.host = isStable ? '正式域名' : '测试域名'
        envConfig.API_URL = isStable ? '正式apiUrl' : '测试apiUrl'
        resolve(envConfig)
    })
}
```
### puppeteer 
puppeteer 模块，启用浏览器，进行页面相关设置
node的puppeteer模块会安装一个chromium浏览器，通过该浏览器进行相关访问操作
```javascript
const puppeteer = require('puppeteer')
const pdfService = require('./pdf')

const openBrowser = async(userConfig) => {
    const browser = await puppeteer.launch({headless: true})   //用指定选项启动一个Chromium浏览器实例, 无头模式为可选项
    const page = await browser.newPage()                       //创建一个页面.
    // 设置浏览器视窗为设备视窗大小
    const dimensions = await page.evaluate(() => {              // Get the "viewport" of the page, as reported by the page.
        return {
            width: 650,
            height: document.documentElement.clientHeight,
            deviceScaleFactor: window.devicePixelRatio
        }
    })
    await page.setViewport(dimensions)
    // 执行pdf打印任务
    await pdfService(page , userConfig)
    await browser.close()                                      //关闭已打开的页面，browser不能再使用。
    console.log(`browser 已关闭`)
}
```
### pdf模块
pdf模块为核心业务模块，负责将puppeteer访问的页面进行打印操作，并最终导出pdf文件

脚本执行思路是：
1. 先检查当前脚本目录中是否存在存放生成pdf的文件夹，无则生成，有则取路径
2. 解析userConfig中的childId: 如果childId === 'all',查询报告接口，获取该类型下所有学生的id数组，否则直接取数组
3. 递归childIds数组，根据id访问对应页面并打印

```javascript
// pdf 模块对外暴露的 service，接收page和userConfig, 处理后传递给业务函数
const pdfService = async function(page , userConfig) {
    const user_ids = userConfig.childId === 'all' ? (await getUserData(userConfig)).user_ids : userConfig.childId
    const pdfDirPath = path.join(path.resolve(__dirname , '..'),'./pdf')
    try{
        await dirExists(pdfDirPath) // 创建pdf文件夹
    }catch(err) {
        throw `创建文件夹失败 ${err}`
    }
    await recurPdf(page , user_ids , userConfig) // 递归生成pdf文件
}
```

```javascript
// ./utils/fsUtil.js
// 文件夹判断
// 在 pdfPath = `./pdf/${childName}_${childId}.pdf`下生成对应的文件，需要借助 fs 模块和 path 模块

const fs = require('fs')
const path = require('path')
function getStat(path) {
    return new Promise((resolve, reject) => {
        fs.stat(path, (err, stats) => {
            if (err) {
                resolve(false)
            } else {
                resolve(stats)
            }
        })
    })
}
function mkdir(dir){
    return new Promise((resolve, reject) => {
        fs.mkdir(dir, err => {
            if(err){
                resolve(false)
            }else{
                resolve(true)
            }
        })
    })
}
async function dirExists(dir) {
    let isExists = await getStat(dir)
    //如果该路径且不是文件，返回true
    if (isExists && isExists.isDirectory()) {
        return true
    } else if (isExists) {     //如果该路径存在但是文件，返回false
        return false
    }
    //如果该路径不存在
    let tempDir = path.parse(dir).dir      //拿到上级路径
    //递归判断，如果上级目录也不存在，则会代码会在此处继续循环执行，直到目录存在
    let status = await dirExists(tempDir)
    let mkdirStatus
    if (status) {
        mkdirStatus = await mkdir(dir)
    }
    return mkdirStatus
}
```

```javascript
// 递归生成
const recurPdf = async function (page , userId_arr , userConfig) {
    const childID = userId_arr.shift() // 每次递归取头id
    try {
        await generatePdf(page , childID , userConfig)
    }catch(err){
        throw `pdf生成过程出错，当前id=${childID}, ${err}`
    }
    if(userId_arr.length !== 0){
        await recurPdf(page , userId_arr , userConfig)
    }else {
        console.log('userId_arr 已遍历完成, pdf 已生成完毕')
    }
}
```

```javascript
// 生成pdf文件
const generatePdf = async function(page , childId , userConfig){
    let hostUrl = `${userConfig.host}?reportId=${userConfig.reportId}&childId=${childId}&token=${userConfig.token}` // 根据userConfig的数据访问对应的页面
    try{
        console.log(`正在前往childId=${childId}`)
        await page.goto(hostUrl,{timeout: 50000})                  //到指定页面的网址.
    }catch(err){
        throw `浏览器访问childId=${childId}出错 ${err}`
    }
    await page.waitFor(15000)                                       //延时15000ms ，等待数据加载完成
    const domHandle = await page.$('.user span');
    const childName = await page.evaluate(body => body.innerHTML, domHandle)          // 根据页面中指定dom元素的值，获取孩子名字
    let pdfPath = `./pdf/${childName}_${childId}.pdf`              // pdf存放路径,pdf命名规则为: 名字_id.pdf
    await page.pdf({
        path: pdfPath,
        format: 'A4' ,
        printBackground:true,
        margin: {
            top: '18mm',
            right: '20mm',
            bottom: '18mm',
            left: '20mm'
        }
    })
}
```
## 经验总结
- 关于Node
    + 以前看Node，知道node是个运行环境，npm依赖node,构建工具依赖node，没有node就没有现在的大前端，没了...
    + 现在看Node,更多了node作为编程语言的一面，操作文件、硬件，从而带来无限的可能，感觉自己的舞台跳出了浏览器
- 关于项目结构
    + 现在的项目结构相对于初版进行了解耦，根据职责不同划分为了不同的功能模块，考虑到了后期更换功能模块的可能，对于提升架构思维是一次非常好的尝试
