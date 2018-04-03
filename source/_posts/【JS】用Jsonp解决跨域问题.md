---
title: 【JS】用Jsonp解决跨域问题
date: 2016-06-04 21:52:15
tags: 
  - jsonp
  - 跨域
categories: 
  - js
---

> 该篇讲述了同源策略，再到跨域的解决。学习分析了流行的jsonp方案从源码上加深了对jsonp的理解和认识

<!-- more -->

## 同源策略

要理解跨域，先要了解一下“同源策略”。所谓同源是指，域名，协议，端口相同。所谓“同源策略“，简单的说就是基于安全考虑，当前域不能访问其他域的东西。

    http://www.abc.com:8080/home?k=v

这么一个url地址，由协议，域名，端口，路径等部分组成。怎么来区分跨域与否呢？ 这里简单列了一下：

**跨域**

* 不同协议
* 域名不同
* 端口不同

**不跨域**

* 协议、域名和端口相同
* 同一域名的不同路径下

**跨域情况**

```js
var btn = document.getElementById('btn');
btn.onclick = function() {
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status === 200) {
      console.log(xhr.responseText);
    }
  };
  xhr.open('get', 'https://api.douban.com/v2/book/search?q=javascript&count=1', true);
  xhr.send();
}
```

![出错](http://blog.fueson.top/img/2017/jsonp_err.jpg)

但`<img>`的src（获取图片），`<link>`的href（获取css），`<script>`的src（获取javascript）这三个都不符合同源策略，它们可以跨域获取数据。这里要介绍的JSONP就是利用`<script>`的src来实现跨域获取数据的。

## Jsonp

JSONP 是 JSON with padding（填充式 JSON 或参数式 JSON）的简写。
JSONP实现跨域请求的原理简单的说，就是动态创建`<script>`标签，然后利用`<script>`的 src 不受同源策略约束来跨域获取数据。
JSONP 由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数。回调函数的名字一般是在请求中指定的。而数据就是传入回调函数中的 JSON 数据。

```js
var script = document.createElement("script");
script.src = "https://api.douban.com/v2/book/search?q=javascript&count=1&callback=handleResponse";
document.body.insertBefore(script, document.body.firstChild);
// 对response数据进行操作代码
function handleResponse(response){}
```

JSONP目前还是比较流行的跨域方式，虽然JSONP使用起来方便，但是也存在一些问题： 
首先， JSONP 是从其他域中加载代码执行。如果其他域不安全，很可能会在响应中夹带一些恶意代码，而此时除了完全放弃 JSONP 调用之外，没有办法追究。因此在使用不是你自己运维的 Web 服务时，一定得保证它安全可靠。

其次，要确定 JSONP 请求是否失败并不容易。虽然 HTML5 给<script>元素新增了一个 onerror事件处理程序，但目前还没有得到任何浏览器支持。为此，开发人员不得不使用计时器检测指定时间内是否接收到了响应。

下面来看看 jsonp 的源码解读 

```js
// const debug = require('debug')('jsonp')
// 记录回调次数
let count = 0
// Noop循环函数
function noop() {}
/**
 * 实现jsonp的主函数，接受3个参数
 * @param {String} url 
 * @param {Object | Function} opts 
 * @param {Function} fn 
 */
function jsonp(url, opts, fn) {
  if (typeof opts === 'function') {
    fn = opts
    opts = {}
  }

  if (!opts) opts = {}

  const prefix = opts.prefix || '__jp'

  // 如果被调用就 使用被提供的回调名称
  // 否则通过增加计数器来生成一个唯一的名称
  let id = opts.name || (prefix + (count++))

  let param = opts.param || 'callback'
  let timeout = null != opts.timeout ? opts.timeout : 60000
  const enc = encodeURIComponent
  let target = document.getElementsByTagName('script')[0] || document.head
  let script
  let timer

  // 边界条件处理
  if (timeout) {
    timer = setTimeout(() => {
      cleanup()
      if (fn) fn(new Error('Timeout'))
    }, timeout)
  }

  function cleanup() {
    // 方法执行时，删除创建的script节点
    if (script.parantNode) script.parantNode.removeChild(script)
    // 且给window对象绑定属性id 为 noop （上面的空函数）
    window[id] = noop
    // 若timer不为空，则先清楚一次计时器
    if (timer) clearTimeout(timer)
  }

  function cancel() {
    // 执行时若window已绑定id则执行 cleanup 清除
    if (window[id]) {
      cleanup()
    }
  }

  /**
   * 为 window[id] 绑定一个匿名函数 
   * 接受data参数，执行回调fn时，将 data作为参数进行传递
   */
  window[id] = function(data) {
    // debug('jsonp got', data)
    cleanup()
    if (fn) fn(null, data)
  }

  // 修改现 url，增加 querystring
  url += `${~url.indexOf('?') ? '&' : '?'}${param}=${enc(id)}`
  // 这针对的是第一个kv对
  url = url.replace('?&', '?')

  // debug('jsonp req "%s"', url)

  // 创建script节点
  script = document.createElement('script')
  script.src = url
  target.parentNode.insertBefore(script, target)

  return cancel
}

module.exports = jsonp
```

## 参考资料

[开源jsonp实现 github](https://github.com/webmodules/jsonp/blob/master/index.js)