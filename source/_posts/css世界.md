---
title: css世界
date: 2018-05-16 16:32:21
tags:
---
从事前端工作一年了，最近越来越觉得自己的css学得不好，虽然基本能实现项目中需要实现的效果，但是还是觉得自己了解得并不深入，最近看了张鑫旭老师写的css世界，真心觉得css真是太博大精深了，张老师讲的东西很多都是我们平时经常遇到却很少时候会知道为什么会这样的一些情况，我也通过这一篇文章来记录一下自己的学习过程，有兴趣的同学还是得买本书看看......
### css中的流

- css世界中的流其实跟现实世界中的水流是类似的，例如:

   1.  水流会自动铺满容器，而div也会自动铺满容器
   2. 在水流中放入木头，木头依次排列，而css世界中的图片文字也会依次排列，空间不足时换行
- 流是怎样影响整个css世界的呢？
   1. 让html的默认表现符合“流”
   2. 通过流的破坏实现特殊布局
   3. 通过改变流向实现不同的布局

### 块级元素
`display: block; display: table; display: list-item;` 的元素都是块级元素，都符合块级元素的显示特征
- 为什么`display: list-item`的元素会出现项目符号？
> 因为生成了一个附加盒子，学名为“标记盒子”，专门用来存放圆点，数字这些项目符号，IE浏览器下伪元素不支持list-item或许是无法创建这个“标记盒子”导致的

- `display: inline-block`是由什么盒子构成的？
> 又生成了一个盒子，即每个元素都有两个盒子，外在盒子和容器盒子，外在盒子负责元素是否是一行显示，容器盒子负责宽高，内容呈现等，那么，`display:inline-block`则是由外在的inline和内在的block组成，因此可以在一行显示又可以设置宽高

- 你真的了解width: auto吗？

    1.元素默认的width值为auto，这就意味着元素会充分利用可用空间，但是，当我们设置了`width: 100%`或其他固定宽度后流动性就丢失了，也失去了margin/border/padding和content内容区域自动分配水平空间的机制
    2. 我们经常犯的错误是设置`display: block`后又设置了`width: 100%`, 其实width: 100%是没有必要的，这会让元素丢失流动性、包裹性等特性
    3. width值默认是作用在`content-box`上的，通过设置`box-sizing`可以改变`width`的作用盒子，如 `padding-box`,`border-box`，但是不存在`margin-box`

- 为何我们经常设置的`height: 100%`会失效？

> 规范中写道：如果包含块的高度没有显式指定，并且该元素不是绝对定位，则计算值为auto, 一句话总结：因为解释成了auto, 而 `auto * 100 / 100 = NaN`

- 如何让height: 100%生效？
    1. 设定显式的高度值
    ```
        html, body {
          height: 100%;
        }
    ```
    2. 使用绝对定位
    ```
      div {
        height: 100%;
        position: absolute;
      }
    ```
注： 绝对定位元素的宽高百分比计算是相对于padding-box的，举个例子如下：
```
<div class="box">
  <div class="child">高度100px</div>
</div>
<div class="box rel">
  <div class="child">高度160px</div>
</div>
```
```
.box {
  height: 160px;
  padding: 30px;
  box-sizing: border-box;
  background-color: #beceeb;
}
.child {
  height: 100%;
  background: #cd0000;
}
.rel {
  position: relative;
}
.rel > .child {
  width: 100%;
  position: absolute;
}
```
![绝对定位和非绝对定位百分比高度对比.png](https://upload-images.jianshu.io/upload_images/11543643-70569847e6c288a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- min-width/max-width和min-height/max-height
1. 初始值：`min-* `初始值是`auto`, `max-*` 初始值是`none`
2. max-width会覆盖!important
3. min-width如果比max-width大，则min-width生效
4. 通过max-height实现任意高度元素的展开收起，点击http://demo.cssworld.cn/3/3-2.php可以查看具体效果，关键代码如下：
  ```
    .element {
      max-height: 0;
      overflow: hidden;
      transition: max-height .25s;
    }
    .element .active {
      max-height: 666px; #一个足够大的最大高度值
    }
  ```

### 内联元素
- 内联盒模型
1. 内容区域：可以理解为文本选中区域
2. 内联盒子：元素的外在盒子，用来决定元素是内联还是块级，可以细分为内联盒子和匿名内联盒子
3. 行框盒子：每一行就是一个行框盒子，每个行框盒子又是由一个个内联盒子组成的
4. 包含盒子：<p>标签就是一个包含盒子，此盒子由一行一行的行框盒子组成

### 盒尺寸四大家族
1. 深入理解content
(1) 替换元素： 通过修改某个属性值，呈现的内容就可以被替换的元素，如`<img>`、`<video>`、`<textarea>`
(2) 替换元素的尺寸计算规则
a. 固有尺寸： 默认尺寸
b. HTML尺寸：如`img`的`width`和`height`属性
c. css尺寸：通过css设定的尺寸
优先级： css尺寸 > HTML尺寸 > 固有尺寸
若上面的尺寸都没设定，则表现为`300px * 150px`
(3) 异步加载图片的占位图片`<img>`通过以下设置可以提高性能
   ``` 
    img {visibility: hidden}
    img[src] {visibility: visible;}
   ```
> firefox会把img当作一个普通的内联元素，css重置时建议加上
> ```
> img {display: inline-block;}
> ```

