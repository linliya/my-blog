---
title: 读书笔记-css揭秘1之背景与边框
date: 2018-10-05 17:56:59
tags:
---
### 1. 半透明边框失效问题
想通过以下代码实现白色背景外面的半透明白色背景，但好像并没有起作用

```
border: 10px solid hsla(0, 0%, 100%);
background: white;
```

原因是: 默认情况下，背景会延伸到边框所在的区域下层，意思是背景是以元素的border-box外边缘包含的内容渲染的，如下图所示，border-edge以内的内容都是背景，因此我们设置的半透明边框也就被白色背景所覆盖了
![](/images/css-secret1)
- 解决方案
通过背景裁剪属性(background-clip)设定背景从padding-box的外边缘开始渲染，这时我们想要实现的效果就出来了
```
border: 10px solid hsla(0, 0%, 100%);
background: white;
background-clip: padding-box;
```
- 效果图
![](/images/css-secret2)

### 2. 多重边框需要添加多余标签？
如果要我们实现多重边框，现在能想到的是不是使用多个元素来模拟？但是这样子的话就多了冗余标签了，其实，还有一个属性可以帮助我们实现这个功能, 那就是`box-shadow`

```
background: yellowgreen;
box-shadow: 0 0 0 10px #655,
            0 0 0 15px deeppink
            0 2px 5px 15px rgba(0, 0, 0, .6)
```
- 效果图
![](/images/css-secret3)

唯一需要注意的是，box-shadow是层层叠加的，第一层投影位于最里层，依次类推，因此需要按照此规律调整扩张半径，比如说，在第一层边框的基础上，我们想在外圈再加一道5px的边框，那就需要指定扩张半径为15px(10px+5px)，还可以在这些边框的外面再加一层常规的投影
- 双重边框也可以用`outline`实现，通过`outline-offset`属性可以控制它跟元素边缘之间的间距，这个属性可以接受负值
### 3. 背景图片偏移
很多时候，我们想针对容器某个角对背景图片做偏移定位，如右下角，通常我们对背景图片使用: `background-position: bottom right`来让背景图片定位右下角，但是紧贴边缘的效果让视觉很不舒服，这时如果想让背景图片距离边缘有一定的间隙，要怎么实现呢?

#### background-position方案
```
background: url(a.svg) no-repeat bottom right #58a;
background-position: right 20px bottom 10px;
```
效果图:
![](/images/css-secret4)

