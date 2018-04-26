---
title: 【web优化】（七）浏览器缓存优化
date: 2018-2-3 21:45:42
layout: slides
tags: 
  - 性能优化
categories: 
  - 性能优化
---

<section>
	<h2>七、浏览器缓存优化</h2>
	<h3>核心点</h3>
	<ul>
		<li>了解缓存策略相关设置</li>
    <li>缓存优化实践</li>
	</ul>
</section>

<section>
  <h2>设置 HTTP Header</h2>
  <ul>
    <li>Cache-Control</li>
    <li>Expires</li>
    <li>Last-Modified / If-Modified-Since</li>
    <li>Etag / If-None-Match</li>
  </ul>
</section>

<section>
  <h3>Cache-Control</h3>
  <p>可出现在request header和response header中，其作用是 让浏览器和客户端相互知道各自的缓存策略情况</p>
  <ul>
    <li>max-age</li>
    <li>private</li>
    <li>public</li>
    <li>no-cache</li>
    <li>no-store</li>
  </ul>
</section>

<section>
  <h3>Cache-Control</h3>
  <ul>
    <li class="fragment">max-age 缓存的最大有效时间，浏览器再次请求该资源时不会向服务端发起请求 </li>
    <li class="fragment">s-maxage 同上，但只能指定public缓存</li>
    <li class="fragment">public</li>
    <li class="fragment">no-cache 发请求到服务端判断浏览器缓存是否过期</li>
    <li class="fragment">no-store 完全不使用缓存策略</li>
  </ul>
</section>

<section>
  <h3>Last-Modified / If-Modified-Since</h3>
  <h4>基于客户端和服务端协商的缓存机制</h4>
  <ul>
    <li class="fragment">Last-Modified - response header</li>
    <li class="fragment">If-Modified-Since - request header</li>
    <li class="fragment">需要与cache-control一起使用</li>
  </ul>
  <h4>缺点</h4>
  <ul>
    <li class="fragment">某些服务端不能获取精确的修改时间</li>
    <li class="fragment">文件修改时间改了，但文件内容没有变</li>
  </ul>
</section>

<section>
  <h3>Etag / If-None-Match</h3>
  <ul>
    <li class="fragment">文件内容的hash值</li>
    <li class="fragment">Etag - response header</li>
    <li class="fragment">If-none-match - request header</li>
    <li class="fragment">需要与cache-control 共同使用</li>
  </ul>
</section>

<section>
  <h2>分级缓存策略</h3>  
</section>

<section>
  <h3>200 HTTP Code</h3>
  <p>当浏览器本地没有缓存或者下一层失效时，可通过ctrl+f5 浏览器去服务器下载最新数据</p> 
</section>

<section>
  <h3>304 HTTP Code</h3>
  <p>这一层由 last-modified / Etag 控制，当下一层失效时或用户点击refresh f5，浏览器发送请求给服务器，若服务端没变化则返回304给浏览器</p> 
</section>

<section>
  <h3>200 HTTP Code (from cache)</h3>
  <p>这一层由 expires/cache-control 控制</p>

  <ul>
    <li class="fragment">expires (http 1.0) 是绝对时间</li>
    <li class="fragment">cache-control (http 1.1) 相对时间，两者都存在，cache-control覆盖expire，只要没失效，浏览器只访问自己的缓存</li>
  </ul>
</section>
