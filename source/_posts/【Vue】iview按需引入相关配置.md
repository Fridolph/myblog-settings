---
title: 【Vue】iview按需引入相关配置
date: 2017-10-14 19:49:03
tags: 
  - vue  
  - webpack
categories: 
  - vue
  - 配置
---

> iview 是一套基于 Vue.js 的高质量 UI 组件库。在项目中使用虽然直接引入整个包挺好的，但打包出来 vendor 非常大，并没必要。所以加入了 iview 按需引入的相关配置，减小包的体积从而提升点性能。

<!-- more -->

首先通过 npm 安装 iview-loader

    npm install iview-loader --save-dev

配置 webpack，改写平时 vue-loader 的配置，之前是这样的：

```js
module: {
  rules: [
    {
      test: /\.vue$/,
      loader: 'vue-loader',
      options: vueLoaderConfig
    },
    {
      test: /iview.src.*?js$/,
      loader: 'babel-loader'
    }
  ]
}
```

改为：

```js
module: {
  rules: [
    {
      test: /\.vue$/,
      use: [
        {
          loader: 'vue-loader',
          options: vueLoaderConfig
        },
        {
          loader: 'iview-loader',
          options: {
            prefix: false
          }
        }
      ]
    },
    {
      test: /\.js$/,
      loader: 'babel-loader', // 注：wepback 3等高版本 loader名称必须写完整
      include: [
        resolve('src'),
        resolve('test'),
        resolve('/node_modules/iview/src')
      ]
    }
    // ... 更多
  ]
}
```

借助插件 babel-plugin-import可以实现按需加载组件，减少文件体积。首先安装，并在文件 .babelrc 中配置：

    npm install babel-plugin-import --save-dev

```js
// .babelrc
{
  "plugins": [["import", {
    "libraryName": "iview",
    "libraryDirectory": "src/components"
  }]]
}
```

然后这样按需引入组件，就可以减小体积了：

```js
import { Button, Table } from 'iview';
Vue.component('Button', Button);
Vue.component('Table', Table);
```

可以直接写 <Switch> 和 <Circle> 这两个标签；
参数 prefix 设置为 true 后，所有 iView 组件标签名都可以使用前缀 i-，例如 <i-row>、<i-select>

注： 按需引用仍然需要导入样式，即在 main.js 或根组件执行 `import 'iview/dist/styles/iview.css'`;

---

**routes 里同样可以异步加载组件**

```js
const routes = [
  {
    path: '/',
    redirect: '/home'
  },
  {
    path: '/home',
    name: 'home',
    // component: Situation,
    component: resolve => require(['@/views/home'], resolve),
  }
  // ...
}
```

在组件 template 中 `import {Button} from 'iview'`， 然后在 components 里注册相应组件即可使用了

使用 :prop传递数据格式为 数字、布尔值或函数时，必须带:(兼容String除外，具体看组件文档)

在非 template/render 模式下（例如使用 CDN 引用时），组件名要分隔，例如 DatePicker 必须要写成 date-picker

注：

locale，Message，Modal 等全局组件需要在主入口文件中引入或声明

```js
import VueI18n from 'vue-i18n'
import ZH from 'locale/local/zh_CN'
import EN from 'locale/local/en_US'
import zhLocale from 'iview/dist/locale/zh-CN'
import enLocale from 'iview/dist/locale/en-US'
// 依赖包
import { locale, Modal, Message } from 'iview'
import 'iview/dist/styles/iview.css'
let lang = localStorage.getItem('language')
Vue.config.productionTip = false
Vue.config.lang = lang
Vue.use(VueI18n)
// 这里的locale国际化组件引入后，需将其声明为方法
Vue.locale = () => {}
const messages = {
  zh: Object.assign(ZH, zhLocale),
  en: Object.assign(EN, enLocale)
}
const i18n = new VueI18n({
  locale: Vue.config.lang,
  messages
})
Vue.use(locale, {
  i18n(path, options) {
    let value = i18n.t(path, options)
    if (value !== null && value !== undefined) {
      return value
    }
    return ''
  }
})
Vue.prototype.$Modal = Modal
Vue.prototype.$Message = Message
```

