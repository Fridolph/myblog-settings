---
title: 【Vue】实现原生类Vue的双向绑定
date: 2018-4-11 23:49:03
tags: 
  - vue
  - mvvm
categories: 
  - vue
---


逛掘金看到篇讲[Vue双向绑定的文章](https://juejin.im/post/5acc17cb51882555745a03f8)，很不错，就拿过来了，里面的代码抄了一遍，当然不是初抄啦，加入我自己的理解和总结。虽然看了几次这类文章，但真不嫌多，温故而知新嘛。

<!-- more -->

## 原理

Vue的双向绑定通过 Object对象的defineProperty属性，重写data的set和get函数来实现。

<img src="https://user-gold-cdn.xitu.io/2018/4/10/162ad3d5be3e5105?imageView2/0/w/1280/h/960/format/webp/ignore-error/1">

简单实现

```html
<div id="app">
  <form>
    <input type="text"  v-model="number">
    <button type="button" v-click="increment">增加</button>
  </form>
  <h3 v-bind="number"></h3>
</div>
```

1. 一个input，使用v-model指令
2. 一个button，使用v-click指令
3. 一个h3，使用v-bind指令。

通过类似于vue的方式来使用我们的双向数据绑定，结合我们的数据结构添加注释

```js
var app = new MyVue({
  el: '#app',
  data: {
    number: 0
  },
  methods: {
    increment() {
      this.number++
    }
  }
})
```

首先定义一个MyVue构造函数

```js
function MyVue(options) {}
```

为了初始化这个构造函数，给它添加一 个_init属性

```js
function MyVue(options) {
  this._init(options)
}
MyVue.prototype._init = function(options) {
  this.$options = options // options为上面使用时传入的结构体，包括el data methods
  this.$el = document.querySelector(options.el) // this.$el 即 id为app的 dom元素
  this.$data = options.data // this.$data = {number: 0}
  this.$methods = options.methods // { increment() {this.number++} }
}
```

接下来实现_observe函数，对data进行处理，重写data的set和get函数，并改造 _init函数

```js
MyVue.prototype._observe = function(obj) { // 将要实现监听的对象 obj = {number: 0}
  var value
  for (key in obj) {
    if (obj.hasOwnProperty(key)) {
      value = obj[key]
      if (typeof value === 'object') { // 若值还是 object类型，则继续遍历
        this._observe(value)
      }
      Object.defineProperty(this.$data, key, { // 关键
        enumerable: true,
        configurable: true,
        get: function() {
          console.log(`获取${value}`)
          return value
        },
        set: function(newVal) {
          console.log(`更新${newVal}`)
          if (value !== newValue) {
            value = newVal
          }
        }
      })
    }
  }
}

MyVue.prototype._init = function(options) {
  this.$options = options
  this.$el = document.querySelector(options.el)
  this.$data = options.data
  this.$methods = options.methods
  
  this._observe(this.$data)
}
```

接下来我们写一个指令类 Watcher 用来绑定更新函数，实现对DOM元素的更新

```js
function Watcher(name, el, vm, exp, attr) {
  this.name = name  // 指令名称，例如文件节点，该值设为 text
  this.el = el      // 指令对应的DOM元素
  this.vm = vm      // 指令所属的MyVue实例
  this.exp = exp    // 指令对应的值，本例为 number
  this.attr = attr  // 绑定的属性值，本例为 innerHTML

  this._update()
}

Watcher.prototype._update = function() {
  this.el[this.attr] = this.vm.$data[this.exp]
  // 比如 h3.innerHTML = this.data.number
  // 当number改变时，会触发_update函数，保证对应的DOM进行更新
}
``` 

更新_init函数以及observe函数

```js
MyVue.ptototype._init = function(options) {
  /* 省略 */
  this._binding = {}
  // _binding保存着model与view的映射关系, 也就是我们前面定义的Watcher的实例
  // 当model改变时，我们会触发其中的指令类更新，保证view也能实时更新
  /* ... */ 
}

MyVue.prototype._observe = function(obj) {
  /* 省略 */
  if (obj.hasOwnProperty(key)) {
    this._binding[key] = {
      _directives: []
    }
    // ...
    var binding = this._binding[key]
    Object.defineProperty(this.$data, key, {
      // ...
      set: function(newVal) {
        console.log(`更新${newVal}`)
        if (value !== newVal) {
          value = newVal
          binding._directives.forEach(function(item) {
            // 当number改变时，触发_binding[number]._directives 中绑定的Watcher类的更新
            item._update()
          })
        }
      }
    })
  }
  /* ... */
}
```

那么如何将 view 与 model进行绑定呢？ 接下来我们定义一个_compiler函数，用来解析我们的指令（v-bind, v-model, v-click）等，并在这个过程中与view与model进行绑定

```js
MyVue.prototype._init = function(options) {
  // 省略
  this._compile(this.$el)
}

MyVue.prototype._compile = function(root) { // root为 id为app的element，即根元素
  var _this = this
  var nodes = root.children
  for (var i = 0; i < nodes.length; i++) {
    var node = nodes[i]
    if (node.chilren.length) { // 若存在子元素则进行递归处理
      this._compile(node)
    }
    // 如果有v-click属性，我们监听它的onclick事件，触发increment事件，即number++
    if (node.hasAttribute('v-click')) {
      node.onclick = (function() {
        var attrVal = nodes[i].getAttribute('v-click')
        return _this.$methods[attrVal].bind(_this.$data)
        // bind是使data的作用域与method函数的作用域保持一致
      })()
    }
    // 如果有v-model属性，且元素是input或textarea，就监听它的input事件
    if (node.hasAttribute('v-model') && (node.tagName === 'input' || node.target === 'textarea')) {
      node.addEventListener('input', (function(key) {
        var attrVal = node.getAttribute('v-model')
        // _this._binding[number]._directives = [一个Watcher的实例]
        // 其中Watcher.prototype.update = functoin() {
        //   node['value'] = _this.$data['number']
        //   这就将node的值保持与number一致了
        // }
        _this._bniding[attrVal]._directives.push(new Watcher(
          'input',  // 结合上面的 Watcher构造函数来看 
          node,     // 分别传参 指令名, dom
          _this,    // Vue绑定的实例
          attrVal,  // 指令对应值
          'value'   // 绑定的对应属性
        ))

        return function() {
          _this.$data[attrVal] = nodes[key].value
          // 使number的值与node的value保持一致，这就实现了双向绑定
        }
      })(i))
    }
    // 如果有v-bind属性，我们只要使node的值及时更新为data中的number值即可
    if (node.hasAttribute('v-bind')) {
      var attrVal = node.getAttribute('v-bind')
      _this._binding[attrVal]._directives.push(new Watcher(
        'text',
        node,
        _this,
        attrVal,
        'innerHTML'
      ))
    }
  }
}
```

## 附上完整代码

```html
<div id="app">
  <form>
    <input type="text"  v-model="number">
    <button type="button" v-click="increment">增加</button>
  </form>
  <h3 v-bind="number"></h3>
</div>
<script>
  function MyVue(options) {
    this._init(options)
  }

  MyVue.prototype._init = function(options) {
    this.$options = options
    this.$el = document.querySelector(options.el)
    this.$data = options.data
    this.$methods = options.methods

    this._binding = {}
    this._observe(this.$data)
    this._compile(this.$el)
  }

  MyVue.prototype._observe = function(obj) {
    var value
    for (key in obj) {
      if (obj.hasOwnproperty(key)) {
        this._binding[key] = {
          _directives: []
        }
        value = obj[key]
        if (typeof value === 'object') {
          this._observe(value)
        }
        var binding = this._binding[key]
        Object.defineProperty(this.$data, key, {
          enumerable: true,
          configurable: true,
          get() {
            console.log(`获取${value}`)
            return value
          },
          set(newVal) {
            console.log(`更新${newVal}`)
            if (value !== newVal) {
              value = newVal
              binding._directives.forEach(function(item) {
                item.update()
              })
            }
          }
        })
      }
    }
  }

  MyVue.prototype._compile = function(root) {
    var _this = this
    var nodes = root.children
    for (var i = 0; i < nodes.length; i++) {
      var node = nodes[i]
      if (node.children.length) {
        this._compile(node)
      }

      if (node.hasAttribute('v-click')) {
        node.onclick = (function() {
          var attrVal = nodes[i].getAttribute('v-click')
          return _this.$methods[attrVal].bind(_this.$data)
        })()
      }

      if (node.hasAttribute('v-model') && (node.tagName === 'input' || node.tagName === 'textarea')) {
        node.addEventListener('input', (function(key) {
          var attrVal = node.getAttribute('v-model')
          _this._binding[attrVal]._directives.push(new Watcher(
            'input',
            node,
            _this,
            attrVal,
            'value'
          ))

          return function() {
            _this.$data[attrVal] = nodes[key].value
          }
        })(i))
      }

      if (node.hasAttribute('v-bind')) {
        var attrVal = node.getAttribute('v-bind')
        _this._binding[attrVal]._directives.push(new Watcher(
          'text',
          node,
          _this,
          attrVal,
          'innerHTML'
        ))
      }
    }
  }

  function Watcher(name, el, vm, exp, attr) {
    this.name = name    //指令名称，例如文本节点，该值设为"text"
    this.el = el        //指令对应的DOM元素
    this.vm = vm        //指令所属myVue实例
    this.exp = exp      //指令对应的值，本例如"number"
    this.attr = attr    //绑定的属性值，本例为"innerHTML"

    this.update()
  }

  Wathcer.prototype.update = function() {
    this.el[this.attr] = this.vm.$data[this.exp]
  }

  window.onload = function() {
    var app = new MyVue({
      el: '#app',
      data: {
        number: 0
      },
      methods: {
        increment: function() {
          this.number++
        }
      }
    })
  }
</script>
```

## 总结

最后来梳理一下逻辑：

1. 写一个构造函数 MyVue 可传入一个options参数，实例化时执行其 _init方法
2. _init 其核心是将 options的各种属性 放到其实例上，并执行 _observe 和 _compile 方法
3. _observe 作为一个代理方法，监听传入的options.data属性的改变，（这里会将data里的每个key放入 _bingding 对象中，然后用Object.defineProperty对每一个key进行监听, 若属性值改变了，就实时改变）
4. 上面说的实时改变就是 update 方法，那观察的对象从哪来？ 指令上对应的值？ 这一过程其实就是 _compile做的. 让 MyVue能知道 el 上的指令代表什么
5. _compile让我们知道了指令是干嘛的，具体触发改变就是由Watcher来做的了，我们传入name(指令名), el(对应dom), vm(MyVue实例), exp(指令对应值), attr(绑定的属性)，然后调用其原型上的update方法。 触发了 _observe其最终是执行 update方法来更改值

说得好像很啰嗦，但这就是双向绑定的过程了，知根知底，这样又往前进了一小步咯。以上就实现了文本与input的双向绑定

---

4.17更新~ 添加完整可运行的 es6版

理下思路：

1. MyVue实例时将传入的options的给实例，完成初始化
2. _binding 进行依赖收集，每次设置会触发 Watcher 实例的update
3. _observe 监听data数据，实现数据响应式化
4. _compile 将模版编译为抽象语法树AST

```js
class MyVue {
  constructor(options) {
    this.$options = options
    this.$el = document.querySelector(options.el)
    this.$data = options.data
    this.$methods = options.methods

    this._binding = {}           // 依赖收集
    this._observe(this.$data)    // 观察data数据添加到Watcher中
    this._compile(this.$el)      // 编译为抽象语法树AST 这里要简单得多
  }

  _observe(obj) {    
    for (let key in obj) {
      if(obj.hasOwnProperty(key)) {
        this._binding[key] = {
          _directives: []
        }
        console.log('this._binding[key]', this._binding[key])
        let value = obj[key]
        if (typeof value === 'object') {
          this._observe(value)
        }

        let binding = this._binding[key]
        Object.defineProperty(this.$data, key, {
          enumerable: true,
          configurable: true,
          get() {
            console.log(`${key}获取${value}`)
            return value
          },
          set(newVal) {
            console.log(`${key}设置${newVal}`)
            if (value !== newVal) {
              value = newVal
              binding._directives.forEach(item => item.update())
            }
          }
        })
      }
    }
  }

  _compile(root) {
    // root为根节点，传入的el
    let _this = this
    let nodes = root.children
    for (let i = 0; i < nodes.length; i++) {
      let node = nodes[i]
      if (node.children.length) {
        this._compile(node)
      }

      if (node.hasAttribute('v-click')) {
        node.onclick = (function() {
          let attrVal = nodes[i].getAttribute('v-click')
          return _this.$methods[attrVal].bind(_this.$data)
        })()
      }

      if (node.hasAttribute('v-model') && (node.tagName === 'INPUT' || node.tagName === 'TEXTAREA')) {
        node.addEventListener('input', (function(key) {
          let attrVal = nodes[i].getAttribute('v-model')
          _this._binding[attrVal]._directives.push(new Watcher(
            'input',
            node,
            _this,
            attrVal,
            'value'
          ))

          return function() {
            _this.$data[attrVal] = nodes[key].value
          }
        })(i))
      }

      if (node.hasAttribute('v-bind')) {
        let attrVal = nodes[i].getAttribute('v-bind')
        _this._binding[attrVal]._directives.push(new Watcher(
          'text',
          node,
          _this,
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
    this.vm = vm        // 指令所属实例
    this.exp = exp      // 指令对应值
    this.attr = attr    // 绑定属性值

    this.update()
  }

  update() {
    this.el[this.attr] = this.vm.$data[this.exp]
  }
}
```