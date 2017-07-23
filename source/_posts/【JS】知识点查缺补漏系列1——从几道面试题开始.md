---
title: 【JS】知识点查缺补漏系列1——从几道面试题开始
date: 2017-07-23 09:01:03
tags: 
  - js
categories: 
  - JavaScript
---

> 最近整理公司文档，对前段时间的面试经验也算作了些总结，从中也很有多体会，毕竟是人生中的第一次经历嘛（一直都是被面试，现在是面试别人，那感觉还不错…）作为纪念于是就把这些记录下来 … ps 也算为之后作点铺垫… 咳咳你们懂的

系列暂拟了大纲（不确定，随时会改，但最终会统一整理）：

1——从几道面试题开始
2——JS必知(原型相关、作用域、执行环境、闭包等等)
3——JS异步相关
4——JS 常用API与DOM相关
5——JS DOM操作与Event事件
6——Ajax、跨域与本地存储
7——性能优化与安全相关
8——前端自动化相关

<!-- more -->

好的，欢迎回来，这是本系列的第一篇，作为引子，想先从几道常见的面试题入手由浅入不那么浅。。(误)~

## 思考问题的结论

拿到面试题 -> 看考点

看不完的题海 -> 以不变应万变

面对面试题 -> 由题目到知识再回归题目本身

### 简单思考

* js总使用typeof能得到的变量类型？ 考点：js变量类型

* 何时用 === 何时用 == ？ 考点：强制类型转换

* window.onload和 DOMContentLoaded 区别？ 考点：浏览器渲染过程

* 用js创建10个<a>标签，点击时弹出对应序号？ 考点：作用域

* 实现数组的随机排序？ 考点：JS基础算法

先看几道题:

1. js总使用typeof能得到的变量类型？ 考点：js变量类型

2. 何时用 === 何时用 == ？ 考点：强制类型转换

3. js有哪些内置函数

4. js变量按照存储方式区分为哪些类型，并描述其特点

5. 如何理解JSON

---

## 变量类型与变量计算

*值类型

*引用类型  -  对象、数组、函数

### typeof 运算符

### 变量计算 - 强制类型转换

* 字符串拼接

```js
var a = 100 + 10 // 110
var b = 100 + '10' // 10010
```

* == 运算符

```js
100 == '100' // true
0 == ''  // true
null == undefined  // true
```

* if语句

```js
var a = true
if (a) {
  // ...
}
var b = 100
if (b) {
  // ...
}
var c = ''
if (c) {
  // ...
}
```

* 逻辑运算

```js
console.log(10 && 0) // 0
console.log('' || 'abc') // 'abc'
console.log(!window.abc) // true
//判断一个变量会被当作true还是false
var a = 100
console.log(!!a)
```

## 解题

1. typeof类型

```js
typeof undefined // undefined
typeof 'abc'  // string
typeof 123  // number
typeof true // boolean
typeof {}  // object
typeof []  // object
typeof null // object
typeof console.log  // function
```

2. 何时使用 === 和 ==

    if (obj.a == null) {
      // ...  
    }

这里相当于 `obj.a === null || obj.a === undefined` 的简写形式
这是jQuery源码中推荐的写法

3. JS中的内置函数 - 数据封装类对象

Object
Array
Boolean
Number
String
Function
Date
RegExp
Error

4. js变量按照存储方式区分为哪些类型，并描述其特点

```js
/* 值类型 */
var a = 10
var b = a
a = 11
console.log(b)  // 11
/* 引用类型 */
var obj1 = { x: 100 }
var obj2 = obj1
obj1.x = 200
console.log(obj2.x) // 200
```

5. 如何理解JSON

```js
// - JSON是一个JavaScript对象，有自己的api
// - 数据格式
JSON.stringify({a: 10, b: 20})
JSON.parse(`{"a": 10, "b": 20}`)
```

## 小结

我们拿到题目，首先阅读仔细要明确考点，懂得面试意图，再来针对性作答。当然，答题不是我们追求的本质，更重要的是通过面试调整好自己的心态，查缺补漏不断完善自己的知识点，如果这次不成，放好心态迎接接下来的挑战就好。

通过这个引子，我们可以看到JavaScript确实有很多坑，但同时也是一门超级灵活，书写爽快的语言，作为一门很有“统治性的语言”（任何可以用JS写的程序最终都将用JS来实现）与其埋怨，不如深究下去，让我们之后的工作更从容些。