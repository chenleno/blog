---
title: nvm的安装及使用
date: 2023-07-11 15:09:06
tags:
---
# nvm的安装及使用

## 前言
重新拾起吃灰的blog，发现执行`hexo d`时给了报错
> TypeError [ERR_INVALID_ARG_TYPE]: The "mode" argument must be integer. Received an instance of Object

本人的hexo版本
```
hexo: 4.2.0
hexo-cli: 3.1.0
os: Darwin 17.7.0 darwin x64
node: 16.20.0
```

查询资料了解到是由于node版本过高导致，需要降级为低版本（13.x.x）才能使用。但是卸载重装这事也太low了，而且保不齐有其他的项目又需要高版本node。想起nvm这个大名鼎鼎的node管理工具，借此机会正好体验一下。

## 安装

查阅了多方资料，安装过程不尽相同。下面就只以本人的安装过程介绍。（MacOS系统，没有卸载原有的node）
首先附上官方文档地址[nvm](https://github.com/nvm-sh/nvm/blob/master/README.md)

MacOS下可以通过`curl`方便地安装nvm,只需要开启终端，输入以下命令即可
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

届时会出现一个进度说明，耐心等待安装完成即可

[![pCWNkND.png](https://s1.ax1x.com/2023/07/11/pCWNkND.png)](https://imgse.com/i/pCWNkND)

PS: 如果进度说明中各项都为0，只有`Time Spent`在增加，可能是终端请求资源地址不通，需要配置终端代理（科学上网）

安装完成后，会出现如下所示的说明

[![pCWNaD0.png](https://s1.ax1x.com/2023/07/11/pCWNaD0.png)](https://imgse.com/i/pCWNaD0)

这时候如果你在控制台使用nvm，大概率是不行的，会提示`This is not the package you are looking for: please go to http://nvm.sh`

这是因为nvm命令还未被配置到环境变量中，这时候查看安装完成后的说明，作者已经告诉了我们处理方法：

```
// 你可以选择关闭终端后重新打开来使用nvm，或者使用以下命令来立即应用
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

这里我们试试命令的方式，终端中输入

```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

之后输入`nvm --version`，可以发现nvm命令已经可以使用

## 使用

有了nvm我们自然要感受一下他的威力，这里我选择安装当前hexo所需的node版本13.14.0

```
nvm i 13.14.0
```

[![pCWU8Z6.png](https://s1.ax1x.com/2023/07/11/pCWU8Z6.png)](https://imgse.com/i/pCWU8Z6)

可以看到顷刻间nvm就帮我们安装好了所需node版本，并且自动切换到了该版本，接下来试试部署命令`hexo d`，OK，完成部署！

切换到其他node版本也很简单，首先我们可以输入`nvm list`查看所有当前已安装的node版本

[![pCWaAld.png](https://s1.ax1x.com/2023/07/11/pCWaAld.png)](https://imgse.com/i/pCWaAld)

切换到其他的node版本(例：v18.16.1)，只需要输入 `nvm use 18.16.1`即可

