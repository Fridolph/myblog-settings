---
title: 【小程序】mpvue开发小程序初体验
date: 2018-5-7 21:52:15
tags:
  - mpvue
  - 小程序
categories:
  - 小程序
---

> mpvue （github 地址请参见）是一个使用 Vue.js 开发小程序的前端框架。框架基于 Vue.js 核心，mpvue 修改了 Vue.js 的 runtime 和 compiler 实现，使其可以运行在小程序环境中，从而为小程序开发引入了整套 Vue.js 开发体验。

具体可以参考官方文档，之前虽然也尝试过小程序，就觉得是vue + react，半杂着es6和模版代码，由于没有css预处理器写起来样式和组件化感觉都很麻烦，正好趁着火热，把一直想做的小程序继续完成了。

<!-- more -->

周末做了2天总算是熬出来了 ……实在是不想写博客啊。 好了不发牢骚，开始正文

小程序的初始化这里就不介绍了，搬网上的文字也没意思， 直接看这里 http://mpvue.com/mpvue/quickstart/

项目跑起来打包出来的dist就是我们需要的文件了，我们先看下根目录下的配置文件 project.config.json

```json
{
	"description": "项目配置文件。",
	"setting": {
		"urlCheck": true,
		"es6": false,
		"postcss": true,
		"minified": true,
		"newFeature": true
	},
	"miniprogramRoot": "./dist/",
	"client": "./dist",
	"svr": "./server",
	"qcloudRoot": "./server",
	"appid": "你的小程序id",
	"projectname": "my-miniapp",
	"compileType": "miniprogram"
  // condition 不知道干啥的，这里没贴
}
```

一看就明白的配置，还是要说说。 setting里是针对打包后的dist目录，这里没有开启es6（毕竟babel本来就是编译为es5的）

miniprogramRoot 就是小程序开发工具 读的app入口了。client和svr是我根据官方的demo改的，这样，可以上传server到测试环境里，直接mock数据。

---

![](http://blog.fueson.top/article/img/miniapp1.jpg)
![](http://blog.fueson.top/article/img/miniapp2.jpg)
![](http://blog.fueson.top/article/img/miniapp3.jpg)

小程序还在开发中。直接拿vue来写，上手超快，有其他坑再上来补充好来。。

主要是小程序还未发布，只能本地玩，之后再把项目踩坑等传上来。
