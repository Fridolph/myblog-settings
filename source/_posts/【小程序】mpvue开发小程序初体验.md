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

> 很有意思的点在于，小程序每个文件都有一个json配置文件，可理解为这是多页的SPA组合而成。如果要引入vuex需在每`页`的entry中引入。那问题是会不会出现类似于flux多个store导致状态管理混乱的情况？ 这个具体的还没有试过，等有时间把应用做大点，想尝试下。

一看就明白的配置，还是要说说。 setting里是针对打包后的dist目录，这里没有开启es6（毕竟babel本来就是编译为es5的）

miniprogramRoot 就是小程序开发工具 读的app入口了。client和svr是我根据官方的demo改的，这样，可以上传server到测试环境里，直接mock数据。

---

不用写那啥wxss和小程序原生的模版还是挺好，用多了和React、Vue挺混乱的，感觉后期会精分。

npm run dev 其实会打包出来 dist 而小程序找的是这个dist，其内容被mpvue转成了 微信小程序的原生模版映射。小程序会在开发环境下分配测试域名，我们要mock数据貌似只能通过该方式来（没了解太多，有其他方式请批评指教）

server/routes/index.js

		router.get('/books', ccontrollers.books)

server端是express，看着还是挺熟悉，这是定义接口，于是我们顺藤摸瓜到 server/controllers 下

`index.js`文件是作路由映射，`module.exports = mapDir(path.join(__dirname))` 找到当前文件夹下的文件或文件夹。我们随便打开个文件看看，如 user.js

```js
module.exports = async (ctx, next) => {
	// 通过 Koa 中间件进行登录态校验之后
	// 登录信息会被存储到 ctx.state.$wxInfo
	// 具体查看：
	if (ctx.state.$wxInfo.loginState === 1) {
			// loginState 为 1，登录态校验成功
			ctx.state.data = ctx.state.$wxInfo.userinfo
	} else {
			ctx.state.code = -1
	}
}
```

解释得很清楚，若访问服务端 `/user` 就能在`ctx.state`中拿到我们想到的数据了。

于是编写一个 books.js

```js
module.exports = ctx => {
  ctx.state.data = {
    books: [
			// 数据省略
		]
	}
}
```

在前台访问

```js
const HOST = '用腾讯云分配的域名即可'

export default {
	// 省略
	methods: {
		getCgiBooks() {
      // util.showBusy('请求中...')
      let that = this
      wx.getStorage({
        key: 'booklist',
        success(res) {
          // console.log('从缓存拿booklist', res.data)
          that.books = res.data
        },
        fail(err) {
          console.log(err)
          wx.request({
            url: `${HOST}/weapp/books`,
            login: false,
            success(result) {
              // util.showSuccess('请求成功完成')
              // console.log('请求服务端拿到booklist', result.data.data.books)
              that.books = result.data.data.books
              wx.setStorage({
                key: 'booklist',
                data: result.data.data.books
              })
            },
            fail(error) {
              // util.showModel('请求失败', error)
              console.log('request fail', error)
            }
          })
        }
      })
    }
	}
}
```

异步和回调很多，随着业务场景的复杂度提升感觉后期的回调会很可怕…… 所以可根据场景自行封装Promise，使用async/await的方式

---

于是假设做好了……

![](http://blog.fueson.top/article/img/miniapp1.jpg)
![](http://blog.fueson.top/article/img/miniapp2.jpg)
![](http://blog.fueson.top/article/img/miniapp3.jpg)


## 先挖坑

小程序还在开发中。直接拿vue来写，上手超快，有其他坑再上来补充好来。。

主要是小程序还未发布，只能本地玩，之后再把项目踩坑等传上来。
