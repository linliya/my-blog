---
title: element树形控件踩坑日记
date: 2018-08-05 17:23:12
tags:
---
最近在做一个管理系统，页面左侧需要一个目录树，便于文件的操作，不想从头开始造轮子，于是就考虑采用iview或者element的tree，调研后发现iview的tree还是有点局限，没有拖拽移动功能，没有懒加载子目录的功能等等，而element则比较符合我们的需求，虽然坑也是有点多...

### lazy & load
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

### props
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

### renderContent

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

### 浏览器渲染问题

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


暂时就遇到这几个问题，如果有后续会再补充...

