---
title: 【React】从基础开始再来温习React，相关学习整理
date: 2017-12-20 23:29:03
tags:
  - react
categories:
  - react
---

> React虽然从14版就在学习使用了，- - 换公司之后一直在用Vue，以前用Redux，服务端渲染总是觉得很坑，虽然跟着前辈写似懂非懂的，业务逻辑也能做，但Redux那块始终用得不好。也算是曲线救国，Vue、Vuex的使用让我慢慢理解了React的核心和优美之处。正好现在新版发布了，再来重学一次吧

<!-- more -->

# React

前端UI本质问题是： 如何将来源于服务器的动态数据和用户的交互行为高效地反映复杂的用户界面上。React通过引入虚拟DOM、状态、单向数据流等设计理念，形成以组件为核心，用组件搭建UI的开发模式，完美地将数据、组件状态和UI映射到一起，极大提高了开发大型Web应用的效率。

React的特点：

1. 声明式的视图层

使用React不用担心数据、状态和视图交错。

2. 简单的更新流程

定义UI状态，React框架会复杂把它渲染成最终的UI。当状态数据发生变化时，React也会根据最新的状态渲染出最新的UI

3. 灵活地渲染实现

先渲染成虚拟DOM，简单的JS对象然后渲染成对应UI

4. 高效的DOM操作

操作JS对象比起操作真实DOM在效率上有了巨大提升。基于Diff算法，React可以尽量减少虚拟DOM到真实DOM的渲染次数

尽管如此，React并不是MVC框架，从分层上看，React属于V层，关注如何根据状态创建可复用UI组件。当应用复杂时，需结合Redux等才能最大发挥


## React新特性相关

这里介绍了16的新特性，包括render方法新支持的返回类型、新的错误处理机制和Error Boundary组件，可以将组件挂载到任意DOM树的Portals特性以及自定义DOM属性的支持。

基于新的fiber架构。还有如 setState传入null不触发组件更新，更加高效的服务端渲染等方式等。


### render新的返回类型

返回数组

```jsx
class ListComponent extends Component {
  render() {
    return [
      <li key="a">a</li>
      <li key="b">b</li>
      <li key="c">c</li>
    ]
  }
}
```

返回字符串

```jsx
class StringComponent extends Component {
  render() {
    return 'Just a strings
  }
}
```

App 组件的render方法渲染 ListComponent 和 StringComponent

```jsx
export default class App extends Component {
  render() {
    return [
      <ul>
        <ListComponent />
      </ul>,
      <StringComponent />
    ]
  }
}
```

---

### 错误处理

16版本前，组件运行期间执行错误会阻塞应用渲染。16引入新的错误处理机制，组件抛错时，会就爱你个组件从组件树中卸载，从而避免整个应用的崩溃。

16还提供了一种更友好的错误处理方式——错误边界。错误边界是能够捕获子组件的错误并对其优雅处理的组件，输出错误日志、显示出错提示等这比卸载组件更友好

定义了 componentDidCatch(error, info) 方法的组件将成为一个错误边界，现在我们创建一个组件ErrorBoundary

```jsx
class ErrorBoundary extends Component {
  constructor(props) {
    super(props)
    this.state = {hasError: false}
  }
  componentDidCatch(error, info) {
    // 显示错误UI
    this.setState({ hasError: true })
    // 同时输出错误日志
    console.log(error, info)
  }
  render() {
    if (this.state.hasError) {
      return <h1>Oopts, something went wrong.</h1>
    }
    return this.props.children
  }
}
```

然后在App中使用ErrorBoundary

```jsx
class App extends Component {
  constructor(props) {
    super(props)
    this.state = {
      user: { name: 'react' }
    }
  }
  // 将user置为null，模拟异常
  onClick = () => {
    this.setState({ user: null })
  }
  render() {
    return (
      <div>
        <ErrorBoundary>
          <Profile user={this.state.user} />
        </ErrorBoundary>
        <button onClick={this.onClick}>更新</button>
      </div>
    )
  }
}

const Profile = ({user}) => <div>name: {user.name}</div>
```

点击更新按钮后，Profile接受的属性user为null，程序会抛错TypeError，这个错误被ErrorBoundary捕获，并在界面上显示出错提示

### Portals

Portals特性让我们可以把组件渲染到当前组件树以外的DOM节点上。该特性的典型应用场景是，渲染应用的全局弹框，使用Portals后，任意组件都可以将弹框组件渲染到根节点上，以方便弹框的显示。Portals的实现依赖于ReactDOM的一个新API

    ReactDOM.createPortal(child, container)

