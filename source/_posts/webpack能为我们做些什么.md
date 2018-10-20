---
title: webpack能为我们做些什么
date: 2018-09-17 10:39:50
tags:
---

### 为我们管理文件的加载顺序
有些文件需要依赖上一个文件是否加载完成，当项目比较庞大时，这个工作就会变得比较繁琐，webpack能为我们处理好文件的加载顺序，减少我们代码的出错率，提高工作效率
### 能进行代码压缩
- 简单使用
可以在`package.json`的`script`中添加-p参数
- `Plugins`模块可以通过`webpack`内置的模块进行代码压缩等工作

先要将`webpack` `require`进来，然后写上
```
new webpack.optimize.UglifyJsPlugin({
    // option code
})
```
webpack就会帮我们把bundle.js进行压缩了
### Module模块可以对文件进行处理
将`css,js`等文件整合进`bundle.js`中，`rules`的解析顺序是从下往上的，例如要引入css文件需要用到`css-loader`, 将css文件插入到`bundle.js中`需要用到`style-loader`,因此应该在`module->rules->use`数组中先写`style-loader`再写`css-loader`
