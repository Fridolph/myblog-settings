---
title: 【框架】Reveal.js一个Web演示文稿框架
date: 2018-4-3 13:17:52
tags: 
  - reveal.js
categories: 
  - 前端框架
---

> A framework for easily creating beautiful presentations using HTML. Check out the live demo.

> 首先安利一波 [melody](https://molunerfinn.com/hexo-theme-melody-doc/#/) 一个漂亮的hexo主题，集合了N多主题的优点于一身，可配置性强、简洁、美观，在知乎刷到后马上就动手把主题换了，读了读文档，有很多好玩的东西，这就算其中之一吧，遂有了这篇博客。

<!-- more -->

## 介绍

> A framework for easily creating beautiful presentations using HTML. Check out the live demo.
> reveal.js comes with a broad range of features including nested slides, Markdown contents, PDF export, speaker notes and a JavaScript API. There's also a fully featured visual editor and platform for sharing reveal.js presentations at slides.com.

reveal.js是一个能够帮助我们很轻易地使用HTML来创建漂亮的演示效果，也就是我们常见的PPT幻灯片。reveal.js不依赖其他任何javascript库，是一个独立的javascript插件库。它提供了多种幻灯片过渡效果，是一个非常棒的在线演示库。

### 在html里使用

我们先引入主要的CSS文件以及js文件。CSS文件要在head内就载入，而reveal.js可以在`</body>`前载入

```html
<link rel="stylesheet" href="css/reveal.css">
<link rel="stylesheet" href="css/theme/moon.css">
<script src="js/reveal.js"></script>
```

HTML标记的层次结构需要是 `.reveal > .slides > section` 这样的，`<section>` 代表一个幻灯片并且能够被无限地重复。如果我们将多个 `<section>` 放到另一个 `<section>` 的内部，它们将会以垂直幻灯片的方式显示。`<section>` 内部可以是文本、图片、多媒体等任意HTML内容。例如：

```html
<div class="reveal">
  <div class="slides">
    <section>slide1</section>
    <section>slide2</section>
  </div>
</div>
```

在页面最后，我们需要运行下面的代码来初始化幻灯片。注意，所有的配置的值都是可选的，下面展示的都是默认值：

```html
<script>
Reveal.initialize({
	// 是否在右下角展示控制条
	controls: true,
	// 是否显示演示的进度条
  progress: true,
  // 是否显示当前幻灯片的页数
  slideNumber: 'c/t'
});
</script>
```

在melody主题中，作者大大已经为我们封装好了，在 `_config.yml` 里的 slide 选项下直接配置就好

### hexo主题

```yaml
slide:
  separator: ---
  separator_vertical: --
  charset: utf-8
  theme: black
  # optional
  mouseWheel: false
  transition: slide
  transitionSpeed: default
  parallaxBackgroundImage: ''
  parallaxBackgroundSize: ''
  parallaxBackgroundHorizontal: null
  parallaxBackgroundVertical: null
```

## 配置项

下面介绍一下可选配置：（对应上面的api，可自行添加、修改）

|参数|描述|默认值|
|:--:|:--|:----:|
|controls|是否在右下角展示控制条|true|
|progress|是否显示演示的进度条|true|
|slideNumber|是否显示当前幻灯片的页数编号，也可以使用代码slideNumber: 'c / t' ，表示当前页/总页数|false|
|history|是否将每个幻灯片改变加入到浏览器的历史记录中去|false|
|keyboard|是否启用键盘快捷键来导航|true|
|overview|是否启用幻灯片的概览模式，可使用"Esc"或"o"键来切换概览模式|true|
|center|是否将幻灯片垂直居中|true|
|touch|是否在触屏设备上启用触摸滑动切换|true|
|loop|是否循环演示|false|
|rtl|是否将演示的方向变成RTL，即从右往左|false|
|fragments|全局开启和关闭碎片|true|
|autoSlide|两个幻灯片之间自动切换的时间间隔（毫秒），当设置成 0 的时候则禁止自动切换，该值可以被幻灯片上的 `data-autoslide` 属性覆盖|0|
|transition|切换过渡效果，有none/fade/slide/convex/concave/zoom|'default'|
|transitionSpeed|过渡速度，default/fast/slow|'default'|
|mouseWheel|是否启用通过鼠标滚轮来切换幻灯片|true|

---

## 其他

reveal.js还有一个片段概念，片段被用来在一个幻灯片中来突出显示单独的一个元素。每一个带有 fragment 样式的元素将会在切换到下一个幻灯片之前被走过。默认的片段样式是开始不可见，然后淡入，我们可以将同一张幻灯片里的多个段落分作多个片段，并给他们加上.fragment样式即可，就像DEMO演示中的：

```html
<section>
	<h2>幻灯片切换方式</h2>
	<p class="fragment">右下角控制条控制切换</p>
	<p class="fragment">可以使用键盘方向键操作</p>
	<p class="fragment">可以设置使用鼠标滚轮切换</p>
	<p class="fragment">移动端滑动切换</p>
</section>
```

关于幻灯片切换效果，是通过transition配置值来设定的。我们也可以通过指定data-transition属性来重写全局配置。例如：

```html
<section data-transition="zoom">
  <h2>This slide will override the presentation transition and zoom!</h2>
</section>
```

光说不练，怎么也搞不明白，遂请看下一篇，直接拿reveal写篇博客就好。

## 参考文档

更多请参考官方文档 [https://github.com/hakimel/reveal.js#configuration](https://github.com/hakimel/reveal.js#configuration)

