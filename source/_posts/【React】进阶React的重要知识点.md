---
title: 【React】进阶React的重要知识点
date: 2017-12-28 22:29:03
tags:
  - react
categories:
  - react
---

> 这里只是React的相关知识点整理，也算是老生常谈了吧，通过这样的整理让自己温故知新。说是进阶，其实也算是基础的重要知识点。好了，于是我们一起来看看吧：

* 组件
  * 组件与state
  * 几种组件通信
  * ref
* 虚拟DOM相关
* 性能优化
* 高阶组件

<!-- more -->



## 组件间的通信

React里的组件通信方式大致有以下几种情况：

### 父子组件的通信通过props

```jsx
// UserList.jsx
class UserList extends Component {
  render() {
    return (
      <div>
        <ul className="user-list">
          {this.props.users.map(user => (
            <li key={user.id}>
              <span>{user.name}</span>
            </li>
          ))}
        </ul>
      </div>
    )
  }
}

// UserListContainer.jsx
import UserList from './UserList'

class UserListContainer extends Component {
  constructor(props) {
    super(props)
    this.state = {
      users: []
    }
  }
  render() {
    return (
      {/* 通过props传递users */}
      <UserList users={this.state.users} />
    )
  }
  componentDidMount() {
    fetch('/path/api').then(res => {
      res.json().then(data => {
        this.setState({
          users: data
        })
      })
    })
  }
}
```

### 子向父组件通信

父组件可以通过子组件的props传递给子组件一个回调函数，子组件在需要改变父组件数据时，调用该回调即可。下面为UserList增加一个添加新用户功能：

```jsx
class UserList extends Component {
  constructor(props) {
    super(props)
    this.state = {
      newUser: ''
    }
  }
  handleChange = e => {
    this.setState({
      newUser: e.target.value
    })
  }
  handleClick = () => {
    if (this.state.newUser && this.state.newUser.length > 0) {
      this.props.onAddUser(this.state.newUser)
    }
  }
  render() {
    return (
      <div>
        <ul className="user-list">
          {this.props.users.map(user => (
            <li key={user.id}>
              <span>{user.name}</span>
            </li>
          ))}
        </ul>
        <input onChange={this.handleChange} value={this.state.newUser} />
        <button onClick={this.handleClick}>add uesr</button>
      </div>
    )
  }
}
```

### 兄弟组件通信

兄弟组件不能直接相互传送数据，需要通过状态提升的方式实现兄弟组件的通信：即把组件之间需要共享的state保存到距离它们最近的共同父组件内，任意一个兄弟组件都可以通过父组件传递的回调修改共享状态。

组件从服务器上获取数据，不包含组件向服务器提交数据的情况。

### 组件挂载阶段通信

React组件的正常运转本质上是组件不同生命周期方法的有序执行，因此组件与服务器的通信也必定依赖组件的声明周期方法。

```jsx
export default class UserListContainer extends Component {
  // 省略无关代码
  render() {
    return (
      <div>
        <h1>UserListContainer</h1>
      </div>
    )
  }
  componentDidMount() {
    fetch('/path/api').then(res => {
      res.json().then(data => {
        this.setState({
          users: data
        })
      })
    })
  }
}
```

componentDidMount是官方推荐的通信周期，当然componentWillMount也是可以的。

1. componentDidMount中执行服务器通信可以保证获取数据时，组件已经处于挂载状态，即便操作DOM也是安全的，而componentWillMount无法保证

2. 当组件在服务端渲染时，componentWillMount会被调用两次，一次是服务端，一次是浏览器端，而componentDidMount能保证在任何情况下只调用一次，从而不会发送多余的请求

### 组件更新阶段通信

componentWillReceiveProps非常适合做更新阶段的服务器通信。

```jsx
class UserListContainer extends Component {
  // 省略
  componentWillReceiveProps(nextProps) {
    if (nextProps.category !== this.props.category) {
      fetch(`/path/api?category=${nextProps.category}`).then(res => {
        res.json().then(data => {
          this.setState({
            users: data
          })
        })
      })
    }
  }
}
```

