---
title: 【TS】装饰器模式与vue-property-decorator
date: 2018-4-26 22:29:03
tags:
  - js
  - ts
categories:
  - js
  - ts
---

在用typescript时，vue-property-decorator 是在 vue-class-component 上增强了更多的结合 Vue 特性的装饰器，新增了这 7 个装饰器：

* @Emit
* @Inject
* @Model
* @Prop
* @Provide
* @Watch
* @Component (从 vue-class-component 继承)

源码也就200来行，于是就有信心来慢慢读了

<!-- more -->

为什么需要vue-class-component？ 在typescript里写vue 每次都需要写很多额外的形式代码：

而装饰器就是解决这些冗余代码的（实质上并没有减少，只是用一层函数包装了，后面有源码会讲解）

> 可自行了解一下[装饰模式](http://blog.csdn.net/zhshulin/article/details/38665187)

## AOP 面向切面编程

示例：

* 首先创建一个普通的Man类，它的抵御值 2，攻击力为 3，血量为 3；
* 然后我们让其带上钢铁侠的盔甲，这样他的抵御力增加 100，变成 102；
* 让其带上光束手套，攻击力增加 50，变成 53；
* 最后让他增加“飞行”能力

```js
class Man {
  constructor(def = 2, atk = 3, hp = 3) {
    this.init(def, atk, hp)
  }

  init(def, atk, hp) {
    this.def = def //防御
    this.atk = atk // 攻击
    this.hp = hp
  }

  toString() {
    return `防御力: ${this.def}，攻击力: ${this.atk}，血量：${this.hp}`
  }
}

var tony = new Man()
console.log(`当前状态 ===> ${tony}`)
// 输出：当前状态 ===> 防御力:2,攻击力:3,血量:3
```

然后 创建 decorateArmour 方法，为钢铁侠装配盔甲——注意 decorateArmour 是装饰在方法init上的

```js
function decorateArmour(target, key, descriptor) {
  const method = descriptor.value
  let moreDef = 100
  let ret
  descriptor.value = (...args) => {
    args[0] += moreDef
    ret = method.apply(target, args)
    return ret
  }
  return descriptor
}

class Man {
  constructor(def = 2, atk = 3, hp = 3) {
    this.init(def, atk, hp)
  }

  @decorateArmour
  init(def, atk, hp) {
    this.def = def
    this.atk = atk
    this.hp = hp
  }

  toString() {
    return `防御力:${this.def},攻击力:${this.atk},血量:${this.hp}`
  }
}

var tony = new Man();
console.log(`当前状态 ===> ${tony}`);
// 输出：当前状态 ===> 防御力:102,攻击力:3,血量:3
```

Decorators 的本质是利用了 ES5 的 Object.defineProperty 属性，这三个参数其实是和 Object.defineProperty 参数一致的，因此不能更改，详细分析请见 [细说 ES7 JavaScript Decorators](http://greengerong.com/blog/2015/09/24/es7-javascript-decorators/)

同样的代码复制一份，增加攻击力：

```js
function decorateLight(target, key, descriptor) {
  const method = descriptor.value;
  let moreAtk = 50;
  let ret;
  descriptor.value = (...args)=>{
    args[1] += moreAtk;
    ret = method.apply(target, args);
    return ret;
  }
  return descriptor;
}

class Man{
  constructor(def = 2,atk = 3,hp = 3){
    this.init(def,atk,hp);
  }

  @decorateArmour
  @decorateLight
  init(def,atk,hp){
    this.def = def; // 防御值
    this.atk = atk;  // 攻击力
    this.hp = hp;  // 血量
  }
...
}
var tony = new Man();
console.log(`当前状态 ===> ${tony}`);
//输出：当前状态 ===> 防御力:102,攻击力:53,血量:3
```

---

按装饰模式所言，装饰模式有：纯粹装饰模式和半透明装饰模式

上面两个属于纯粹装饰模式，它不增加对原有类的接口。而下面给普通人增加飞行能力，给类增加新方法，属于半透明的装饰模式，类似适配器模式：

1. 增加一个方法

```js
function addFly(canFly) {
  return function(target) {
    target.canFly = canFly
    let extra = canFly ? '(技能加成：飞行能力)' : ''
    let method = target.prototype.toString

    target.prototype.toString = (...args) => {
      return method.apply(target.prototype, args) + extra
    }
    return target
  }
}
```

2. 用这个方法去直接装饰类

```js
function addFly(canFly) {
  // 接上， 省略了
}
@addFly(true)
class Man {
  constructor(def = 2, atk = 3, hp = 3) {
    this.init(def, atk, hp)
  }
  @decorateArmour
  @decorateLight
  init(def, atk, hp) {
    this.def = def
    this.atk = atk
    this.hp = hp
  }
  // 省略
}
console.log(`当前状态 ===> ${tony}`);
// 输出：当前状态 ===> 防御力:102,攻击力:53,血量:3(技能加成:飞行能力)
```

作用在方法上的decorator接收第一个参数(target)是类的prototype；如果把一个decorator作用到类上，则它的第一个参数target是类本身

## 经典实现 Logger

有了上面的基础，下面我们来写一个简易版`日志系统`

```js
let log = type => (target, name, descriptor) => {
  const method = descriptor.value
  descriptor.value = (...args) => {
    console.info(`(${type} 正在执行：${name}(${args}) = ?`)
    let ret
    try {
      ret = method.apply(target, args)
      console.info(`(${tytpe} 成功：${name}(${args}) => ${ret})`)
    } catch (error) {
      console.error(`(${tytpe} 失败：${name}(${args}) => ${error})`)
    }
    return ret
  }
}

class IronMan {
  @log('IronMan 自检阶段')
  check() {
    return '检查完毕'
  }

  @log('IronMan 攻击阶段')
  attack() {
    return '击倒敌人'
  }

  @log('IronMan 机体报错')
  error() {
    throw 'something is wrong!'
  }
}

var tony = new IronMan()
tony.check();
tony.attack();
tony.error();
// 输出：
// (IronMan 自检阶段) 正在执行: check() = ?
// (IronMan 自检阶段) 成功 : check() => 检查完毕
// (IronMan 攻击阶段) 正在执行: attack() = ?
// (IronMan 攻击阶段) 成功 : attack() => 击倒敌人
// (IronMan 机体报错) 正在执行: error() = ?
// (IronMan 机体报错) 失败: error() => Something is wrong!
```

Logger方法的关键在于：

* 首先使用 `const method = descriptor.value` 将原有的方法提取出来，保障原有方法的纯净
* 在`try catch`语句是调用 `ret = method.apply(target, args)` 在调用之前之后分别进行日志汇报
* 最后返回 `return ret` 原始的调用结果

### vue-property-decorator

vue-property-decorator 是在 vue-class-component 上增强了更多的结合 Vue 特性的装饰器，新增了这 7 个装饰器：

* @Emit
* @Inject
* @Model
* @Prop
* @Provide
* @Watch
* @Component (从 vue-class-component 继承)

我们来读读源码上是怎样来实现的吧：

```ts
import Vue, {PropOptions, WatchOptions} from 'vue'
import Component, {createDecorator} from 'vue-class-component'
import 'reflect-metadata'

export type Constructor = {
  new(...args: any[]): any
}
// 我们在 vue-property-decorator 可调用 Vue 和 Component 是这样继承下来的
export { Component, Vue }

/**
 * decorator of an inject
 * @param key key
 * @return PropertyDecorator
 */
export function Inject(key?: string | symbol): PropertyDecorator {
  return createDecorator((componentOptions, k) => {
    if (typeof componentOptions.inject === 'undefined') {
      // inject默认为 空对象
      componentOptions.inject = {}
    }
    if (!Array.isArray(componentOptions.inject)) {
      // 第二个参数 就是我们对 inject的注入
      componentOptions.inject[k] = key || k
    }
  })
}

/**
 * decorator of an provide
 * @param key key
 * @return PropertyDecorator
 */
export function Provide(key?: string | symbol): PropertyDecorator {
  return createDecorator((componentOptions, k) => {
    let provide: any = componentOptions.provide
    if (typeof provide !== 'function' || !provide.managed) {
      const original = componentOptions.provide
      provide = componentOptions.provide = function(this: any) {
        let rv = Object.create((typeof original === 'function' ? original.call(this) : original) || null)
        for (let i in provide.managed) rv[provide.managed[i]] = this[i]
        return rv
      }
      provide.managed = {}
    }
    provide.managed[k] = key || k
  })
}

/**
 * decorator of model
 * @param  event event name
 * @return PropertyDecorator
 */
export function Model(event?: string, options: (PropOptions | Constructor[] | Constructor) = {}): PropertyDecorator {
  return function (target: Vue, key: string) {
    if (!Array.isArray(options) && typeof (options as PropOptions).type === 'undefined') {
      (options as PropOptions).type = Reflect.getMetadata('design:type', target, key)
    }
    createDecorator((componentOptions, k) => {
      (componentOptions.props || (componentOptions.props = {}) as any)[k] = options
      componentOptions.model = { prop: k, event: event || k }
    })(target, key)
  }
}

/**
 * decorator of a prop
 * @param  options the options for the prop
 * @return PropertyDecorator | void
 */
export function Prop(options: (PropOptions | Constructor[] | Constructor) = {}): PropertyDecorator {
  return function (target: Vue, key: string) {
    if (!Array.isArray(options) && typeof (options as PropOptions).type === 'undefined') {
      (options as PropOptions).type = Reflect.getMetadata('design:type', target, key)
    }
    createDecorator((componentOptions, k) => {
      (componentOptions.props || (componentOptions.props = {}) as any)[k] = options
    })(target, key)
  }
}

/**
 * decorator of a watch function
 * @param  path the path or the expression to observe
 * @param  WatchOption
 * @return MethodDecorator
 */
export function Watch(path: string, options: WatchOptions = {}): MethodDecorator {
  const { deep = false, immediate = false } = options

  return createDecorator((componentOptions, handler) => {
    if (typeof componentOptions.watch !== 'object') {
      componentOptions.watch = Object.create(null)
    }
    (componentOptions.watch as any)[path] = { handler, deep, immediate }
  })
}

// Code copied from Vue/src/shared/util.js
const hyphenateRE = /\B([A-Z])/g
const hyphenate = (str: string) => str.replace(hyphenateRE, '-$1').toLowerCase()

/**
 * decorator of an event-emitter function
 * @param  event The name of the event
 * @return MethodDecorator
 */
export function Emit(event?: string): MethodDecorator {
  return function (target: Vue, key: string, descriptor: any) {
    key = hyphenate(key)
    const original = descriptor.value
    descriptor.value = function emitter(...args: any[]) {
      if (original.apply(this, args) !== false)
        this.$emit(event || key, ...args)
    }
  }
}

/**
 * decorator of $off
 * @param event The name of the event
 * @param method The name of the method
 */
export function Off(event?: string, method?: string): MethodDecorator {
  return function (target: Vue, key: string, descriptor: any) {
    key = hyphenate(key)
    const original = descriptor.value
    descriptor.value = function offer(...args: any[]) {
      if (original.apply(this, args) !== false) {
        if (method) {
          if (typeof this[method] === 'function') {
            this.$off(event || key, this[method])
          } else {
            throw new TypeError('must be a method name')
          }
        } else if (event) {
          this.$off(event || key)
        } else {
          this.$off()
        }
      }
    }
  }
}

/**
 * decorator of $on
 * @param event The name of the event
 */
export function On(event?: string): MethodDecorator {
  return createDecorator((componentOptions, k) => {
    const key = hyphenate(k)
    if (typeof componentOptions.created !== 'function') {
      componentOptions.created = function () { }
    }
    const original = componentOptions.created
    componentOptions.created = function () {
      original()
      if (typeof componentOptions.methods !== 'undefined') {
        this.$on(event || key, componentOptions.methods[k])
      }

    }
  })
}

/**
 * decorator of $once
 * @param event The name of the event
 */
export function Once(event?: string): MethodDecorator {
  return createDecorator((componentOptions, k) => {
    const key = hyphenate(k)
    if (typeof componentOptions.created !== 'function') {
      componentOptions.created = function () { }
    }
    const original = componentOptions.created
    componentOptions.created = function () {
      original()
      if (typeof componentOptions.methods !== 'undefined') {
        this.$once(event || key, componentOptions.methods[k]);
      }
    }
  })
}

/**
 * decorator of $nextTick
 *
 * @export
 * @param {string} method
 * @returns {MethodDecorator}
 */
export function NextTick(method: string): MethodDecorator {
  return function (target: Vue, key: string, descriptor: any) {
    const original = descriptor.value
    descriptor.value = function emitter(...args: any[]) {
      if (original.apply(this, args) !== false)
        if (typeof this[method] === 'function') {
          this.$nextTick(this[method])
        } else {
          throw new TypeError('must be a method name')
        }
    }
  }
}
```
