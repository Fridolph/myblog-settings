---
title: 【ES6】深入理解ES6——块级作用域绑定
date: 2017-07-17 21:41:03
tags: 
  - ES6
  - js
categories: 
  - JavaScript
---

想了很久还是决定一边看书一边把所记多得用博客的形式记录下来。毕竟好记性不如烂笔头嘛，而且，ES6这么多的内容很适合记博客啊，哈哈~~ 可以产出很多篇不说，还可以顺便练练文笔什么的啦... 额，废话就直接跳过了

> ES6的新语法可以让我们更好地控制作用域。这里我们先理解var声明的一些坑，然后从ES6新引入的块级作用域说起，然后到其绑定机制和最佳实践。

<!-- more -->

## var声明及变量提升机制

```js
function getValue(something) {
  if (something) {
    var value = 'hello world';
    // ...
    result value;
  } else {
    return null;
  }
}
```

我们读着代码，可能会认为只有something为true(存在)时，value才会被声明，但事实上，在`预编译`阶段，JavaScritp引擎会将上面的getValue函数修成成下面这样

```js
// 这里是全局作用域 window或global下
// getValue也会有函数声明提升，所以直接调用是能访问并正确执行getValue函数的
getValue();
function getValue(something) {
  // getValue的函数作用域顶层
  var value;  
  if (something) {
    value = 'hello world';
    // ...
    result value;
  } else {
    // 此处可访问变量value，其值为undefined
    return null;
  }
  // 此处可访问变量value，其值为undefined
}
```

变量value的声明被提升至顶部。所以在注释处我们都可以在`getValue函数内部`访问到value。那ES6引入块级作用域就是来强化对变量声明周期的控制的。

## 块级声明

> 块级声明用于声明在指定块的作用域之外无法访问的变量，块级作用域存在于：函数内部和块中（字符`{ }`之前的区域）

### let和const

还是以刚才的代码为例：

```js
// 这里是全局作用域 window或global下
function getValue(something) {
  // getValue的函数作用域顶层  
  if (something) {
    let value = 'hello world';
    const PI = 3.14;
    // ...
    result { value, PI };
  } else {
    // 变量value，PI在此处不存在
    return null;
  }
  // 变量value，PI在此处不存在
}
```

用let或const进行的声明不再被提升至函数顶部。执行流离开if块后，value、PI被立即销毁，如果something为false，就永不会声明及初始化变量。

**二者间的相同点** 禁止重声明

`在同级的作用域内，若变量用let或const声明过后，再次声明会直接报错`。但如果当前作用域嵌另一个作用域便可在内嵌的作用域中用let或const声明同名变量：

```js
var name = 'fri';
if (true) {
  // 不会报错
  let name = 'fridolph';
  // ...
}
```

## 临时死区 (Temporal Dead Zone)

> JavaScript引擎在扫描代码时发现变量声明，要么将它们提升至作用域顶部（遇到var声明），要么将声明放到TDZ中（遇到let或const声明）

访问TDZ中的变量会触发运行时错误。只有执行过变量声明语句后，变量才会从TDZ中移出，然后才可正常访问。

## 总结

* 块级作用域绑定的let和const为JavaScript引入了词法作用域（它们声明的变量不会提升），而且只可以在声明这些变量的代码块中使用。

* 在for-in和for-of循环中，let和const都会为每次迭代时创建新绑定（从而使循环体内创建的函数可以访问到相应迭代的值，而非最后一次迭代后的值）

注：在for循环中使用const声明可能引发错误，要尽量避免

* 使用块级作用域绑定的最佳实践是：默认使用const，只有在确实需要改变变量时才使用let
