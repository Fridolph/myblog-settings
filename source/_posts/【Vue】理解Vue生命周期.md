---
title: 【Vue】理解Vue生命周期
date: 2018-4-14 12:49:03
tags: 
  - vue
  - mvvm
categories: 
  - vue
---

早上刷手机在思否上看到 闰土大叔的这篇[如何解释vue的生命周期才能令XXX满意？](https://segmentfault.com/a/1190000014376915?_ea=3615607)，觉得挺好就老套路了。当然，看别人的文章，在此基础上注释，添加自己的东西，才有写博客的价值，希望能站在大佬的肩膀上往前走得更快吧！

<!-- more -->

这张图肯定是必上的，说得相当详尽和清楚了。

<img src="https://segmentfault.com/img/bV4xju?w=1200&h=3039">

在理解上，一直觉得Vue生命周期就是四个阶段 创建 -> 挂载 -> 更新 -> 销毁，每个阶段对应2个钩子函数 `beforeCreate` `created` ， 带 before 和 ed 的，同时也写过demo，无非就是以下： https://jsfiddle.net/0dzvcf4d/10495/ 可看下官网的例子

```js
var vm = new Vue({
  el: '#editor',
  data: {
    input: '# hello'
  },
  computed: {
    compiledMarkdown: function () {
      return marked(this.input, { sanitize: true })
    }
  },
  methods: {
    update: _.debounce(function (e) {
      this.input = e.target.value
    }, 300)
  },
  beforeCreate() {
    console.log('beforeCreate')
  },
  created() {
    console.log('created')
  },
  beforeMount() {
    console.log('beforeMount')
  },
  render() {
    console.log('render')
  },
  mounted() {
    console.log('mounted')
    setTimeout(() => {
      this.input = 'vue lifecycle'
    }, 2000)    
  },
  beforeUpdate() {
    console.log('beforeUpdate')
  },
  updated() {
    console.log('updated')
  },
  beforeDestory() {
    console.log('beforeDestory')
  },
  destroyed() {
    console.log('destroyed')
  }
})
```

控制台会依次输出 

beforeCreate
created
beforeMount
render
mounted

过了两秒后，再打印
beforeUpdate
updated

那么，信息量还是挺多的，Vue的生命周期描述不是说8个钩子函数，这显然也不是问题的本质。

---

那么我们来理一理这个过程吧，请关注随时对比大图：

在一开始，我们创建了一个Vue的实例对象 vm，在控制台打印 beforeCreate 之前，首先执行了 `_init` 方法  （正好之前写了篇 【Vue】实现原生双向绑定 可参考一下） `_init` 就是将我们所传入的 el、data、methods等等各种属性绑定到Vue原型对象上。 而在init的过程中首先调用`beforeCreate`钩子。然后在`injections`[注射]和`reactivity`[反应作用]时，它会再去调用`created`钩子。

也就是说在init时，事件已经被调用了。所以，我们在`beforeCreate`时千万别去修改data里面的赋值，最早也要放在`created`里去做。

在created完成后，它会去判断`instance`实例里是否含有 `el` 属性 ，若没有 就会调用 vm.$mount(el)这个方法，然后继续执行后续逻辑；

若有el属性，则会判断是否有 `template`属性，并将template解析成一个 `render function`

---

这里插播一下 render function 相关的科普

```js
render(createElement) {
  /*
   * param {String} tagName
   * param {Object} element properties
   * param {String} text node content
   */

  return createElement('div', {}, this.text)
}
```

我们经常看到的 h 的写法，其实就是代表 createElement 的一个实参而已。 可传入三个参数， 参考代码方法注释。其中，如果要渲染更多子节点的话，可在 properties 中继续传入 createElement。

最终我们的template将被解析为 render 函数所返回的内容，也就是 Virtial DOM。关于Virtial DOM相关的下次会在了解后再写。

---

回到生命周期的话题中，render 函数发生在 `beforeMount` 和 `mounted`之间的，这也侧面说明在`beforeMount`时，$el还只是我们在HTML里面写的节点。（可自行 在生命周期函数中添加 console.log(this.$el) ） `beforeMount` 和 `render`都打印了节点内容，到`mounted`时已被渲染出来并将之挂载到了DOM节点上。在此间经历的过程其实是执行了 render funcion的内容。

在使用.vue文件开发的过程当中，我们在里面写了template模板，在经过了vue-loader的处理之后，就变成了render function，最终放到了vue-loader解析过的文件里面。这样做有什么好处呢？原因是由于在解析template变成render function的过程，是一个非常耗时的过程，vue-loader帮我们处理了这些内容之后，当我们在页面上执行vue代码的时候，效率会变得更高。

`beforeMount`在有了render function的时候才会执行，当执行完render function之后，就会调用`mounted`这个钩子，在`mounted`挂载完毕之后，这个实例就算是走完流程了。

后续的钩子函数执行的过程都是需要外部的触发才会执行。比如说有数据的变化，会调用beforeUpdate，然后经过Virtual DOM，最后updated更新完毕。当组件被销毁的时候，它会调用beforeDestory，以及destoryed。

**这就是vue实例从新建到销毁的一个完整流程**

---

说点题外话：

钩子函数是什么意思？ 特别是在学习 React生命周期，Vue生命周期这一词频繁出现。那么，看到此相信你大致有感觉了。
没错，钩子函数可理解为回调函数，当代码执行到某处时，检查是否有回调，简单代码如下：

```js
function gouzi1 () {
  console.log('g1')
}
function gouzi2 () {
  console.log('g2')
}

for (let i = 0; i < 10000; i++) {
  if (i === 10) {
    gouzi1()    
  }
  if (i > 100 && i < 105) {
    console.log(i)
  }
  if (i === 1000) {
    gouzi2()
  }
  if(i === 9999) {
    console.log('end')
  }
}
// g1
// 101
// 102
// 103
// 104
// g2
// end
```

当然JS的执行环境不是这么简单的。这里大致了解一下就好，最后的最后，再简单总结收个尾吧

## 小结

Vue的完整生命周期流程：

1. new Vue(options) 时执行 init初始化将options的值绑到Vue原型中
2. 从init起，生命周期已开始，beforeCreate 调用后通过注入等方式拿到值后走到created钩子，data重赋值等操作最早在该钩子中进行
3. 检查实例是否传入el来解析template或是执行vm.$mount，其实质是一个render function，将template写的节点，变成Virtial DOM
4. beforeMount较早于render，一旦进入mounted钩子，虚拟节点将生成真正的DOM节点并被挂载
5. 之后的数据改变 会触发 beforeUpdate 和 updated 钩子
6. 通过路由移入移出组件等可触发 destroy destroyed 钩子

这就是Vue实例的完整声明周期流程！ 完~