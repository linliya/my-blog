---
title: 用vue写一个轮播图
date: 2018-04-26 16:20:01
tags: 前端
---

今天接到一个写轮播图的需求，话不多说，直接开写：
#### html部分
```
<template>
  <div class="carousel-container">
    <header>轮播图</header>
    <section @mouseenter="_stop" @mouseleave="_begin">
      <ul class="carousel-text" @click="changePic">
        <li
          :class="{active: currentIndex === index}"
          v-for="(item, index) in carouselList" 
          :data-index="index" 
          :key="index">
          {{item.text}}
        </li>
      </ul>
      <transition-group tag="ul" class='carousel-img-container' name="fade">
        <li v-for="(item, index) in carouselList" 
            :key="index" 
            v-show="index===currentIndex">
          <img :src="item.src" :alt="item.text" class="carousel-img">
        </li>
      </transition-group>
    </section>
  </div>
</template>
```

> 核心是使用`transition-group`，这是vue提供的动画切换组件，我们可以通过这个组件方便地实现轮播图图片动画的各种切换效果，当然，如果不用这个组件，也可以通过css动画来完成，代码会复杂一点，但也是可以实现的。

#### js部门

代码不多，就直接贴上来了：
```
export default {
  name: 'c-carousel',
  data () {
    return {
      /**
       * 图片src
       */
      src: require('@/assets/images/D2.1_1@2x.png'),
      /**
       * 轮播图数据
       */
      carouselList: [
        {
          text: '1. 第一张图片', 
          src: require('@/assets/images/D2.1_1@2x.png')
        },
        {
          text: '2. 第二张图片',
          src: require('@/assets/images/D2.1_2.png'),
        },
         {
          text: '3. 第三张图片',
          src: require('@/assets/images/D2.1_3.png'),
        }
      ],
      /**
       * 当前正在显示的图片
       */
      currentIndex: 0,
      /**
       * 切换图片定时器
       */
      carouselTimer: null
    }
  },
  mounted () {
    this._begin()
  },
  methods: {
    /**
     * 点击切换图片
     */
    changePic (e) {
      this.currentIndex = parseInt(e.target.dataset.index)
    },
    /**
     * 定时切换图片
     */
    autoPlay () {
      this.currentIndex++
      if (this.currentIndex >= this.carouselList.length) {
        this.currentIndex = 0
      }
    },
    /**
     * 开始定时切换图片
     */
    _begin () {
      this.carouselTimer = setInterval (this.autoPlay, 4000)
    },
    /**
     * 停止定时切换图片
     */
    _stop () {
      clearInterval(this.carouselTimer)
    }
  }
}
```

> 因为使用的是`nuxt`，所以图片的引入需要用`require`,  [nuxt](https://nuxtjs.org/) 的好处就是有利于SEO,轮播图一般出现的场景是公司官网，对搜索引擎友好是必要条件，这是`spa`(单页web应用)无法达到的，而且`nuxt`预设了利用`Vue.js`开发服务端渲染的应用所需要的各种配置，服务端渲染能大大提高网页对加载速度

#### css部分核心代码

```
 .carousel-img-container {
    overflow: hidden;
    width: 100%;
    position: relative;
    li {
      width: 100%;
      height: 100%;
      position: absolute;
      left: 0;
      top: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      img {
        width: 100%;
      }
    }
  }
// 动画
.fade-enter-active, .fade-leave-active {
  transition: all 2s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
  transition: translateY(20px);
}
```
使用`transition-group`时遇到一个问题，当添加和移除元素的时候，周围的元素会瞬间移动到他们的新布局的位置，而不是平滑的过渡，解决方案是所有图片需要相对于容器绝对定位，让每张图片叠加在一起，然后再根据判断条件显示对应的图片，关于`transition-group`的其他详细内容请查看[官方文档](https://cn.vuejs.org/v2/guide/transitions.html)





