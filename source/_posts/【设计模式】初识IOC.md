---
title: 【设计模式】初识IOC
date: 2017-12-13 17:49:03
tags: 
  - js
  - 设计模式
categories: 
  - js
  - 设计模式
---

> 看到群里的大佬们在讨论着，冒出了好多“生词”，竟然完全不知，惭愧惭愧，于是各种谷歌百度后将之整理一下。近几年，前端应用（WebApp）正朝着大规模方向发展，在这个过程中我们会对项目拆解成多个模块/组件来组合使用，以此提高我们代码的复用性，最终提高研发效率。在编写一个复杂组件的时候，总会依赖其他组件来协同完成某个逻辑功能。组件越复杂，依赖越多，可复用性就越差，我们可以借助软件工程中优秀的编程理念来提高复杂组件的可复用性，以下将详述其中之一的依赖倒置理念。

<!-- more -->

之前自学React，看到了高阶组件这么一个概念。那么理解之前，我们首先要理解高阶函数这个概念

```js
function hello() {
  console.log('Hello World')
}
hello() // 'Hello World'
// hello()
function WrapperHello(fn) {
  return function() {
    console.log('before ------ say hello')
    fn()
    console.log('after ------- say hello')
  }  
}
hello = WrapperHello(hello)
hello()
// 'before ------ say hello'
// 'Hello World'
// 'after ------- say hello'
```

函数既可以当参数，也可以当返回值

### 属性代理

```jsx
class Hello extends Component {
  render() {
    return <h2>Hello World</h2>
  }
}
function WrapHello(Comp) {
  class WrapComp extends Component {
    render() {
      return (
        <div>
          <p>高阶组件 - 属性代理</p>
          <Comp {...this.props} />
        </div>
      )
    }
  }
  return WrapComp
}
Hello = WrapHello(Hello)
<Hello />
// 这个Hello 组件 已经有p标签了
```

上面的 WrapComponent 就是属性代理
可以在其添加更多的属性或组件等

### 反向继承

```js
class Hello extends Component {
  render() {
    return <h2>Hello World</h2>
  }
}
function WrapComponent(Comp) {
  class Wrap extends Comp {
    render() {
      return <Comp />
    }
    componentDidMount() {
      console.log('高阶函数 - 反向继承')
    }
  }
  return Wrap
}
Hello = WrapComponent(Hello)
```

### 

## 依赖反转 Inversion of Control

其实这词好像经常听到过，很高端的样子。我们先来百科的解释： 控制反转把创建对象的权利交给框架，是框架的重要特征，并非面向对象编程的专业术语。
它包括依赖注入`Dependency Injection`和依赖查找`Dependency Lookup`

它包含两个准则：
1. 高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象
2. 抽象不应该依赖于具体实现，具体实现应该依赖于抽象

其背后的核心思想是：面向接口编程

### 优缺点

IoC最大的好处是什么？因为把对象生成放在了XML里定义，所以当我们需要换一个实现子类将会变成很简单（一般这样的对象都是实现于某种接口的），只要修改XML就可以了，这样我们甚至可以实现对象的热插拔（有点像USB接口和SCSI硬盘了）。

IoC最大的缺点是什么？（1）生成一个对象的步骤变复杂了（事实上操作上还是挺简单的），对于不习惯这种方式的人，会觉得有些别扭和不直观。（2）对象生成因为是使用反射编程，在效率上有些损耗。但相对于IoC提高的维护性和灵活性来说，这点损耗是微不足道的，除非某对象的生成对效率要求特别高。（3）缺少IDE重构操作的支持，如果在Eclipse要对类改名，那么你还需要去XML文件里手工去改了，这似乎是所有XML方式的缺陷所在。

### 实现方式

**实现数据访问层**

数据访问层有两个目标。第一是将数据库引擎从应用中抽象出来，这样就可以随时改变数据库—比方说，从微软SQL变成Oracle。不过在实践上很少会这么做，也没有足够的理由未来使用实现数据访问层而进行重构现有应用的努力。[3] 
第二个目标是将数据模型从数据库实现中抽象出来。这使得数据库或代码开源根据需要改变，同时只会影响主应用的一小部分——数据访问层。这一目标是值得的，为了在现有系统中实现它进行必要的重构。

**模块与接口重构**

依赖注入背后的一个核心思想是单一功能原则（single responsibility principle）。该原则指出，每一个对象应该有一个特定的目的，而应用需要利用这一目的的不同部分应当使用合适的对象。这意味着这些对象在系统的任何地方都可以重用。但在现有系统里面很多时候都不是这样的。[3] 

**随时增加单元测试**

把功能封装到整个对象里面会导致自动测试困难或者不可能。将模块和接口与特定对象隔离，以这种方式重构可以执行更先进的单元测试。按照后面再增加测试的想法继续重构模块是诱惑力的，但这是错误的。

**使用服务定位器而不是构造注入**

实现控制反转不止一种方法。最常见的办法是使用构造注入，这需要在对象首次被创建是提供所有的软件依赖。然而，构造注入要假设整个系统都使用这一模式，这意味着整个系统必须同时进行重构。这很困难、有风险，且耗时。

---