#### background-origin方案
若我们想让背景图片的偏移量与内边距一致，如果使用`background-position`方案，代码看起来是这样的:
```
padding: 10px;
background: url(a.svg) no-repeat #58a;
background-position: right 10px bottom 10px;
```
这段代码不够DRY(don't repeat yourself)，每次改内边距时，我们都需要改动3个地方，这时可以考虑使用`background-origin`来优化这段代码

> 相信很多人都写过类似`background-position: top left`这样的代码，那这里的top left到底是哪里的左上角，每个元素身上都存在3个矩形框`(border box, padding box, content box)`,而`background-position`是以`padding box`为准的，top left默认指的是padding box的左上角

`background-origin`的默认值是`padding-box`, 通过将`background-origin`设定为`content-box`,我们在`background-position`中设定的边角关键字将会以内容区的边缘作为基准，这时就与内边距保持一致了，代码如下:
```
padding: 10px;
background: url(a.svg) no-repeat #58a bottom right;
background-origin: content-box;
```

#### calc方案
calc函数允许我们对距离进行运算，代码如下:
```
background: url(a.svg) no-repeat;
background-position: calc(100% - 20px) calc(100% - 10px);
```
### 4. 边框内圆角
这是我们想要实现的效果图:

![](/images/css-secret5)
这个效果如果采用两个元素，我们可以很容易地实现，那如果想利用前面的知识用一个元素实现呢？
我们需要利用`outline`描边不会跟着圆角走，而`box-shadow`则是会的这两个特性，代码如下:
```
background: tan;
border-radius: .8em;
padding: 1em;
box-shadow: 0 0 0 .6em #655;
outline: .6em solid #655;
```
这里的box-shadow的扩张值可以直接使用圆角值的一半

> 这个方案依赖的特性是描边不跟着圆角走，但以后可能会修改，所以无法保证这种行为会永远不变
### 5. 条纹背景

#### 水平条纹
利用css线性渐变属性`linear-gradient`属性和`background-size`，我们可以实现本来需要用背景图片才能实现的好看的条纹背景

基础的渐变情况是这样的:
```
background: linear-gradient(#fb3, #58a)
```
![](/images/css-secret6)
如果我们将这两个色标拉近一点:
```
background: linear-gradient(#fb3 20%, #58a 80%)
```
![](/images/css-secret7)
20% 80%表示#fb3到#58a的渐变区域为20%-80%,其它位置都为实色，
40% 60%表示渐变区域仅为40%-60%之间,
50% 50%则表示全部区域都为实色区域了
```
background: linear-gradient(#fb3 50%, #58a 50%);
```

![](/images/css-secret8)
渐变是一种由代码生成的图像，我们能像对待其它任何背景图像那样来对待它，增加`background-size`来调整尺寸:
```
background: linear-gradient(#fb3 50%, #58a 50%);
# 100% 表示宽度， 30px表示高度
background-size: 100% 30px;
```
默认是重复平铺的,效果图如下：

![](/images/css-secret9)
创建不等宽的条纹:
```
background: linear-gradient(#fb3 30%, #58a 30%);
background-size: 100% 30px;
```

![](/images/css-secret10)
为了避免每次改动条纹宽度都需要改动两个数字，我们可以根据以下规范对代码进行调整
> 如果某个色标的位置值比整个列表中在它之前的色标的位置值都要小，则该色标的位置值会被设置为它前面所有色标位置值的最大值
```
background: linear-gradient(#fb3 30%, #58a 0);
background-size: 100% 30px;
```
这样可以实现跟上图一样的结果，但代码会更加DRY
如果要创建三种颜色但水平条纹，也是很简单的:
```
background: linear-gradient(#fb3 33.3%, #58a 0, #58a 66.6%, yellowgreen 0);
background-size: 100% 45px;
```

![](/images/css-secret11)

#### 垂直条纹
只需要增加一个参数就可以实现，即渐变的角度(方向)
```
background: linear-gradient(to right, #fb3 50%, #58a 0);
background-size: 30px 100%;
```

![](/images/css-secret12)

#### 斜向条纹
按照上面的思路，我们写出了下面的代码:
```
background: linear-gradient(45deg, #fb3 50%, #58a 0);
background-size: 30px 30px;
```

![](/images/css-secret13)

好像结果跟想象的不太一样，正确的思路应该是增加2个色块，一个贴片包含4条条纹，而不是两条
```
background: linear-gradient(45deg, #fb3 25%, #58a 0, #58a 50%, #fb3 0, #fb3 75% #58a 0);
background-size: 30px 30px;
```

![](/images/css-secret14)
这时我们又发现另一个问题，条纹比预想的要细，那么background-size的宽高应该如何计算呢?
根据勾股定理计算得(具体过程不详细说明),如果想让条纹宽度为15px,就需要把background-size指定为2x15√2, 大约为42.4px
```
background: linear-gradient(45deg, #fb3 25%, #58a 0, #58a 50%, #fb3 0, #fb3 75% #58a 0);
background-size: 42px 42px;
```
![](/images/css-secret15)

#### 更好的斜向条纹
根据上面创造斜向条纹的方法，如果我们想让条纹的角度是60度的话怎么办，若是单纯只改变角度似乎不能实现

![](/images/css-secret16)

其实我们有更好的方法来创建斜向条纹, 即`repeating-linear-gradient()`，它的工作方式与`linear-gradient()`类似，只有一点不同：色标是无限循环重复的，直到填满整个背景,因此我们想实现的60度斜向条纹这样写即可:
```
background: repeating-linear-gradient(60deg, #fb3, #fb3 15px, #58a 0, #58a 30px);
```
