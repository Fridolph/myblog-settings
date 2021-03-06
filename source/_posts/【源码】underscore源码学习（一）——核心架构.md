---
title: 【源码】underscore源码学习（一）——核心架构
date: 2017-06-02 19:11:46
categories: 
  - JS
tags: 
  - 源码
  - underscore
---

> 读源码是一个提升自己编码和架构的有效方式，明明知道但一直没敢尝试，就算是学习jQuery的源码也是跟着大神的视频，博客一点点填坑的，之前也苦于写的博客没什么技术含量，要不就是自己都知道的，要不就是摘录书中的一些经验，久之反倒没了写博客的兴趣。underscore.js是个优秀的工具库，总代码行也就1600行+ 每天读50行+的速度一个月也能搞定，这么一想完全就不怕了。

多点信心，不怕犯错，多记录和搜索这样利于自己巩固，嗯嗯… 也为后面看Vue、React、Redux源码什么的打点基础，当然那是后话了… 希望这个系列能把坑填完就好

<!-- more -->

---

`注：underscore 叫着太啰嗦- - 下文都用 _ 来指代了， 既然是读underscore想必也能接受`

那么直接开始正文吧~~

## 命名空间管理

最外层是一个IIFE(立即调用函数)，各大框架常用，如jQuery就是典型，这是为了模拟块级作用域，防止污染全局命名空间。具体的好处和由来可以去谷歌一下

```js
((function() {
  // ...
})());
```

## 变量声明 14-168行

### 巧妙地变量声明

巧妙地变量声明可以让程序执行地更加效率，关于这些点，在后文中我会力所能及地表达清楚
在说这点前，大家可以看看JS高级程序的第7章函数部分有提到

```js
var root = typeof self == 'object' && self.self === self && self ||
  typeof global == 'object' && global.global === global && global ||
  this ||
  {};
```

### 了解运算符优先级

声明root变量，首先我们要了解JS的运算符优先级，[运算符优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)

以下运算符从高到低:

* 16  typeof
* 10  ==、=== 
* 6   逻辑与 &&
* 5   逻辑或 ||

那么我们重新审视这串代码：

```js
var root = 
  ((typeof self == 'object') && (self.self === self) && self) ||
  ((typeof global == 'object') && (global.global === global) && global) ||
  this ||
  {};
```

这样会好理解一些了，把root定义为全局对象，如果不是在浏览器或者Node.js环境中就设为一个空对象

    var previousUnderscore = root._;

previousUnderscore作为全局对象的属性存储，其名为 `_` ，说不清，但很简单，举个例：

    ((function() {
      var root = window || {};
      var pu = root._;

      pu = {
        'name': 'fri',
        'age': 24
      }

    })());

    console.log(_);

内部作用域的私有变量是pu，但通过root._ 的形式暴露给了全局对象，所以在全局通过 `_` 变量就能访问到内部pu对象了

`注：这段代码在严格模式下会报错 ReferenceError: _ is not defined`


```js
var ArrayProto = Array.prototype, 
    ObjProto = Object.prototype;
var SymbolProto = typeof Symbol !== 'undefined' ? Symbol.prototype : null;
var push = ArrayProto.push,
    slice = ArrayProto.slice,
    toString = ObjProto.toString,
    hasOwnProperty = ObjProto.hasOwnProperty;
var nativeIsArray = Array.isArray,
    nativeKeys = Object.keys,
    nativeCreate = Object.create;
```

### 原型链

这里的用处差不多，我就放到一块了，之前我提到过作用域链，而JS是没有类这一说的，其实是通过对象，对象的原型这样一层一层建立起对象的引用关系从而拿到活动对象上的值。

我们看几个原型变量的声明，其实就是简化原型链的查找过程，还是举个例子：

    var arr = [];
    arr.push('a');

声明arr变量为一个空数组，添加一个元素'a'，这个push方法是在arr上的吗？并不是，arr变量上找不到push方法，会搜索arr的原型prototype(arr的constructor指向的对象 -> 这里是Array)。
通过原型链一层层往上找，最终在Array.prototype上找到了push方法，也就是说程序执行 arr.push('a')时，检索到push方法经历了3次搜索即 `arr本身 -> Array对象 -> Array.prototype`
执行效率并不高，所以 var ArrPro = Array.prototype 这样的声明就是减少了搜索过程，从而提升了最终的运行速度

    var SymbolProto = typeof Symbol !== 'undefined' ? Symbol.prototype : null;

这里顺便提一下，是对ES6新增的基本数据类型Symbol进行兼容处理

我们接着读

    var Ctor = function(){};    

这里声明了`Ctor`对象 为一个匿名函数，用于代理原型交换


### 安全引用

这里为 `_` 创建一个安全的引用，这里 `_` 是作为一个方法存在，我们结合后面的例子再来说这块

```js
  var _ = function(obj) {
    if (obj instanceof _) return obj;
    if (!(this instanceof _)) return new _(obj);
    this._wrapped = obj;
  };
```

