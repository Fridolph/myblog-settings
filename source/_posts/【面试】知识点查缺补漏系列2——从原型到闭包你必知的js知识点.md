---
title: 【面试】知识点查缺补漏系列2——从原型到闭包你必知的js知识点
date: 2017-07-23 10:16:33
tags: 
  - 面试
categories: 
  - 面试
---

记得之前知乎有一个问题很火？ 问的大概是一个五年的程序员居然不知道原型链，空口说会vue react什么的巴拉巴拉… 然后大佬们讨论得很激烈

<a href="https://www.zhihu.com/question/60165921">面试一个5年的前端，却连原型链也搞不清楚，满口都是Vue，React之类的实现，这样的人该用吗？</a>感兴趣可以看看这篇里大牛们的回答

<!-- more -->

以下只是自我感想… 见笑了……题设是觉得一名5年的前端工程师至少从基本功来说应该掌握这些知识，而不仅仅是业务逻辑熟练，会用各种框架。
有一部分认为工作应该服务于需求，拿来主义，满足需求第一，至于细节让他去吧。
还有一部分认为知识的不全面并不足以否定面试者其他方面的优秀……
总之仁者见仁吧，如果对自己的定位是一名资深前端工程师，或是以技术为核心发展方向，（我现在就以此为目标为之努力）那么就不应该放过这些微小的细节。

原型为何而来，解决什么样的问题？从中衍生出了哪些更好的解决方案？ jQuery中的原型等等，就像是问css多少种垂直水平居中一样，绝对定位 margin transform flexbox grid很多种解决方案，适合于某些特定的情形，我们了解了各自的差异，才是从众多解决方案中得到`最适合的解决方案`。与茴字有多少种写法是完全不同的… 于是不多BB了。。。

---

那我们说回来，还是从题目入手、思考，寻找知识点最后解答

## 引子题目

* 如果准确判断一个变量是数组类型
* 写一个原型链继承的例子
* 描述new一个对象的过程
* zepto或其他框架源码中如何使用原型链

## 知识点

### 构造函数

```js
function Foo(name, age) {
  this.name = name;
  this.age = age;
  this.class = 'class-1';
  // return this 默认有这行
}
var f = new Foo('zhangsan', 20)
// var f1 = new Foo('lisi', 22) // 创建多个对象
```

### 构造函数 - 扩展

* `var a = {}`其实是`var a = new Object()`的语法糖
* `var a = []`其实是`var a = new Array()`的语法糖
* `function Foo() {...}`其实是`var Foo = new Function(...)`
* 使用instanceof判断一个函数是否是一个变量的构造函数

### 原型规则和示例

5条原型规则 - 原型规则是学习原型链的基础

* 所有的引用类型（对象、数组、函数）都具有对象特性，即可自由扩展属性(除null)

```js
var obj = {}; obj.a = 100;
var arr = []; arr.a = 100;
function fn() {}; fn.a = 100;
// 
console.log(obj.__proto__);
console.log(arr.__proto__);
console.log(fn.__proto__);
//
console.log(fn.prototype)
console.log(obj.__proto__ === Object.prototype)
```

* 所有的引用类型都有一个`__proto__`(隐式原型)属性，属性值是一个普通的对象

* 所有的函数，都有一个prototype属性，属性值也是一个普通的对象

* 所有的引用类型(数组、对象、函数), __proto__属性值指向它的构造函数的`prototype`属性

* 当试图得到一个对象的某个属性时，如果这个对象本身没有这个属性，那么会去它的__proto__（即它的构造函数的prototype）中寻找

```js
// 构造函数
function Foo(name, age) {
  this.name = name
}
Foo.prototype.alertName = function() {
  alert(this.name)
}
// 创建实例
var f = new Foo('zhangsan')
f.printName = function() {
  console.log(this.name)
}
// 测试
f.printName()
f.alertName()
```

**循环对象自身的属性**

```js
var item
for (item in f) {
  // 高级浏览器已经在for in 中屏蔽了来自原型的属性
  // 但是这里还是建议加上这个判断，保证程序健壮性
  if (f.hasOwnProperty(item)) {
    // console.log(item)
  }
}
```

### 原型链

```js
// 构造函数
function Foo(name, age) {
  this.name = name
}
Foo.prototype.alertName = function() {
  alert(this.name)
}
// 创建实例
var f = new Foo('zhangsan')
f.printName = function() {
  console.log(this.name)
}
// 测试
f.printName()
f.alertName()
f.toString()  // 要去 f.__proto__.__proto__ 中去找
```


### instanceof

用于判断引用类型属于哪个构造函数的方法

* f instanceof Foo 的判断逻辑

* f的__proto__一层一层往上，能否对应到Foo.prototype

* 再试着判断 f instanceof Object

## 解题

### 如果准确判断一个变量是数组类型

```js
var arr = []
arr instanceof Array // true
typeof arr // object, typeof无法判断是否为数组
```

### 写一个原型链继承的例子

