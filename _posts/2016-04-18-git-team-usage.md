---
layout: post
title: GIT团队协作使用规范
tags: git
categories: git
---


# GIT团队协作使用规范

## Commit 规范
参考 [AngularJS](https://github.com/angular/angular/pulls)


### Commit Message 格式

```
<type>(<scope>): <subject>
<空行>
<body>
<空行>
<footer>
```

上面是一次Commit后Message格式规范，分成标题，内容详情，结尾三个部分，各有各的用处，没有多余项。

头部即首行，是可以直接在页面中预览的部分，入上面图中所示，一共有三个部分<type>，<scope>，<subject>，含义分别如下

**Type**

- feat：新功能（feature）

- fix：修补bug

- docs：文档（documentation）

- style： 不影响功能的代码变化（空格，格式化，丢失分号等）

- refactor：重构（即不是新增功能，也不是修改bug的代码变动）

- test：增加测试

- chore：构建过程或辅助工具的变动

**Scope**
用来说明本次Commit影响的范围，即简要说明修改会涉及的部分。这个本来是选填项，但从AngularJS实际项目中可以看出基本上也成了必填项了。

**Subject**
用来简要描述本次改动，概述就好了，因为后面还会在Body里给出具体信息。并且最好遵循下面三条:

以动词开头，使用第一人称现在时，比如change，而不是changed或changes
首字母不要大写
结尾不用句号(.)。

**Body**
body 里的内容是对上面subject里内容的展开，在此做更加详尽的描述，内容里应该包含修改动机和修改前后的对比。

**Footer**
footer里的主要放置不兼容变更和Issue关闭的信息，

**Revert**
此外如果需要撤销之前的Commit，那么本次Commit Message中必须以revert：开头，后面紧跟前面描述的Header部分，格式不变。并且，Body部分的格式也是固定的，必须要记录撤销前Commit的SHA值。

### 实践

**安装利器Commitiz**

为了让我们能把这些规范应用到实际使用中，我们要借助于Commitizen这个Node工具，它会在我们Commit的过程中更具规范的内容来引导我们如何一步一步实施规范。当然，规范这种东西就没有唯一的，各家有各家的不同，这一点当然也被该工具想到了，你也可以自定义一份自己的规范，以插件的形式让Commitizen来根据自家规范提醒你。

>npm install -g commitizen

**配置**
上一步我们在全局范围内安装了commitizen，之后我们就可以在Git仓库中配置我们的Commit规范了。打开项目执行如下命令:

>commitizen init cz-conventional-changelog --save --save-exact

上面的cz-conventional-changelog就是AngularJS的规范，其它的规范你可以自行到官网上找找看，不行就自己花时间拟定一份吧。此命令帮你完成了下载cz-conventional-changelog规范，配置package.json(添加依赖和配置应用规范)，想看具体改动打开package.json即可。


## GIT 分支命名规范：　

|分支|命名|说明|
|:-----  |:-------|:----- |
|主分支 | master | 主分支，所有提供给用户使用的正式版本，都在这个主分支上发布|
|开发分支 | dev | 开发分支，永远是功能最新最全的分支|
|功能分支 | feature-* | 新功能分支，某个功能点正在开发阶段|
|发布版本 | release-* | 发布定期要上线的功能|
|修复分支  | bug-* | 修复线上代码的 bug|


### 主分支  Master
首先，代码库应该有一个、且仅有一个主分支。所有提供给用户使用的正式版本，都在这个主分支上发布。

Git主分支的名字，默认叫做 Master 。它是自动建立的，版本库初始化以后，默认就是在主分支在进行开发。

### 开发分支 Dev

主分支只用来分布重大版本，日常开发应该在另一条分支上完成。我们把开发用的分支，叫做  Dev

这个分支可以用来生成代码的最新隔夜版本（nightly）。如果想正式对外发布，就在 Master 分支上，对 Dev 分支进行”合并”（merge）。

Git创建 Dev 分支的命令：

>git checkout -b dev  master

将 Dev 分支发布到 Master 分支的命令：

切换到 Master 分支

>git checkout master

对 Dev 分支进行合并

>git merge –no–ff dev

这里稍微解释一下，上一条命令的–no–ff参数是什么意思。默认情况下，Git执行”快进式合并”（fast-farward merge），会直接将 Master 分支指向 Dev 分支。

使用–no–ff参数后，会执行正常合并，在 Master 分支上生成一个新节点。为了保证版本演进的清晰，我们希望采用这种做法。


### 功能分支 Feature

功能分支的名字，可以采用feature-*的形式命名。

创建一个功能分支：

>git checkout -b feature-x dev

开发完成后，将功能分支合并到dev 分支：

>git checkout dev

>git merge –no-ff feature-x

删除feature分支：

>git branch -d feature-x

### 预发布分支 Release

第二种是预发布分支，它是指发布正式版本之前（即合并到 Master 分支之前），我们可能需要有一个预发布的版本进行测试。

预发布分支是从 Dev 分支上面分出来的，预发布结束以后，必须合并进 Dev 和 Master 分支。它的命名，可以采用release-*的形式。

创建一个预发布分支：

>git checkout -b release-1.2 dev

确认没有问题后，合并到master分支：

>git checkout master

>git merge –no-ff release-1.2

#### 对合并生成的新节点，做一个标签

>git tag -a 1.2

再合并到dev 分支：

>git checkout dev

>git merge –no-ff release-1.2

最后，删除预发布分支：

>git branch -d release-1.2


### 修补分支 Bug

最后一种是修补bug分支。软件正式发布以后，难免会出现bug。这时就需要创建一个分支，进行bug修补。

修补bug分支是从 Master 分支上面分出来的。修补结束以后，再合并进 Master 和 Dev 分支。它的命名，可以采用fixbug-*的形式。

创建一个修补bug分支：

>git checkout -b fixbug-0.1 master

修补结束后，合并到master分支：

>git checkout master

>git merge –no-ff fixbug-0.1

>git tag -a 0.1.1

再合并到dev 分支：

>git checkout dev

>git merge –no-ff fixbug-0.1

最后，删除”修补bug分支”：

>git branch -d fixbug-0.1


