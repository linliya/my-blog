---
title: js中的神奇代码
date: 2018-08-27 14:47:37
tags:
---
js中，有一些解决方案很值得斟酌，有时候简单的几行代码，能解决很复杂的问题，我在这里做一下记录，就当作是一个工具包吧

#### 请用代码实现test(2)(3)(4)(5) = 120这个函数
```
function test(x){
  var sum = x;
  var mod = function(y) {
    sum = sum * y;
    return mod;
  };
  mod.toString =function(){
    return sum;
  };
  return mod;
}
```
#### 格式化日期，包含 `年-月-日`
```
 # @param {Date|String} date 时间戳
 # @returns {String} 按照`-`分割开的日期显示字符串

const formatDate = date => {
  let dateFormat = new Date(date)
  if (!dateFormat) return ''
  const year = dateFormat.getFullYear()
  const month = dateFormat.getMonth() + 1
  const day = dateFormat.getDate()

  return [year, month, day].map(formatNumber).join('-')
}

const formatNumber = n => {
  n = n.toString()
  return n[1] ? n : '0' + n
}
```

#### 格式化时间，包含 `年-月-日 时-分-秒`
```
 # @param {Date|String} date 时间戳
 # @returns {String} 按照`-`分割开的时间显示字符串
 
const formatTime = date => {
  if (!date) return ''
  let dateFormat = new Date(date)
  if (!dateFormat) return ''
  const year = dateFormat.getFullYear()
  const month = dateFormat.getMonth() + 1
  const day = dateFormat.getDate()
  const hour = dateFormat.getHours()
  const minute = dateFormat.getMinutes()
  const second = dateFormat.getSeconds()

  return [year, month, day].map(formatNumber).join('-') + ' ' + [hour, minute, second].map(formatNumber).join(':')
}
```

#### 格式化文件大小显示

```
 # @param {Number} size 大小
 # @returns {String} 格式化后的容量大小

const formatSize = size => {
  if (typeof size !== 'number' || size < 0) {
    return '0 B'
  }
  if (size < 1024) return `${size} B`
  size = size % 1024 === 0 ? size / 1024 : (size / 1024).toFixed(2)
  if (size < 1024) return `${size} KB`
  size = size % 1024 === 0 ? size / 1024 : (size / 1024).toFixed(2)
  if (size < 1024) return `${size} MB`
  size = size % 1024 === 0 ? size / 1024 : (size / 1024).toFixed(2)
  if (size < 1024) return `${size} GB`
  size = size % 1024 === 0 ? size / 1024 : (size / 1024).toFixed(2)
  if (size < 1024) return `${size} TB`
}
```
