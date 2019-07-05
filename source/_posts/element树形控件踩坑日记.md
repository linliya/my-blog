---
title: element树形控件踩坑日记
date: 2018-08-05 17:23:12
tags:
---
最近在做一个管理系统，页面左侧需要一个目录树，便于文件的操作，不想从头开始造轮子，于是就考虑采用iview或者element的tree，调研后发现iview的tree还是有点局限，没有拖拽移动功能，没有懒加载子目录的功能等等，而element则比较符合我们的需求，虽然坑也是有点多...

## lazy & load
在`<el-tree>`中加入lazy属性，可以让树变成懒加载的tree,即默认渲染左边的下拉小箭头，点击每个小箭头可以触发一次load操作，可以实现动态获取树下面节点的操作

> 这里遇到了第一个问题：怎么获取每个节点对应的路径？

我们需要根据每个节点所在的路径向后台发送请求，节点的路径就是我们请求的资源路径，当然拿到这个路径的方法就是字符串拼接，拿到当前节点的node对象，根据它是否存在parent，将它的parent推入我们的currentPath数组中，每次推进数组之后需要将当前节点设置为它的parent,当然这个思路还是费了一点时间才想到的-_-!!

- 获取每个节点对应路径的方法
```
# 获取当前文件所在路径
getCurrentPath (node) {
  if (node && node.data && node.data.name) {
    let nodeParent = node.parent
    this.currentPath = [node.data.name]
    while (nodeParent && nodeParent.data && nodeParent.data.name
           && typeof nodeParent.data === 'object') {
      this.currentPath.unshift(nodeParent.data.name)
      nodeParent = nodeParent.parent
    }
  }
}
```

## props
我们创建文件的时候，是不需要`lazy load`时生成的小箭头的，因为文件下面是不能创建文件的，因此，需要做一下配置，在创建文件的时候给el-tree传一下类型，跟它说我要创建的是文件，不要给我渲染一个小箭头了，那要怎么配置呢？其实这个问题element官方文档有具体的例子
> 第二个问题：怎么选择性渲染`lazy load`生成的小箭头?
```
<el-tree
  :props="props1"
  :load="loadNode1"
  lazy>
</el-tree>

<script>
  export default {
    data() {
      return {
        props1: {
          label: 'name',
          children: 'zones',
          isLeaf: 'leaf'
        },
      };
    },
    methods: {
      loadNode1(node, resolve) {
        if (node.level === 0) {
          return resolve([{ name: 'region' }]);
        }
        if (node.level > 1) return resolve([]);

        setTimeout(() => {
          const data = [{
            name: 'leaf',
            leaf: true
          }, {
            name: 'zone'
          }];

          resolve(data);
        }, 500);
      }
    }
  };
</script>
```

## renderContent

renderContent会监听data里面的属性值来决定是否渲染和渲染对应的视图，如果有对某个data的属性值判断的需要，需要对那个属性值进行初始化

例如:
根据node节点的data.type决定渲染的内容,一开始需要给data.type赋初始值，如果不赋值则监听不到变化(我就是因为一开始没有初始化type,直接设置data.type='edit',然后视图一直没更新......)

> 第三个问题: 为什么data.type变化了，视图一直没更新?

```
  # 重命名编辑框
  if (data.type === 'edit') {
    return h('input', {
      attrs: {
        id: 'treeInput',
        value: this.currentNodeData.name
      },
      on: {
        blur: (e) => {
          this.updateCurrentNode(e.target.value || data.name)
        },
        keyup: (e) => {
          if (e.keyCode === 13 || e.keyCode === 27) {
            e.target.blur()
          }
        }
      }
    })
  }
  # 新建编辑框
  if (data.type === 'input') {
    return h('input', {
      attrs: {
        id: 'treeInput'
      },
      on: {
        blur: (e) => {
          this.createNewNode(e.target.value)
        },
        keyup: (e) => {
          if (e.keyCode === 13 || e.keyCode === 27) {
            e.target.blur()
          }
        }
      }
    })
  }
```
这里还有一个小问题:`on-blur`和`on-keyup`本来我写的都是`this.createNewNode(e.target.value)`,但是触发了两次`create`操作，原来是`keyup`的同时输入框也会失去焦点，所以就触发了`blur`，因此就用`e.target.blur()`代替了原本的写法，统一用`blur`来实现触发`create`的操作

