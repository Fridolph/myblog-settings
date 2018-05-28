---
title: 【TS】使用vuex-class
date: 2018-5-01 10:29:03
tags:
  - js
  - ts
categories:
  - js
  - ts
---

> `vuex-class`是`vue-class-component`作者写的，质量有保证。且目前这类的库也不多，用法和之前的vue-property-decorator差不多，主要是通过装饰器模式，一来支持ts里的vuex，二来减少冗余的代码量。

![](http://blog.fueson.top/18-5-17/8785480.jpg)

<!-- more -->

## demo

我们直接来看官方的例子就好，引入相关依赖

```js
import Vue from 'vue'
import Component from 'vue-class-component'
import {
  State,
  Getter,
  Action,
  Mutation,
  namespace
} from 'vuex-class'

const someModule = namespace('path/to/module')

@Component
export class MyComp extends Vue {
  @State('foo') stateFoo
  @State(state => state.bar) stateBar
  @Getter('foo') getterFoo
  @Action('foo') actionFoo
  @Mutation('foo') mutationFoo
  @someModule.Getter('foo') moduleGetterFoo

  // 若省略参数，直接使用参数名称
  @State foo
  @Getter bar
  @Action baz
  @Mutation qux

  created () {
    this.stateFoo // -> store.state.foo
    this.stateBar // -> store.state.bar
    this.getterFoo // -> store.getters.foo
    this.actionFoo({ value: true }) // -> store.dispatch('foo', { value: true })
    this.mutationFoo({ value: true }) // -> store.commit('foo', { value: true })
    this.moduleGetterFoo // -> store.getters['path/to/module/foo']
  }
}
```

看着非常简单… 好像懂了 大概就是这种感觉？！

然而 namespace 这个就找不到，需要自己在 `*.d.ts`文件里定义接口

和我们之前用的vuex提供的 mapMutations mapGetters 这些一样，只是换了种形式进来了而已。

源码也更少，增加点信心一探究竟吧~

vuex-class/src/index.ts

```ts
export {
  State,
  Getter,
  Action,
  Mutation,
  namespace
} from './bindings'
```

vuex-class/src/bindings.ts

```ts
import Vue from 'vue'
import {createDecorator} from 'vue-class-component'
import {
  mapState,
  mapGetters,
  mapActions,
  mapMutations
} from 'vuex'

// 我们在使用时，ts有各种烦人的检查，若类型不一致报错是无法运行的
// 把其他的省略了- - 只保留了看得懂和我们关心，使用的那几个
export interface BindingHelpers {
  State: StateBindingHelper
  Getter: BindingHelper
  Mutation: BindingHelper
  Action: BindingHelper
}

// 有了这几个方法后，我们就可以在ts里使用了 那具体是怎么实现的？
export const State = createBindingHelper('computed', mapState) as StateBindingHelper

export const Getter = createBindingHelper('computed', mapGetters)

export const Action = createBindingHelper('methods', mapActions)

export const Mutation = createBindingHelper('methods', mapMutations)
```

核心处理函数 `createBindingHelper`单独拿出来分析了

```ts
function createBindingHelper(
  // 方法或者属性，它已经告诉我们了 在computed或methods上
  bindTo: 'computed' | 'methods',
  // 这个后面再说，用过应该能想个大概
  mapFn: MapHelper
) BindingHelper: {
  // 返回类型因属于 BindingHelper

  function makeDecorator(may: any, namespace: string | undefined) {
    return createDecorator((componentOptions, key) => {
      // 初次处理, 初始化为对象
      if (!componentOptions[bindTo]) {
        componentOptions[bindTo] = {}
      }
      const mapObject = {[key]: map}
      // 这个!号突如其来，没懂…
      componentOptions[bindTo]![key] = namespace !== undefined
        ? mapFn(namespace, mapObject)[key]
        : mapFn(mapObject)[key]
    })
  }
}
```

只能大致说下意思了…

可耻地写个 Todo 了，我去看TypeScript最新的官方文档了。。。
