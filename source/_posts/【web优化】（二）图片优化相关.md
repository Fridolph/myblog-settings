---
title: 【web优化】（二）图片优化相关
date: 2018-1-22 19:17:52
layout: slides
tags: 
  - 性能优化
categories: 
  - 性能优化
---

<section>
	<h2>二、图片相关知识与优化</h2>
	<h3>核心点</h3>
	<ul>
		<li>超过一半的流量和时间都用来下载图片</li>
		<li>图片优化效果明显将大大提高网站性能</li>
	</ul>
</section>

<section>
	<h3>有损压缩，一张JPG图片的解析过程</h3>
	<p class="fragment">Raw Image Data -> Color Transform -> Down sampling -> Forward DCT -> Quantization -> Encoding</p>
	<hr>
	<p class="fragment">源数据 -> 颜色转换 -> 颜色采样（区分高频低频颜色变化） -> 对高频颜色进行压缩DCT -> 质量控制 -> 编码</p>
</section>

<section>
	<h3>不同格式图片常用的业务场景</h3>
	<ul>
		<li class="fragment">1. jpg有损压缩，压缩率高，不支持透明。用于大部分不需要透明图片的业务场景</li>	
		<li class="fragment">2. png支持透明，浏览器兼容性好。用于大部分需要透明图片的业务场景</li>	
		<li class="fragment">3. webp压缩程度更好，在ios webview有兼容性问题。适用所有高版安卓</li>	
		<li class="fragment">4. svg矢量图，代码内嵌，相对较小，图片样式相对简单的场景</li>	
	</ul>
</section>

<section>
	<h3>图片压缩</h3>
	<ul>
		<li class="fragment">1. CSS雪碧图 减少HTTP请求数（至少1次）整张图较大时也会存在加载慢的现象</li>	
		<li class="fragment">2. Image inline base64 （将图片的内容内嵌到html中 -> 减少网站的HTTP请求数(0)）</li>	
		<li class="fragment">3. 在安卓下使用webp</li>	
		<li class="fragment">webp 优势在于更好的图像压缩压缩算法带来的更小图片体积。同时具备有损和无损压缩模式、alpha透明及动画特性，在jpg和png上转化效果都很优秀、统一和稳定</li>
	</ul>	
</section>