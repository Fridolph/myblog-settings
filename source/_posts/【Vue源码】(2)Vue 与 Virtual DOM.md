---
title: 【Vue源码】（二）Vue 与 Virtual DOM
date: 2019-1-18 21:52:59
tags:
  - vue
  - Virtual DOM
  - 虚拟dom
categories:
  - vue源码学习
---

> React 好像已经火了很久很久，以致于我们对于 Virtual DOM 这个词都已经很熟悉了，网上也有非常多的介绍 React、Virtual DOM 的文章。 而 Vue2 也是加入了 Virtual DOM 来提高性能。

[**Change And Its Detection In JavaScript Frameworks**
Monday Mar 2, 2015 by Tero Parviainen](http://teropa.info/blog/2015/03/02/change-and-its-detection-in-javascript-frameworks.html)

更多 Virtual DOM 的介绍和前生可以参考这篇文章，这里就直接进入正题——Vue 与 virtual dom

![Virtual DOM](http://blog.fueson.top/article/img/2Virtual-DOM.jpg)

> 1 月 25 有大佬谈到这个话题，受益匪浅，于是准备就这个话题再展开一下，并记录下来，也算是再次深入学习。具体的请跳到最后哦~

<!-- more -->

以下为 1 月 25 日大佬发言笔记：

> 什么是虚拟 DOM？虚拟 DOM 的原理及它的价值是什么？
> diff 算法是怎样的?树结构的 diff 和对象数组的 diff 有什么不一样的？JSON patch 是什么？怎么应用？
> 浏览器的重排呢？也是 diff 么？那是的话又是什么结构的数据在 diff 呢？如何减少浏览器的重排？再来。通讯录的同步与 diff 的关系。线下与云端的同步。把一个小原理，看成一个场景问题，解决一大片问题！

从上文也能找出一些线索，这里就作个整理。

### 什么是虚拟 DOM

虚拟 DOM 就是用一个原生的 JS 对象去描述一个真实的 DOM 节点

### 虚拟 DOM 的原理

虚拟 DOM 是在 DOM 的基础上建立了一个抽象层，我们对数据和状态所做的任何改动，都会被自动且高效的同步到虚拟 DOM，最后再批量同步到 DOM 中。

### 虚拟 DOM 的价值

虚拟 DOM 会使得应用只关心数据和组件的执行结果，中间产生的 DOM 操作不会受到干预，且通过虚拟 DOM 来生成 DOM，会有一项非常可观收益——性能（在这个过程中就是省下了回流重绘的渲染开销）

虚拟 DOM 的出现改变了前端的开发思想和生态，对今后的业界也有积极影响

### 什么是 diff 算法

个人理解就是比较新旧老节点之间的差异，做到最优化的更新，所以中间涉及到较复杂的算法。（方向选择问题，现阶段没对该问题进行深究）

暂不想对这部分进行展开，毕竟知识盲区：

- 如何判断新旧节点是相同的，相同就跳过（做到优化）
- 若不相同时会有很多种情况：tag? key? 有无 children? text 有差异? 等等等

### 什么是 JSON patch？ 如何应用

参考自

- [http://jsonpatch.com/](http://jsonpatch.com/)
- [JSON patch 使用简述](https://www.imooc.com/article/36843)

它是一种用于描述对 JSON 文档的更改的格式。它可以用来避免在只有一个部件已经更改时发送整个文档。当与 HTTP 修补程序方法结合使用时，它允许以符合标准的方式对 HTTPAPI 进行部分更新。

### 浏览器的重排？ 什么的改变触发了重排？ 如何减少？

这样一想，浏览器的重排本质上也是一种 diff。

页面在初次渲染后，DOM 树在结构（增删）改变，或者获取某些属性或操作等：

- offsetTop
- offsetLeft
- getComputedStyle()
- 改变 DOM 元素位置
- 改变 DOM 元素的集合属性（width、height、padding、margin 等）
- 元素内容的改变
- 调整缩放、触发 resize、scroll 等

优化：

- 将改变样式的属性尽可能集中放置
- 对获取时会触发重排的属性进行缓存（运行时允许的情况下）
- 需要改变多个样式的时候，利用切换 class 来实现
- 可以将 position 设置为 absolute 或 fixed，脱离文档流

下面是之前的正文部分：

---

# Virtual DOM

Virtual DOM 这个概念相信大部分人都不会陌生，它产生的前提是浏览器中的 DOM 是很“昂贵"的，为了更直观的感受，我们可以简单的把一个简单的 div 元素的属性都打印出来

```js
const div = document.createElement('div');
let str = '';
for (const key in div) {
  str += `${key}\n`;
}
console.log(str); // 结果超乎你的想象
```

可见，真正的 DOM 元素是非常庞大的，因为浏览器的标准就把 DOM 设计的非常复杂。当我们频繁的去做 DOM 更新，会产生一定的性能问题。

而 Virtual DOM 就是用一个原生的 JS 对象去描述一个 DOM 节点，所以它比创建一个 DOM 的代价要小很多。在 Vue.js 中，Virtual DOM 是用 VNode 这么一个 Class 去描述，它是定义在 src/core/vdom/vnode.js 中的。

```ts
export default class VNode {
  tag: string | void;
  data: VNodeData | void;
  children: ?Array<VNode>;
  text: string | void;
  elm: Node | void;
  ns: string | void;
  context: Component | void; // rendered in this component's scope
  key: string | number | void;
  componentOptions: VNodeComponentOptions | void;
  componentInstance: Component | void; // component instance
  parent: VNode | void; // component placeholder node

  // strictly internal
  raw: boolean; // contains raw HTML? (server only)
  isStatic: boolean; // hoisted static node
  isRootInsert: boolean; // necessary for enter transition check
  isComment: boolean; // empty comment placeholder?
  isCloned: boolean; // is a cloned node?
  isOnce: boolean; // is a v-once node?
  asyncFactory: Function | void; // async component factory function
  asyncMeta: Object | void;
  isAsyncPlaceholder: boolean;
  ssrContext: Object | void;
  fnContext: Component | void; // real context vm for functional nodes
  fnOptions: ?ComponentOptions; // for SSR caching
  fnScopeId: ?string; // functional scope id support

  constructor (
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.fnContext = undefined
    this.fnOptions = undefined
    this.fnScopeId = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }

  // DEPRECATED: alias for componentInstance for backwards compat.
  /* istanbul ignore next */
  get child (): Component | void {
    return this.componentInstance
  }
}
```

可以看到 Vue.js 中的 Virtual DOM 的定义还是略微复杂一些的，因为它这里包含了很多 Vue.js 的特性。这里千万不要被这些茫茫多的属性吓到，实际上 Vue.js 中 Virtual DOM 是借鉴了一个开源库 snabbdom 的实现，然后加入了一些 Vue.js 特色的东西。我建议大家如果想深入了解 Vue.js 的 Virtual DOM 前不妨先阅读这个库的源码，因为它更加简单和纯粹。

## 小结

其实 VNode 是对真实 DOM 的一种抽象描述，它的核心定义无非就几个关键属性，标签名、数据、子节点、键值等，其它属性都是都是用来扩展 VNode 的灵活性以及实现一些特殊 feature 的。由于 VNode 只是用来映射到真实 DOM 的渲染，不需要包含操作 DOM 的方法，因此它是非常轻量和简单的。

Virtual DOM 除了它的数据结构的定义，映射到真实的 DOM 实际上要经历 VNode 的 create、diff、patch 等过程。那么在 Vue.js 中，VNode 的 create 是通过之前提到的 createElement 方法创建的，我们接下来分析这部分的实现。
