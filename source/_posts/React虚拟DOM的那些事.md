---
title: React虚拟DOM的那些事
date: 2019-03-18 18:12:06
tags: [react, diff]
id: l4
categories: 学习笔记
---

## 前言

一直在用 react，什么是虚拟 dom，什么是 diff 算法，这篇文章我觉得大概能说清楚这两个问题

## 虚拟 DOM

如下一个正常的 html

```html
<div id="thisdiv">
  <ul id="thisul">
    <li>item1</li>
    <li>item2</li>
  </ul>
</div>
```

就是虚拟 DOM 用 js 对象，存储了 DOM 节点信息。如下就是一个简易虚拟 DOM 的结构

```js
const dom = {
  type: 'div',
  props: {
    id: 'thisdiv',
    children: {
      type: 'ul',
      props: {
        id: 'thisul',
        children: [
          {
            type: 'li',
            props: { children: 'item1' }
          },
          {
            type: 'li',
            props: { children: 'item2' }
          }
        ]
      }
    }
  }
}
```

实现一个 creactElement 的方法，创建虚拟 dom

```js
//生成虚拟dom
const creactElement = (tag, props, ...children) => {
  return {
    type: tag,
    props: {
      ...props,
      children: children
    }
  }
}
```

实现一个 render 方法，将以上的虚拟 dom 渲染成真实 dom

```js
//渲染成真实dom
const render = (vdom, dom) => {
  const renderEl = vdom => {
    if (typeof vdom === 'string') {
      return document.createTextNode(vdom)
    }
    const el = document.createElement(vdom.type)
    if (vdom && vdom.props) {
      const props = vdom.props
      props.id && el.setAttribute('id', props.id)
      props.children.map(renderEl).forEach(el.appendChild.bind(el))
    }
    return el
  }
  dom.appendChild(renderEl(vdom))
}
```

jsx 语法创建虚拟 dom，babel 后其实就是如下的 creactElement

```js
const ele = creactElement(
  'div',
  { id: 'thisdiv' },
  creactElement(
    'ul',
    { id: 'thisul' },
    creactElement('li', null, 'line1'),
    creactElement('li', null, 'line2')
  )
)
render(ele, document.getElementById('root'))
```

## Diff 算法

当有操作要改变 UI 展示的时候就要用到 diff 来计算需要更新哪些节点  
两树的完全 diff 时间复杂度为 O(n<sup>3</sup>)，实际上 DOM 的跨层操作是非常少的，React 就采用了时间复杂度为 O(n)的同层 diff  
![diff tree](/images/diff1.jpg)

### 记录差异

通过深度遍历，记录差异，生成 patch

```js
let patchs = {}
//深度遍历,获取差异部分
const diff = (oldTree, newTree, patchs, index) => {
  if (!newTree) {
    //新树不存在的节点
    patchs[index] = {
      type: '删除'
    }
  } else if (oldTree.type !== newTree.type) {
    //类型不同直接替换
    patchs[index] = {
      type: '替换',
      node: newTree
    }
  } else {
    //类型相同
    //比较属性，如果属性不同重制属性
    diffAttr(oldTree.props, newTree.props, patchs, index)
    diffChildren(oldTree.props.childern, newTree.props.childern, index) //比较子树
  }
}
const diffAttr = (oldProps, newProps, patchs, index) => {
  // 如果属性不相同，重制属性。这里会比所有的属性，就不写代码实现了
  // if(oldProps不等于newProps){
  //   patchs[index]={
  //     type:'属性',
  //     attribute: {}
  //   }
  // }
}
const diffChildren = (oldChildren, newChildren, index) => {
  for (let i = 0; i < oldChildren.length; i++) {
    dfs(oldChildren[i], newChildren[i], index++)
  }
}
```

> 如果是列表，以 key 为索引  
> 原列表 key 为 a,b,c,d；现列表为 b,a,d,c,e,f  
> 知道新旧列表，求最小插入删除操作，可以用动态规划来解决，这里不展开说明

### 作用差异

```js
const patch=(node, patches)=>{
  let walker = {index: 0}
  dfs(node,patches,walker)
}
const dfs=(node,patchs,walker)=>{
  let patch = patches[walker.index] // 从patches拿出当前节点的差异
  let len = node.childNodes ? node.childNodes.length : 0
  for (let i = 0; i < len; i++) { // 深度遍历子节点
    let child = node.childNodes[i]
    walker.index++
    dfs(child, patchs, walker)
  }
  if (patch) {
    applyPatch(node, patch) // 对当前节点进行DOM操作
  }
}
const applyPatch=(node, patch)=>{
  //根据不同类型做不同操作
  switch(patch.type){
    case '删除':
      ...
    case '替换':
      ...
    case '属性':
      ...
    case '移动':
      ...
  }
}
```

综上，简单来说 React 虚拟 dom 的事儿就是一个构建虚拟 dom，比较差异，记录差异，作用差异的过程  
creactElement->render->update(diff->patch)

下一篇我们来看一下 React16 的 Fiber
