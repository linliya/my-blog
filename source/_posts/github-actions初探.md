---
title: github actions初探
date: 2019-01-22 15:52:43
tags:
---
## 简单介绍

`Github Actions`是`Github`推出的一个新的功能，可以为我们的项目自动化地构建工作流，例如代码检查，自动化打包，测试，发布版本等等。入口在项目`pull request`的旁边。
![action入口](/images/actions-header)
私有仓库现在应该很多同学都可以看到了，公有的因为现在还是beta版，名额还在陆续开放，需要主动去申请排队，有需要的同学可以[去申请一下](https://github.com/features/actions)，点击主页的`sign up for beta`即可。

> 由于每个`action`都是在一个独立的`docker`容器里面运行的，所以需要我们有一点`docker`方面的知识储备，关于`docker`本文就不多加介绍了...

## 提出问题

### 1. workflow和action是什么？
字面上理解，`workflow`就是一个工作流，而`action`就是这个工作流中的一个个步骤，`github`规定每次最多只能并发执行两个工作流，每个工作流中的`actions`会按照我们需要的顺序运行，`actions`怎样执行可以根据我们的需要进行定制。

### 2. 怎么创建一个workflow？

#### 方法一：通过`github`提供的可视化界面进行创建

- 点击`actions`入口，一开始我们可以看到这个界面：

![创建workflow](/images/actions-create)

- 点击上图的`create a new workflow`按钮，进入自定义`workflow`页面：

![](/images/actions-workflow)
在这个界面，我们从`workflow`拉下来一条线，下面那个就是`action`，`GitHub官方`提供了几个`action`可以让我们直接用，点击右侧的几个`action`对应的`use`按钮即可。

例如我们想使用`npm`，点击`npm`对应的`use`，出现以下界面，假如我们每次`push`分支之后想执行`npm install`，在`runs`对应的输入框输入`npm install`，那么，执行`npm install`这个`action`就写好了。

#### 方法二: 编写main.workflow文件

在项目根目录新建`.github`文件夹，在`.github`文件夹新建`main.workflow`文件，编写完成后推送到`master`分支即可。

这是一个简单的`main.workflow`文件:
```
workflow "Build, Test, and Publish" {
  on = "push"
  resolves = ["Publish"]
}

action "Build" {
  uses = "actions/npm@master"
  args = "install"，
  env = {
    LOG_FILE = "log.txt"
  }
}

action "Test" {
  needs = "Build"
  uses = "actions/npm@master"
  args = "test"
}

action "Publish" {
  needs = "Test"
  uses = "actions/npm@master"
  args = "publish --access public"
  secrets = ["NPM_AUTH_TOKEN"]
}
```
其实，我们大概可以看出这段代码想要表达的意思，这里我简单介绍一下几个属性：

`workflow`的属性：
- on： 定义什么情况下会触发这个workflow，例如，push, pull_request等。
- resolves： 定义要调用的`actions`，可以是字符串或者一个字符串数组，若是只有一个字符串，表示最后调用的`action`，若是一个字符串数组，则表示完成这个`workflow`需要执行完这几个`actions`。

`action`的属性：
- needs：定义执行本`action`前需要成功执行的`action`，可以是字符串或者一个字符串数组，如果`needs`的`action`不止一个，那么这些`actions`会并行执行。

- uses：定义执行本`action`需要运行的`docker`镜像，如`uses = "node:10"`，如果不是使用`docker hub`提供的镜像，而是选择自己编写`Dockerfile`的话（后面会具体说明），这里的路径则为本地`Dockerfile`的路径。

- runs：指定在`docker`镜像中要执行的命令，若指定，则会覆盖`Dockerfile`里面的`ENTRYPOINT`，若不指定，则默认执行`Dockerfile`里面`ENTRYPOINT`的命令。

> 假如我们想要在action里面执行npm install，那么只要指定: `runs: "npm install"`即可。

- args：指定要传递给`action`的参数，可以是一个字符串或者一个字符串数组，若指定，则会覆盖`Dockerfile`里面的`CMD`。

> 假如我们想要在action里面执行npm run lint，那么只要指定: `args: "run lint"`即可。

- env：设置action运行时需要的环境变量，一般是自己写`Dockerfile`时会需要用到，具体说明请看[官方文档](https://developer.github.com/actions/creating-workflows/workflow-configuration-options/)。

### 3. 怎么创建一个action？

#### 方法一. 使用官方提供的几个action（比较简单）
在`github action`界面点击右侧的`actions`列表，通过点击use按钮使用该`action`，根据需求填写`runs`, `args`, `env`等参数即可。

#### 方法二. 自己写dockerfile(自定义程度高)
可以参考[Github actions for npm](https://github.com/actions/npm/tree/4633da3702a5366129dca9d8cc3191476fc3433c/)等官方提供的`action`源码进行编写，主要是编写以下两个文件：
```
 Dockerfile
 entrypoint.sh
```

## 踩坑日记

### 1. entrypoint.sh需要执行权限
通过自己写`dockerfile`的方式来创建`action`的话，需要执行以下代码，不然会出错。
```
chmod +x ./actions/entrypoint.sh（替换成自己项目entrypoint.sh的路径）
```

### 2. npm install yarn -g ?
本来是想通过安装`yarn`，然后使用`yarn`来安装依赖的，但是因为容器的独立性，每个`action`都是独立的，不存在全局环境，所以无法实现，而`github`官方提供了一个公共的存储空间，`npm install`下载完成的文件就放在公共的存储空间，因此可以提供给后面的`action`使用，这也意味着如果需要在`action`之间传递信息，暂时也只能利用公共的存储空间，使用文件读写的方式来传递。

### 3. npm run test需要使用浏览器来跑单元测试，启动失败
浏览器无法直接在docker容器里面启动，使用`xvfb-run npm test` 成功解决。

## 总结

由于`github action`还算比较新的功能，网上的教程不是很多，以上大多是参考官方文档，加上自己摸索尝试得出的结论，如果有什么错漏之处，欢迎指出~