```js
// 动物
function Animal() {
  this.eat = function() {
    console.log('animal eat')
  }
}
// 狗
function Dog() {
  this.bark = function() {
    console.log('dog bark')
  }
}
Dog.prototype = new Animal()
// 哈士奇
var hashiqi = new Dog() // 这不是最好的方式
```

### 描述new一个对象的过程

* 创建一个新对象
* this指向这个新对象
* 执行代码，即对this赋值
* 返回this

### zepto或其他框架源码中如何使用原型链

阅读源码是高效提高技能的方式，但不能“埋头苦钻”，有技巧在其中

> 可以在慕课网搜索“zepto设计和源码分析”

**写一个封装DOM查询的例子**

```js
function Elem(id) {
  this.elem = document.getElementById(id)
}
Elem.prototype.html = function(val) {
  var elem = this.elem
  if (val) {
    elem.innerHTML = val
    return this // return this是为了链式调用
  } else {
    return elem.innerHTML
  }
}
Elem.prototype.on = function(type, fn) {
  var elem = this.elem
  elem.addEventListener(type, fn)
}
var div1 = new Elem('div1')
console.log(div1.html())
div1.html('<h1>Hello world!</h1>').on('click', function() {
  alert('html changed...')
})
```

从题目入手、思考，寻找知识点最后解答

* 说一下对变量提升的理解
* 说明this几种不同的使用场景
* 创建10个`<a>`标签，点击时弹出对应的序号
* 如何理解作用域
* 实际开发中闭包的应用

---

### 执行上下文

```js
console.log(a) // undefined
var a = 100
fn('zhangsan') // 'zhangsan' 20
function fn(name) {
  age = 20
  console.log(name, age)
  var age
}
```

范围：一段`<script>`或者一个函数
全局：变量定义、函数声明
函数：变量定义、函数声明、this、arguments

### this

要在执行时才能确认值，定义时无法确认

```js
var a = {
  name: 'A',
  fn: function() {
    console.log(this.name)
  }
}
a.fn()  // 'A'  this === a
a.fn.call({name: 'B'})  // this === {name: 'B'}
var fn1 = a.fn
fn1() // undefined  this === window
```

* 作为构造函数执行
* 作为普通属性执行
* 作为普通函数执行
* call apply bind
* es6箭头函数 

### 作用域

es5及以前是没有块级作用域的（所以我们一般用IIFE来模拟）

### 作用域链

```js
/* 例1 */
var a = 100
function fn() {
  var b = 200
  // 当前作用域没有定义的变量，即“自由变量”
  console.log(a)
  console.log(b)
}
fn()
/* 例2 */
var a = 100
function f1() {
  function f2() {
    var c = 300
    console.log(a) // a是自由变量
    console.log(b) // b是自由变量
    console.log(c)
  }
  f2()
}
f1()
```

作用域链就是 … （我理解的）这层作用域找不到的向父级找，父级找不到继续往上直到顶层作用域（window或global）

### 闭包

**闭包的使用场景：**

* 函数作为返回值
* 函数作为参数传递

```js
/* 1 函数作为返回值 */
function f1() {
  var a = 100
  // 返回一个函数，函数作为返回值
  return function() {
    console.log(a) // 自由变量，父作用域寻找
  }
}
// f1得到一个函数
var f2 = f1()
var a = 200
f1() // 100
/* 2 函数作为参数传递 */
function Fn1() {
  var a = 100
  return function() {
    console.log(a) // 自由变量，父作用域寻找
  }
}
var fn1 = Fn1()
function Fn2(fn) {
  var a = 200
  fn()
}
Fn2(f1) // 100
```

## 解题

* 说一下对变量提升的理解 - 见上文

* 说明this几种不同的使用场景 - 见上文

* 创建10个`<a>`标签，点击时弹出对应的序号

```js
for (var i = 0; i < 10; i++) {
  (function(i) {
    var el = document.createElement('a');
    el.innerHTML = i + '<br>';
    el.addEventListener('click', function(e) {
      e.preventDefault()
      console.log(i)
    })
    document.body.appendChild(el);
  })(i)
}
```

* 如何理解作用域

自由变量、作用域链（自由变量查找机制）、闭包的使用场景

* 实际开发中闭包的应用

```js
// 闭包实际应用中主要用于封装变量，收敛权限
function isFirstLoad() {
  // _作为私有变量、方法的前缀，这算是一个约定
  var _list = []
  return function(id) {
    if (_list.indexOf(id) >= 0) {
      return false
    } else {
      _list.push(id)
      return true
    }
  }
}
// 使用
var firstLoad = isFirstLoad()
firstLoad(10) // true
firstLoad(10) // false
firstLoad(20) // true
```

## 小结

… 其实杂七杂八说了一堆，只是浅尝辄止但作为迈出的一小步放在这里我觉得够了，明年再看到这篇时不知又是怎样的理解…  前端技术发展日新月异，但万变不离其宗，基本功是一切的根基，抛js基础大谈框架顶层建筑全是耍流氓~ 
坚持自己的路，与其和无聊之人瞎BB不如把时间拿来写写代码看看书提升下自己~

ps 最后推荐一下 `《你不知道的JavaScript》`上与中~