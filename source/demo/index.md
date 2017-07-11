---
title: 欢迎访问我的作品集 —— 展示了我参与的项目及个人作品等
date: 2017-06-15 16:28:43
---

[具体细节请移步我的Github仓库](https://github.com/Fridolph)

这里的介绍分为两部分：

* [实战项目](https://github.com/Fridolph/my-program) 这里收录了本人实际线上的项目或其他具有一定参考意义的作品

* [练习Demo](https://github.com/Fridolph/my-demo) 静态展示为主，该仓库里的内容大多独立，可以单独跑在jsFiddle或CodePen中

<style>
@media (min-width: 600px) {
  .item {display: flex;}  
}
.item .item-content{flex: 0 0 1;margin: 10px;padding: 10px;}
.item .item-img{flex: 0 0 50%;margin: 10px;padding: 10px;}

.content-title {font-size: 20px;font-weight:bold;}
</style>

## 线上 / 运营项目

<div class="item">
  <div classs="item-content">
    <div class="content-title"><a href="https://www.liyi.top">环球礼仪</a></div>
    <div class="content-desc">上线运营中，四川爱礼科技有限公司的主业务，打造最大最强的线上礼仪平台，利用大数据和互联网基础设施,研发具有时代气息的礼仪文化产品,提供专业、实际、轻松的礼仪教育资讯。</div><ul class="content-dev"><li>语言：HTML/CSS/JavaScript</li><li>框架：React/Redux</li><li>框架：jQuery/Bootstrap/Antd</li><li>环境：Node v6.11.0/npm 4</li></ul><div class="content-my">16年5月开始项目的核心建设，项目前端由jquery+bootstrap的整页构建，一直迭代到以React+Redux为核心的组件化开发，根据设计规范完成了aili-ui的UI库的开发</div>
  </div>
  <div class="item-img">
    <img src="https://img1.doubanio.com/lpic/s29478358.jpg" alt="我的demo1" />
  </div>
</div>


环境参数
* 语言 HTML CSS JavaScript Node.js
* 框架 Think.js
* 前端框架 React React-router Redux jQuery
* UI库 Bootstrap AntDesign
* 打包工具 Webpack

`技术栈格式大同小异，下面的就省略了直接说重点`

### [四川爱礼科技有限公司官网](http://www.ili.top) 
上线运营中，四川爱礼科技有限公司官网，以展示为主。

`技术栈 Vue2 + vue-router + vue-resource`

官网正逢改版，之前是外包做的一个简单页，正好拿之前自学的vue练练手，vue-cli搭了环境自己写了个简单的express服务器就上线跑起了，获取专家信息用了vue-resource，v-for渲染数据快捷，不涉及过多数据交互就没用到Vuex。