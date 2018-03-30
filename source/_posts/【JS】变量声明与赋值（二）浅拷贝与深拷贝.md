---
title: 【JS】变量声明与赋值（二）浅拷贝与深拷贝
date: 2017-12-04 21:52:15
tags: 
  - js
  - es6
  - 对象拷贝
  - 转载
categories: 
  - js
---

> 该篇介绍了浅拷贝、深拷贝及复合类型的拷贝的几种方式，对其进行了整理和总结

<!-- more -->

## Copy Composite Data Types: 复合类型的拷贝

### Shallow Copy 浅拷贝

浅拷贝是指复制时，指对第一层键值对进行独立的赋值，一个简单的实现如下：

```js
function shallowCopy(target, source) {
  if (!source || typeof source !== 'object') return
  // 这个方法有点问题，target一定要事先定好，不然就不能改变实参了
  // 具体原因解释可看参考资料中 JS是值传递还是引用传递
  if (!target || typeof target !== 'object') return

  // 这边最好区别一下对象和数组的复制
  for (var key in source) {
    if (source.hasOwnProperty(key)) {
      target[key] = source[key]
    }
  }
}
```

### Object.assign

Object.assign() 方法可以把任意多个源对象所拥有的自身可枚举属性拷贝给目标对象，然后返回目标对象。
Object.assign 方法只会拷贝 源对象自身的并且可枚举的属性 到目标对象身上。注意，对于访问器属性，该方法会执行那个访问器属性的 getter 函数，然后把得到的值拷贝给目标对象如果你想拷贝访问器属性本身，请使用 `Object.getOwnPropertyDescriptor` 和 `Object.difineProperties()`方法

注意，字符串类型和 symbol 类型的属性都会被拷贝。

注意，在属性拷贝过程中可能会产生异常，比如目标对象的某个只读属性和源对象的某个属性同名，这时该方法会抛出一个 TypeError 异常，拷贝过程中断，已经拷贝成功的属性不会受到影响，还未拷贝的属性将不会再被拷贝。

注意， Object.assign 会跳过那些值为 null 或 undefined 的源对象。

    Object.assign(target, ...sources)

浅拷贝一个对象

    var obj = {a: 1}
    var copy = Object.assign({}, obj)
    console.log(copy) // {a:1}

合并若干个对象

```js
var o1 = { a: 1 }
var o2 = { b: 2 }
var o3 = { c: 3 }
var obj = Object.assign(o1, o2, o3)
console.log(obj) // { a: 1, b: 2, c: 3 }
console.log(o1) // { a: 1, b: 2, c: 3 }, 注意目标对象自身也会改变。
```

拷贝 symbol 类型的属性

```js
var o1 = { a: 1 }
var o2 = { [Symbol('foo')]: 2 }
var obj = Object.assign({}, o1, o2)
console.log(obj) // { a: 1, [Symbol("foo")]: 2 }
```

继承属性和不可枚举属性是不能拷贝的

```js
var obj = Object.create(
  { foo: 1 },
  {
    // foo是个继承属性
    bar: { value: 2 }, // bar是个不可枚举属性
    baz: {
      value: 3,
      enumerable: true // baz是个自身可枚举属性
    }
  }
)
var copy = Object.assign({}, obj)
console.log(copy) // {baz: 3}
```

原始值会被隐式转换成其包装对象

```js
var v1 = '123'
var v2 = true
var v3 = 10
var v4 = Symbol('foo')

var obj = Object.assign({}, v1, null, v2, undefined, v3, v4)
// 源对象如果是原始值，会被自动转换成它们的包装对象，
// 而 null 和 undefined 这两种原始值会被完全忽略。
// 注意，只有字符串的包装对象才有可能有自身可枚举属性。
console.log(obj) // { "0": "1", "1": "2", "2": "3" }
```

例子：拷贝属性过程中发生异常

```js
var target = Object.defineProperty({}, 'foo', {
  value: 1,
  writeable: false
}) // target 的 foo 属性是个只读属性。
Object.assign(target, { bar: 2 }, { foo2: 3, foo: 3, foo3: 3 }, { baz: 4 })
// TypeError: "foo" is read-only
// 注意这个异常是在拷贝第二个源对象的第二个属性时发生的。
console.log(target.bar) // 2，说明第一个源对象拷贝成功了。
console.log(target.foo2) // 3，说明第二个源对象的第一个属性也拷贝成功了。
console.log(target.foo) // 1，只读属性不能被覆盖，所以第二个源对象的第二个属性拷贝失败了。
console.log(target.foo3) // undefined，异常之后 assign 方法就退出了，第三个属性是不会被拷贝到的。
console.log(target.baz) // undefined，第三个源对象更是不会被拷贝到的。
```

