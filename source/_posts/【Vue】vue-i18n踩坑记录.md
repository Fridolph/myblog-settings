---
title: 【Vue】vue-i18n踩坑记录
date: 2017-09-05 15:49:03
tags: 
  - vue
  - vue-i18n
categories: 
  - vue
  - 国际化
---

> 跟着官网来看着并不难，放项目里，这出错那不行的… 慢慢看文档一边整理一边尝试，最终得到了我想要的效果。暂时记下来写成博客，可能不是最好的，但目前先把问题解决了来后面再谈细节和优化的事儿吧

<!-- more -->

折腾了一上午，犯了很多错… 最终捣鼓出来，算是完成了需求。这里说下自己的理解。Vue里的国际化，其实只是把 所谓的静态内容，标题，文字段落，提示说明等等用vue的响应式data来存储，再根据我们的配置，动态改变这些data，从而实现无刷新语言切换，这和vue自身的响应式数据是一个道理。

## 渲染原理

```js
VueI18n.prototype._t = function _t (key, _locale, messages, host) {
  var values = [], len = arguments.length - 4;
  while ( len-- > 0 ) values[ len ] = arguments[ len + 4 ];

  if (!key) { return '' }

  var parsedArgs = parseArgs.apply(void 0, values);
  var locale = parsedArgs.locale || _locale;
  var ret = this._translate(
    messages, locale, this.fallbackLocale, key,
    host, 'string', parsedArgs.params
  );
  if (this._isFallbackRoot(ret)) {
    if ("development" !== 'production' && !this._silentTranslationWarn) {
      warn(("Fall back to translate the keypath '" + key + "' with root locale."));
    }
    /* istanbul ignore if */
    if (!this._root) { throw Error('unexpected error') }
    return (ref = this._root).t.apply(ref, [ key ].concat( values ))
  } else {
    return this._warnDefault(locale, key, ret, host)
  }
  var ref;
};
```

```js
Vue.prototype.$t = function (key) {
  var values = [], len = arguments.length - 1;
  while ( len-- > 0 ) values[ len ] = arguments[ len + 1 ];

  var i18n = this.$i18n;
  return i18n._t.apply(i18n, [ key, i18n.locale, i18n._getMessages(), this ].concat( values ))
};
```

这是目前用到的一个重要函数，从源码里复制出来。这里Vue原型里添加了一个$t方法，它可传入一个key参数，最后连同信息一起返回。

我们使用的$t即将想要的key 拿到写好的 locale目录下的data中查询，返回对应value这样一个过程，理解了这个思路后后面的就要好办多了。

---

## 项目运用

首先安装依赖 npm install vue-i18n -D

先贴一下我的配置，然后慢慢说：

```js
import Vue from 'vue';
import VueI18n from 'vue-i18n';
import App from './index';
// import router from '../../router/edrPage';
import store from '../../store/index';
import router from '../../router/edrPage/index';
// 依赖包
import iView from 'iview';
import en from 'locale/local/en_US';
import zh from 'locale/local/zh_CN';
import zhLocale from 'locale/iviewLocale/zh-CN';
import enLocale from 'locale/iviewLocale/en-US';
import 'iview/dist/styles/iview.css';
Vue.config.productionTip = false;
Vue.config.lang = 'en';  // 语言控制的开关
Vue.use(VueI18n);        // Vue实例中注册
// 定义数据
const messages = {
    en: Object.assign({}, en, enLocale),  // 这里浅拷贝也可
    zh: Object.assign({}, zh, zhLocale)
}
// 新生成i18n实例，并写入相应配置
const i18n = new VueI18n({
    locale: Vue.config.lang, // 语言
    fallbackLocale: 'en',    // 这是iView用到的
    messages                 // 把上面定义好的数据传入i18n
})
// 注册iView的同时，把i18n的配置一同传入
// 这样就可以在vue中无缝使用i18n和iView的国际化组件了, element同理
Vue.use(iView, {
    i18n(path, options) {
      let value = i18n.t(path, options);
      if (value !== null && value !== undefined) return value;

      return '';
    }
});
/* eslint-disable no-new */
new Vue({
  el: '#app',
  // i18n已有上面定义的messages了，这里在Vue实例中注入
  // 就可以在组件中 this.$i18n 这样来用了(这一步是i18n已帮我们做的)
  i18n,
  router,
  store,
  template: '<App/>',
  components: { App }
});
```

我用的iView 2.2版本i18n版本只支持到5.0+明显不成，各种问了之后看到了一个解决方案挺不错，就是上面这个。思路很简单，就是本地创建自己的语言包，然后与iView提供的包合并，其他流程走i18n自己的流程即可

---

在单文件组件的模版中:  (用来测试的随便写了)

```html
<template>
  <div id="login" class="bg">
    <button @click="switchLang">切换语言</button>
    <h1>{{$t('edr.footer.copyright')}}</h1>
  </div>
</template>

<script>
  export default {
    name: 'page-login',
    data() {
      return {
        locale: 'en',
      }
    },
    mounted() {
      this.locale = this.$i18n.locale = localStorage.getItem('language') // 从 localStorage 中获取语言状态
    },
    methods: {
      switchLang() {
        if (this.locale === 'en') {
          this.locale = this.$i18n.locale = 'zh'
          localStorage.setItem('language', 'zh')
        } else {
          this.locale = this.$i18n.locale = 'en'
          localStorage.setItem('language', 'en')
        }
      }
    }
  }
</script>
```

现在，点击切换按钮，语言就可实时更改了~ 

好了，其他的同理，在locale目录下，把我们需要切换的中英文数据管理好即可~ 如果内容多拆开来管理更好

---

但感觉很麻烦的是，以后写静态数据都不能直接写在模版里了… 也算是长姿势了
至于后端返回的数据…… 暂时不知道，让后端自己先折腾吧

还有一个问题…  整个template里， 全是$t()这样的…  也不知道写了什么，占位符谁是谁… 于是，之前写好的最好还是来一层注释，或者利用良好的class命名来避免开发时的尴尬… 不然拿到同事手中只有两眼懵比了...