第一个参数child是可以被渲染的React节点，例如React元素、由React元素组成的数组、字符串等。
container是一个DOM元素，child将被挂载到这个DOM节点

```jsx
class Modal extends Component {
  constructor(props) {
    super(props)
    // 根节点下创建一个div节点
    this.container = document.createElement('div')
    document.body.appendChild(this.container)
  }
  render() {
    return ReactDOM.createPortal(
      <div className="modal">
        <span className="close" onClick={this.props.onClose}>&times;</span>
        <div className="content">
          {this.props.children}
        </div>
      </div>,
      this.container
    )
  }
  componentWillUnmount() {
    document.body.removeChild(this.container)
  }
}
```

在App中使用Modal

```jsx
export default class App extends Component {
  constructor(props) {
    super(props)
    this.state = {showModal: true}
  }
  closeModal = () => {
    this.setState({showModal: false})
  }
  render() {
    return (
      <div>
        <h2>Dashboard</h2>
        {this.state.showModal && (
          <Modal onClose={this.closeModal}>Modal Dialog</Modal>
        )}
      </div>
    )
  }
}
```

### 自定义DOM属性

16版 React会把不识别的属性传递给DOM元素。例如， 在16前，下面的React元素

    <div custom-attribute="something" />

在浏览器中渲染出的DOM节点为：

    <div />

而React16版渲染出的DOM节点为：

    <div custom-attribute="something" />

---

## 生命周期

组件从创建到被销毁的过程称为组件的生命周期。通常，有以下三个阶段：

### 挂载阶段

* constructor

* componentWillMount 用得较少，因为此阶段的都可移到constructor里，该方法调用this.setState不会引起重渲

* render 根据组件的props和state返回一个React元素。需注意，render并不负责组件的实际渲染，它只是返回一个UI描述，真正渲染DOM由React本身负责，该阶段不能执行有副作用的操作

* componentDidMount 组件被挂载后只调用一次，依赖DOM节点操作可放该阶段中，还有服务端请求等操作，this.setState会引起重渲

### 更新阶段

* componentWillReceiveProps(nextProps) 引起组件更新过程调用。state引起组件更新不会触发该方法的执行。nextProps是父组件传递给当前组件的新props. 因此往往需比较nextProps和this.props来决定是否执行props发生变化后的逻辑

> (1)在componentWillReceiveProps中调用setState，只有在组件render及其之后的方法中，this.setState指向的才是更新后的state。在render之前的方法shouldComponentUpdate、componentWillUpdate中，this.setState依然指向的是更新前的state

> (2)通过调用setState更新组件状态并不会触发componentWillReceiveProps的调用，否则会进入一个死循环 componentWillReceiveProps -> this.setState -> componentWillReceiveProps ...

* shouldComponentUpdate(nextProps, nextState) 该方法决定组件是否继续执行更新过程。当方法返回true时(默认)，组件会继续更新过程。当返回false，组件的更新过程停止，后续的componentWillUpdate、render、componentDidUpdate也不会被调用，这是优化性能的一个重要钩子

* componentWillUpdate(nextProps, nextState) 在组件render前调用，可作为组件更新前执行某些工作的地方

> shouldComponentUpdate和componentWillUpdate中都不能调用setState，否则会引起循环调用问题

* render 同上，省略了

* componentDidUpdate(prevProps, prevState) 组件更新后被调用，可作为更新后操作DOM的地方，其参数prevProps, prevState代表组件更新前的props和state

### 卸载阶段

组件从DOM中被卸载的过程，只有一个声明周期方法

* componentWillUnmount 在组件卸载前调用，可在这里执行一些清理工作。如定时器或手动创建的DOM元素等，以避免内存泄漏

> 只有类才具有生命周期方法，函数组件是没有生命周期钩子的。

## ref相关
绝大部分场景应避免使用ref，因为它破坏了React中以props为数据传递介质的典型数据流。

下面介绍下ref的常用使用场景

## 在DOM上使用ref

ref接受一个回调函数作为值，在组件被挂载或卸载时，回调函数会被调用，在组件被挂载时，回调函数会接受当前DOM元素作为参数；组件被卸载时回调函数会接受null作为参数

