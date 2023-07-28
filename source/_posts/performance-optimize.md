---
layout: post
title: 性能优化
date: 2023-07-27 11:52:04
tags: 前端
---

# 性能优化

## 1.document.createDocumentFragment()创建文档碎片

### 定义

用例是创建文档片段，将元素附加到文档片段，然后将文档片段附加到 DOM 树，文档片段存在于内存中，并不在 DOM 树中，所以将子元素插入到文档片段时不会引起页面回流（对元素位置和几何上的计算）。因此，使用文档片段通常会带来更好的性能。

举例：

当我们需要往 body 中添加大量节点时，按下面这种方式，每次循环都往 body 中添加节点会导致浏览器频繁重排重绘，非常影响性能

```
for (let i = 0; i < 500; i++) {
  let node = document.createElement('span')
  let iNode = document.createElement('i')
  node.appendChild(iNode)
  document.body.appendChild(node)
}
```

改进，创建 div 节点，循环创建的节点统一存在这个 div 中，然后再将 div 添加到 body 中，但缺点是要在 body 中多添加了一层 div 包裹，而 document.createDocumentFragment()就不会产生额外的节点

```
let oDiv = document.createElement('div')
for (let i = 0; i < 500; i++) {
  let node = document.createElement('span')
  let iNode = document.createElement('i')
  node.appendChild(iNode)
  oDiv.appendChild(node) // 动态创建的节点统一存在oDiv中
}
document.body.appendChild(oDiv)
```

使用 document.createDocumentFragment()进行优化，这种方式不会产生额外的 DOM 元素

```
let fragment = document.createDocumentFragment();
for (let i = 0; i < 500; i++) {
  let node = document.createElement('span')
  let iNode = document.createElement('i')
  node.appendChild(iNode)
  fragment.appendChild(node) // 动态创建的节点统一存在fragment中
}
document.body.appendChild(fragment)
```