### 使用 [].concat 来复制数组

同样类似于对于对象的复制，我们建议使用[].concat 来进行数组的深复制:

    ar list = [1, 2, 3];
    var changedList = [].concat(list);
    changedList[1] = 2;
    list === changedList; // false

同样的，concat 方法也只能保证一层深复制:

    > list = [[1,2,3]]
    [ [ 1, 2, 3 ] ]
    > new_list = [].concat(list)
    [ [ 1, 2, 3 ] ]
    > new_list[0][0] = 4
    4
    > list
    [ [ 4, 2, 3 ] ]

### 浅拷贝的缺陷

不过需要注意的是，assign 是浅拷贝，或者说，它是一级深拷贝，举两个例子说明：

```js
const defaultOpt = {
  title: {
    text: 'hello world',
    subtext: "It's my world"
  }
}
const opt = Object.assign({}, defaultOpt, {
  title: {
    subtext: 'Yes, your world'
  }
})
console.log(opt) // { title: { subtext: 'Yes, your world.' } }
```

上面这个例子中，对于对象的一级子元素而言，只会替换引用，而不会动态的添加内容。那么，其实 assign 并没有解决对象的引用混乱问题，参考下下面这个例子：

```js
const defaultOpt = {
  title: {
    text: 'hello world',
    subtext: "It's my world."
  }
}
const opt1 = Object.assign({}, defaultOpt);
const opt2 = Object.assign({}, defaultOpt);
opt2.title.subtext = 'Yes, your world.';
console.log(opt1); // { title: { text: 'hello world', subtext: 'Yes, your world.' } }
console.log(opt2); // { title: { text: 'hello world', subtext: 'Yes, your world.' } }
```

### DeepCopy: 深拷贝

**递归属性遍历**

一般来说，在JavaScript中考虑复合类型的深层复制的时候，往往就是指对于Date、Object与Array这三个复合类型的处理。我们能想到的最常用的方法就是先创建一个空的新对象，然后递归遍历旧对象，直到发现基础类型的子节点才赋予到新对象对应的位置。不过这种方法会存在一个问题，就是JavaScript中存在着神奇的原型机制，并且这个原型会在遍历的时候出现，然后原型不应该被赋予给新对象。那么在遍历的过程中，我们应该考虑使用hasOenProperty方法来过滤掉那些继承自原型链上的属性:

```js
function clone(obj) {
  var copy
  // 处理基本类型 null undefined
  if (null === obj || 'object' !== typeof obj) return obj
  // 若为 Date 对象时
  if (obj instanceof Date) {
    copy = new Date()
    copy.setTime(obj.getTime())
    return copy
  }
  // 处理数组
  if (obj instanceof Array) {
    copy = []
    for (var i = 0, len = obj.length; i < len; i++) {
      copy[i] = clone(obj[i])
    }
    return copy
  }
  // 处理对象
  if (obj instanceof Object) {
    copy = {}
    for (var attr in obj) {
      if (obj.hasOwnProperty(attr)) {
        copy[attr] = clone(obj[arr])
      }
    }
    return copy
  }
  throw new Error('Unable to copy obj! Its type isn\'t supported!')
}
```

调用如下：

```js
// This would be cloneable:
var tree = {
    "left"  : { "left" : null, "right" : null, "data" : 3 },
    "right" : null,
    "data"  : 8
};
// This would kind-of work, but you would get 2 copies of the 
// inner node instead of 2 references to the same copy
var directedAcylicGraph = {
    "left"  : { "left" : null, "right" : null, "data" : 3 },
    "data"  : 8
};
directedAcyclicGraph["right"] = directedAcyclicGraph["left"];
// Cloning this would cause a stack overflow due to infinite recursion:
var cylicGraph = {
    "left"  : { "left" : null, "right" : null, "data" : 3 },
    "data"  : 8
};
cylicGraph["right"] = cylicGraph;
```

### 利用JSON深拷贝

    JSON.parse(JSON.stringify(obj))

对于一般的需求是可以满足的，但是它有缺点，下例中，可以看到JSON赋值会忽略掉值为undefined以及函数表达式

```js
var obj = {
  a: 1,
  b: 2,
  c: undefined,
  sum: function() {
    return a + b
  }
}
var obj2 = JSON.parse(JSON.stringify(obj))
console.log(obj2) // Object {a: 1, b: 2}
```

---

### 参考资料

[基于 JSX 的动态数据绑定](https://zhuanlan.zhihu.com/p/28313321)
[ECMAScript 2017（ES8）特性概述](https://zhuanlan.zhihu.com/p/27844393)
[WebAssembly 初体验：从零开始重构计算模块](https://zhuanlan.zhihu.com/p/27410280)