```jsx
class AutoFocusTextInput extends Component {
  render() {
    return (
      <div>
        <input type="text" ref={input => this.textInput = input} />
      </div>
    )
  }
  componentDidMount() {
    // 通过ref让input自动获取焦点
    this.textInput.focus()
  }
}
```

AutoFocusTextInput中为input定义ref，在组件挂载后，通过ref获取input元素，让其自动获取焦点，否则就很难实现该功能

### 在组件上使用ref

例，在使用AutoFocusTextInput组件的外部组件Containter中控制：

```jsx
class AutoFocusTextInput extends Component {
  constructor() {
    super()
  }
  blur = () => {
    this.textInput.blur()
  }
  render() {
    return (
      <div>
        <input type="text" ref={input => this.textInput = input} />
      </div>
    )
  }
}

// AutoFocusTextInputContainer.jsx
class AutoFocusTextInputContainer extends Component {
  handleClick = () => {
    // 通过ref调用 组件的方法
    this.inputInstance.blur()
  }
  render() {
    return (
      <div>
        <AutoFocusTextInput ref={this.inputInstance = input} />
        <button onClick={this.handleClick}>失去焦点</button>
      </div>
    )
  }
}
```

通过ref获取到了AutoFocusTextInput组件的实例对象，并把它赋值给Container的inputInstance属性，这样就可以通过inputInstance调用AutoFocusTextInput中的blur方法，让已经处于获取焦点状态的input失去焦点。

### 父组件访问子组件的DOM节点

某些场景可能会需要。例如父组件需知道这个DOM元素的尺寸或位置信息，直接使用ref是无法实现的。

这时，可在子组件的DOM元素上定义ref，ref的值是父组件传递给子组件的一个回调函数，回调函数可以通过一个自定义的属性传递，例如inputRef, 这样父组件的回调函数中就能获取到这个DOM元素

```jsx
function Children(props) {
  // 子组件使用父组件传递的inputRef, 为input的ref赋值
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  )
}

class Parent extends Component {
  render() {
    // 自定义一个属性inputRef，值是一个函数
    return (
      <Children inputRef={el => this.inputElement = el} />
    )
  }
}
```

从该例中还可发现，即时子组件是函数组件，这种方式同样有效。

## 事件处理

* React的事件命名采用驼峰命名
* 处理事件的响应函数要以对象形式赋值给事件属性

---

React事件是合成事件，并不是原生DOM事件。在React事件中必须显式调用事件对象的preventDefault方法来阻止事件的默认行为。

在 React组件中处理事件最容易出错的是事件处理函数中this指向问题，因为ES6 Class不会为方法自动绑定到当前对象。

1. 使用箭头函数

箭头函数的this指向的是函数定义时的对象，可保证this总是指向当前组件的实例对象。
直接在render方法为元素事件定义事件处理函数，最大问题是每次render调用时都会重新创建一个新的事件处理函数，带来额外的性能开销，组件所处层级越低，开销就越大

```jsx
class Child extends Component {
  handleClick(e) {
    console.log(e.target)
  }

  render() {
    return (
      <div>
        <button onClick={e => this.handleClick(e)}></button>
      </div>
    )
  }
}
```

2. 使用组件方法

直接将组件的方法赋值给元素的事件属性，同时在类的构造函数中，将这个方法的this绑定当当前对象

```jsx
class Child extends Component {
  constructor() {
    super()
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick(e) {
    console.log(e.target)
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick(e)}></button>
      </div>
    )
  }
}
```

好处是每次render不会重新创建一个回调函数，没有额外的性能损失。但模版较为繁琐，还有下一种改良：

```jsx
class Child extends Component {
  handleClick(e) {
    console.log(e.target)
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick.bind(this)}></button>
      </div>
    )
  }
}
```

使用bind会创建一个新函数，因此该写法毅然存在每次render都会创建一个新函数问题

**3. 属性初始化语法** property initializer syntax

使用ES7的property initializers会自动为class中定义的方法绑定this：

```jsx
class Child extends Component {
  constructor(props) {
    super(props)
    this.state = { num: 0 }
  }
  // ES7的属性初始化方法，实际上也是使用了箭头函数
  handleClick = e => {
    const num = ++this.state.num
    this.setState({
      num
    })
  }
  render() {
    return (
      <div>
        <p>{this.state.num}</p>
        <button onClick={this.handleClick}>click</button>
      </div>
    )
  }
}
```

这种方式既不需要在构造函数中手动绑定this，也不需要担心组件重复渲染导致的函数重复创建问题。
