---
title: 【web优化】（六）浏览器存储
date: 2018-3-28 21:17:32
layout: slides
tags: 
  - 性能优化
categories: 
  - 性能优化
---

<section>
	<h2>六、浏览器存储</h2>
	<h3>核心点</h3>
	<ul>
		<li>理解localStorage、cookie、sessionStorage、indexDB的概念和使用</li>
		<li>学习理解PWA和Service Worker的应用</li>
    <li>多种浏览器存储方式并存</li>
	</ul>
</section>

<section>
  <h3>什么是Cookie</h3>
  <p>Cookie 是在 HTTP 协议下，服务器或脚本可以维护客户工作站上信息的一种方式，也可叫作浏览器缓存</p>
  <h4>Cookie 生成方式</h4>
  <ul>    
    <li>http response header 中的 set-cookie</li>
  </ul>
</section>

<section>
  <h4>Cookie 用途</h4>
  <ul>
    <li class="fragment">用于浏览器端和服务器端的交互</li>
    <li class="fragment">客户端自身数据的存储</li>
  </ul>
  <h4 class="fragment">过期时间 expire</h4>
  <p class="fragment">因为HTTP请求无状态，所以需要Cookie去维持客户端状态</p>
</section>

<section>
  <h3>Cookie存储限制</h3>
  <ul>
    <li class="fragment">浏览器存储 大小4KB左右</li>
    <li class="fragment">需要设置过期时间 expire</li>
    <li class="fragment">cookie属性有 httponly 是不支持js读写</li>
    <li class="fragment">CDN的流量损耗(故CDN域名不要携带cookie)</li>
    <li class="fragment">CDN的域名和主站的域名要分开</li>
  </ul>
</section>

<section>
  <h3>LocalStorage</h3>
  <ul>
    <li class="fragment">HTML5设计出来专门用于浏览器存储的</li>
    <li class="fragment">本地存储大小为5M左右</li>
    <li class="fragment">仅在客户端使用，不和服务端进行通信</li>
    <li class="fragment">接口封装较好</li>
    <li class="fragment">浏览器本地缓存方案</li>
  </ul>
</section>

<section>
  <h3>SessionStorage</h3>
  <ul>
    <li class="fragment">会话级别的浏览器存储</li>
    <li class="fragment">大小为5M左右</li>
    <li class="fragment">仅在客户端使用，不和服务端进行通信</li>
    <li class="fragment">接口封装较好</li>
    <li class="fragment">对于表单信息的维护</li>
  </ul>
</section>

<section>
  <h3>IndexDB</h3>
  <p  class="fragment">IndexDB是一种低级API，用于客户端存储大量数据化数据。该API使用索引来实现对该数据的高性能搜索。</p>
  <p  class="fragment">虽然Web Storage对于存储较少量数据很有用，但对于存储更大量结构化数据来说，这种方法不太有用，IndexDB提供了一个解决方案为应用创建离线版本</p>
</section>

<section>
  <h3>Service Workers</h3>
  <p  class="fragment">Service Worker 是一个脚本，浏览器独立于当前网页，将其在后台运行，为实现一些不依赖页面或者用户交互的特性打开了一扇大门。</p>
  <p  class="fragment">在未来这些特性将包括推送消息，背景后台同步，geofencing(地理围栏定位)，但它将推送的第一个首要特性就是拦截和处理网络请求的能力，包括以编程方式来管理被缓存的响应。</p>
</section>

<section>
  <h3>PWA (Progressive Web App)</h3>
  <p  class="fragment">是一种Web APP新模型，并不是具体指某一种前沿技术或者某一个单一的知识点。这是一系列新的Web特性，配合优秀的UI交互设计，逐步地增强Web APP的用户体验</p>
  <ul>
    <li class="fragment">可靠：在没有网络的环境也能提供基本的页面访问，而不会出现未连接到互联网的页面</li>
    <li class="fragment">快速：针对网页渲染及网络数据访问有较好地优化</li>
    <li class="fragment">融入：应用可以被增加到手机桌面，并且和普通应用一样有全屏、推送等特性</li>
  </ul>
</section>