```js
  if (typeof exports != 'undefined' && !exports.nodeType) {
    if (typeof module != 'undefined' && !module.nodeType && module.exports) {
      exports = module.exports = _;
    }
    exports._ = _;
  } else {
    root._ = _;
  }
```

这里用于支持Node.js，若在浏览器中就让`_`成为全局对象
xxx && xxx 这样的语句其实有断句，首先 typeof exports != 'undefinded' 若为false 函数将直接跳过这段语句

    _.VERSION = '1.8.3';

`_` 的属性， 存储版本号，你看到这里可能很奇怪，之前不是让 `var _ = function() { ... }`了吗，这并不冲突，因为在JS这门语言中一切皆对象，函数也是对象仅此而已`(Function instanceof Object  // true)`  `_` 本身声明为一个匿名函数，但它本身是一个引用类型的数据(对象)，所以继续声明其他属性，或是删除属性也是没有问题的。

68-87行

### optimizeCb 与 函数柯里化

```js
  var optimizeCb = function(func, context, argCount) {
    if (context === void 0) return func;
    switch (argCount) {
      case 1: return function(value) {
        return func.call(context, value);
      };
      // The 2-parameter case has been omitted only because no current consumers
      // made use of it.
      case null:
      case 3: return function(value, index, collection) {
        return func.call(context, value, index, collection);
      };
      case 4: return function(accumulator, value, index, collection) {
        return func.call(context, accumulator, value, index, collection);
      };
    }
    return function() {
      return func.apply(context, arguments);
    };
  };
```

`optimizeCb`是`_`的一个内部函数，该框架内的函数都是以变量的形式来声明，这么做的好处有几点：

1. 避免了不必要的函数声明提升，流程控制性更强
2. 更利于实现内存的回收

我们先暂时跳过流程控制语句，该函数会返回一个匿名函数（闭包），该匿名函数执行并返回第一个参数func，`这个func就是要进行柯里化的函数`，此时的func作为 optimizeCb的一个参数，其this指向会上一层寻找，即整个 `_` 的内部作用域，这显然是不行的，所以func.apply的作用即是将func的作用域绑定到所传的context中，arguments为剩下的argCount


参考 高级程序设计198页

