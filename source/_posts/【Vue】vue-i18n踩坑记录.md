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
Vue.config.lang = 'en';
Vue.use(VueI18n);

const messages = {
    en: Object.assign(en, enLocale),
    zh: Object.assign(zh, zhLocale)
}

const i18n = new VueI18n({
    locale: Vue.config.lang,
    fallbackLocale: 'en',
    messages
})

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

麻烦的是，以后写静态数据都不能直接写在模版里了… 也算是长姿势了，至于后端返回的数据…… 暂时不知道，让后端自己先折腾吧