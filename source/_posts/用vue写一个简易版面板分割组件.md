---
title: 用vue写一个简易版面板分割组件
date: 2018-08-05 17:21:27
tags:
---
最近需要用到一个面板分割组件，于是动手自己写了一个简易版的，有以下功能:

- trigger样式自定义
- 左面板宽度可以设置默认值
- 左面板宽度可以设置最大最小值

一般使用够用的了，如果以后需要继续可以增加功能...

### 实现原理

- 设置`cursor: col-resize`可以让鼠标变成↔️这个图标(这个很多人都不知道吧^_^)
- 通过监听`mousedown`,获取鼠标开始拖拽`trigger`的位置
- 通过监听`mousemove`实时获取鼠标的位置，设置左右两边的宽度
- 通过监听`mouseup`结束trigger的拖拽，左右两边宽度不再随着鼠标移动而变化

### html部分
```
<div 
  class="split-pane" 
  ref="wraper"
  @mouseup="handleMouseup"
  @mousemove="handleMouseMove">
    <div class="split-pane-container">
      <div class="split-pane-left-area" :style="{width: leftSize}">
        <slot name="left"></slot>
      </div>
    
      <slot name="split-pane-trigger">
        <div 
          ref="trigger"
          class="split-pane-trigger"
          :style="horizontalTriggerStyle"
          @mousedown="handleMousedown">
        </div>
      </slot>
      <div class="split-pane-right-area" :style="{width: rightSize}">
        <slot name="right"></slot>
      </div>
    </div>
</div>
```
核心是slot元素，插槽功能可以让我们在使用的时候设置要左右分割的内容区域

### js部分

- 自定义trigger样式
```
horizontalTriggerStyle () {
  return Object.assign({left: this.leftSize}, this.triggerStyle)
}
```
通过`Object.assign`将组件内部设置的样式和通过`prop`传入的`triggelStyle`结合成一个`horizontalTriggerStyle`对象

- 初始化左侧宽度
```
computed () {
    leftSize () {
      return this.triggerOffset + 'px'
    }
}
```

```
mounted () {
  if (this.value) {
    this.$nextTick(() => {
      this.triggerOffset = this.transValue(this.value)
    })
  }   
}
# 将值转为Number并转换成百分比
transValue (val) {
  return typeof val === 'number'
    ? val
    : Math.floor(parseInt(val))
  }
}
```

组件挂载完成后通过判断是否传入`value`参数，设置组件的`triggerOffset`值，而左侧的宽度则是通过`triggerOffset`值`computed`而来的，因此就可以达到设置默认值的目的

- 处理鼠标在trigger上按下事件
```
handleMousedown (e) {
  this.canMove = true
  this.preTriggerOffset = this.triggerOffset
  # 记录开始按下的位置
  this.offset = {
    x: e.pageX,
    y: e.pageY
  }
  e.preventDefault()
}
```
将`canMove`属性设置为true,表示从该位置可以开始移动，同时通过`this.offset`记录当前鼠标所在坐标位置
- 处理鼠标移动
```
handleMouseMove (e) {
  if (this.canMove) {
    let moveSize = this.transValue(e.clientX - this.offset.x)
    let offset = this.preTriggerOffset + moveSize

    if (offset <= this.minTransed) {
      this.triggerOffset = Math.max(offset, this.minTransed)
    } else {
      this.triggerOffset = Math.min(offset, this.maxTransed)
    }
  }
}
```
根据`e.clientX - this.offset.x`得到鼠标移动的距离，将这个距离和开始移动的坐标相加得到trigger应该距离左边的距离，再与面板最小最大值比较，得到最终的`triggerOffset`
- 处理鼠标抬起
```
handleMouseup () {
  this.canMove = false
}
```
将canMove设置为false，结束拖拽

### 怎么使用?
```
<c-split-pane :value="260" min="260" max="600">
  <div slot="left"></div>

  <div slot="right">
    <router-view></router-view>
  </div>
</c-split-pane>
```

总结: 这就是简易面板分割组件的具体实现过程啦~本来第一版是通过百分比实现的，但是因为百分比的宽度会根据页面大小变化，比如当你把页面最大化后将左侧拖动到最大，但是缩小后它又不是在最大的位置上了，所以会有点奇怪，最后还是改成用`px`