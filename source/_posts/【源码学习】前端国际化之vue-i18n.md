---
title: 【源码学习】前端国际化之vue-i18n
date: 2018-12-28 22:33:00
tags:
  - vue-i18n
  - 国际化
categories:
  - 源码学习
---

> 最近的项目都要求做国际化，把模板替换后注释更多了，\$t 看着也见怪不怪了。之前在掘金看到有人在写类似的文章趁着手热，自己也借鉴借鉴，写点东西来也算复习一下了。

![前端国际化解决方案之 —— VueI18n](http://blog.fueson.top/19-1-9/64247896.jpg)

<!-- more -->

一般地，我们提供一个按钮，当用户点击后，就可以把内容切换成对应的语言了。这种切换有几种方案：

1. 有很多站是利用跳转到新的站点，如 [antd](https://ant.design/index-cn)、 [iview](https://www.iviewui.com/)
2. 在线翻译（腾讯、谷歌啥的，依赖网络环境）
3. 统一模板文件，前端根据相应的语言映射表去做文案的替换（vue-i18n 就是这样）

---

## 基本使用

```js
// 入口main.js文件
import VueI18n from 'vue-i18n'

// 通过插件的形式挂载
Vue.use(VueI18n)

let lang = 'zh'

const i18n = new VueI18n({
  // 可理解为语言切换的开关
  locale: lang,
  messages: {
    'zh-CN': require('./locale/zh'),
    'en-US': require('./locale/en'),
    'jp-JP': require('./locale/jp'),
    // 可加入多种语言
  }
})

const app = new Vue({
  i18n,
  ...App
}).$mout('#root')

// 单vue文件
<template>
  <span>{{$t('test')}}</span>
</template>
```

Vue-i18n 是以插件的形式配合 Vue 进行工作的。通过全局的 mixin 的方式将插件提供的方法挂载到 Vue 的实例上。

> ps `<img>`标签的 src 等都可以这样来切换，以达到不同语言使用不同图片的目的…

## 部分源码分析

### vue-i18n src 目录结构

```bash
|__ component.js   当作 组件 来使用
|__ directive.js   当作 指令 来使用
|__ extend.js      将各方法挂载到 Vue 实例
|__ format.js      转化函数 封装了format, 里面包含了template模板替换的方法
|__ index.js       主文件，向外暴露 VueI18n类
|__ install.js     对Vue的挂载函数,主要是为了将mixin.js里面的提供的方法挂载到Vue实例当中:
|__ mixin.js       对Vue对象上混合了钩子函数.
|__ path.js        存了各类常量、规则、映射条件和路径等
|__ util.js        通用的工具函数文件，如warn函数、类型判断函数等
```

### install.js

```js
import { warn } from './util';
import mixin from './mixin';
import Asset from './asset';

export let Vue;

// 注入root Vue
export function install(_Vue) {
  // 通过上面的 基本用例 可知，原来 _Vue 就是 import Vue 这个Vue对象
  // 然后赋值给了当前的 变量Vue。 这样就能取到 Vue 上的属性和方法了
  Vue = _Vue;

  const version = (Vue.version && Number(Vue.version.split('.')[0])) || -1;
  if (process.env.NODE_ENV !== 'production' && install.installed) {
    warn('already installed.');
    return;
  }
  install.installed = true;

  if (process.env.NODE_ENV !== 'production' && version < 2) {
    warn(
      `vue-i18n (${install.version}) need to use Vue 2.0 or later (Vue: ${
        Vue.version
      }).`,
    );
    return;
  }
  // 通过mixin的方式，将插件提供的methods，钩子函数等注入到全局
  // 之后每次创建的vue实例都用拥有这些methods或者钩子函数
  Vue.mixin(mixin);

  Asset(Vue);
}
```

### mixin.js

```js
// VueI18n构造函数
import VueI18n from './index';
import { isPlainObject, warn } from './util';

// $i18n 是每创建一个Vue实例都会产生的实例对象
// 调用以下方法前都会判断实例上是否挂载了$i18n这个属性
// 最后实际调用的方法是插件内部定义的方法
export default {
  // 这里混合了computed计算属性, 注意这里计算属性返回的都是函数,这样就可以在vue模板里面使用{{ $t('hello') }},
  // 或者其他方法当中使用 this.$t('hello')。这种函数接收参数的方式
  computed: {
    // 翻译函数, 调用的是VueI18n实例上提供的方法
    $t() {
      if (!this.$i18n) {
        throw Error(`Failed in $t due to not find VueI18n instance`);
      }
      // add dependency tracking !!
      const locale: string = this.$i18n.locale; // 语言配置
      const messages: Messages = this.$i18n.messages; // 语言包
      // 返回一个函数. 接受一个key值. 即在map文件中定义的key值, 在模板中进行使用 {{ $t('你好') }}
      // ...args是传入的参数, 例如在模板中定义的一些替换符
      // 具体的支持的形式可翻阅文档https://kazupon.github.io/vue-i18n/formatting.html
      return (key: string, ...args: any): string => {
        return this.$i18n._t(key, locale, messages, this, ...args);
      };
    },

    // tc方法可以单独定义组件内部语言设置选项, 如果没有定义组件内部语言，则还是使用global的配置
    $tc() {
      if (!this.$i18n) {
        throw Error(`Failed in $tc due to not find VueI18n instance`);
      }
      // add dependency tracking !!
      const locale: string = this.$i18n.locale;
      const messages: Messages = this.$i18n.messages;
      return (key: string, choice?: number, ...args: any): string => {
        return this.$i18n._tc(key, locale, messages, this, choice, ...args);
      };
    },

    // te方法
    $te() {
      if (!this.$i18n) {
        throw Error(`Failed in $te due to not find VueI18n instance`);
      }
      // add dependency tracking !!
      const locale: string = this.$i18n.locale;
      const messages: Messages = this.$i18n.messages;
      return (key: string, ...args: any): boolean => {
        return this.$i18n._te(key, locale, messages, ...args);
      };
    },
  },

  // 钩子函数
  // 被渲染前，在vue实例上添加$i18n属性
  // 在根组件初始化的过程中:
  /**
   * new Vue({
   *   i18n   // 这里是提供了自定义的属性 那么实例当中可以通过this.$option.i18n去访问这个属性
   *   // xxxx
   * })
   */
  beforeCreate() {
    // 因为 Vue 被install进来 所以 能通过 this拿到Vue对象，进而访问到 $options 属性
    const options: any = this.$options;
    // 如果有i18n这个属性. 根实例化的时候传入了这个参数
    if (options.i18n) {
      if (options.i18n instanceof VueI18n) {
        // 如果是VueI18n的实例，那么挂载在Vue实例的$i18n属性上
        // 这里是往 Vue 对象上新加了一个属性 $i18n
        this.$i18n = options.i18n;
        // 如果是个object
      } else if (isPlainObject(options.i18n)) {
        // 如果是一个pobj
        // component local i18n
        // 访问root vue实例。
        if (
          this.$root &&
          this.$root.$i18n &&
          this.$root.$i18n instanceof VueI18n
        ) {
          options.i18n.root = this.$root.$i18n;
        }
        this.$i18n = new VueI18n(options.i18n); // 创建属于component的local i18n
        if (options.i18n.sync) {
          this._localeWatcher = this.$i18n.watchLocale();
        }
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn(`Cannot be interpreted 'i18n' option.`);
        }
      }
    } else if (
      this.$root &&
      this.$root.$i18n &&
      this.$root.$i18n instanceof VueI18n
    ) {
      // root i18n
      // 如果子Vue实例没有传入$i18n方法,且root挂载了$i18n,那么子实例也会使用root i18n
      this.$i18n = this.$root.$i18n;
    }
  },

  // 实例被销毁的回调函数
  destroyed() {
    if (this._localeWatcher) {
      this.$i18n.unwatchLocale();
      delete this._localeWatcher;
    }

    // 组件销毁后，同时也销毁实例上的$i18n方法
    this.$i18n = null;
  },
};
```

接下来就看看 `VueI18n` 构造函数及原型上提供了哪些 可以被实例继承的属性或者方法

```js
import { install, Vue } from './install';
import { warn, isNull, parseArgs, fetchChoice } from './util';
import BaseFormatter from './format';
import getPathValue from './path';
import type { PathValue } from './path';

// VueI18n构造函数
export default class VueI18n {
  // 静态方法
  static install: () => void;
  // 静态属性
  static version: string;

  _vm: any;
  _formatter: Formatter;
  _root: ?I18n;
  _sync: ?boolean;
  _fallbackRoot: boolean;
  _fallbackLocale: string;
  _missing: ?MissingHandler;
  _exist: Function;
  _watcher: any;

  // 实例化参数配置
  constructor(options: I18nOptions = {}) {
    // vue-i18n初始化的时候语言参数配置
    const locale: string = options.locale || 'en-US';
    // 本地配置的所有语言环境都是挂载到了messages这个属性上
    const messages: Messages = options.messages || {};
    // ViewModel
    this._vm = null;
    // 缺省语言配置
    this._fallbackLocale = options.fallbackLocale || 'en-US';
    // 翻译函数
    this._formatter = options.formatter || new BaseFormatter();
    this._missing = options.missing;
    this._root = options.root || null;
    this._sync = options.sync || false;
    this._fallbackRoot = options.fallbackRoot || false;

    this._exist = (message: Object, key: string): boolean => {
      if (!message || !key) {
        return false;
      }
      return !isNull(getPathValue(message, key));
    };

    this._resetVM({ locale, messages });
  }

  // VM 重置viewModel
  _resetVM(data: { locale: string, messages: Messages }): void {
    const silent = Vue.config.silent;
    Vue.config.silent = true;
    this._vm = new Vue({ data });
    Vue.config.silent = silent;
  }

  // 根实例的vm监听locale这个属性
  watchLocale(): any {
    if (!this._sync || !this._root) {
      return null;
    }
    const target: any = this._vm;
    // vm.$watch返回的是一个取消观察的函数，用来停止触发回调
    this._watcher = this._root.vm.$watch(
      'locale',
      val => {
        target.$set(target, 'locale', val);
      },
      { immediate: true },
    );
    return this._watcher;
  }

  // 停止触发vm.$watch观察函数
  unwatchLocale(): boolean {
    if (!this._sync || !this._watcher) {
      return false;
    }
    if (this._watcher) {
      this._watcher();
      delete this._watcher;
    }
    return true;
  }

  get vm(): any {
    return this._vm;
  }

  // get 获取messages参数
  get messages(): Messages {
    return this._vm.$data.messages;
  }
  // set 设置messages参数
  set messages(messages: Messages): void {
    this._vm.$set(this._vm, 'messages', messages);
  }

  // get 获取语言配置参数
  get locale(): string {
    return this._vm.$data.locale;
  }
  // set 重置语言配置参数
  set locale(locale: string): void {
    this._vm.$set(this._vm, 'locale', locale);
  }

  //  fallbackLocale 是什么?
  get fallbackLocale(): string {
    return this._fallbackLocale;
  }
  set fallbackLocale(locale: string): void {
    this._fallbackLocale = locale;
  }

  get missing(): ?MissingHandler {
    return this._missing;
  }
  set missing(handler: MissingHandler): void {
    this._missing = handler;
  }

  // get 转换函数
  get formatter(): Formatter {
    return this._formatter;
  }
  // set 转换函数
  set formatter(formatter: Formatter): void {
    this._formatter = formatter;
  }

  _warnDefault(locale: string, key: string, result: ?any, vm: ?any): ?string {
    if (!isNull(result)) {
      return result;
    }
    if (this.missing) {
      this.missing.apply(null, [locale, key, vm]);
    } else {
      if (process.env.NODE_ENV !== 'production') {
        warn(
          `Cannot translate the value of keypath '${key}'. ` +
            'Use the value of keypath as default.',
        );
      }
    }
    return key;
  }

  _isFallbackRoot(val: any): boolean {
    return !val && !isNull(this._root) && this._fallbackRoot;
  }

  // 插入函数
  _interpolate(message: Messages, key: string, args: any): any {
    if (!message) {
      return null;
    }

    // 获取key对应的字符串
    let val: PathValue = getPathValue(message, key);
    if (Array.isArray(val)) {
      return val;
    }
    if (isNull(val)) {
      val = message[key];
    }
    if (isNull(val)) {
      return null;
    }
    if (typeof val !== 'string') {
      warn(`Value of key '${key}' is not a string!`);
      return null;
    }

    // TODO ??
    // 检查转换后的字符串中是否存在链接
    if (val.indexOf('@:') >= 0) {
      // 匹配所有在本地链接, 我们将用它的译文替换它们
      const matches: any = val.match(/(@:[\w|.]+)/g);
      for (const idx in matches) {
        const link = matches[idx];
        // 删除前面的@
        const linkPlaceholder = link.substr(2);
        // 翻译链接
        const translatedstring = this._interpolate(
          message,
          linkPlaceholder,
          args,
        );
        // 用转换后的字符串替换链接
        val = val.replace(link, translatedstring);
      }
    }

    // 如果没有传入需要替换的obj, 那么直接返回字符串
    // 否则调用this._format进行变量等的替换, 获取替换后的字符
    return !args ? val : this._format(val, args);
  }

  _format(val: any, ...args: any): any {
    return this._formatter.format(val, ...args);
  }

  // 翻译函数
  _translate(
    messages: Messages,
    locale: string,
    fallback: string,
    key: string,
    args: any,
  ): any {
    let res: any = null;
    /**
     * messages[locale] 使用哪个语言包
     * key 语言映射表的key
     * args 映射替换关系
     */
    res = this._interpolate(messages[locale], key, args);
    if (!isNull(res)) {
      return res;
    }

    res = this._interpolate(messages[fallback], key, args);
    if (!isNull(res)) {
      if (process.env.NODE_ENV !== 'production') {
        warn(
          `Fall back to translate the keypath '${key}' with '${fallback}' locale.`,
        );
      }
      return res;
    } else {
      return null;
    }
  }

  // 翻译的核心函数
  /**
   * 这里的方法传入的参数参照 mixin.js里面的定义的方法
   * key map的key值 (为接受的外部参数)
   * _locale 语言配置选项: 'zh-CN' | 'en-US' (内部变量)
   * messages 映射表 (内部变量)
   * host为这个i18n的实例 (内部变量)
   */
  _t(
    key: string,
    _locale: string,
    messages: Messages,
    host: any,
    ...args: any
  ): any {
    if (!key) {
      return '';
    }

    // parseArgs函数用以返回传入的局部语言配置
    // 及映射表接收的参数 { locale, params(映射表) }
    const parsedArgs = parseArgs(...args);
    // 语言配置
    const locale = parsedArgs.locale || _locale;

    // 字符串替换
    /**
     * @params messages  语言包
     * @params locale  语言配置
     * @params fallbackLocale 缺省语言配置
     * @params key 替换的key值
     * @params parsedArgs.params 需要被替换的参数map表
     */
    const ret: any = this._translate(
      messages,
      locale,
      this.fallbackLocale,
      key,
      parsedArgs.params,
    );
    if (this._isFallbackRoot(ret)) {
      if (process.env.NODE_ENV !== 'production') {
        warn(`Fall back to translate the keypath '${key}' with root locale.`);
      }
      if (!this._root) {
        throw Error('unexpected error');
      }
      return this._root.t(key, ...args);
    } else {
      return this._warnDefault(locale, key, ret, host);
    }
  }

  // 转化函数
  t(key: string, ...args: any): string {
    return this._t(key, this.locale, this.messages, null, ...args);
  }

  _tc(
    key: string,
    _locale: string,
    messages: Messages,
    host: any,
    choice?: number,
    ...args: any
  ): any {
    if (!key) {
      return '';
    }
    if (choice !== undefined) {
      return fetchChoice(
        this._t(key, _locale, messages, host, ...args),
        choice,
      );
    } else {
      return this._t(key, _locale, messages, host, ...args);
    }
  }

  tc(key: string, choice?: number, ...args: any): any {
    return this._tc(key, this.locale, this.messages, null, choice, ...args);
  }

  _te(key: string, _locale: string, messages: Messages, ...args: any): boolean {
    const locale = parseArgs(...args).locale || _locale;
    return this._exist(messages[locale], key);
  }

  te(key: string, ...args: any): boolean {
    return this._te(key, this.locale, this.messages, ...args);
  }
}

VueI18n.install = install;
VueI18n.version = '__VERSION__';

// 如果是通过CDN或者外链的形式引入的Vue
if (typeof window !== 'undefined' && window.Vue) {
  window.Vue.use(VueI18n);
}
```

### format.js

```js
/**
 *  String format template
 *  - Inspired:
 *    https://github.com/Matt-Esch/string-template/index.js
 */
// 变量的替换, 在字符串模板中写的站位符 {xxx} 进行替换
const RE_NARGS: RegExp = /(%|)\{([0-9a-zA-Z_]+)\}/g;
// let reg = /(%|)\{([0-9a-zA-Z_]+)\}/g
// reg.test(`{xxx}`) -> true
// reg.test(`{a: 'bbb'}`) -> false

/**
 * template
 *
 * @param {String} string
 * @param {Array} ...args
 * @return {String}
 */

// 模板替换函数
export function template(str: string, ...args: any): string {
  // 如果第一个参数是一个对象
  if (args.length === 1 && typeof args[0] === 'object') {
    args = args[0];
  } else {
    args = {};
  }

  if (!args || !args.hasOwnProperty) {
    args = {};
  }

  // str.prototype.replace(substr/regexp, newSubStr/function)
  // 第二个参数如果是个函数的话，每次匹配都会调用这个函数
  // match 为匹配的子串
  return str.replace(RE_NARGS, (match, prefix, i, index) => {
    let result: string;

    // match是匹配到的字符串
    // prefix 检测前缀 ?? 但没看后面用啊
    // i 括号中需要替换的字符换
    // index是偏移量

    // 字符串中如果出现{xxx}不需要被替换。那么应该写成{{xxx}}
    if (str[index - 1] === '{' && str[index + match.length] === '}') {
      return i;
    } else {
      // 判断args obj是否包含这个key值
      // 返回替换值, 或者被匹配上的字符串的值
      result = hasOwn(args, i) ? args[i] : match;
      if (isNull(result)) {
        return '';
      }

      return result;
    }
  });
}
```

## 总结

- 学习优秀开源库文件的一些架构和组织方式
- 前端根据后端下发的语言标识字段来调用不同的语言包
- 文本内容和图片 src 等可使用 vue-i18n 进行替换
- 样式需要根据多语言进行定制。比如在 body 上添加多语言的标识 class 属性(如英语翻译过长等，需要提前想好布局等)
