---
layout: post
title: el-tree懒加载树 手动触发load更新
date: 2024-02-19 09:59:37
tags: element-ui
---
element-ui el-tree懒加载树组件，当对树组件进行增删改时需要手动触发load更新节点数据，遇到一些问题，特此记录下面三种方法，避免下次踩坑

## 方法1：通过保存currentNode和currentResolve来刷新（不推荐）
```
<template>
  <el-tree
    :props="defaultProps"
    :load="loadLazyTree"
    lazy
  ></el-tree>
</template>

<script>
export default {
  data() {
    return {
      defaultProps: {
        label: 'name',
        children: 'children',
        isLeaf: (data, node) => {
          return !data.hasChild
        }
      },
    }
  },
  methods: {
    loadLazyTree(node, resolve) {
      const isRootNode = node.level === 0 ? true : false
  
      // 保存当前展开的节点数据
      this.currentNode = node
      this.currentResolve = resolve
    
      const sendData = {
        parentId: isRootNode ? null : node.data.id
      }
      getPrepPlanOrganTree(sendData).then(res => {
        resolve(res.data)
      })
    },
    loadCurrentNode() {
      this.currentNode.childNodes = []
      this.loadLazyTree(this.currentNode, this.currentResolve)
    }
  }
}
</script>
```
缺点：
loadLazyTree只会针对某个节点加载一次数据，下次再展开不重新加载数据，此时保存的this.currentNode和this.currentResolve与当前点击节点不匹配，这种情况调用节点的刷新当前节点数据就是错的

## 方法2：通过node.expand()来刷新（推荐）

```
<template>
  <el-tree
    :props="defaultProps"
    :load="loadLazyTree"
    lazy
    ref="tree"
    @node-click="handleNodeClick"
  ></el-tree>
</template>

<script>
export default {
  data() {
    return {
      defaultProps: {
        label: 'name',
        children: 'children',
        isLeaf: (data, node) => {
          return !data.hasChild
        }
      },
      currentNode: null
    }
  },
  methods: {
    loadLazyTree(node, resolve) {
      const isRootNode = node.level === 0 ? true : false
      const sendData = {
        parentId: isRootNode ? null : node.data.id
      }
      getPrepPlanOrganTree(sendData).then(res => {
        resolve(res.data)
      })
    },
    // 重载当前节点
    loadCurrentNode() {
      let node = this.$refs.tree.getNode(this.currentNode.id) // 获取当前节点
      node.loaded = false// 设置未进行懒加载状态
      node.expand()// 重新展开节点就会间接重新触发load达到刷新效果
    },
    // 获取选中的子节点
    handleNodeClick(data, node) {
      this.currentNode = node
      this.$emit('node-click', data)
    }
  }
}
</script>
```
该方法不需要缓存resolve这个不好拿到的入参, 避免调用loadLazyTree(node, resolve)函数时传入缓存的node和resolve不匹配当前点击的节点

## 方法3：通过node.loadData()来刷新
```
<template>
  <el-tree
    :props="defaultProps"
    :load="loadLazyTree"
    lazy
    ref="tree"
    @node-click="handleNodeClick"
  ></el-tree>
</template>

<script>
export default {
  data() {
    return {
      defaultProps: {
        label: 'name',
        children: 'children',
        isLeaf: (data, node) => {
          return !data.hasChild
        }
      },
      currentNode: null
    }
  },
  methods: {
    loadLazyTree(node, resolve) {
      const isRootNode = node.level === 0 ? true : false
      const sendData = {
        parentId: isRootNode ? null : node.data.id
      }
      getPrepPlanOrganTree(sendData).then(res => {
        resolve(res.data)
      })
    },
    // 重载当前节点
    loadCurrentNode() {
      this.currentNode.loaded = false
      this.currentNode.childNodes = []
      this.currentNode.loadData((data) => {
        // 懒加载完数据后做一些操作
      })
    },
    // 重载当前节点的父节点
    loadParentNode() {
      const treeNode = this.$refs.tree.getNode(this.currentNode.id)
      treeNode.parent.loaded = false
      treeNode.parent.childNodes = []
      treeNode.parent.loadData() // 刷新父节点数据
    },
    // 获取选中的子节点
    handleNodeClick(data, node) {
      this.currentNode = node
      this.$emit('node-click', data)
    }
  }
}
</script>
```
亲试node.loadData()有时候获取不到导致报错


[el-tree 懒加载load 手动触发load更新的三种方法](https://www.jianshu.com/p/0e1d4d28104e)
[ElementUi Tree树形控件的使用（增、删）](https://www.dianjilingqu.com/190963.html) 
