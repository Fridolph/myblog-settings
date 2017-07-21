---
title: 【DEMO】纯CSS3手工打造变形金刚
date: 2017-02-20 17:29:49
tags: 
  - css3
  - demo
categories: 
  - CSS
---

老外写的一个很有意思的demo~ 就copy到本地了，地址找不到了… 请原谅我就不贴网址了
我把代码扒下来重新写了遍，整理了下就记录下来了，还是挺有意思的

只要有想法，真的是各种强啊， 简直不敢相信这居然是纯CSS代码完成的，里面有很多 transform 及 css三角的应用，代码读起来很顺， 理解起来也不难，但想到…那是真的难… 很多地方需要细细拆分才能琢磨透，先上图上DEMO

<iframe style="width:100%; height:460px;" src="https://jsfiddle.net/vho3j50x/embedded/result,html,css,js/"></iframe>

<!-- more -->

那开始我们的编程吧：

```html
<!-- 外层容器结构可以根据自己的情况来调整 -->
<div id="batianhu-wrapper">
    <div class="batianhu">
        <div class="head-body"></div>
        <div class="head-topcover"></div>
        <div class="head-toplogo-bg"></div>
        <div class="head-toplogo"></div>
        <div class="head-toplogo-2"></div>
        <div class="head-bottom-left"></div>
        <div class="head-bottom-right"></div>
        <div class="head-sw-left"></div>
        <div class="head-sw-right"></div>
        <div class="head-eye"></div>
        <div class="head-eye-2"></div>
    </div>
</div>
```

```css
#batianhu-wrapper {
  width: 250px;
  height: 325px;
  overflow: hidden;
  margin: 50px auto 20px auto;
}
```

```css
.head-body {
  height: 0;
  width: 200px;
  position: relative;
  margin: 25px 0 50px 0;
  border-top: 150px solid #555;
  border-left: 25px solid transparent;
  border-right: 25px solid transparent;
  z-index: 2;
}
```

```css
.head-topcover {
  position: relative; z-index: 5; top: -225px; left: -19px;
  border-top: 80px solid #333; 
  border-left: 60px solid transparent; 
  border-right: 60px solid transparent;
  width: 169px; height: 0;
}
```

```css
.head-topcover:after {
  content: ""; position: absolute; top: 0px; left: 0px;
  border-style: solid;
  border-color: #333 transparent transparent transparent;
  border-width: 15px 85px 0 85px;
  width: 0; height: 0;
}
```

```css
.head-toplogo-bg {
  position: relative; z-index: 6; top: -267px; left: 67px;
  border-top: 110px solid #333;
  border-left: 35px solid transparent;
  border-right: 35px solid transparent;
  width: 46px; height: 0;
}
```

```css
.head-toplogo-bg:after {
  content: ""; position: absolute; top: 0px; left: 0px;
  border-style: solid;
  border-color: #333 transparent transparent transparent;
  border-width: 30px 23px 0 23px;
  width: 0; height: 0;
}
```

```css
.head-toplogo {
  position: relative; z-index: 7; top: -377px; left: 75px;
  border-top: 105px solid #555;
  border-left: 30px solid transparent;
  border-right: 30px solid transparent;
  width: 40px; height: 0;
}
```

```css
.head-toplogo:before {
  content: ''; display: block;
  position: absolute; top: -42px; left: 10px;
  border-top: 30px solid #333;
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  width: 0; height: 0;
}
```

```css
.head-toplogo:after {
  content: "";
  position: absolute; top: 0px; left: 0px;
  border-style: solid;
  border-color: #555 transparent transparent transparent;
  border-width: 28px 20px 0 20px;
  width: 0; height: 0;
}
```

```css
.head-toplogo-2 {
  position: relative; z-index: 8; top: -482px; left: 75px; 
  border-top: 37px solid #333;
  border-left: 35px solid transparent;
  border-right: 35px solid transparent;
  width: 30px; height: 0;
}
```

```css
.head-bottom-left {
  display: block; margin: 50px 0;
  position: relative; top: -365px; left: -31px; z-index: 10;
  border-right: 107px solid transparent;
  border-bottom: 44px solid #555;
  border-left: 60px solid transparent;
  width: 0px; height: 0px;
  color: #555;
  -webkit-transform: rotate(236deg);
          transform: rotate(236deg);
}
```

```css
.head-bottom-right {
  display: block;
  position: relative; top: -460px; left: 115px;
  border-right: 60px solid transparent;
  border-bottom: 44px solid #555;
  border-left: 107px solid transparent;
  width: 0px; height: 0px;
  color: #555;
  -webkit-transform: rotate(484deg);
          transform: rotate(484deg);
}
```

如果有困惑的地方，可以把容器的`overflow:hidden`属性去掉， 一行一行去掉我们所写的css效果，看看是由怎样的变化得到的，如上图，我们就是用两个三角拼出来的脸颊，利用overflow属性得到了我们想要的效果 

```css
.head-sw-left {
  position: relative; z-index: 15; left: 45px; top: -629px;
  width: 49px; height: 8px;
  background: #333;  
  -webkit-transform: skew(-149deg) rotate(9deg);
          transform: skew(-149deg) rotate(9deg);
}

.head-sw-left:after {
  content: "";
  position: absolute; top: 24px; left: -6px;
  width: 53px; height: 8px;
  background: #333;
  -webkit-transform: skew(-174deg) rotate(1deg);
          transform: skew(-174deg) rotate(1deg);
}
```

```css
.head-sw-right {
  position: relative; z-index: 15; left: 155px; top: -637px;
  width: 49px; height: 8px;
  background: #333;
  -webkit-transform: skew(279deg) rotate(10deg);
          transform: skew(279deg) rotate(10deg);
}

.head-sw-right:after {
  content: "";
  position: absolute; top: -2px; left: 132px;
  width: 45px; height: 9px;
  background: #333;
  -webkit-transform: skew(-212deg) rotate(0deg);
          transform: skew(-212deg) rotate(0deg);
}
```

```css
.head-eye {
  position: relative; z-index: 12; left: 67px; top: -585px;
  box-shadow: rgba(255, 255, 255, 0.4) 0 0 15px, rgba(76, 190, 255, 0.95) 0 0 10px;
  width: 32px; height: 22px;
  background: #4CBEFF;
  -webkit-transform: skew(46deg) rotate(14deg);
          transform: skew(46deg) rotate(14deg);
}

.head-eye-2 {
  position: relative; z-index: 12; left: 150px; top: -607px;
  box-shadow: rgba(255, 255, 255, 0.4) 0 0 15px, rgba(76, 190, 255, 0.95) 0 0 10px;
  width: 32px; height: 22px;
  background: #4CBEFF;
  -webkit-transform: skew(-46deg) rotate(-14deg);
          transform: skew(-46deg) rotate(-14deg);
}
```

`文章到此的完整展示代码请参考上面demo的jsfiddle里的css部分`


至此，我们的纯css所写的变形金刚的静态效果就完成了，是不是很有趣呢？
这个结构按照身体各部分拆成了很多份，感兴趣还可以做成动画~~ 待填坑...