这里需注意：在执行fetch请求时，要先对新老props中的category作比较，有不一致才进行更新。componentWillReceiveProps并不能保证props一定发生了修改


## 深入理解组件

state所代表的一个组件UI呈现的完整状态又可以分成两类数据：用作渲染组件时使用到的数据的来源以及用作组件UI展现形式的判断依据。

---

### 如何定义state

在组件中需要用到一个变量，且与组件的渲染无关时，就应该把这个变量定义为组件的普通属性，直接挂载到this下，而不是作为组件的state。还有就是看render方法有无用到该变量，没有就同样挂载到this下

state和props都直接和组件的UI渲染有关，它们的变化会触发组件的重新渲染，但props对于使用它的组件来说是只读的，是通过父组件传递过来的，想要修改props，只能在父组件中修改，而state是组件内部自己维护的状态，是可变的

总结一下，组件中用到一个变量是不是应该作为state可通过下面4条依据进行判断：

1. 该变量是否通过props从父组件中获取？若是，则不是state
2. 该变量是否在组件的整个生命周期中保持不变？若是，则不是state
3. 该变量是否可通过其他state或props计算得到？若是，则不是state
4. 该变量是否在render方法中使用？若是，则为state，反之则不是，该情况下，变量更适合定义为组件的一个普通属性

---

* 不能直接修改state，需要用setState
* state的更新是异步的

调用setState时，组件的state不会立即改变，setState只是把要修改的状态放入一个队列中，React会优化真正的执行时机，出于性能考虑，可能会将多次setState状态修改合并成一次状态修改。所以不要依赖当前state去计算下一个state。当真正执行修改时，依赖的this.state并不能保证是最新的state。同样不能依赖当前props计算下一状态，因为props的更新也是异步的。

---

### state 与不可变对象

1. state类型是不可变类型（数字、字符串、布尔值、null、undefined）

因state是不可变类型，所以直接给要修改的状态赋一个新值即可

```jsx
this.setState({
  count: 1,
  title: 'React',
  success: true
})
```

2. state类型是数组

法一：使用preState、concat创建新数组

```jsx
this.setState(preState => ({
  books: preState.books.concat(['React Guide'])
}))
```

法二：ES6 spread syntax

```jsx
this.setState(preState => ({
  books: [...preState.books, 'React Guide']
}))
```

当从books中截取部分元素作为新状态时，可使用数组的slice方法

```jsx
this.setState(preState => ({
  books: preState.books.slice(1, 3)
}))
```

当从books中过滤部分元素后作为新状态，可使用数组的filter方法

```jsx
this.setState(preState => ({
  books: preState.books.filter(item => {
    return item !== 'React'
  })
}))
```

注意，不要使用push、pop、shift、unshift、splice等方法修改数组类型的状态，因为这些方法都是在原数组的基础上修改的，而concat、slice、filter会返回一个新的数组

### state的类型是普通对象(不包含字符串、数组)

1. 使用ES6的Object.assign

```jsx
this.setState(preState => ({
  owner: Object.assign({}, preState.owner, {name: 'Jason'})
}))
```

2. 使用对象扩展语法 Object spread properties

```jsx
this.setState(preState => ({
  owner: {...preState.owner, name: 'Jason'}
}))
```

总结下，创建新的state的关键是，避免使用会直接修改原对象的方法，而是使用可以返回一个新对象的方法。当然也可以使用一些Immutable的JS库来实现

`React状态为什么要是不可变对象呢？` 一方面是因为不可变对象的修改会返回一个新对象，不用担心原有对象在不小心情况下被修改导致的错误，方便程序管理与调试。另一方面是出于性能考虑，当对象组件状态都是不可变对象时，在组件的shouldComponentUpdate方法中仅需要比较前后两次状态引用就可以判断状态是否真的改变，从而避免不必要的render调用。

React之所以执行效率高，其重要原因是虚拟DOM机制。React应用常用的性能优化也与虚拟DOM机制有关。

## 虚拟DOM相关

在Web环境中，DOM也就是对HTML文本的一种抽象描述。在传统开发中，通过调用浏览器提供API对DOM执行增删查改操作。这些操作看似只执行了一条JS语法，但其效率要慢得多。因为对DOM的修改会引起页面的重布局和重渲染，这过程很耗时，这也是前端性能优化的一条原则，尽量减少DOM操作。

