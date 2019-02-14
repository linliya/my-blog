---
title: github actions初探
date: 2019-01-22 15:52:43
tags:
---
## 简单介绍

`Github Actions`是`Github`推出的一个新的功能，可以为我们的项目自动化地构建工作流，例如代码检查，自动化打包，测试，发布版本等等。入口在项目`pull request`的旁边。
![action入口](https://user-gold-cdn.xitu.io/2019/1/22/1687441b5ac73115?w=1830&h=128&f=png&s=27493)
私有仓库现在应该很多同学都可以看到了，公有的因为现在还是beta版，名额还在陆续开放，需要主动去申请排队，有需要的同学可以[去申请一下](https://github.com/features/actions)，点击主页的`sign up for beta`即可。

> 由于每个`action`都是在一个独立的`docker`容器里面运行的，所以需要我们有一点`docker`方面的知识储备，关于`docker`本文就不多加介绍了...

## 提出问题

### 1. workflow和action是什么？
字面上理解，`workflow`就是一个工作流，而`action`就是这个工作流中的一个个步骤，`github`规定每次最多只能并发执行两个工作流，每个工作流中的`actions`会按照我们需要的顺序运行，`actions`怎样执行可以根据我们的需要进行定制。

### 2. 怎么创建一个workflow？

#### 方法一：通过`github`提供的可视化界面进行创建

- 点击`actions`入口，一开始我们可以看到这个界面：

![创建workflow](https://user-gold-cdn.xitu.io/2019/1/22/16874755e1a043bb?w=3584&h=1578&f=png&s=442468)

- 点击上图的`create a new workflow`按钮，进入自定义`workflow`页面：

![](https://user-gold-cdn.xitu.io/2019/1/22/168747c6abb7dd2f?w=3584&h=1570&f=png&s=589968)
在这个界面，我们从`workflow`拉下来一条线，下面那个就是`action`，`GitHub官方`提供了几个`action`可以让我们直接用，点击右侧的几个`action`对应的`use`按钮即可。

例如我们想使用`npm`，点击`npm`对应的`use`，出现以下界面，假如我们每次`push`分支之后想执行`npm install`，在`run`对应的输入框输入`install`，那么，执行`npm install`这个`action`就写好了。

![](https://user-gold-cdn.xitu.io/2019/1/22/16874890f71c1f5a?w=3576&h=1668&f=png&s=559712)

#### 方法二: 编写main.workflow文件

在项目根目录新建`.github`文件夹，在`.github`文件夹新建`main.workflow`文件，编写完成后推送到`master`分支即可

这是`GitHub Action for npm`的`main.workflow`文件:
```
workflow "Build, Test, and Publish" {
  on = "push"
  resolves = ["Publish"]
}

action "Build" {
  uses = "actions/npm@master"
  args = "install"
}

action "Test" {
  needs = "Build"
  uses = "actions/npm@master"
  args = "test"
}
```
其实，我们大概可以看出这段代码想要表达的意思，这里我简单介绍一下：

`workflow`的属性：
- on： 定义什么情况下会触发这个workflow，例如，push, pull_request等
- resolves： 定义要调用的`actions`，可以是字符串或者一个字符串数组，若是只有一个字符串，表示最后调用的`action`，若是一个字符串数组，则表示完成这个`workflow`需要执行完这几个`actions`

`action`的属性：
- needs：定义执行本`action`前需要成功执行的`action`，可以是字符串或者一个字符串数组，如果`needs`的`action`不止一个，那么这些`actions`会并行执行

- uses：定义执行本`action`需要运行的`docker`镜像，如`uses = "node:10"`，如果不是使用`docker hub`提供的镜像，而是选择自己编写`Dockerfile`的话（后面会具体说明），这里的路径则为本地`Dockerfile`的路径

- runs：指定在`docker`镜像中要执行的命令，若指定，则会覆盖`Dockerfile`里面的`ENTRYPOINT`，若不指定，则默认执行`Dockerfile`里面`ENTRYPOINT`的命令

例如：还是用上面举的例子，我们想要在action里面执行npm install，那么只要指定: `runs: "npm install"`即可

- args
- secrets
- env

具体编写教程见[官方文档](https://developer.github.com/actions/creating-workflows/workflow-configuration-options/)

### 3. 怎么创建一个action？

#### 方法一. 使用官方提供的几个action（比较简单）
在`github action`界面点击右侧的`actions`列表，通过点击use按钮即可使用该`action`

#### 方法二. 自己写dockerfile(自定义程度高)
   - Dockerfile
   - entrypoint.sh

遇到的几个坑：
npm install yarn -g ?


npm run test 需要使用浏览器来跑单元测试，多次启动失败

entrypoint.sh需要执行权限

chmod +x ./actions/entrypoint.sh

docker常用的命令：
```
docker build -t docker-name-tag
docker run --rm -it -v $(pwd)/:dw docker-name-tag bash
docker ps -a
docker rm
```

使用xvfb-run npm test 成功解决