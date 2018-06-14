---
title: 【FEStar】(一)从订阅发布模式说起
date: 2018-06-13 23:19:03
tags:
  - 观察者
  - 发布订阅
  - 设计模式
categories:
  - 设计模式
---

> 先扯点闲话。一时兴起，遂决定之后采用这个前缀来写这个十多篇的系列总结了。在之前的学习和博客整理中，感觉都挺没方向，很散乱。除了看书的总结外，其他时候都是零碎性地学习，我想，若能系统性地带着目的去散落这些点，再类似依赖收集一样把这些零碎的东西汇总整理出来，那应该也挺不错的~

![](http://blog.fueson.top/18-6-14/4361135.jpg)

这周的复习知识点主要是以下几方面：

* 设计模式之发布订阅/观察者模式
* 事件相关，事件、模型、处理机制
* Ajax
* 异步

<!-- more -->

我将这些点梳理到了脑图中，并作了一些批注，感觉一边整理了知识点，也让自己有了一个整体的脉络和方向感，树越分散也就是所谓的深度，让树更深也就是知识的深度。

![知识点梳理](http://blog.fueson.top/18-6-14/52685215.jpg)

## 从设计原则说起

一个优秀的程序员通常由其**操作技能**、**知识水平**，**经验层力**和**能力**四个方面组成。这些原则，也就是巨人的肩膀，总结而出流传至今，每一个程序员都应该了解，运用于开发和生活中。

要我来说的话就是，先去了解，知道有这些东西，也许不能马上运用到实际项目中，但当我们遇到困难时，就可以回顾一下图里的各原则，结合以下经验：

1. 粗浅地了解这些原则
2. 在（公司或开源）项目中观察或总结他人或自己的设计
3. 实践、回顾，再总结
4. 总结后再回到第1步深入了解以此往复

### 观察者模式

比较概念的解释是，目标和观察者是基类，目标提供维护观察者的一系列方法，观察者提供更新接口。具体观察者和具体目标继承各自的基类，然后具体观察者把自己注册到具体目标里，在具体目标发生变化时候，调度观察者的更新方法

![观察者模式](http://blog.fueson.top/18-6-14/636423.jpg)

简易版观察者模式实现

```js
class ObserverList {
  constructor() {
    this.observerList = [];
  }

  add(observer) {
    this.observerList.push(observer)
  }

  get(index) {
    if (index >= 0 && index < this.count()) {
      return this.observerList[index]
    }
  }

  removeAt(index) {
    this.observerList.splice(index, 1)
  }

  count() {
    return this.observerList.length
  }

  indexOf(observer, startIndex = 0) {
    let i = startIndex

    while (i < this.count()) {
      if (this.observerList[i] === observer) {
        return i
      }
      i++
    }
    return -1
  }
}

class Subject {
  constructor() {
    this.observers = new ObserverList();
  }

  addObserver(observer) {
    this.observers.add(observer)
  }

  removeObserver(observer) {
    this.observers.removeAt(this.observers.indexOf(observer))
  }

  notify(...args) {
    let observerCount = this.observers.count()
    for (let i = 0; i < observerCount; i++) {
      this.observers.get(i).update(args)
    }
  }
}
```


### 发布订阅模式

比较概念的解释是，订阅者把自己想订阅的事件注册到调度中心，当该事件触发时候，发布者发布该事件到调度中心（顺带上下文），由调度中心统一调度订阅者注册到调度中心的处理代码。

![发布订阅模式](http://blog.fueson.top/18-6-14/91374451.jpg)

简易版发布订阅模式实现

```js
module.exports = class PubSub {

  constructor() {
    this.subscribers = {}
  }

  subscribe(type, fn) {
    let listeners = this.subscribers[type] || []
    listeners.push(fn)
    this.subscribers[type] = listeners
  }

  unsubscribe(type, fn) {
    let listeners = this.subscribers[type]
    if (!listeners) return
    this.subscribers[type] = listeners.filter(v => v !== fn)
  }

  publish(type, ...args) {
    let listeners = this.subscribers[type]
    if (!listeners) return
    listeners.forEach(item => (item.apply(type, args)))
  }
}
```

### 两者区别

* 两者最大的区别是调度的地方
虽然两种模式都存在订阅者和发布者（具体观察者可认为是订阅者，具体目标可认为是观察者），但是观察者模式是由具体目标调度的，而发布/订阅模式是统一由调度中心调度的。所以观察者模式的订阅者和发布者之间是存在依赖的，而发布/订阅模式则不会
* 两种模式都可以用于松散耦合，改进代码管理和潜在的复用

#### 优点

观察者和发布/订阅模式鼓励我们认真思考应用程序不同部分之间的关​​系。他们还帮助我们确定包含直接关系的层次，而这些关系可以用主题和观察者集合取而代之。这可以有效地将应用程序分解为更小，更松散的块，以改善代码管理和重用的潜力。

使用Observer模式的进一步动机是我们需要保持相关对象之间的一致性，而不需要使类紧密耦合。例如，当一个对象需要能够通知其他对象而不做这些对象的假设时。

使用这两种模式时，观察者和主体之间可能存在动态关系。这提供了很大的灵活性，当我们的应用程序的不同部分紧密耦合时，这可能不容易实现。

尽管对于每个问题来说，这并不总是最好的解决方案，但这些模式仍然是设计分离系统的最佳工具之一，应该被视为任何JavaScript开发人员的工具带中的重要工具。

#### 缺点

因此，这些模式的一些问题实际上源于其主要益处。在“发布/订阅”中，通过将发布者与订阅者分离，有时可能难以获得我们应用程序的特定部分正常运行的保证。

例如，发布商可能会假定一个或多个订阅者正在倾听他们。假设我们正在使用这样的假设来记录或输出关于某些应用程序过程的错误。如果执行日志记录的用户崩溃（或出于某种原因无法运行），由于系统的解耦特性，发布者将无法看到这种情况。

这种模式的另一个缺点是用户对彼此的存在并不知情，并且对转换发布商的成本视而不见。由于用户和发布者之间的动态关系，更新依赖性很难追踪。

---

### 发布订阅模式实现Vue的双向绑定

这么一看的话，其实Vue里的双向绑定的实现也是发布订阅模式吧。为什么不是观察者？ 因为多了一个Watcher，这就是相当于调度中心，通过这一层为中间人进行的更新。

这类的代码看多少次都不嫌多，这是之前网上一个不错的实现，我拿过来学习了

```js
class MyVue {
  // new Vue({ ... }) 这个options就是这里的对象啦
  // 初始化 将 options 赋值到实例对象上
  constructor(options) {
    this.$options = options
    this.$el = document.querySelector(options.el)
    this.$data = options.data
    this.$methods = options.methods
    // data的内容会放到这里来， 这也就说明了 初始化结束后，手动加的值不会被监听到
    this._binding = {}
    // 初始化时会监听data上的属性
    this._observe(this.$data)
    // 初始化时 编译 $el 里的子节点
    this._compile(this.$el)
  }

  _observe(obj) {
    let that = this
    Object.keys(obj).forEach(key => {
      if (obj.hasOwnProperty(key)) {
        that._binding[key] = {
          // 添加依赖收集， 是个数组
          _directives: []
        }
        console.log(that._binding[key])
        let val = obj[key]
        if (typeof val === 'object') {
          // 若是对象就递归
          that._observe(val)
        }

        let binding = that._binding[key]
        // 响应式 双向绑定的核心操作
        Object.defineProperty(that.$data, key, {
          enumerable: true,
          configurable: true,
          get() {
            console.log(`${key} 获取 ${val}`)
            return val
          },
          set(newVal) {
            console.log(`${key} 更新 ${newVal}`)
            if (val !== newVal) {
              val = newVal
              binding._directives.forEach(item => {
                item.update()
              })
            }
          }
        })
      }
    })
  }

  _compile(root) {
    let that = this
    let nodes = root.children
    for (let i = 0; i < nodes.length; i++) {
      let node = nodes[i]
      if (node.children.length) {
        this._compile(node)
      }

      if (node.hasAttribute('v-click')) {
        node.onclick = (function() {
          let attrVal = nodes[i].getAttribute('v-click')
          return that.$methods[attrVal].bind(that.$data)
        })()
      }

      if (node.hasAttribute('v-model') && (node.tagName === 'INPUT' || node.tagName === 'TEXTAREA')) {
        node.addEventListener('input', (function(key) {
          let attrVal = node.getAttribute('v-model')
          that._binding[attrVal]._directives.push(new Wathcer(
            'input',
            node,
            that,
            attrVal,
            'value'
          ))

          return function() {
            that.$data[attrVal] = nodes[key].value
          }
        })(i))
      }

      if (node.hasAttribute('v-bind')) {
        let attrVal = node.getAttribute('v-bind')
        that._binding[attrVal]._directives.push(new Watcher(
          'text',
          node,
          that,
          attrVal,
          'innerHTML'
        ))
      }
    }
  }
}

class Watcher {
  constructor(name, el, vm, exp, attr) {
    this.name = name    // 指令名
    this.el = el        // 指令对应dom
    this.vm = vm        // 指令所属MyVue实例
    this.exp = exp      // 指令对应值
    this.attr = attr    // 绑定的属性值
    this.update()
  }

  update() {
    this.el[this.attr] = this.vm.$data[this.exp]
  }
}
```

---

于是到codepen上试试吧

<iframe height='300' scrolling='no' title='simple Vue ~ MVVM' src='//codepen.io/fridolph/embed/gKxrBO/?height=300&theme-id=dark&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/fridolph/pen/gKxrBO/'>simple Vue ~ MVVM</a> by fridolph (<a href='https://codepen.io/fridolph'>@fridolph</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