![活动对象1](http://blog.fueson.top/img/2017/underscore02.png)
![活动对象2](http://blog.fueson.top/img/2017/underscore01.png)

`optimizeCb`若执行，所传参数，都将作为optimizeCb内部的活动对象，函数执行完毕所返回的匿名函数是一个闭包，它能访问到当前活动对象上的参数，即函数传参。

比如在 case 4 里的代码段

    ...
    return function(accumulator, value, index, collection) {
      return func.call(context, accumulator, value, index, collection);
    };

function(accumulator, value, index, collection) 能访问到上层，传参进行活动对象，执行后返回的func是之前传入，这里调用call方法，以传入的`context`来替换当前的活动对象，并将 accumulator, value, index, collection 作为参数传递给 当前传入的 `context` ，这里有点绕，大家最好自己写点代码在浏览器中测试

### 函数柯里化

这里顺便理一下这个话题，因为下面会用到，且强行克服一下这个一直没攻破的难关。
函数柯里化`用于创建`已经设置好了一个或多个参数的`函数`，其基本方法与函数绑定一样：使用闭包返回一个函数。
与apply,call,bind等绑定方法的区别在于：返回的函数还需要设置一些传入的参数

柯里化函数通常由以下步骤动态创建：`调用另一个函数`并为它`传入`要`柯里化的函数`和`必要参数`，代码如下：

```js
/**
 * @param fn 表示要柯里化的函数
 * @return 返回一个闭包，这个闭包能够访问当前的函数作用域
 */
function curry(fn) {
  // args包含了来自外部函数的参数
  var args = Array.prototype.slice.call(arguments, 1)
  return function() {
    // innerArgs 用于存放所有传入的参数
    var innerArgs = Array.prototype.slice.call(arguments)
    // finalArgs 将内外参数合并到一起
    var finalArgs = args.concat(innerArgs)
    // 使用apply将结果传递给该函数，这里由于没考虑执行环境所以传了null
    return fn.apply(null, finalArgs)
  }
}
```

### cb 安全的上下文环境

这个cb函数用于生成可以应用到集合中的每个元素的回调函数，返回想要的结果或标识`（一个任意的回调，一个属性matcher，或者一个属性访问器）`
iteratee和builtinIteratee赋值为一个匿名函数，返回了在这个执行环境中的一个标识
要注意的是 `Infinity`这是一个系统常量

```js
var builtinIteratee;

var cb = function(value, context, argCount) {
  if (_.iteratee !== builtinIteratee) return _.iteratee(value, context);
  if (value == null) return _.identity;
  if (_.isFunction(value)) return optimizeCb(value, context, argCount);
  if (_.isObject(value) && !_.isArray(value)) return _.matcher(value);
  
  return _.property(value);
}

_.iteratee = builtinIteratee = function(value, context) {
  return cb(value, context, Infinity);
};
```

内部作用域是 4个 if语句，若继续执行会 返回 `_.property(value)`(1408行) 这里还是贴一下

```js
_.property = function(path) {
  if (!_.isArray(path)) {
    return shallowProperty(path);
  }
  return function(obj) {
    return deepGet(obj, path);
  };
};
```

我们来具体分析一下：

    if (_.iteratee !== builtinIteratee) return _.iteratee(value, context);

也就是说上一句 `_.iterate = builtinIteratee = function(value, context) {...}` 未生效就重新赋值一次，

    if (value == null) return _.identity;

没有传value就默认返回最大

    if (_.isFunction(value)) return optimizeCb(value, context, argCount);

`_`自己实现的方法，判断传入对象的引用类型是否为 function 是就用optimizeCb函数柯里化处理所传value

    if (_.isObject(value) && !_.isArray(value)) return _.matcher(value);

同上，若判断成功则用 matcher方法来处理（1429行）
通过这样的判断及调用，可保证我们处理的value值是合法且可控的

### restArgs方法

该方法作用类似于ES6的 function(...args) {...} 展开符，注释上是这么说的：（来自我的渣翻译）
这些剩余的参数会进到一个数组，并对应给一个索引，我们还是来看代码吧

```js
  var restArgs = function(func, startIndex) {
    startIndex = startIndex == null ? func.length - 1 : +startIndex;
    return function() {
      var length = Math.max(arguments.length - startIndex, 0),
          rest = Array(length),
          index = 0;
      for (; index < length; index++) {
        rest[index] = arguments[index + startIndex];
      }
      switch (startIndex) {
        case 0: return func.call(this, rest);
        case 1: return func.call(this, arguments[0], rest);
        case 2: return func.call(this, arguments[0], arguments[1], rest);
      }
      var args = Array(startIndex + 1);
      for (index = 0; index < startIndex; index++) {
        args[index] = arguments[index];
      }
      args[startIndex] = rest;
      return func.apply(this, args);
    };
  };
```

    startIndex = startIndex == null ? func.length - 1 : +startIndex;

`startIndex`是第二个参数， startIndex == null 这一句很有意思，很多框架里都喜欢这么用，我们可能经常听说要用 === 少用 == 吗，这里有个隐式转换， 这里判断了startIndex变量是否为空对象，毕竟进入函数作用域，startIndex作为参数是定义过的，这个函数会返回一个匿名函数（闭包），其作用是循环传递的func，将其作用域作为参数传递给调用者

### baseCreate

用于创建新对象的内部函数，该对象继承自另一个对象。

```js
var baseCreate = function(prototype) {
  if (!_.isObject(prototype)) return {};
  if (nativeCreate) return nativeCreate(prototype);
  Ctor.prototype = prototype;
  var result = new Ctor;
  Ctor.prototype = null;
  return result;
};
```

`_.isObject(prototype)` 这是 `_` 的一个内置方法用于判断传入参数是否为对象，这里加上!就是说，若不是对象则返回一个空对象 {}。
nativeCreate之前已声明过，这是ES5原生的创建对象方法，我们把之前匿名函数的原型赋给了传入的prototype，然后把这里的结果赋给一个新对象result并返回出来。 
这里的 Ctor.prototype = null 是精髓，用于内存回收，`简单说一下，函数运行会创建自己的活动对象，执行环境，运行结束后会自行销毁，但闭包中的变量和方法将存于内存中得不到回收，具体可读一下高级程序设计`
这样Ctor这个匿名函数我们重复使用且不会造成内存泄漏了。

### shallowProperty

```js
var shallowProperty = function(key) {
  return function(obj) {
    return obj == null ? void 0 : obj[key];
  };
};
```

意同名，浅属性~ shallowProperty返回了一个匿名函数，该匿名函数根据所传入的key的有无来决定是返回一个空对象还是对象的值

### deepGet

```js
var deepGet = function(obj, path) {
  var length = path.length;
  for (var i = 0; i < length; i++) {
    if (obj == null) return void 0;
    obj = obj[path[i]];
  }
  return length ? obj : void 0;
};
```

内部辅助方法，这里要说下第二个参数path，比如一个 `var arr = [1, [2,3]]` 二维数组，要拿到3，只能通过arr[1][1]来完成，再看下上面的代码是不是懂了呢？

### isArrayLike 

一个处理类数组的方法，注释上说 —— 根据所传参数`collection`确定是否应该将一个集合作为数组或对象进行迭代
该方法返回一个布尔值，该判断结果用于后续的执行

```js
var MAX_ARRAY_INDEX = Math.pow(2, 53) - 1;
var getLength = shallowProperty('length');
var isArrayLike = function(collection) {
  var length = getLength(collection);
  return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
};
```

Math.pow()方法可返回 x 的 y 次幂的值。这里是一个安全值的处理，防止内存溢出
这里的isArrayLike将传入的collection类数组进行这样的处理：
通过getLength辅助函数拿到collection的长度，之前的shallowProperty就是用于此处，getLength返回一个闭包，从而拿到 collection.length

    return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;

根据之前的经验，这里 && 优先级平级，即 `typeof length == 'number' && length >= 0` 比较完后的结果 再拿去和 `length <= MAX_ARRAY_INDEX` 进行逻辑与运算即可


## 总结
