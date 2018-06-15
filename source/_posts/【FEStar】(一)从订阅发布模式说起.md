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

### 实践应用 - 实现Vue的双向绑定

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

这里说一下，我们通过 new 生成了一个 MyVue实例，而传入的参数（对象）作为配置，在初始化阶段做了这些事情：

1. 赋值，实例的 $options $el $data $method 都是从options里获取到的
2. _observe 为每一个实例上的data属性添加监听 （通过Object.defineProperty）
3. _compile 编译节点，指令对应的value对应一个新的Watcher，进行依赖收集从而触发数据响应

于是到codepen上试试吧

<iframe height='300' scrolling='no' title='simple Vue ~ MVVM' src='//codepen.io/fridolph/embed/gKxrBO/?height=300&theme-id=dark&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/fridolph/pen/gKxrBO/'>simple Vue ~ MVVM</a> by fridolph (<a href='https://codepen.io/fridolph'>@fridolph</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

---

## event.js 的实现

这是一个典型的发布订阅模式的应用

```js
// 定义R对象 如果支持es6用原生的Reflect，没就相当于提供polyfill
'use strict';
var R = typeof Reflect === 'object' ? Reflect : null
var ReflectApply = R && typeof R.apply === 'function'
  ? R.apply
  : function ReflectApply(target, receiver, args) {
    return Function.prototype.apply.call(target, receiver, args);
  }
var ReflectOwnKeys
if (R && typeof R.ownKeys === 'function') {
  ReflectOwnKeys = R.ownKeys
} else if (Object.getOwnPropertySymbols) {
  ReflectOwnKeys = function ReflectOwnKeys(target) {
    return Object.getOwnPropertyNames(target)
      .concat(Object.getOwnPropertySymbols(target));
  };
} else {
  ReflectOwnKeys = function ReflectOwnKeys(target) {
    return Object.getOwnPropertyNames(target);
  };
}

// console.warn的封装
function ProcessEmitWarning(warning) {
  if (console && console.warn) console.warn(warning);
}

// Number.isNaN 是es6加入的，后面是并联的兼容处理，Number类型 自身与自身不相等的只有NaN了
var NumberIsNaN = Number.isNaN || function NumberIsNaN(value) {
  return value !== value;
}

function EventEmitter() {
  EventEmitter.init.call(this);
}
// 将EventEmitter构造函数作为默认导出
module.exports = EventEmitter;
// 向后兼容node 0.10.x 版本 看来这个库相当老了啊，不过也不妨碍我们学习

EventEmitter.EventEmitter = EventEmitter;
EventEmitter.prototype._events = undefined;
EventEmitter.prototype._eventsCount = 0;
EventEmitter.prototype._maxListeners = undefined;

// 默认情况下，如果超过10个侦听器，EventEmitters将会输出警告添加到它
// 这是一个有用的默认值，它有助于查找内存泄漏
var defaultMaxListeners = 10;
Object.defineProperty(EventEmitter, 'defaultMaxListeners', {
  enumerable: true,
  get: function() {
    return defaultMaxListeners;
  },
  set: function(arg) {
    if (typeof arg !== 'number' || arg < 0 || NumberIsNaN(arg)) {
      throw new RangeError('The value of "defaultMaxListeners" is out of range. It must be a non-negative number. Received ' + arg + '.');
    }
    defaultMaxListeners = arg;
  }
});

// 构造类的静态方法
EventEmitter.init = function() {
  // 相当于constructor的构造函数调用 初始化
  if (this._events === undefined ||
      this._events === Object.getPrototypeOf(this)._events) {
    this._events = Object.create(null);
    this._eventsCount = 0;
  }
  this._maxListeners = this._maxListeners || undefined;
};

// 显然不是所有的Emitters侦听器都应该限制为10个。这个功能允许增加
EventEmitter.prototype.setMaxListeners = function setMaxListeners(n) {
  if (typeof n !== 'number' || n < 0 || NumberIsNaN(n)) {
    throw new RangeError('The value of "n" is out of range. It must be a non-negative number. Received ' + n + '.');
  }
  this._maxListeners = n;
  return this;
};

// 给下面的 getMaxListeners方法用  EventEmitter原型方法里的 this指向实例对象
// 所以可从实例中 拿到 属性值 _maxListeners
function $getMaxListeners(that) {
  if (that._maxListeners === undefined)
    return EventEmitter.defaultMaxListeners;
  return that._maxListeners;
}
EventEmitter.prototype.getMaxListeners = function getMaxListeners() {
  return $getMaxListeners(this);
};

EventEmitter.prototype.emit = function emit(type) {
  var args = [];
  // emit第一个参数一般是方法名，所以 从arguments[1]开始
  for (var i = 1; i < arguments.length; i++) args.push(arguments[i]);
  // 右边的表达式返回一个布尔值 ，type报错就为true。doError可理解为错误标记的flag
  var doError = (type === 'error');

  // 拿到 实例的 _events 对象
  var events = this._events;
  if (events !== undefined)
    // 没有错就直接返回 false
    doError = (doError && events.error === undefined);
  else if (!doError)
    return false;

  // 下面是错误处理
  if (doError) {
    var er;
    if (args.length > 0)
      er = args[0];
    if (er instanceof Error) {
      // 注意：“throw”这一行的注释是有意的，他们表示 如果这导致未处理的异常，则在Node的输出中输入。
      throw er;
    }
    // 错误上下文为er 这里的er就是传进来的type的报错环境上下文
    var err = new Error('Unhandled error.' + (er ? ' (' + er.message + ')' : ''));
    err.context = er;
    throw err;
  }

  // 从events里找type对应的函数 拿到 并赋给handler
  var handler = events[type];
  if (handler === undefined)
    return false;
  if (typeof handler === 'function') {
    // 绑定上下文
    ReflectApply(handler, this, args);
  } else {
    // handler多次调用  把多个handler 放到数组里来处理
    var len = handler.length;
    var listeners = arrayClone(handler, len);
    for (var i = 0; i < len; ++i)
      ReflectApply(listeners[i], this, args);
  }
  return true;
};

function _addListener(target, type, listener, prepend) {
  var m;
  var events;
  var existing;
  // listener类型判断
  if (typeof listener !== 'function') {
    throw new TypeError('The "listener" argument must be of type Function. Received type ' + typeof listener);
  }
  events = target._events;
  if (events === undefined) {
    events = target._events = Object.create(null);
    target._eventsCount = 0;
  } else {
    // 为了避免在 type === "newListener" 这种情况下递归
    // 在将其添加到侦听器之前，首先触发 "newListener"
    if (events.newListener !== undefined) {
      target.emit('newListener', type,
                  listener.listener ? listener.listener : listener);
      // 重新分配 `events`，因为newListener处理程序可能导致了 this._events被分配给一个新的对象
      events = target._events;
    }
    existing = events[type];
  }
  if (existing === undefined) {
    // 优化一个listener的情况。不需要额外的数组对象
    existing = events[type] = listener;
    ++target._eventsCount;
  } else {
    if (typeof existing === 'function') {
      // 添加第二个元素，需要更改为数组
      existing = events[type] =
        prepend ? [listener, existing] : [existing, listener];
      //如果我们已经有了一个数组，只需追加
    } else if (prepend) {
      existing.unshift(listener);
    } else {
      existing.push(listener);
    }
    // 检查监听器泄漏
    m = $getMaxListeners(target);
    // m > 0 && (exiting.length > m && !existing.warned )
    if (m > 0 && existing.length > m && !existing.warned) {
      existing.warned = true;
      // 没有错误代码，因为它是一个warning
      var w = new Error(
        'Possible EventEmitter memory leak detected. ' +
        existing.length + ' ' + String(type) + ' listeners ' +
        'added. Use emitter.setMaxListeners() to ' +
        'increase limit'
      )
      w.name = 'MaxListenersExceededWarning';
      w.emitter = target;
      w.type = type;
      w.count = existing.length;
      ProcessEmitWarning(w);
    }
  }
  return target;
}

// 结合上面的函数，这是高阶函数的用法 普通地，我们一般会传 事件名和回调
// 这里返回 _addListner方法 第一个参数是 this 实例对象作为上下文环境
// 然后是 事件名，回调，prepend默认为false
EventEmitter.prototype.addListener = function addListener(type, listener) {
  return _addListener(this, type, listener, false);
};
// 同名方法 addListner 和用 on等效
EventEmitter.prototype.on = EventEmitter.prototype.addListener;

// 和on的区别在于 prepend为true，当前已经是数组了才会这样用（不同我们来，已经封装在内部逻辑里了）
EventEmitter.prototype.prependListener = function prependListener(type, listener) {
  return _addListener(this, type, listener, true);
};

function onceWrapper() {
  // 把参数放到 args中。 我们先看下面的 _onceWrap方法
  var args = [];
  for (var i = 0; i < arguments.length; i++) args.push(arguments[i]);
  // 显然，这个if逻辑会走到里
  if (!this.fired) {
    // removeListener会被调用
    this.target.removeListener(this.type, this.wrapFn);
    // 然后再把fired 设回true
    this.fired = true;
    // 这里因为ReflectApply的原因，this上下文回到了 实例对象上
    ReflectApply(this.listener, this.target, args);
  }
}
function _onceWrap(target, type, listener) {
  // 初始state
  var state = { fired: false, wrapFn: undefined, target: target, type: type, listener: listener };
  // 好 有了这个bind后，上面onceWrapper的this就知道了，我们再跳回去
  var wrapped = onceWrapper.bind(state);
  wrapped.listener = listener;
  state.wrapFn = wrapped;
  return wrapped;
  // 回到wrapped这个上下文中
}

// 该方法可理解为调用一次后 自动把方法注销掉
EventEmitter.prototype.once = function once(type, listener) {
  if (typeof listener !== 'function') {
    throw new TypeError('The "listener" argument must be of type Function. Received type ' + typeof listener);
  }
  this.on(type, _onceWrap(this, type, listener));
  // 后同，return this 是返回实例对象自身，方便链式调用
  return this;
};

EventEmitter.prototype.prependOnceListener = function prependOnceListener(type, listener) {
  if (typeof listener !== 'function') {
    throw new TypeError('The "listener" argument must be of type Function. Received type ' + typeof listener);
  }
  this.prependListener(type, _onceWrap(this, type, listener));
  return this;
};

// 删除监听方法
EventEmitter.prototype.removeListener = function removeListener(type, listener) {
  var list, events, position, i, originalListener;
  // 错误判断，这里先略过了
  if (typeof listener !== 'function') {
    throw new TypeError('The "listener" argument must be of type Function. Received type ' + typeof listener);
  }
  events = this._events;
  if (events === undefined)
    return this;

  // list被赋值为evnets[type] 即 一个注册的事件
  list = events[type];
  if (list === undefined)
    return this;

  // 当前list 和所传的注册事件若一致就进入这个逻辑里
  if (list === listener || list.listener === listener) {
    // 处理 注册事件 为单个函数 的情况
    if (--this._eventsCount === 0)
      this._events = Object.create(null);
    else {
      delete events[type];
      if (events.removeListener)
        this.emit('removeListener', type, list.listener || listener);
    }
  } else if (typeof list !== 'function') {
    // 处理 注册事件 为数组的情况
    position = -1;
    for (i = list.length - 1; i >= 0; i--) {
      if (list[i] === listener || list[i].listener === listener) {
        originalListener = list[i].listener;
        position = i;
        break;
      }
    }
    if (position < 0)
      return this;
    if (position === 0)
      list.shift();
    else {
      spliceOne(list, position);
    }
    if (list.length === 1)
      events[type] = list[0];
    if (events.removeListener !== undefined)
      this.emit('removeListener', type, originalListener || listener);
  }
  return this;
};

// 同名方法，等同于 removeListener
EventEmitter.prototype.off = EventEmitter.prototype.removeListener;

// 删除所有的监听
EventEmitter.prototype.removeAllListeners = function removeAllListeners(type) {
  var listeners, events, i;
  events = this._events;
  if (events === undefined)
    return this;

  // 不侦听removeListener，不需要触发emit
  if (events.removeListener === undefined) {
    if (arguments.length === 0) {
      // 通过赋值的方式 让 实例对象的 _events 为空
      this._events = Object.create(null);
      // 事件数清0
      this._eventsCount = 0;
    } else if (events[type] !== undefined) {
      if (--this._eventsCount === 0)
        this._events = Object.create(null);
      else
        delete events[type];
    }
    return this;
  }

  // 为所有事件上的所有侦听器发出removeListener
  if (arguments.length === 0) {
    var keys = Object.keys(events);
    var key;
    for (i = 0; i < keys.length; ++i) {
      key = keys[i];
      if (key === 'removeListener') continue;
      this.removeAllListeners(key);
    }
    this.removeAllListeners('removeListener');
    this._events = Object.create(null);
    this._eventsCount = 0;
    return this;
  }
  listeners = events[type];
  if (typeof listeners === 'function') {
    this.removeListener(type, listeners);
  } else if (listeners !== undefined) {
    // LIFO order
    for (i = listeners.length - 1; i >= 0; i--) {
      this.removeListener(type, listeners[i]);
    }
  }
  return this;
};

// 同样的，我们先看 原型方法
function _listeners(target, type, unwrap) {
  var events = target._events;
  if (events === undefined)
    return [];
  // 参数type为事件名
  var evlistener = events[type];
  if (evlistener === undefined)
    return [];
  if (typeof evlistener === 'function')
    return unwrap ? [evlistener.listener || evlistener] : [evlistener];
  return unwrap ?
    unwrapListeners(evlistener) : arrayClone(evlistener, evlistener.length);
}

EventEmitter.prototype.listeners = function listeners(type) {
  // 返回 _listeners调用的结果，回到 _listener函数
  return _listeners(this, type, true);
};
EventEmitter.prototype.rawListeners = function rawListeners(type) {
  return _listeners(this, type, false);
};
EventEmitter.listenerCount = function(emitter, type) {
  if (typeof emitter.listenerCount === 'function') {
    return emitter.listenerCount(type);
  } else {
    return listenerCount.call(emitter, type);
  }
};
EventEmitter.prototype.listenerCount = listenerCount;

function listenerCount(type) {
  var events = this._events;
  if (events !== undefined) {
    var evlistener = events[type];
    if (typeof evlistener === 'function') {
      return 1;
    } else if (evlistener !== undefined) {
      return evlistener.length;
    }
  }
  return 0;
}
EventEmitter.prototype.eventNames = function eventNames() {
  return this._eventsCount > 0 ? ReflectOwnKeys(this._events) : [];
};

// 辅助方法，浅拷贝数组
function arrayClone(arr, n) {
  var copy = new Array(n);
  for (var i = 0; i < n; ++i)
    copy[i] = arr[i];
  return copy;
}

// 从数组中删除某项
function spliceOne(list, index) {
  for (; index + 1 < list.length; index++)
    list[index] = list[index + 1];
  list.pop();
}

// listner的拷贝
function unwrapListeners(arr) {
  var ret = new Array(arr.length);
  for (var i = 0; i < ret.length; ++i) {
    ret[i] = arr[i].listener || arr[i];
  }
  return ret;
}
```
