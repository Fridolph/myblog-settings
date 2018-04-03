---
title: 【web优化】（三）JS加载与执行
date: 2018-3-27 21:17:52
layout: slides
tags: 
  - 性能优化
categories: 
  - 性能优化
---

<section>
	<h2>三、JS脚本加载与执行</h2>
	<h3>核心点</h3>
	<ul>
		<li>理解浏览器端html、css、js的加载过程</li>
		<li>css、js加载过程中的优化点</li>
    <li>深入理解和学习JS相关优化点</li>
	</ul>
</section>

<section>
	<h3>网站在浏览器端是如何进行渲染的？</h3>
	<ul>
    <li class="fragment">1. 通过HTTP请求拿回的html文件，读取html</li>
    <li class="fragment">2. 字节流(服务器端) -> 字符流(浏览器端) -> 语法分析 拿到相应 token 添加到DOM树</li>
    <li class="fragment">3. （html -> DOM树）link方式 并发 同时css -> CSSOM树)</li>
    <li class="fragment">4. V8引擎解析JavaScript(阻塞)</li>
    <li class="fragment">5. Render Tree (DOM CSSOM都准备好才做这步)</li>
    <li class="fragment">6. 回流 reflow 重绘 repaint</li>
  </ul>
</section>

<section>
  <h3>HTML渲染过程的一些特点</h3>
  <ul>
    <li class="fragment">顺序执行、并发加载</li>
    <li class="fragment">是否阻塞</li>
    <li class="fragment">依赖关系</li>
    <li class="fragment">引入方式</li>
  </ul>
</section>

<section>
  <h4>顺序执行、并发加载</h4>
  <ul>
    <li class="fragment">词法分析 - token分析从上到下，从而html是从上到下解析</li>
    <li class="fragment">并发加载 - html中引入的外部资源是并发加载的</li>
    <li class="fragment">并发上限 - 根据浏览器不同 一般来说HTTP1.1下是6个</li>
  </ul>
</section>

<section>
  <h4>CSS阻塞</h4>
  <ul>
    <li class="fragment">css加载会阻塞页面的渲染  -> css文件在head标签中通过link引入</li>
    <li class="fragment">css阻塞js的执行 ->  在加载css时后续js的执行是会被阻塞的</li>
    <li class="fragment">css不阻塞外部脚本的加载 （阻塞执行js而不阻塞加载js）</li>
  </ul>
</section>

<section>
  <h4>js阻塞</h4>
  <ul>
    <li class="fragment">直接引入的js阻塞页面的渲染</li>
    <li class="fragment">js的执行不阻塞资源的加载</li>
    <li class="fragment">js顺序执行，阻塞后续js逻辑的执行</li>
  </ul>
</section>