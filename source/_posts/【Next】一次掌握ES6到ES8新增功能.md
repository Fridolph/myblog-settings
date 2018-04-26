---
title: 【Next】一次掌握ES6到ES8新增功能
date: 2018-4-22 21:19:23
tags: 
  - js
  - es6
categories: 
  - js
---

> 转自 https://medium.freecodecamp.org/here-are-examples-of-everything-new-in-ecmascript-2016-2017-and-2018-d52fa3b5a70e
> 作者 rajaraodv
> 感觉挺好的，就搬过来了，很多代码值得敲一遍多多回顾理解

<!-- more -->

## Array

### Array.prototype.includes 

找到数组中是否包含某项，直接返回布尔值（能找到NaN）

```js
const arr = [1,2,3,4,NaN]
// 之前
if (arr.indexOf(3) >= 0) {
  console.log(true)
}
// 推荐
if (arr.includes(3)) {
  console.log(true)
}
// ps: includes还可以用来判断是否包含NaN
arr.includes(NaN) // true
arr.indexOf(NaN) // -1
```

## Object

### 1. Object.values()

这个还好，我们已经在使用了，原来还是es7的。Object.values()是一个类似于Object.keys()的新函数，但是它返回了对象自身属性的所有值，不包括原型链中的任何值。

```js
const cars = {BMW: 3, Tesla: 2, Toyota: 1}
// es6
const vals = Object.keys(cars).map(key => cars[key])
console.log(vals) // [3,2,1]
// use
const values = Object.values(cars) // [3,2,1]
```

### 2. Object.entries()

同上，返回的是 由键值对组成的数组的数组集合~ 好像解释得绕了，我们还是看代码吧：

```js
const cars = {BMW: 3, Tesla: 2, Toyota: 1}
const ret = Object.entries(cars) // [ [BWM, 3], [Tesla, 2], [Toyota, 1] ]
// 场景1 - 利用解构赋值，直接拿到 k v 
for (let [key, val] of Object.entries(cars)) {
  console.log(`key: ${key}, value: ${val}`)
}
// 场景2 - 将拿到的 kv对存到 Map中
const map = new Map(Object.entries(cars))
console.log(map) // Map(3) {"BMW" => 3, "Tesla" => 2, "Toyota" => 1}
```

### 3. Object.getOwnPropertyDescriptors

该方法返回给定对象的所有属性的所有细节(包括getter getand setter set方法)。
添加这一功能的主要动机是允许浅层复制/克隆一个对象到另一个对象，该对象也复制getter和setter函数。

