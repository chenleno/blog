---
title: 代码pr审核流程说明
date: 2020-05-12 22:21:09
updated: 2020-05-12 23:32:57
categories: 
- 工作流
tags:
- git
---
# 代码pr审核流程说明
## 简介
pr (pull request)或称 mr（merge request）是一种git代码合并工作流程。意即将分支代码合并过程放在远端，藉由提交者发起pr的形式，由协作者共同进行代码评审，评审通过后由合并者在远端进行分支合并操作的流程。
## 优缺点
优点：
- 代码变动和更新公开透明，规避了本地合并提交的黑盒操作
- 代码合并质量有保障，每个协作者都可以查看提交内容并给出建议
- 提高开发者的代码优化意识，对每次提交保持敬畏之心

缺点：
- 涉及多人评审时会略微占用开发时间，影响工作节奏
## 提交，审核及合并规范
- pr审核至少要有两人参与（发起者和一名协作者）
- 原则上pr发起者**不能**`Merge`自己的pr，特殊情况下除外，但也需要通知其他协作者并得到同意
- 所有pr提交均需要通知到所有相关协作者并视情况同步本地代码

Tips: git 暂存操作 git stash
## 操作流程
以gitlab为例，进行一次完整的模拟pr提交过程
### clone远端仓库
> $ git clone git@origin/pr-example.git

假定项目主分支为 **master** 分支
### 建立本地个人开发分支
pr提交要求每个开发者都有基于主分支的个人开发分支，开发者在个人开发分支开发功能需求
> $ git checkout -b private_dev(开发分支名自取)
### 在个人分支开发功能需求
some coding...
> $ git add .

> $ git commit -m 'feat: something new'
### 推送本地个人开发分支至远端
> $ git push

如果是第一次提交分支到远端git会有提示
[![YUYUud.md.jpg](https://s1.ax1x.com/2020/05/12/YUYUud.md.jpg)](https://imgchr.com/i/YUYUud)
照做即可
> $ git push --set-upstream origin private_dev
### 查看远端仓库个人分支
此时可以在远程仓库分支列表中查看到自己刚刚推送的个人分支
[![YUtjyQ.md.jpg](https://s1.ax1x.com/2020/05/12/YUtjyQ.md.jpg)](https://imgchr.com/i/YUtjyQ)
### 发起pr
1. 点击图示按钮
[![YUNkSU.md.png](https://s1.ax1x.com/2020/05/12/YUNkSU.md.png)](https://imgchr.com/i/YUNkSU)
2. 选择 Source branch (即提交分支 private_dev) 和 Target branch (即目标分支 master),点击下方按钮
[![YUNGOH.md.png](https://s1.ax1x.com/2020/05/12/YUNGOH.md.png)](https://imgchr.com/i/YUNGOH)
3. 检查source和target 分支是否正确，可以在changes中查看当前提交的git diff，作最后走查。确认无误点击按钮提交。
[![YUUegS.md.png](https://s1.ax1x.com/2020/05/12/YUUegS.md.png)](https://imgchr.com/i/YUUegS)
### 查看pr
提交后的pr会出现在`Merge Requests`页面中，届时所有项目协作者均可点击该pr查看pr内容
[![YUURDH.md.png](https://s1.ax1x.com/2020/05/12/YUURDH.md.png)](https://imgchr.com/i/YUURDH)
### pr审核
pr查看者可以在`changes`页签中查看完整提交内容和变动项。如果对某块代码持有建议可以通过点击对应行数前的评论图标，在输入框中进行评论。
[![YUaSP0.md.png](https://s1.ax1x.com/2020/05/12/YUaSP0.md.png)](https://imgchr.com/i/YUaSP0)
评论后的内容会显示在对应代码行数下方供pr发起者参考修改。如无异议则可点击`Merge`按钮合并pr
[![YUanG6.md.png](https://s1.ax1x.com/2020/05/12/YUanG6.md.png)](https://imgchr.com/i/YUanG6)
pr合并后可以看到master分支已经应用了该change
[![YUaIeJ.md.png](https://s1.ax1x.com/2020/05/12/YUaIeJ.md.png)](https://imgchr.com/i/YUaIeJ)
### 协作者分支同步
假设有A,B两个协作者，此时远端仓库的master分支已经包含了A的提交代码，则B需要将他的个人开发分支同远端同步
B:
> $ git stash (暂存当前未add的代码)（视情况使用）

> $ git checkout master

> $ git pull（拉取包含最新change的master分支）

> $ git checkout b_dev(b的个人开发分支)

> $ git rebase master（变基操作）

> $ git stash pop (释放之前暂存的代码)（视情况使用）

通过将个人分支变基到master上的形式来进行代码merge
Tips:
1. 如果两人修改了同一文件，则此时会发生代码冲突，B需要在本地解决冲突
2. 参考资料：[git rebase和git merge](https://www.jianshu.com/p/c17472d704a0)
git merge的commit流：
![git merge](https://upload-images.jianshu.io/upload_images/305877-c4ddfcf679821e2f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
git rebase 的 commit 流
![git rebase](https://upload-images.jianshu.io/upload_images/305877-467ba180733adca1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
