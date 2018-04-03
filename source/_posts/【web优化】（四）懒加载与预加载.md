---
title: 【web优化】（四）懒加载与预加载
date: 2018-3-27 22:11:24
layout: slides
tags: 
  - 性能优化
categories: 
  - 性能优化
---

<section>
	<h2>四、懒加载和预加载</h2>
	<h3>核心点（这里只以图片举例）</h3>
	<ul>
		<li>理解懒加载和预加载的原理</li>
		<li>懒加载与预加载案例</li>
    <li>懒加载与预加载实战</li>
	</ul>
</section>

<section>
  <h3>什么是懒加载</h3>
  <ul>
    <li class="fragment">图片进入可视区之后请求图片资源</li>
    <li class="fragment">对于电商等图片很多、页面很长的业务场景适用</li>
    <li class="fragment">减少无效资源的加载</li>
    <li class="fragment">并发加载的资源过多会阻塞js的加载，影响网站的正常使用</li>
  </ul>
</section>

<section>
  <h3>图片懒加载</h3>
  <ul>
    <li class="fragment">需要监听scroll事件，在scroll事件的回调中</li>
    <li class="fragment">去判断设置的懒加载图片是否进入可视区域</li>
    <li class="fragment">到可视区通过data-src，将src赋给图片从而发出请求</li>
  </ul>
</section>

<section>
  <h3>什么是预加载</h3>
  <ul>
    <li class="fragment">图片等静态资源在使用之前提前请求</li>
    <li class="fragment">资源使用到时能从缓存中加载，提升用户体验</li>
    <li class="fragment">页面展示的依赖关系维护</li>
  </ul>
</section>

<section>
  <h3>预加载的几种实践方案</h3>
  <ul>
    <li class="fragment">通过设置样式，默认是通过DOM加载的，但不显示</li>
    <li class="fragment">使用Image对象，并设置src，这样脚本运行便会加载，动态添加到需要的DOM中即可</li>
    <li class="fragment">通过XHR加载（可能会有跨域问题）</li>
  </ul>
</section>