## 浏览器渲染问题

> 第四个问题: 为什么需要setTimeout?

我们对树的操作的过程经常需要使用到setTimeout(fn, 0)，例如:
```
tryToCreateNode (type) {
  this.$refs.tree.append({
    id: 'treeInput',
    type: 'create'
  }, this.currentNodeData.id)
  this.currentNode.expanded = true
  this.createNewWay = 'append'
  setTimeout(() => {
    this.$el.querySelector('#treeInput').focus()
  }, 0)
}
```
这是因为当我们执行了append操作时，触发了浏览器的重排和重绘，需要重新构建dom树，这个过程是比较耗费时间的，如果我们接下来直接执行`this.$el.querySelector('#treeInput').focus()`,这时候dom树是还没有treeInput这个元素的，setTimeout会将querySelector操作放进任务队列中去，等到主进程完成dom树的构建后再执行setTimeout里面的操作，这时候我们就可以拿到我们的treeInput了

## 从后端递归获取目录树数据

> 第五个问题：如果不用懒加载，我们怎么渲染目录树？

因为我们这个项目后端存储文件的方式就是一个文件系统，就跟我们在本地看到的一样，一层一层地存文件和文件夹，因此我们前端获取文件也是得一层一层地发请求，拿到对应层级的文件，这就得考虑通过递归的方式，将每次获取得到的数据保存在一个treeData对象中，如第一层的数据就是`treeData[0]`,`treeData[1]`,第二层的数据就是`treeData[0].children`, `treeData[1].children`等等

#### 那这个递归函数要怎么实现呢？

- 获取某一层数据

- 将上一层获取到的文件夹类型的数据再传入递归函数

- 当获取那个层级的文件数为0， 或者都是文件的时候，结束递归

```
# 递归获取目录树数据
async getTreeDataRecursively (path) {
  # getDirByPath是我们自己定义的获取对应目录文件的函数
  let dataList = await this.getDirByPath(path)
  if (dataList && dataList.length >= 0) {
    if (!dataList || dataList.length === 0 || dataList.every(el => el.type === 'file')) {
      return dataList
    } else {
      for (let i = 0; i < dataList.length; i++) {
        let path = dataList[i].id
        if (dataList[i].isDir) {
          this.$set(dataList[i], 'children', await this.getTreeDataRecursively(path))
        }
      }
      return dataList
    }
  }
}
```
#### 在vue中，如果直接通过赋值的方式`myObj.name = 'aaa'`这样的方式为一个对象的新增某个属性，不会触发视图的更新，可以通过`$set`来新增，从而触发更新，详细见[官方文档](https://cn.vuejs.org/v2/guide/reactivity.html)

## 在目录树中插入子节点

> 递归获取到的数据要怎么插入到对应的节点呢？

在上一个问题中，我们解决了每个节点下面子节点的获取，得到了某个路径下包含所有子节点的一个对象，这个对象需要插入到对应的目录结构，例如:

```
--- a
   --- aa
     --- aaa
     --- aaa1
        --- aaaa
```
我们通过`getTreeDataRecursively('/a/aa')`获取到了`/a/aa`下面的目录结构: `aaa`和`aaa1.children(aaaa)`, 那么我们现在想要把它插入到对应的路径`/a/aa`下面，要怎么实现呢？

一开始的思路是通过路径跟每个节点的id比较，因为我们在上面的递归函数中，把路径赋给了每个节点的id，如果id = '/a/aa'，那么我们就将数据dataList插入到下面，整体思路是没有问题的，但是要改变treeData对应层级的数据这一步卡住了，无法实现...

既然没法通过改变treeData的数据结构，我就翻看起了element tree的官方文档，终于找到了解决方案...


![](../../images/element-ui)

我们可以官方提供的这个方法，这里的`key`就是我们的`id`(`/a/aa`), 而`value`则是dataList

完美～


### 写在最后

这就是这一个庞大的组件我们遇到的坑，当然还有很多没有写下来，本文只是作为纪录重要的几个点，也方便有遇到同样的问题的同学查看，能提供一点小小的思路也是很荣幸哈~
  