而`Object.assign 浅复制原始源对象的getter和setter函数外的所有细节。

```js
let Car = {
  name: 'BWM',
  price: 10000,
  set discount(x) {
    this.d = x
  },
  get discount() {
    return this.d
  }
}
console.log(Object.getOwnPropertyDescriptor(Car, 'discount')) // 返回某一属性
console.log(Object.getOwnPropertyDescriptors(Car) // 返回所有属性
```

```js
let oCar = Object.assign({}, Car)
console.log(Object.getOwnPropertyDescriptors(oCar)) // 没有了discount的 getter setter
```

### 4. 剩余对象属性

点点点的扩展，- - 其实用得很多了，比如写redux里经常~~

```js
let {firstName, age, ...remaining } = {
  firstName: 'hello',
  lastName: 'world',
  age: 24,
  height: 170,
  hoby: 'coding'
}
firstName // hello
age // 24
remaining // { lastName: 'world', height: 170, hoby: 'coding' }
```

> 可以用这一点，删除对象中不需要的属性

```js
let {SSN, ...cleanObj} = {
  SSN: 'xxx',
  name: 'fri',
  age: 24
}
cleanObj // {name: 'fri', age: 24}
```

> 当然，还可以利用这一点扩展对象属性

```js
let p1 = {name: 'fri', age: 24}
let p2 = {name: 'yk', hobby: 'coding'}
let person = {...p1, ...p2} // {name: "yk", age: 24, hobby: "coding"} 有同名key后面的会覆盖前面的
```


## String

### 1. String padding 字符串填充

允许将空字符串或其他字符串添加到原始字符串的开头或结尾。直接上代码：

```js
'5'.padStart(10)  // "         5" 不传第二个参数默认填充空格
'5'.padStart(10, '=*')  // "=*=*=*=*=5" 第一个参数决定返回字符串的length，第二个参数用于填充，但会保留初始字符串

'fri'.padEnd(10)  // "fri       "  和padStart类似，不过是在末尾加
'fri'.padEnd(10, '|--|')  // "fri|--||--" 字符串最大长度为10，所以最终成这样
```

我们来看一个高级点的用法，用于游戏，或是渲染字符串图片等很有用：

```js
const formatted = [0, 1, 12, 123, 1234, 12345].map(num => (
  num.toString().padStart(10, '0')
))
console.log(formateed)
/* [
  "0000000000", 
  "0000000001", 
  "0000000012", 
  "0000000123", 
  "0000001234", 
  "0000012345"
] */

const cars = {
  '😀BWM': '10'
  '😎Tesla': '5',
  '😝Lamborghini': '0'
}
Object.entries(cars).map(([name, count]) => {
  console.log(`${name.padEnd(20, ' -')} Count: ${padStart(3, '0')}`)
})
// 😀BWM - - - - - - - Count: 010
// 😎Tesla - - - - - - Count: 010
// 😝Lamborghini - - - Count: 010
```

> 注：在Emojis和其他双字节字符上使用padStart和padEnd，一个emoji表情是2个字符

## 正则扩展


### 1. `s` 模式

这种增强使得点操作符可以匹配任何单个字符。为了确保这不会破坏任何东西，我们需要使用\s标志，当我们创建RegEx来工作时。

```js
/first.second/s.test('first\nsecond'); //true
```

### 2. RegExp命名组捕获

在下面的例子中，我们使用`(?<year>) (?<month>) (?<day>)`名称来分组日期RegEx的不同部分。产生的对象现在将包含具有属性年、月、日和相应值的组属性。

```js
// before
let re1 = /(\d{4})-(\d{2})-(\d{2})/
let ret1 = re1.exec('2015-01-02')
console.log(ret1) // ["2015-01-02", "2015", "01", "02", index: 0, input: "2015-01-02", groups: undefined]
// after
let re2 = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
let ret2 = re2.exec('2015-01-02')
console.log(ret2.goups) // {year: "2015", month: "01", day: "02"}
```

### 3. 使用命名组

```js
let sameWords = /(?<fruit>apple|orange)==\k<fruit>/u;
sameWords.test('apple==apple')   // true
sameWords.test('orange==orange') // true
sameWords.test('apple==orange')  // false
```

### 4. 在字符串中使用命名组

```js
let re = /(?<firstName>[A-Za-z]+) (?<lastName>[A-Za-z]+$)/u;
'Raja Rao'.replace(re, '$<lastName>, $<firstName>') // 'Rao Raja'
```

### 5. 正则向后插入断言

```js
/(?<=#).*/.test('winning') // false
/(?<=#).*/.test('#winning') // true
```

之前我们写(?=) 这类匹配到的一般从[1]开始，这样写从[0]就可以匹配到，习惯了用之前的也行，毕竟这样会增加学习成本。

### 6. 正则匹配unicode

```js
//The following matches multiple hindi character
/^\p{Script=Devanagari}+$/u.test('हिन्दी'); //true  
//PS:there are 3 hindi characters h
```

## Other

### 1. `**` 用来替代 `Math.pow`

```js
Math.pow(7, 2) // 49  7的2次方
// use
7**2 // 49 
```

### 2. 在函数参数中添加尾随逗号，现在不会报错了

### 3. async/await 

这个用得也挺多了，同步写法的异步函数

```js
async function doubleAndAdd(a, b) {
  a = await doubleAfterSec(a)
  b = await doubleAfterSec(b)

  return a + b
}
doubleAndAdd(1, 2).then(console.log)
function doubleAfterSec(param) {
  return new Promise(resolve => {
    setTimeout(resolve(param * 2), 1000)
  })
}
```

来看一个高级用法

```js
async function doubleAndAdd(a, b) {
  try {
    [a, b] = await Promise.all([doubleAfterSel(a), doubleAfterSec(b)])
  } catch (e) {
    return throw new Error(e)
  }
  return a + b
}
doubleAndAdd(1, 2).then(console.log)
function doubleAfterSec(param) {
  return new Promise(resolve => {
    setTimeout(resolve(param * 2), 1000)
  })
}
```

### 4. Promise.prototype.finally()

主要的想法是允许在解决或拒绝帮助清理问题之后运行回调。最后回调被调用，没有任何值，无论如何都要执行。可以理解为增加了一个生命周期钩子

```js
let started = true
let myPromise = new Promise((resolve, reject) => {
  resolve('all good')
}).then(val => {
  console.log(val) // 'all good'
}).catch(err => {
  console.log(err)
}).finally(() => {
  console.log('finally')
  started = false
})
```

### 5. 异步迭代器

这是一个非常有用的特性。基本上，它允许我们轻松地创建异步代码的循环！
这个特性添加了一个新的“for-awaof”循环，它允许我们调用async函数，它在循环中返回承诺（或带有一堆承诺的数组）。
很酷的一点是，循环等待每个承诺在执行下一个循环之前解决。

```js
const promises = [
  new Promise(resolve => resolve(1)),
  new Promise(resolve => resolve(2)),
  new Promise(resolve => resolve(3))
]
// 之前这么用
async function test1() {
  for (const obj of promises) {
    console.log(obj) // 打印3个promise
  }
}
// 现在可以
async function test2() {
  for await (const obj of promises) {
    console.log(obj) // 直接打印结果 1 2 3
  }
}
```

## 总结

* Array方法
  1. Array.prototype.includes 是否包含某项，返回Boolean

* Object方法
  1. Object.values() 迭代器 返回value组成的对象
  2. Object.entries() 迭代器，返回 k v组成的数组
  3. Object.getOwnPropertyDescriptors? 取某个属性能拿到getter setter
  4. 对象扩展 ... 点点点

* String方法
  1. padStart
  2. padEnd

* 正则扩展
  1. s模式 匹配换行等而不破坏字符串
  2. 命名组捕获
  3. 插入断言
  4. 匹配unicode

* 函数扩展
  1. async/await
  3. Promise.finally 钩子
  3. async函数里 for 可以使用 await
  4. 原生方法扩展 Math.pow -> `**`