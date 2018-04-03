---
title: 【web优化】（五）回流与重绘
date: 2018-3-28 11:57:52
layout: slides
tags: 
  - 性能优化
categories: 
  - 性能优化
---

<section>
	<h2>五、回流与重绘</h2>
	<h3>核心点</h3>
	<ul>
		<li>理解浏览器重绘与回流的机制</li>
		<li>对于一些经典案例进行分析</li>
    <li>重绘与回流的案例相关解读</li>
	</ul>
</section>

<section>
  <h3>CSS性能让JavaScript变慢？</h3>
  <p class="fragment">因为CSS放在head中，加载CSS也会阻塞浏览器资源加载</p>
  <ul>
    <li class="fragment">一个线程 => UI渲染</li>
    <li class="fragment">一个线程 => JS引擎</li>
  </ul>
  <p class="fragment">频繁触发重绘与回流，会导致UI频繁渲染，从而阻塞JS执行，最终导致JS变慢</p>
</section>

<section>
  <h3>回流 Reflow</h3>
  <p class="fragment">当render tree中的一部分或全部因为元素的规模尺寸、布局、隐藏等改变而需要重新构建。这就称为回流，Reflow</p>
  <p class="fragment">当页面布局和几何属性改变时就需要Reflow。Reflow涉及到页面布局、大小变化等</p>
</section>

<section>
  <h3>重绘 Repaint</h3>
  <p class="fragment">当render tree中的一些元素需要更新属性，而这些属性只是影响元素的外观、风格，而不影响布局的，比如backgrond-color 则就称为重绘 Repaint</p>
  <p class="fragment">页面发生（任何）变化，都会发生重绘</p>
  <p class="fragment"><b>回流必将引起重绘，而重绘不一定引起回流</b></p>
</section>

<section>
  <h3>触发Repaint的属性 - 盒模型相关</h3>
  <ul>
    <li>display</li>
    <li>padding margin</li>
    <li>border-width border</li>
    <li>width height min-height</li>
  </ul>
</section>

<section>
  <h3>触发Repaint的属性 - 定位、浮动</h3>
  <ul>
    <li>clear</li>
    <li>float</li>
    <li>position</li>
    <li>top bottom left right</li>
  </ul>
</section>

<section>
  <h3>触发Repaint的属性 - 改变文字结构</h3>
  <ul>
    <li>overflow overflow-y</li>
    <li>vertical-align</li>
    <li>line-height</li>
    <li>font-size font-family font-weight</li>
    <li>text-align</li>
    <li>white-space</li>
  </ul>
</section>

<section>
  <h3>只触发Reflow的属性</h3>
  <ul>
    <li>visibility</li>    
    <li>color box-shadow</li>
    <li>border-style border-radius</li>
    <li>text-decoration</li>
    <li>background background-image background-position background-repeat background-size</li>
    <li>outline outline-color outline-style outline-width</li>
  </ul>
</section>

<section>
  <h3>Reflow、Repaint优化</h3>
  <ul>
    <li class="fragment">避免使用触发回流、重绘的CSS属性</li>    
    <li class="fragment">将回流、重绘的影响范围限制在单独的图层之内</li>    
    <li class="fragment">将频繁重绘回流的DOM元素单独作为一个独立图层，那么这个DOM元素的重绘和回流影响只会在这个图层中</li>    
  </ul>
</section>

<section>
  <h3>新建DOM的过程</h3>
  <ol>
    <li>获取DOM后分隔为多个图层</li>    
    <li>对每个图层的节点计算样式结果 Recalculate style</li>    
    <li>为每个节点生成图形和位置 Layout</li>    
    <li>将每个节点绘制填充到图层位图中</li>    
    <li>图层作为纹理上传至GPU</li>    
    <li>符合多个图层到页面上生成最终屏幕图像</li>    
  </ol>
</section>

<section>
  <h3>Chrome 创建图层的条件</h3>
  <ol>
    <li>3D或透视变换的CSS属性 perspective transform</li>    
    <li>使用加速视频解码 video节点</li>    
    <li>拥有3D (WebGL) 上下文或加速的2D上下文的canvas节点</li>    
    <li>混合插件 如Flash</li>    
    <li>对自己的opacity 做CSS动画或使用一个动画webkit变换的元素</li>    
    <li>拥有加速CSS过滤器的元素</li>    
    <li>元素有一个包含复合层的后代节点</li>    
    <li>元素有一个 z-index 较低且包含一个复合层的兄弟元素</li>    
  </ol>
</section>

<section>
  <h3>实战优化点 1</h3>
  <ol>
    <li>用 translate 替代 top 改变</li>    
    <li>用opacity 替代 visibility</li>    
    <li>不直接修改DOM样式，而是预先定义好class</li>    
    <li>把DOM离线后修改（先隐藏计算后再显示）</li>    
    <li>不要把DOM结点的属性值放在循环里当成循环里的变量</li>
  </ol>
</section>

<section>
  <h3>实战优化点 2</h3>
  <ol>
    <li>不要使用table布局</li>    
    <li>动画实现的速度的选择</li>    
    <li>对于动画新建图层</li>    
    <li>合理启用GPU硬件加速</li>   
  </ol>
</section>