虚拟DOM是一层抽象（对真实DOM），建立在真实的DOM上。（虚拟DOM是一项独立的技术）

    <div class="foo">
      <h1>Hello React</h1>
    </div>

可以用JS对象来描述

```js
{
  type: 'div',
  props: {
    className: 'foo',
    children: {
      type: 'h1',
      props: {
        children: 'Hello React'
      }
    }
  }
}
```

有了虚拟DOM这一层，当我们需要操作DOM时，就可以操作虚拟DOM而不是操作真实DOM。

### Diff算法

React采用声明式地API描述UI结构，每次组件的状态或属性更新，组件的render方法都会返回一个新的虚拟DOM对象，用来表述新的UI结构。如果每次render都直接使用新的虚拟DOM来生成真实DOM结构，那么会带来大量对真实DOM的操作，影响执行效率。

事实上，React通过比较两次虚拟DOM的变化找出差异部分，更新到真实DOM上，从而减少最终要在真实DOM上执行的操作，提高程序执行效率。这一过程就是React的调和过程Reconcliliation，其中的关键就是比较两个树型结构的diff算法。

> 在diff算法中，比较的两方是新的虚拟DOM和旧的虚拟DOM，而表示虚拟DOM和真实DOM，只不过Diff的结果会更新到真实的DOM上。

正常情况下，比较两个树形结构差异的算法时间复杂度是O(N^3)。React通过总结DOM的实际使用场景提出了两个在绝大多数实践场景下都成立的假设，基于这两个假设，React实现了O(N)时间复杂度内完整两棵虚拟DOM树的比较：

1. 如果两个元素的类型不同，那么它们将生成两棵不同的树
2. 为列表中的元素设置key，用key标识对应的元素在多次render过程中是否发生变化

### 当根节点是不同类型时

从div变成p，ComponentA变成ComponentB，或者从ComponentA变成div这些都是节点类型发生变化的情况。

根节点类型变化，React会认为新的树和旧的树完全不同，不会再继续比较其他属性和子节点，而是把整棵树拆掉重建（包括虚拟DOM树和真实DOM树）。需要注意的是，虚拟DOM节点类型分为两类：一类是DOM元素类型，一类是React组件类型。

在旧的虚拟DOM树被拆除过程中，旧的DOM元素类型的节点会被销毁，旧的React组件的实例componentWillUnmount会被调用，在重建过程中，新的DOM元素会被插入到DOM树中，新的组件实例的componentWillMount和componentDidMount方法会被调用。重建后的新的虚拟DOM树会被整体更新到真实DOM树中，这种情况需要大量DOM操作，更新效率最低。

### 当根节点是相同的DOM元素类型时

React会保留根节点，而比较根节点的属性，然后只更新那些变化了的属性。

### 当根节点是相同的组件类型时

对应的组件实例不会被销毁，只是会执行更新操作，同步变化的属性到虚拟DOM树上，这一过程组件实例的componentWillReceiveProps和componentWillUpdate会被调用。注意，对于组件类型的节点，React是无法直接知道如何更新真实DOM树的，需要在组件更新并且render方法执行完成后，根据render返回的虚拟DOM结构决定如何更新真实DOM树。

比较完根节点后，React会以同样的原则继续递归比较子节点，每一个子节点相对于其层级以下的节点来说又是一个根节点。如此递归比较，直到比较完两棵树上的所有节点，计算得到最终差异，更新到DOM树中。

## 高阶组件

高阶组件是React中一个重要而复杂的概念。主要用来实现组件逻辑的抽象和复用，在很多第三方库被使用到。合理使用高阶组件也能显著提高代码质量。

**基本概念** 高阶函数是函数作为参数，并且返回值也是函数的函数。类似地，高阶组件简称（HOC）接受React组件作为参数，并且返回一个新的React组件。高阶组件本质上也是一个函数，并不是一个组件，高阶组件的函数形式如下：

    const EnhancedComponent = higherOrderComponent(WrappedComponent)

