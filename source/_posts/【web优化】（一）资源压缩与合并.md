---
title: 【web优化】（一）资源压缩与合并
date: 2018-3-26 13:17:52
layout: slides
tags: 
  - 性能优化
categories: 
  - 性能优化
---

<section>
	<h2>一、资源的合并与压缩</h2>
	<h3>核心点</h3>
	<ul>
		<li>减少HTTP数量请求</li>
		<li>减少请求资源大小</li>
	</ul>
</section>

<section>
	<h3>浏览器的一个请求从发送到返回经历了什么</h3>
	<p class="fragment">1. 浏览器输入url -> 回车</p>
	<p class="fragment">2. 解析 domain -> DNS服务器 -> 返回ip</p>
	<p class="fragment">3. IP地址打在协议中 -> 发送到网络 (经过局域网 -> 交换机 -> 路由器 -> 主干网络 -> 服务端)</p>
	<p class="fragment">4. 服务端返回资源（GET请求）浏览器进行渲染</p>
	<p class="fragment">5. render -> DOM -> CSSOM -> 下载并加载css、js、img等资源 -> reflow & repaint</p>
</section>

<section>
	<h3>针对以上，可作的优化工作</h3>
	<p class="fragment">1. DNS 资源缓存 -> 减少初次请求数及时间</p>
	<p class="fragment">2. 网络请求 CDN -> 解决网络高并发和缓存（不带主站cookie）</p>
	<p class="fragment">3. 接口优化 -> 浏览器的缓存策略 (相同资源、接口在浏览器读取缓存)</p>
</section>

<section>
	<h3>减少HTTP请求大小和HTTP请求次数</h3>
	<p><b>浏览器端渲染</b> vue、react 走框架代码渲染数据，而不是直出HTML（该过程对首屏有损耗，不利于性能）</p>
	<p class="fragment">服务端渲染 SSR，整个HTML直出到浏览器端</p>
</section>

<section>
	<h3>请求过程中一些潜在的性能优化点</h3>
	<ul>
		<li class="fragment">DNS是否可以通过缓存减少DNS查询时间？</li>
		<li class="fragment">网络请求的过程中走最近的网络环境？</li>
		<li class="fragment">相同的静态资源是否可以缓存？</li>
		<li class="fragment">能否减少请求HTTP请求大小</li>
		<li class="fragment">减少HTTP请求数</li>
		<li class="fragment">服务端渲染</li>
	</ul>
	<hr>
	<h3 class="fragment">深入理解 HTTP请求过程 是前端性能优化的核心</h3>
</section>

<section>
	<h3>通过案例学习，理解以下优化点</h3>
	<hr>
	<ul>
		<li>HTML文件压缩</li>
		<li>CSS压缩</li>
		<li>JS的压缩和混乱</li>
		<li>文件合并</li>
		<li>图片压缩并使用CDN</li>
		<li>开启Gzip</li>
	</ul>
</section>

<section>
	<h3>HTML压缩</h3>
	<p style="text-align: left">HTML代码压缩就是压缩这些在文本文件中有意义，但是在HTML中不显示的字符，包括`空格、制表符、换行符`等，还有一些其他意义的字符，如`HTML注释`也可以被压缩</p>
	<hr>
	<h3>如何进行HTML压缩</h3>
	<ul>
		<li>使用在线网站进行压缩</li>
		<li>Node.js提供了`html-minifier`工具(Node.js的压缩方案： 1构建阶段 2服务端)</li>
		<li>后端模版引擎渲染压缩</li>
	</ul>
</section>

<section>
	<h3>CSS压缩</h3>
	<ul>
		<li>无效代码删除</li>
		<li>css语义合并</li>
	</ul>
	<hr>
	<h3>如何进行css压缩</h3>
	<ul>
		<li>使用在线网站进行压缩</li>
		<li>使用`html-minifier`对html中的css进行压缩</li>
		<li>配置postcss等进行压缩</li>
	</ul>
</section>

<section>
	<h3>JS压缩与混乱</h3>
	<ul>
		<li>无效字符的删除</li>
		<li>剔除注释</li>
		<li>代码语义的缩减和优化</li>
		<li>代码保护</li>
	</ul>
	<hr>
	<h3>如何进行JS压缩</h3>
	<ul>
		<li>使用在线网站进行压缩</li>
		<li>使用 uglifyjs2 对js代码进行压缩</li>
		<li>在webpack、gulp中配置js压缩</li>
	</ul>
</section>

<section>
	<h3>文件合并</h3>	
	<p>不合并带来的弊端：</p>
	<ul>
		<li>文件与文件之间有插入的上行请求，增加了 N-1 个网络延迟</li>
		<li>受丢包问题影响更严重</li>
		<li>经过代理服务器时可能会被断开</li>
	</ul>
	<p>合并所带来的问题思考：1.首屏渲染问题  2.缓存失效问题</p>
</section>

<section>
	<h3>如何进行文件合并</h3>
	<ul>
		<li>使用在线网站进行文件合并</li>
		<li>使用Node.js实现文件合并(webpack、gulp)</li>
	</ul>
</section>