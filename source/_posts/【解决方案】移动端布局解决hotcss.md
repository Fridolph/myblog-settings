---
title: 【解决方案】移动端布局解决hotcss
date: 2018-03-29 20:40:06
tags: 
  - 移动端
  - rem
  - 自适应
categories: 
  - 解决方案
---

> 号称移动端布局终极解决方案。其实现非常简洁，但功能却不是盖的。在最近准备做的一个项目中会用到该解决方案，知根知底最好的办法便是学习官方文档和敲源码了。嗯，感觉这法还好，于是废话不多说，开始正文吧

<!-- more -->

## 官方文档

[hotcss](http://imochen.github.io/hotcss/) 官网称这是一个移动端布局开发的解决方案，使用动态HTML根字体大小和动态viewport scale。之前有一篇很火的讲述移动端布局的，介绍了淘宝、网易几家大厂的实现方式，这里的不同也就是在这个动态viewport scale上了吧。

其优势在于：

* 保证不同设备下的统一视觉体验。
* 不需要你再手动设置viewport，根据当前环境计算出最适合的viewport。
* 支持任意尺寸的设计图，不局限于特定尺寸的设计图。
* 支持单一项目，多种设计图尺寸，专为解决大型，长周期项目。
* 提供px2rem转换方法，CSS布局，零成本转换，原始值不丢失。
* 有效解决移动端真实1像素问题。

**用法**

直接引入

    <script src="/path/to/hotcss.js"></script>

### css写法

推荐使用scss来编写css，在scss文件的头部使用import将px2rem导入

    @import '/path/to/px2rem.scss';

如果项目是单一尺寸设计图，那么你需要去px2rem.scss中定义全局的designWidth。

    @function px2rem( $px ){
      @return $px*320/$designWidth/20 + rem;
    }
    $designWidth : 750; //如设计图是750

如果你的项目是多尺寸设计图，那么就不能定义全局的designWidth了。需要在你的业务scss中单独定义。如以下是style.scss

    @import '/path/to/px2rem.scss';
    $designWidth : 750; //如设计图是750

$designWidth必须要在使用px2rem前定义。否则scss编译会出错。

## hotcss 源码学习

上面的是用来初（凑）步（字）认（数）识的，果断还是要从源码着手，才能拨云见雾。

项目目录

```js
├── example	//所有的示例都在这个目录下
│   ├── duang
│   ├── normal
│   └── wolf
│
└── src	//主要文件在这里
    ├── hotcss.js
    ├── px2rem.less
    ├── px2rem.scss
    └── px2rem.styl
```

---

```js
;(function(widnow, document) {
  'use strict';
  // 给hotcss开辟个命名空间，暴露给外部访问
  var hotcss = {}
  ;(function() {
    // 根据devicePixelRatio自定计算scale
    // 可以有效解决移动端1px问题
    var viewportEl = document.querySelector('meta[name="viewport"]'),
      hotcssEl = document.querySelector('meta[name="hotcss"]'),
      dpr = window.devicePixelRatio || 1, // 默认给1
      maxWidth = 540,  // 这个最大宽度是可以手动调整的，表示我们支持的最大宽度
      designWidth = 0  // 该值最终为设计稿宽度 比如 ip5 320 实际计算得 640    
    // 这里有点绕，一句话说就是，dpr超过3的按3算，2-3之间按2算，1-2之间按1算
    dpr = dpr >= 3 ? 3 : (dpr >= 2 ? 2 : 1)
    // 允许通过自定义name为hotcss的meta头，通过initial-dpr来强制页面缩放
    if (hotcssEl) {
      var hotcssCon = hotcssEl.getAttribute('content') // 拿meta的content
      if (hotcssCon) {
        // 搞不清楚运行一下就好， 'initial-dpr=2.333'.match(/initial\-dpr=([\d\.]+)/)
        // 返回数组 ['initial-dpr=2.333', '2.333'] 
        var initialDprMatch = hotcssCon.match(/initial\-dpr=([\d\.]+)/)
        if (initialDprMatch) {
          // 实际上取 上数组的第1项 即具体数值
          dpr = parseFloat(initialDprMatch[1])
        }
        var maxWidthMatch = hotcssCon.match(/max\-width=(\d\.)+/)
        if (maxWidthMatch) {
          // 同上，拿到了最大宽度
          maxWidth = parseFloat(maxWidthMatch[1])
        }
        var designWidthMatch = hotcssCon.match(/degisn\-width=([\d\.]+)/)
        if (designWidthMatch) {
          // 这个设计宽度很重要，下面会将到
          designWidth = parseFloat(designWidthMatch[1])
        }
      }
    }
    // 给html设置 data-dpr属性
    document.documentElement.setAttribute('data-dpr', dpr)
    // 同时为hotcss对象 添加属性 dpr
    hotcss.dpr = dpr
    // 同上
    document.documentElement.setAttribute('max-width', maxWidth)
    hotcss.maxWidth = maxWidth
    if (designWidth) {
      document.documentElement.setAttribute('design-width', designWidth)
    }
    // 保证 px2rem 和 rem2px 不传第二个参数时，获取 hotcss.desgnWidth 是 undefined导致的NaN
    hotcss.designWidth = designWidth
    var scale = 1 / dpr,
      // 注：我觉得麻烦就改成es6写法了，作者为兼容性用的 + 来拼接，这段大家写viewport都很熟悉吧
      content = `width=device-width, initial-scale=${scale}, minimum-scale=${scale}, maximum-scale=${scale}, user-scalable=no`
    // 容错处理，如果没写viewport作者大大很和善地帮加上了
    if (viewportEl) {
      viewportEl.setAttribute('content', content)
    } else {
      viewportEl = document.createElement('meta')
      viewportEl.setAttribute('name', 'viewport')
      viewportEl.setAttribute('content', 'content')
      document.head.appendChild(viewportEl)
    }
  })()
  // 其实到这里为止还没开始重头戏，前戏为，我们将 html文档设置了需要的前置属性
  // meta-dpr max-width design-width 通过这3个值就能进行以下的适配工作了
  hotcss.px2rem = function(px, designWidth) {
    // 预判你将会在JS中用到尺寸，特提供一个方法助你在JS中将px转为rem
    if (!designWidth) {
      // 如果在JS中大量用到此方法，建议直接定义 hotcss.designWidth 来定义设计图尺寸
      // 否则可以在第二个参数告诉设计图是多大
      designWidth = parseInt(hotcss.designWidth, 10)
    }
    // 按320宽 与 设计图比例 进行 20份等比分配
    return parseInt(px, 10) * 320 / designWidth / 20
  }
  hotcss.rem2px = function(rem, designWidth) {
    // 新增一个rem2px的方法，用法和 px2rem 一致
    if (!designWidth) {
      designWidth = parseInt(hotcss.designWidth, 10)
    }
    // rem可能为小数，这里暂不作处理
    // rem是 以320标准和设计图尺寸比 除以20份 得到的，这里反过来求得 px
    return rem * 20 * designWidth / 320
  }
  // 核心方法，给html 设置 font-size, 因为 rem是基于 根元素的fontsize
  hotcss.mresize = function() {
    // getBoundingClientRect()有少数兼容性问题
    var innerWidth = document.documentElement.getBoundingClientRect().width || window.innerWidth;
    
    // 我们之前给hotcss设了最大宽度，这是一个处理边界
    // 假设 苹果xxx 宽度是 2560 / 3 = 853，我们处理的最大宽为 600， 则说明走以下逻辑， 反之则重设innerWidth
    if (hotcss.maxWidth && (innerWidth / hotcss.dpr > hotcss.maxWidth)) {
      innerWidth = hotcss.maxWidth * hotcss.dpr
    }

    if (!innerWidth) return false
    // 还记得我们将设计图尺寸切分成20份吗。dpr已用于innerWidth计算了。这里按320宽还原回去
    document.documentElement.style.fontSize = (innerWidth * 20 / 320) + 'px'
    // 有回调则执行回调
    hotcss.callback && hotcss.callback()
  }
  // 先直接调用一次，调用后， html根元素被设置了：
  // data-dpr、max-width、design-width、font-size
  hotcss.mresize() 
  // 接下来的 即函数节流了，用于性能优化
  window.addEventListener('resize', function() {
    clearTimeout(hotcss.tid)
    // 这个时间可按实际需求更改
    hotcss.tid = setTimeout(hotcss.mresize, 33)
  }, false)
  // 这里可理解为初始化，load后先执行一次
  window.addEventListener('load', hotcss.mresize, false) 
  setTimeout(function() {
    hotcss.mresize()
    // 这里作者为保险又调用了一次，真是小心谨慎啊
  }, 333)
  window.hotcss = hotcss
  // 命名空间暴露出来，控制权也暴露了，需要时可手动调用以处理某些特别需求
})(window, document)
```

还不错，加上注释也没多少行代码，为以后读大段源码打下了基础…… 误。。。 看到这里，其实和之前淘宝 rem 解决方案差不多，这里只是进行整合。
我们可以简单回顾一下，脚本执行后 为window添加全局变量hotcss。在其中可以调用 hotcss.mresize方法，设置几大重要值，随着屏幕尺寸的改变，font-size也会跟着变，从而让我们写在代码中的rem值也随之调整，这样就实现了移动端的适配。

### 接口说明

**initial-dpr**

可以通过强制设置dpr。来关闭响应的viewport scale。使得viewport scale始终为固定值。

```html
<meta name="hotcss" content="initial-dpr=1">
<script src="/path/to/hotcss.js"></script>
<!--
如iphone微信强设dpr=1，则可以长按识别二维码。
注意，强制设置dpr=1后，css中的1px在2x，3x屏上则不再是真实的1px。
-->
```

**max-width**
通过设置该值来优化平板/PC访问体验，注意该值默认值为540。设置为0则该功能关闭。 为了配合使用该设置，请给body增加样式width:16rem;margin:0 auto;。

```html
<meta name="hotcss" content="max-width=640">
<script src="/path/to/hotcss.js"></script>
<!--
默认为540，可根据具体需求自己定义
-->
<style>
body{
    width: 16rem;
    margin: 0 auto;
}
</style>
```

**design-width**
通过对design-width的设置可以在本页运行的JS中直接使用hotcss.px2rem/hotcss.rem2px方法，无需再传递第二个值。

<meta name="hotcss" content="design-width=750">
<script src="/path/to/hotcss.js"></script>

**hotcss.mresize**
用于重新计算布局，一般不需要你手动调用。

    hotcss.mresize();


**hotcss.callback**
触发mresize的时候会执行该方法。

    hotcss.callback = function(){
      //your code here
    }

单位转换 `hotcss.px2rem` / `hotcss.rem2px`
hotcss.px2rem 和 hotcss.rem2px。你可以预先设定hotcss.designWidth可以在meta中设置design-width，则之后使用这两个方法不需要再传递第二个参数。

迭代后仍然支持在js中设置hotcss.designWidth，推荐使用meta去设置。

```js
/**
 * [px2rem px值转换为rem值]
 * @param  {[number]} px          [需要转换的值]
 * @param  {[number]} designWidth [设计图的宽度尺寸]
 * @return {[number]}             [返回转换后的结果]
 */
hotcss.px2rem( px , designWidth );
/**
 * 同上。
 * 注意：因为rem可能为小数，转换后的px值有可能不是整数，需要自己手动处理。
 */
hotcss.rem2px( rem , designWidth );
//你可以在meta中定义design-width，此后使用px2rem/rem2px，就不需要传递designWidth值了。同时也支持旧的设置方式，直接在JS中设置hotcss.designWidth
hotcss.px2rem(200);
hotcss.rem2px(350);
```

## 参考资料 

官方文档 http://imochen.github.io/hotcss/

github https://github.com/imochen/hotcss/blob/master/src/hotcss.js