从例子来看。MyComponent组件需要从LocalStorage中读取数据然后渲染到界面，一般来说：

```jsx
class MyComponent extends Component {
  render() {
    return (
      <div>{this.state.data}</div>
    )
  }
  componentWillMount() {
    let data = localStorage.getItem('data')
    this.setState({ data })
  }
}
```

但当其他组件也需要从LocalStorage中获取同样的数据展示时，每个组件都需要重写一次componentWillMount中的代码，这是很冗余的。于是用高阶组件来改写这部分：

```jsx
function withPersistentData(WrappedComponent) {
  return class extends Component {
    componentWillMount() {
      let data = localStorage.getItem('data')
      this.setState({ data })
    }
    render() {
      // 通过 {...this.props}把传递给当前组件的属性传递给被包装的组件
      return <WrappedComponent data={this.state.data} {...this.props} />
    }
  }
}

class MyComponent extends Component {
  render() {
    return <div>{this.props.data}</div>
  }
}
const MyComponentWithPersistenData = withPersistentData(MyComponent)
```

withPersistenData就是一个高阶组件，它返回一个新的组件，在新组件的componentWillMount中统一处理从localStorage中获取数据的逻辑，然后将获取到的数据通过props传递给被包装的租金啊WrappedComponent。这样在WrappedComponent中就可以直接使用this.props.data获取需要展示的数据。

高阶组件的主要功能是封装被分离组件的通用逻辑，让通用逻辑在组件间更好地被复用。高阶组件的这种实现方式的本质其实就是装饰者设计模式。

---

**使用场景**

1. 操作props
2. 通过ref访问组件实例
3. 组件状态提升
4. 从其他元素包装组件

### 操作props

在被包装组件接受props前，高阶组件可以先拦截props，对props执行增、删、改等操作，然后将处理后的props再传递给被包装组件。

### 通过ref访问组件实例

高阶组件通过ref获取被包装组件实例的引用，然后高阶组件就具备了直接操作被包装组件的属性或方法的能力。

```jsx
function withRef(WrappedComponent) {
  return class extends Component {
    someMehod = () => {
      this.wrappedInstance = this.someMethodInWrappedComponent()
    }
    render() {
      // 为被包装组件添加ref ， 从而获取该组件实例并赋值给this.wrappedInstance
      return <WrappedComponent ref={instance => this.wrappedInstance = instance} {...this.props} />
    }
  }
}
```

当WrappedComponent被渲染时，执行ref的回调函数，高阶组件通过this.wrappedInstance保存WrappedComponent实例的引用，在someMethod方法中，通过this.wrappedInstance调用WrappedComponent实例中的方法。

### 组件状态提升

无状态组件更容易被复用。高阶组件可以通过将被包装组件的状态及相应的状态处理方法提升到高阶组件自身内部实现被包装组件的无状态化。一个典型场景是，利用高阶组件将原本受控组件需要自己维护的状态统一提升到高阶组件中：


```jsx
function withControlledState(WrappedComponent) {
  return class extends Component {
    constructor() {
      super()
      this.state = {
        value: ''
      }
    }
    handleValueChange = e => {
      this.setState({
        value: e.target.value
      })
    }
    render() {
      // newProps保存受控组件需要使用的属性和事件处理函数
      const newProps = {
        controlledProps: {
          value: this.state.value,
          onChange: this.handleValueChange
        }
      }
      return <WrappedComponent {...this.props} {...newProps} />
    }
  }
}
```

这个例子把受控组件的value属性用到的状态和处理value变化的回调都提升到了高阶组件中，当我们再使用受控组件时，可以这么使用：

```jsx
class SimpleControlledComponent extends Component {
  render() {
    // 此时的SimpleControlledComponent 为无状态组件，状态由高阶组件维护
    return <input nmae="simple" {...this.props.controlledProps} />
  }
}
const ComponentWithControlledState = withControlledState(SimpleControlledComponent)
```

### 用其他元素包装组件

我们还可以在高阶组件渲染 WrappedComponent时添加额外的元素，这种情况通常用于为 WrappedCompoent增加布局或修改样式