我们还是由例子来看：要实现一个列表 A，能够加载一系列的信息并展示
于是很自然地我们遵守单一职责功能，将展示和加载两个逻辑分成两个类：

```js
// loading.js
export default class Loader {
  constructor(url) {
    this.url = url
  }
  async load() {
    let result = await fetch(this.url)
    return result.text()
  }
}
/*****************************/
// list.js
import Loader from './Loader
export default class List {
  constructor(container) {
    this.container = container
    this.loader = new Loader('list.json')
  }
  async render() {
    let items = await this.loader.load()
    this.container.textContent = items
  }
}
/*****************************/
// main.js
import List from './List'
let list = new List(document.getElementById('elem'))
List.render()
```

列表A很快开发完毕，于是继续开发下一个列表 B，B 的功能和 A 类似，也是加载数据展示数据，区别在于 B 的数据来源是一个第三方的服务，他们提供一个 js sdk 给你调用能够返回数据信息。
很自然的我们想到 A 的展示逻辑是可以复用的，对于数据加载这个逻辑我们重新实现一个 ThirdLoader 来专门加载第三方服务就是了，但回到 List 模块，我们发现在其构造函数中写死了对 Loader 的依赖：

    this.loader = new Loader('/list'); 

导致无法对 List 设置第三方数据加载逻辑。这个问题就在于 List 依赖了具体的实现而不是依赖一个 Loader 接口。

IOC正是解决这类问题的最佳良药，我们再回顾IOC的两条准则，再看看如何利用IOC理念解决这类问题：

`1. 高层次的模块不应该依赖于低层次的模块，它们都应该依赖于抽象`

上述代码中，列表模块是高层次的模块，Loader是低层次的模块。高层次的List依赖了低层次的Loader，违背了该准则。好在准备也提供了解决方案：**应该依赖于抽象**。
那什么是抽象？即`接口`，放在JS语言中，接口则是隐式的。

我们正好实践下该准则：

1. 我们定义一个隐式的接口ILoader,ILoader声明了一个load方法，该方法签名是返回一个包含请求结果的Promise
2. 将List模块对Loader模块的依赖调整为对ILoader接口的依赖：我们在List模块中移除对Loader模块的依赖（即移除import语句），同时构造函数中增加一个参数，该参数是一个实现了ILoader接口的实例

```js
// list.js
export default class List() {
  constructor(container, loader) {
    this.container = container
    this.loader = loader
  }
  async render() {
    this.container.textContent = await this.loader.load()
  }
}
```

3. 为了完成列表A的功能，我们还要改造main.js，将实现了ILoader的Loader模块实例化传给List模块：

```js
// main.js
import List from './List'
import Loader from './Loader'
let list = new List(document.getElementById('elem'), new Loader('list.json'))
list.render()
```

至此，我们完成了对List模块的一次改造，List从对具体实现Loader的依赖变成了对抽象接口ILoader的依赖，而List模块中对Loader模块的导入和实例化过程转移到了main.js，这一过程就是我们的`依赖倒置`。依赖创建的控制权交给了外部main.js，而在main.js中查找创建依赖并将依赖传递给List模块的这一过程我们称之为依赖注入`Dependency injection`

我们再来看看IOC的第二个准则：`2. 抽象不应该依赖于具体实现，具体实现应该依赖于抽象`，我们的ILoader接口显然不会依赖于任何具体实现，而Loader这个具体实现依赖于ILoader接口，完全符合了IOC的第二准则。

原有系统的依赖关系图结果如下：

<img src="http://blog.fueson.top/img/2017/%E5%8E%9F%E6%9C%89%E5%85%B3%E7%B3%BB%E4%BE%9D%E8%B5%96%E5%9B%BE.jpg" />

基于新的依赖架构，List模块具备了设置不同数据加载逻辑的能力，现在我们可以复用List模块再实现列表B的数据加载逻辑并在main中组装即可完成列表B的功能：

```js
// ThirdLoader.js
import {request} from '../third/sdk'
export default class ThirdServiceLoader {
  async load() {
    return request()
  }
}
// main.js
import List from './List'
import Loader from './Loader'
import ThirdLoader from './ThirdLoader'
let listA = new List(document.getElementById('a'), new Loader('list.json'))
listA.render()
let listB = new List(document.getElementById('b'), new ThirdLoader())
listB.render()
```

最终的一个依赖关系图如下：

<img src="http://blog.fueson.top/img/2017/%E6%9C%80%E7%BB%88%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB%E5%9B%BE.jpg" />

至此我们上面演示了应用 IoC 理念对高层模块的一个依赖架构改造，提高了高层模块的可复用性。

## IOC小结

总结我们最开始遇到的问题：类 A 直接依赖类 B，假如要将类 A 改为依赖类 C，则必须通过修改类 A 的代码来达成。这种场景下，类 A 一般是高层模块，负责复杂的业务逻辑；类 B 和类 C 是低层模块，负责基本的原子操作；假如修改类 A，会给程序带来不必要的风险。

IOC 解决方案：将类 A 修改为依赖接口 I，类 B 和类 C 各自实现接口 I，类 A 通过接口 I 间接与类 B 或者类 C 发生联系，则会大大降低修改类 A 的几率。