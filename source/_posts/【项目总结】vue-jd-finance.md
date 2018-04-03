---
title: 【项目总结】vue-jd-finance
date: 2018-03-30 21:41:03
tags: 
  - 总结
  - 实战
categories: 
  - 总结
---

> 今年3月做的一个小项目，虽然不难，但有很多值得思考和回味的地方，加上自己写东西一向不勤快，趁着小项目完工，思路方法热情都还历历在目，赶紧开篇记录下这些点点滴滴。

<!-- more -->

## 项目开始

这是在慕课实战的一个小项目，我觉得自己最大的进步在于，以前做黄老师的Vue是手把手跟着视频敲代码，而现在是看了两节知道了整体思路后，模块、目录、结构都是由自己进行了拆分和重构，自己折腾出来了，再对比视频，更多的是看老师实现的思路，当然，因项目不复杂才有了上述操作吧。

在实战过程中，提出了很多问题，这个也是我想写博客的原因，问题的回答也是自己慢慢摸索进行了总结

![](http://blog.fueson.top/img/2017/jd01.png)

![](http://blog.fueson.top/img/2017/jd02.png)

![](http://blog.fueson.top/img/2017/jd03.png)

![](http://blog.fueson.top/img/2017/jd04.png)

![](http://blog.fueson.top/img/2017/jd05.png)

![](http://blog.fueson.top/img/2017/jd06.png)

## 需求分析

头部
轮播图
新手特权
极速借贷
理财精选
精品推荐
生活服务
在线客服
公司介绍
导航条

**项目目录**

```js
- src
  - assets            // 静态资源
  - components        // 可拆分的组件
  - layout            // 布局相关组件
  - lib               // 公共方法等，请求，计算等
  - routes            // 路由相关
  - views             // container 页面容器 其相关子组件也放到对应文件夹中
  - App.vue           // 起始文件
  - main.js           // 入口
```

### CSS模块化设计

#### 设计原则

* 可复用、能继承、要完整
* 周期性迭代

#### 设计方法

* 先整体再后部再颗粒化

布局 -> 页面 -> 功能 -> 业务

* 先抽象再具体

### JS组件设计原则

* 高内聚、低耦合
* 尽可能抽象
* 周期性迭代

### 自适应

1. 基本概念

* CSS像素、设备像素、逻辑像素、设备像素比
* viewport
* rem

css里的 1px px就是css像素了，和逻辑单位一致。
设备像素比：css像素和物理像素的比值

2. 原理

* 利用viewport和设备像素比调整基准像素
* 利用px2rem自动转换CSS单位

### SPA设计

#### 设计意义

* 前后端分离
* 减轻服务器压力
* 增强用户体验
* Prerender预渲染优化SEO

### 原理

* History API
* Hash

### 生产构建

* 合并 style javascript
* 抽取 样式要从 javascript中抽取出来
* 压缩 js、css、图片等
* 调试 开启sourceMap

### 发布部署

* 提交 使用git提交代码 管理线上版本
* 部署 从git仓库拉取代码，通过小流量、跨机房、全量部署
* 开启Gzip 开启gzip压缩
* 更新CDN 

### 构建工具

* babel 
* webpack
* dev-server

### 代码校验 eslint

* eslint-plugin-vue
* mystaticatea.github.op/vue-eslint-demo

## 参考

windows https://github.com/coreybutler/nvm-windows
mac https://github.com/creationix/nvm#install-script

**构建环境相关**

webpack 4.10.0  
https://doc.webpack-china.org

npm scripts  
http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html

babel 
https://babeljs.cn/docs/usage/polyfill
https://www.imooc.com/article/21866