```jsx
function withRedBackground(WrappedComponent) {
  return class extends Component {
    render() {
      return (
        <div style={{backgroundColor: 'red'}}>
          <WrappedComponent {...this.props} />
        </div>
      )
    }
  }
}
```

高阶组件的参数并非只是一个组件，它还可以接受其他参数。

```jsx
function withPersistentData(WrappedComponent, key) {
  return class extends Component {
    componentWillMount() {
      let data = localStorage.getItem(key)
      this.setState({
        [`local/${key}`]: data
      })
    }
    render() {
      // 通过{...this.props}把传递给当前组件的属性继续传递给被包装的组件
      return <WrappedComponent data={this.state.data} {...this.props} />
    }
  }
}
```

---

`HOC(...params)`返回值是一个高阶组件，高阶组件需要的参数是先传递给HOC函数的。用这种形式改写withPersistentData如下

```jsx
const withPersistentData = key => WrappedCompnent => {
  return class extends Component {
    componentWillMount() {
      let data = localStorage.getItem(key)
      this.setState({ data })
    }
    render() {
      // 通过 {...this.props} 把传递给当前组件的属性继续传递给被包装的组件
      return <WrappedComponent data={this.state.data} {...this.props} />
    }
  }
}
class MyComponent extends Component {
  render() {
    return <div>{this.props.data}</div>
  }
}
// 获取key='data'的数据
const MyComponentPersistentData = withPersistentData('data')(MyCompoent)
```

实际上，这种形式的高阶组件大量出现在第三方库中，如react-redux的connect函数就是一个典型的例子。connect的简化定义如下：

    connect(mapStateToProps, mapDispatchToProps)(WrappedComponent)

这个函数会将一个React组件连接到Redux的store上，在连接的过程中，connect通过函数参数mapStateToProps从全局store中取出当前组件需要的state，并把state转化成当前组件的props，传递给当前组件。 connect并不会修改传递进去的组件的定义，而是会返回一个新的组件。

> connect的参数mapStateToProps、mapDispatchToProps是函数类型，说明高阶组件的参数也可以是函数类型。


    const connectedComponentA = connect(mapStateToProps, mapDispatchToProps)(ComponentA)

我们可以把它拆分来看

    // connect是一个函数，返回值 enhance也是一个函数
    const enhance = connect(mapState, mapDispatch)
    // enhance是一个高阶组件
    const ConnectedComponentA = enhance(ComponentA)

这种形式的高阶组件易组合使用。因为当多个函数的输出和它的输入类型相同时，这些函数易组合到一起。

    // connect的参数是可选参数，这里省略了mapDispatch参数
    const ConnectedComponentA = connect(mapStateToProps)(withLog(ComponentA))

我们还可以定义一个工具函数 compose(...funcs)

```js
function compose(...fns) {
  if (fns.length === 0) return arg => arg
  if (fns.length === 1) return fns[0]
  return fns.reduce((acc, cur) => (...args) => acc(cur(args)))
```

调用`compose(f, g, h)` 等价于 `(...args) => f(g(h(...args)))`

用compose函数可以把高阶组件嵌套的写法打平：


```js
const enhance = compose(
  connect(mapState),
  withLog()
)
const ConnectedComponentA = enhance(ComponentA)
```

像Redux等第三方库都提供了compose的实现，compose结合高阶组件使用可以显著提高代码的可读性和逻辑的清晰度

---

### 继承方式实现高阶组件

上面这类由高阶组件处理通用逻辑，然后再将相关属性传递给被包装组件，我们称这种实现方式为`属性代理`。

除此外，还可以通过继承方式实现高阶组件，通过继承被包装组件实现逻辑的复用。继承方式实现的高阶组件常用于渲染劫持。例如：当用户处于登录状态，允许组件渲染，否则渲染一个空组件：

```jsx
function withAuth(WrappedComponent) {
  return class extends WrappedComponent {
    render() {
      if (this.props.loggedIn) {
        return super.render()
      } else {
        return null
      }
    }
  }
}
```

根据WrappedComponent的`this.props.loggedIn`判断用户是否已经登录，若登录就通过super.render()调用WrappedComponent的render方法正常渲染，否则返回null。

