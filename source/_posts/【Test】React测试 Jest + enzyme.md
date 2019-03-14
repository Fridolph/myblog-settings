---
title: 【Test】React测试 Jest + Enzyme
date: 2019-2-20 23:11:50
tags:
  - react
  - jest
  - enzyme
categories:
  - 前端测试
---

> 最近在社区看到graphQL相关的，正好捡起了之前想写的小项目 [react-note-all](https://github.com/Fridolph/react-note-all)... 顺便先学学把测试这部分好好温习下。

<!-- more -->

## Jest

之前也有看过阮一峰老师的博客，简略学习过mocha、chai的配置和写法等，按describe it 这样来构建测试单元。而最新的 create-react-app 已包含了Jest——facebook出品对上面内容封装的一个测试库。

可直接 `npm test`，会跑 test文件夹下的 *.test.js 测试文件。

那么在react项目里，我们可单独创建一个 test文件夹，然后创建一个 example.test.js 文件，开始第一个测试用例：

```js
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
})
```

不需配mocha 引入chai什么的了，很好上手，当然，如果是对一个模块或者组件进行测试的话，还是建议写 describe 定义一系列测试用例。

嘛，实际上，要进行测试并不仅仅是这样，更多疑惑是对于业务逻辑和封装的基础组件或状态管理等的测试。所以说，加入了下面的Enzyme来测试业务组件。

## Enzyme

> Enzyme 是一个用于响应式的JavaScript测试实用框架，用它观测响应式组件的输出变得更容易。您还可以操作、遍历，并在某些方面模拟给定输出的运行时。Enzyme的API是通过模仿jQuery的DOM操作和遍历API来实现直观和灵活的。

相比Jest，Enzyme能和React进行更好地搭配，渲染Virtual DOM、对比props、渲染真实DOM，变得更易，且和上面说的Jest也是无缝衔接的。如此，我们直接用实例，开始React测试吧（Vue也有vue-cli脚手架官方提供的相应测试工具，掌握了react的测试后，vue组件也会感觉差不多）

首先要安装依赖

```bash
npm install enzyme enzyme-adapter-react-16 --save-dev
```

在使用Enzyme 前需要先适配React对应的版本，我们在src目录下 新建一个 setupTests.js 文件：

```js
import { configure } from 'enzyme'
import Adapter from 'enzyme-adapter-react-16'

configure({ adapter: new Adapter() })
```

> 注：旧版需要为每个测试文件引入上面的配置，现在已经不需要了

### Enzyme - shallow

Enzyme 的使用之浅渲染shallow。Shallow Rendering（浅渲染）指的是，将一个组件渲染成虚拟DOM对象，但是只渲染第一层，不渲染所有子组件，所以处理速度非常快。它不需要DOM环境，因为根本没有加载进DOM。
shallow的函数输入组件，返回组件的浅渲染结果，而返回的结果可以用类似jquery的形式获取组件的信息。

为 [react-note-all](https://github.com/Fridolph/react-note-all) 现在的项目写下测试吧

这里我们选择为 TotalPrice 组件编写测试

```js
import React from 'react'
// import { shallowMount } from '@vue/test-utils'
import {shallow} from 'enzyme'
import TotalPrice from '../'

// 我们验证 需要用到的对象
const props = {
  income: 1000,
  outcome: 2000
}

// 这是一个组件，所以应为每一个组件都写 describe 再编写相应的 测试用例 it
describe('src/components/TotalPrice', () => {
  it('component should render correct income&outcome number', () => {
    // shallow 浅渲染 比较适合测试纯函数 、纯组件
    const wrapper = shallow(<TotalPrice {...props} />)
    // 这里的写法可对比jquery
    expect(wrapper.find('.income span').text()).toEqual(1000)
    expect(wrapper.find('.outcome span').text()).toEqual(2000)
  })
})
```

然后测试结果居然没通过 `Comparing two different types of values. Expected number but received string.`

这里有个坑， text() 拿到的值类型被转成了 string，所以 我们稍稍调整下:

```js
// ...
expect(wrapper.find('.income span').text() * 1).toEqual(1000)
expect(wrapper.find('.outcome span').text() * 1).toEqual(2000)
//...
```


### toMatchSnapshot

既然shallow是浅渲染，一些复杂的测试场景可能不适用。这里先不提 Full rendering，可结合Jest的 toMatchSnapshot API。这是jest 为我们提供了一种快照的方式，来对比组件的变化

下面我们也用实例来 PriceList.test.js

```js
import React from 'react'
import { shallow } from 'enzyme'
import Ionicon from 'react-ionicons'
import PriceList from '../'
import {items, categories} from '../../../containers/Home'

const itemsWithCategory = items.map(item => {
  item.category = categories[item.cid]
  return item
})

const props = {
  items: itemsWithCategory,
  onModifyItem: () => {},
  onDeleteItem: () => {}
}

let wrapper

describe('@src/components/PriceList', () => {
  beforeEach(() => {
    wrapper = shallow(<PriceList {...props} />)
  })

  it('should render the component to match snapshot', () => {
    expect(wrapper).toMatchSnapshot()
  })

  it('should render correct price items length', () => {
    expect(wrapper.find('.list-group-item').length).toEqual(itemsWithCategory.length)
  })

  it('should render correct icon and price for each item', () => {
    const iconList = wrapper.find('.list-group-item').first().find(Ionicon)
    expect(iconList.length).toEqual(3)
    expect(iconList.first().props().icon)
      .toEqual(itemsWithCategory[0].category.iconName)
  })
})
```

### 事件模拟 simulate

对于事件模拟，可对mount的virtual组件用simulate来模拟原生事件，通过自定义props，把事件用jest.fn()来替代，然后断言事件触发后的值与期望值是否一致来测试。我们还是看看实例吧

```js
// CategorySelect    __test__/CategorySelect.test.js
import React from 'react'
import {mount} from 'enzyme'
import Ionicon from 'react-ionicons'
import CategorySelect from '../'


let categories = [
  {
    id: 1,
    name: '旅行',
    type: 'outcome',
    iconName: 'ios-plane'
  },
  {
    id: 2,
    name: '理财',
    type: 'income',
    iconName: 'ios-paper'
  },
  {
    id: 3,
    name: '挣外块',
    type: 'income',
    iconName: 'ios-paper'
  }
]

let props = {
  categories,
  onSelectCategory: jest.fn()
}

let props_with_category = {
  categories,
  selectedCategoryId: 1,
  onSelectCategory: jest.fn()
}

let wrapper

describe('@src/components/CategorySelect', () => {
  it('能用上面的categories渲染 正确的items', () => {
    const wrapper = mount(<CategorySelect {...props} />)
    expect(wrapper.find('.category-item').length).toEqual(categories.length)
    expect(wrapper.find('.category-item.active').length).toEqual(0)

    const firstIcon = wrapper.find('.category-item').first().find(Ionicon)
    expect(firstIcon.length).toEqual(1)
    expect(firstIcon.props().icon).toEqual(categories[0].iconName)
  })

  it('渲染出的items，默认项带上高亮状态', () => {
    const wrapper = mount(<CategorySelect {...props_with_category} />)
    expect(wrapper.find('.category-item').first().hasClass('active')).toEqual(true)
  })

  it('点击其他项，可以将高亮状态切换到其他项', () => {
    const wrapper = mount(<CategorySelect {...props_with_category} />)
    // wrapper.find('.category-item').at(1).simulate('click', { preventDefault: () => {} })
    wrapper.find('.category-item').at(1).simulate('click')
    expect(props_with_category.onSelectCategory).toHaveBeenCalledWith(2)
  })
})
```

## 总结

这样，我们完成了对组件的简单测试。开发业务也是基于此，所谓单元测试就是我们先事先定好大致的测试用例，根据用例一步步去解决这些用例最终完成业务组件。

看似麻烦，但如果规划好了组件，以后将是一劳永逸。迈出第一步就好，加油。
