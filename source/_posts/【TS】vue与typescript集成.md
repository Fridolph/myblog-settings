---
title: 【TS】vue与typescript集成
date: 2018-4-25 23:29:03
tags: 
  - js
  - ts
categories: 
  - js
  - ts
---

> 一直想找个机会试试typescript，正好看到一个小项目练手，于是直接开始吧

正文分两部分：
* vue与ts集成 
* 项目实战

转自[Vue-TypeScript-DpApp-Demo](https://segmentfault.com/a/1190000012486378)
[Vue + TypeScript 新项目起手式](https://juejin.im/post/59f29d28518825549f7260b6)

---

## Vue与TypeScript集成相关配置

此demo就是用了 typescript 改的~~ https://github.com/Fridolph/colorful-gallery.git

`vue init SimonZhangITer/vue-typescript-template <project-name>`

npm i ts-loader -D

使用别人造好轮子就好。当然，我们还是要看看前人踩了哪些坑

### 配置rules

接着在Webpack的module.rules里面添加对ts的支持

```js
{
  test: /\.vue$/,
  loader: 'vue-loader',
  options: vueLoaderConfig
},
{
  test: /\.ts$/,
  exclude: /node_modules/,
  enforce: 'pre',
  loader: 'tslint-loader'
},
{
  test: /\.tsx?$/,
  loader: 'ts-loader',
  exclude: /node_modules/,
  options: {
    appendTsSuffixTo: [/\.vue$/],
  }
}
```

`ts-loader`会检索当前目录下的 `tsconfig.json`文件，根据里面定义的规则来解析`.ts`文件，就跟`.babelrc的作用一样`

> tslint-loader 作用等同于 eslint-loader

### 配置extensions

添加可识别文件后缀对ts的支持，如：

`extensions: ['.js', '.vue', '.json', '.ts']`

### tsconfig.json

创建tsconfig.json文件，放在根目录下，和package.json同级

配置内容主要也看个人需求，具体可以去typescript的官网查看，但是有一点需要注意：

> 在Vue中，你需要引入 strict: true (或者至少 noImplicitThis: true，这是 strict 模式的一部分) 以利用组件方法中 this 的类型检查，否则它会始终被看作 any 类型。

```json
{
  "include": [// 只对src下面的文件生效
    "src/*",
    "src/**/*"
  ],
  "exclude": [
    "node_modules" // 排除node_modules 加快速度
  ],
  "compilerOptions": {
    // 类型选项之前已经配置好了
    "types": [
      // 添加Node作为选项
      "node"
    ],
    // typeRoots选项之前已经配置好了
    "typeRoots": [
      "node_modules/@types"
    ],
    // 以严格模式解析
    "strict": true,
    // 在.tsx文件里支持JSX
    "jsx": "preserve",
    // 使用的JSX工厂函数 createElement
    "jsxFactory": "h",
    // 允许从没有设置默认导出的模块中默认导入
    "allowSyntheticDefaultImports": true,
    // 启用装饰器
    "experimentalDecorators": true,
    "strictFunctionTypes": false,
    // 允许编译javascript文件
    "allowJs": true,
    // 采用的模块系统
    "module": "esnext",
    // 编译输出目标 ES 版本
    "target": "es5",
    // 如何处理模块
    "moduleResolution": "node",
    // 在表达式和声明上有隐含的any类型时报错
    "noImplicitAny": true,
    "lib": [
      "dom",
      "es5",
      "es6",
      "es7",
      "es2015.promise"
    ],
    "sourceMap": true,
    "pretty": true
  }
}
```

### 修改main.js

把项目主文件main.js修改成main.ts，里面的写法基本不变，但是有一点需要注意：

引入Vue文件的时候需要加上.vue后缀,否则编辑器识别不到

把webpack的entry文件也修改成`main.ts`

### vue-shims.d.ts

TypeScript并不支持Vue文件，所以需要告诉TypeScript*.vue文件交给vue编辑器来处理。解决方案就是在创建一个vue-shims.d.ts文件，建议放在src目录下再创建一个typings文件夹，把这个声明文件放进去，如：src/typings/vue-shims.d.ts，文件内容：

> *.d.ts类型文件不需要手动引入，TypeScript会自动加载

```js
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}
```

意思是告诉 TypeScript `*.vue`后缀的文件可以交给vue模块来处理

而在代码中导入`*.vue`文件时，需要写上.vue后缀，原因还是因为TypeScript默认只识别 *.ts 文件，不识别 *.vue 文件

到这里TypeScript在Vue中配置就完成了，可以愉快的撸代码了~


### 踩坑

引入第三方库需要额外声明文件

比如说我想引入vue-lazyload,虽然已经在本地安装，但是typescript还是提示找不到模块。原因是typescript是从node_modules/@types目录下去找模块声明，有些库并没有提供typescript的声明文件，所以就需要自己去添加

> 解决办法： 在src/typings目前下建一个tools.d.ts文件，声明这个模块即可

```js
declare module 'vue-awesome-swiper' {
  export const swiper: any
  export const swiperSlide: any
}

declare module 'vue-lazyload'
```

### 改造 .vue 文件

#### vue-class-component

vue-class-component是官方维护的TypeScript装饰器，写法比较扁平化。Vue对其做到完美兼容，如果你在声明组件时更喜欢基于类的 API，这个库一定不要错过

```html
<template>
  <div>
    <input v-model="msg">
    <p>msg: {{msg}}</p>
    <p>computed msg: {{computedMsg}}</p>
    <button @click="greet">Greet</button>
  </div>
</template>
<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
export default class App extends Vue {
  // 初始化数据 不用写 data return 了
  msg = 123

  // 计算属性
  get computedMsg() {
    return `computed ${this.msg}`
  }

  // 相同于 methods 里写 方法
  greet(): void {
    alert(`greeting ${this.msg}`)
  }

  // 生命周期钩子 - 像普通函数一样写就好
  mounted() {
    this.greet()
  }
}
</script>
```

#### vuex-class

vuex-class是基于基于vue-class-component对Vuex提供的装饰器。它的作者同时也是vue-class-component的主要贡献者，质量还是有保证的。

```js
import Vue from 'vue'
import Component from 'vue-class-component'
import {State, Action, Getter} from 'vuex-class'

@Component
export default class App extends Vue {
  name: string = 'fu yinsheng'

  @State login: boolean
  @Action initAjax: () => void
  @Getter load: boolean

  get isLogin(): boolean {
    return this.login
  }

  mounted() {
    this.initAjax()
  }
}
```

#### vue-property-decorator

这是在`vue-class-component`上增强了更多结合vue特性的装饰器，新增了7个装饰器

* @Emit
* @Inject
* @Model
* @Prop
* @Provide
* @Watch
* @Component (从vue-class-component中继承)

这里列举几个常用的 `@Prop、@Watch、@Component`

```ts
import {Component, Emit, Inject, Model, Prop, Provide, Vue, Watch} from 'vue-property-decorator'

@Component
export class MyComponent extends Vue {
  @Prop()
  propA: number = 1

  @Prop({default: 'default value'})
  propB: string

  @Prop([String, Boolean])
  PropC: string | boolean

  @Prop({type: null})
  propD: any

  @Watch('child')
  onChildChanged(val: string, oldVal: string) {
    // ...
  }
}
```

上面的代码相当于：

```js
export default {
  props: {
    propA: Number,
    propB: {
      type: String,
      default: 'default value'
    },
    propC: [String, Boolean],
    propD: {type: null}
  },
  watch: {
    child: {
      handler: 'onChildChanged',
      immediate: false,
      deep: false
    }
  },
  methods: {
    onChildChanged(val, oldVal) {
      //
    }
  }
}
```