继承方式实现的高阶组件对被包装组件具有侵入性，当组合多个高阶组件使用时，很容易因为子类组件忘记通过super调用父类组件方法而导致逻辑丢失。因此，在使用高阶组件时，应尽量通过代理方式实现高阶组件。

---

**注意事项**

### 1. 为了在开发和调试阶段更好地区别包装了不同组件的高阶组件，需要对高阶组件的显示名称做自定义处理。

常用的处理方式是，把被包装组件的显示名称也包到高阶组件的显示名称中：

```jsx
function withPersistentData(WrappedComponent) {
  return class extends Component {
    // 结合被包装组件的名称，自定义高阶组件的名称
    static displayName = `HOC(${getDisplayName(WrappedComponent)})`
    render() {
      return  // 省略
    }
  }
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component'
}
```

### 2. 不要在组件的render方法中使用高阶组件，尽量也不要在组件的其他生命周期方法中使用高阶组件。

因为调用高阶组件，每次都会返回一个新的组件，于是每次render前一次高阶组件创建的组件都会被卸载unmount，然后重新挂载(mount)本次创建的新组件，影响效率又丢失子组件状态，如下：

```jsx
render() {
  // 每次render, enhance都会创建一个新租金啊，尽管被包装的组件没有变
  const EnhancedComponent = enhance(MyComponent)
  // 因为是新的组件，所以会经历旧组件的卸载和新组件的重新挂载
  return <EnhancedComopnent />
}
```

所以高阶组件最适合的地方是在组件定义的外部，这样就不会收到组件生命周期的影响。

### 3. 如果需要使用被包装组件的静态方法，那么必须手动复制这些静态方法。

因为高阶组件返回的新组件不包含被包装组件的静态方法，如：


```jsx
// WrappedComponent组件定义了一个静态方法staticMethod
WrappedComponent.staticMethod = function() {
  // ...
}
function withHOC(WrappedComponent) {
  class Enhance extends Component {
    // ...
  }
  // 手动复制静态方法到Enhance上
  Enhance.staticMethod = WrappedComponent.staticMethod
}
```

### 4. refs不会被传递给被包装组件

尽管在定义高阶组件时，我们会把所有的属性都传递给被包装组件，但是ref并不会传递给被包装组件。如果在高阶组件的返回数组中定义了ref，那么它指向的是这个返回的新组件，而不是内部被包装的组件。如果希望获取被包装组件的引用，那么可以自定义一个属性，属性的值是一个函数，传递给被包装的ref。下面的例子就是用inputRef这个属性名替代常规的ref命名：


```jsx
function FocusInput({inputRef, ...rest}) {
  // 使用高阶组件传递的inputRef作为ref的值
  return <input ref={inputRef} {...rest} />
}
// enhance是一个高阶组件
const EnhanceInput = enhance(FocusInput)
// 在一个组件的render方法中，自定义属性inputRef代替ref
// 保证inputRef可以传递给被包装组件
return (
  <EnhanceInput
    inputRef={input => this.input = input}
  />
)
// 组件内 让FocusInput自动获取焦点
this.input.focus()
```

### 5. 与父组件的区别

高阶组件在一些方面与父组件类似。我们可以把高阶组件中的逻辑放到一个父组件中执行，执行完成的结果再传递给子组件，但是高阶组件强调的是逻辑的抽象。

高阶组件是一个函数，函数关注的是逻辑；父组件是一个组件，组件主要关注的是UI/DOM。如果逻辑是与DOM直接相关的，那么这部分逻辑适合放到父组件中实现；如果逻辑是与DOM不直接相关的，那么这部分逻辑使用用高阶组件抽象，如数据校验、请求发送等。

---


高阶组件用于封装组件的通用逻辑，常用在操作组件props、通过ref访问组件实例、组件状态提升和用其他元素包装组件等场景中。

高阶组件可以接受被包装组件以外的其他参数，多个高阶组件还可以组合使用。高阶组件一般通过代理实现，少量场景中也会使用继承等方式实现。灵活地使用高阶组件可以显著提高代码质量和效率。
