---
title: nuxt入门
date: 2018-04-26 16:18:23
tags: 前端
---
有关于nuxt的详细介绍，[官方文档](https://zh.nuxtjs.org/guide)已经写得很好了，我这里也不再赘述，我还是写一下比较难理解的配置项吧:

#### asyncData

在这个方法被调用的时候，第一个参数被设定为当前页面的上下文对象[context](https://zh.nuxtjs.org/api/)(context可以理解为nuxt提供的全局对象)，可以利用 asyncData方法来获取数据并返回给当前组件(返回值即this.data)

> 由于asyncData方法是在组件 **初始化** 前被调用的，所以在方法内是没有办法通过 this 来引用组件的实例对象。

例如：
```
asyncData ({params}) {
  return { params: params }
}
```
####  配置里的build相关配置解析

##### analyze
如果将`analyze`设置为`true`的话，需要在`package.json`里面的script加上`build: nuxt build -analyze`，那么运行`npm run build`的时候在`http://localhost:8888`可以查看整个项目打包后的文件大小等信息，方便我们进行性能优化

##### extend 
`nuxt`集成了`webpack`, 它允许我们对`webpack`的配置进行拓展，通过push方法将配置增加到`webpack`的配置项中，如：
```
extend (config, { isDev, isClient }) {
  if (isDev && isClient) {
    config.module.rules.push({
      enforce: 'pre',
      test: /\.(js|vue)$/,
      loader: 'eslint-loader',
      exclude: /(node_modules)/
    })
  }
},
```
##### postcss
`nuxt`默认为我们添加了`autoprefixer`，即项目打包完后为一些属性自动添加浏览器前缀
```
[
  require('autoprefixer')({
    browsers: ['last 3 versions']
  })
]
```
##### vendor
这个感觉挺有用的，在vendor数组里面添加的模块会被打包到vendor bundle里，之后在组件内对该模块的引用将不会被打包到组件对应的文件内了，这样说感觉有点抽象，举个例子说明一下：
> 我们在`build.vendor`里面添加了`axios`模块:
 ```
module.exports = {
  build: {
    vendor: ['axios']
  }
}
```
> 然后在a.vue里面和b.vue里面都进行了以下操作：
```
import axios from 'axios'
```
> 那么a.vue和b.vue打包后的文件大小将比不加上build.vendor小，如果很多文件都需要引入axios的话，那么对于文件大小的优化效果是比较明显的


暂时就写这几点...其它的可以查看nuxt的文档，可能会继续更新...



