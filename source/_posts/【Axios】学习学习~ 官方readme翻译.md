---
title: 【Axios】学习学习~ 官方readme翻译
date: 2017-07-18 09:55:04
tags: 
  - axios
categories: 
  - 工具
---

# Axios

[Axios的github](https://github.com/mzabriskie/axios) 看这里

<!-- more -->

 <p><a href="https://www.npmjs.org/package/axios"><img src="https://camo.githubusercontent.com/9f600e10007ac86da6a8b90c16ca1e9504901730/68747470733a2f2f696d672e736869656c64732e696f2f6e706d2f762f6178696f732e7376673f7374796c653d666c61742d737175617265" alt="npm version" data-canonical-src="https://img.shields.io/npm/v/axios.svg?style=flat-square" style="max-width:100%;"></a>
<a href="https://travis-ci.org/mzabriskie/axios"><img src="https://camo.githubusercontent.com/5f36e9ec9e05d8160d54f9e2312a1e3059d5e252/68747470733a2f2f696d672e736869656c64732e696f2f7472617669732f6d7a61627269736b69652f6178696f732e7376673f7374796c653d666c61742d737175617265" alt="build status" data-canonical-src="https://img.shields.io/travis/mzabriskie/axios.svg?style=flat-square" style="max-width:100%;"></a>
<a href="https://coveralls.io/r/mzabriskie/axios"><img src="https://camo.githubusercontent.com/04dcbd38251ebdea54b541442efb9dd48cfb297b/68747470733a2f2f696d672e736869656c64732e696f2f636f766572616c6c732f6d7a61627269736b69652f6178696f732e7376673f7374796c653d666c61742d737175617265" alt="code coverage" data-canonical-src="https://img.shields.io/coveralls/mzabriskie/axios.svg?style=flat-square" style="max-width:100%;"></a>
<a href="http://npm-stat.com/charts.html?package=axios"><img src="https://camo.githubusercontent.com/0466334190cd8c680cfa27ff0ace874fae1157e4/68747470733a2f2f696d672e736869656c64732e696f2f6e706d2f646d2f6178696f732e7376673f7374796c653d666c61742d737175617265" alt="npm downloads" data-canonical-src="https://img.shields.io/npm/dm/axios.svg?style=flat-square" style="max-width:100%;"></a>
<a href="https://gitter.im/mzabriskie/axios"><img src="https://camo.githubusercontent.com/013650d23d9eb743d6169b87e2b1b9741709f699/68747470733a2f2f696d672e736869656c64732e696f2f6769747465722f726f6f6d2f6d7a61627269736b69652f6178696f732e7376673f7374796c653d666c61742d737175617265" alt="gitter chat" data-canonical-src="https://img.shields.io/gitter/room/mzabriskie/axios.svg?style=flat-square" style="max-width:100%;"></a></p> 

（真是受不了我蹩脚的英语了- -， 你们还是直接看官方文档吧，让我自high）

> 一个支持浏览器和Node.js的基于Promise的HTTP客户端


## 特点

* 从浏览器发出XMLHttpRequest
* 从Node.js发出HTTP请求
* 支持Promise API
* 拦截请求和响应
* 转换请求和响应数据
* 取消请求
* 自动转换为JSON数据
* 跨站请求伪造的客户端防护

## 浏览器支持一览

 <table>
<thead>
<tr>
<th><a href="https://camo.githubusercontent.com/26846e979600799e9f4273d38bd9e5cb7bb8d6d0/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f6368726f6d652f6368726f6d655f34387834382e706e67" target="_blank"><img src="https://camo.githubusercontent.com/26846e979600799e9f4273d38bd9e5cb7bb8d6d0/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f6368726f6d652f6368726f6d655f34387834382e706e67" alt="Chrome" data-canonical-src="https://raw.github.com/alrra/browser-logos/master/src/chrome/chrome_48x48.png" style="max-width:100%;"></a></th>
<th><a href="https://camo.githubusercontent.com/6087557f69ec6585eb7f8d7bd7d9ecb6b7f51ba1/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f66697265666f782f66697265666f785f34387834382e706e67" target="_blank"><img src="https://camo.githubusercontent.com/6087557f69ec6585eb7f8d7bd7d9ecb6b7f51ba1/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f66697265666f782f66697265666f785f34387834382e706e67" alt="Firefox" data-canonical-src="https://raw.github.com/alrra/browser-logos/master/src/firefox/firefox_48x48.png" style="max-width:100%;"></a></th>
<th><a href="https://camo.githubusercontent.com/6fbaeb334b99e74ddd89190a42766ea3b4600d2c/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f7361666172692f7361666172695f34387834382e706e67" target="_blank"><img src="https://camo.githubusercontent.com/6fbaeb334b99e74ddd89190a42766ea3b4600d2c/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f7361666172692f7361666172695f34387834382e706e67" alt="Safari" data-canonical-src="https://raw.github.com/alrra/browser-logos/master/src/safari/safari_48x48.png" style="max-width:100%;"></a></th>
<th><a href="https://camo.githubusercontent.com/96d2405a936da1fb8988db0c1d304d3db04b8a52/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f6f706572612f6f706572615f34387834382e706e67" target="_blank"><img src="https://camo.githubusercontent.com/96d2405a936da1fb8988db0c1d304d3db04b8a52/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f6f706572612f6f706572615f34387834382e706e67" alt="Opera" data-canonical-src="https://raw.github.com/alrra/browser-logos/master/src/opera/opera_48x48.png" style="max-width:100%;"></a></th>
<th><a href="https://camo.githubusercontent.com/826b3030243b09465bf14cf420704344f5eee991/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f656467652f656467655f34387834382e706e67" target="_blank"><img src="https://camo.githubusercontent.com/826b3030243b09465bf14cf420704344f5eee991/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f656467652f656467655f34387834382e706e67" alt="Edge" data-canonical-src="https://raw.github.com/alrra/browser-logos/master/src/edge/edge_48x48.png" style="max-width:100%;"></a></th>
<th><a href="https://camo.githubusercontent.com/4b062fb12353b0ef8420a72ddc3debf6b2ee5747/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f617263686976652f696e7465726e65742d6578706c6f7265725f392d31312f696e7465726e65742d6578706c6f7265725f392d31315f34387834382e706e67" target="_blank"><img src="https://camo.githubusercontent.com/4b062fb12353b0ef8420a72ddc3debf6b2ee5747/68747470733a2f2f7261772e6769746875622e636f6d2f616c7272612f62726f777365722d6c6f676f732f6d61737465722f7372632f617263686976652f696e7465726e65742d6578706c6f7265725f392d31312f696e7465726e65742d6578706c6f7265725f392d31315f34387834382e706e67" alt="IE" data-canonical-src="https://raw.github.com/alrra/browser-logos/master/src/archive/internet-explorer_9-11/internet-explorer_9-11_48x48.png" style="max-width:100%;"></a></th>
</tr>
</thead>
<tbody>
<tr>
<td>Latest ✔</td>
<td>Latest ✔</td>
<td>Latest ✔</td>
<td>Latest ✔</td>
<td>Latest ✔</td>
<td>8+ ✔</td>
</tr></tbody></table>

<p><a href="https://saucelabs.com/u/axios"><img src="https://camo.githubusercontent.com/626c46cfd86214001b4143cda5d0ef27a25bd69f/68747470733a2f2f73617563656c6162732e636f6d2f6f70656e5f73617563652f6275696c645f6d61747269782f6178696f732e737667" alt="Browser Matrix" data-canonical-src="https://saucelabs.com/open_sauce/build_matrix/axios.svg" style="max-width:100%;"></a></p> 

## 安装

使用npm:

`$ npm install axios`

使用bower:

`$ bower install axios`

使用cdn:

`<script src="https://unpkg.com/axios/dist/axios.min.js"></script>`

## 例子

**执行一个get请求**

```js
// 发出请求给用户一个ID
axios.get('/user?ID=12345')
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });

// 上面的请求也可以用下面的方式
axios.get('/user', {
  params: {
    ID: 12345
  }
}).then(response => {
  console.log(response);
}).catch(error => {
  console.log(error);
});
```

**执行POST请求**

```js
axios.post('/user', {
  firstName: 'fys',
  lastName: 'fridolph'
}).then(response => {
  console.log(response);
}).catch(error => {
  console.log(error);
});
```

**执行多个并发请求**

```js
function getUserAccount() {
  return axios.get('/user/12345');
}
 　
function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}
 　
axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(acct, perms) => {
    // 当两个请求都执行完成后
  })
```

## axios API

请求可以通过相关配置给axios

**axios(config)**

```js
// 发送一个POST请求
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'fys',
    lastName: 'fridolph'
  }
})
```

```js
// GET请求一个远程图像
axios({
  method: 'get',
  url: 'http://bit.ly/2mTM3nY',
  resonseType: 'stream'
}).then(response => {
  response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
});
```

**axios(url[, config])**

```js
// Send a GET request (default method)
axios('/user/12345')
```

### 请求方法别名

为了方便别名为所有支持的请求提供了方法
译者注：类似于jQuery.ajax封装提供的别名，$.get $.post等等

* axios.request(config)
* axios.get(url[, config])
* axios.delete(url[, config])
* axios.head(url[, config])
* axios.options(url[, config])
* axios.post(url[, data[, config]])
* axios.put(url[, data[, config]])
* axios.patch(url[, data[, config]])

`注：当使用url别名方法,方法,和数据属性中指定不需要配置`


### 同时请求（并发性）

以下处理函数用于处理并发请求

* axios.all(iterable)
* axios.spread(callback)

### 创建一个实例

你可以创建一个新的axios实例用于个性化配置

**axios.create([config])**

```js
var instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

### 实例方法

下面列出了可用的实例方法。指定的配置将合并实例配置。

* axios#request(config)
* axios#get(url[, config])
* axios#delete(url[, config])
* axios#head(url[, config])
* axios#options(url[, config])
* axios#post(url[, data[, config]])
* axios#put(url[, data[, config]])
* axios#patch(url[, data[, config]])

## Requese Config 请求配置

下面这些可用的配置选项进行请求，只需要url，若没有指定方法则请求违约。

ps 写到一半发现朋友发了篇… [axios中文文档翻译](https://segmentfault.com/a/1190000008470355?utm_source=tuicool&utm_medium=referral)过来。。额，我就偷下懒了

```js
{
  // `url`用于请求服务端的url
  url: '/user',

  // `method` 是用来处理请求的请求方法, 默认为get请求
  method: 'get', 

  // `baseUrl`将被处理为url，除非是绝对路径
  // axios通过相对url关联到实例，这便于设置 `baseURL`
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest`允许请求的数组传递给服务器，在这之前进行(转化)处理
  // 不过这是只适用于请求方法 'PUT' 'POST' 'PATCH'
  // 数组中最后一个函数必须返回一个字符串、Buffer实例、ArrayBuffer、FormData或Stream
  transformRequest: [function (data) {
    // 处理任何你想要转换的数据
    return data;    
  }],

  // `transformResponse`允许响应的数据传递给 then/catch，在这之前进行处理
  transformResponse: [function (data) {
    // 处理任何你想要转换的数据
    return data;
  }],

  //`headers`是自定义的要被发送的头信息
    headers:{'X-Requested-with':'XMLHttpRequest'},

    //`params`是请求连接中的请求参数，必须是一个纯对象，或者URLSearchParams对象
    params:{
      ID:12345
    },
    
    //`paramsSerializer`是一个可选的函数，是用来序列化参数
    //例如：（https://ww.npmjs.com/package/qs,http://api.jquery.com/jquery.param/)
    paramsSerializer: function(params){
      return Qs.stringify(params,{arrayFormat:'brackets'})
    },

    //`data`是请求提需要设置的数据
    //只适用于应用的'PUT','POST','PATCH'，请求方法
    //当没有设置`transformRequest`时，必须是以下其中之一的类型（不可重复？）：
    //-string,plain object,ArrayBuffer,ArrayBufferView,URLSearchParams
    //-仅浏览器：FormData,File,Blob
    //-仅Node：Stream
    data:{
      firstName:'fred'
    },
    //`timeout`定义请求的时间，单位是毫秒。
    //如果请求的时间超过这个设定时间，请求将会停止。
    timeout:1000,
    
    //`withCredentials`表明是否跨网站访问协议，
    //应该使用证书
    withCredentials:false //默认值

    //`adapter`适配器，允许自定义处理请求，这会使测试更简单。
    //返回一个promise，并且提供验证返回（查看[response docs](#response-api)）
    adapter:function(config){
      /*...*/
    },

    //`auth`表明HTTP基础的认证应该被使用，并且提供证书。
    //这个会设置一个`authorization` 头（header），并且覆盖你在header设置的Authorization头信息。
    auth:{
      username:'janedoe',
      password:'s00pers3cret'
    },

    //`responsetype`表明服务器返回的数据类型，这些类型的设置应该是
    //'arraybuffer','blob','document','json','text',stream'
    responsetype:'json',

    //`xsrfHeaderName` 是http头（header）的名字，并且该头携带xsrf的值
    xrsfHeadername:'X-XSRF-TOKEN'，//默认值

    //`onUploadProgress`允许处理上传过程的事件
    onUploadProgress: function(progressEvent){
        //本地过程事件发生时想做的事
    },

    //`onDownloadProgress`允许处理下载过程的事件
    onDownloadProgress: function(progressEvent){
        //下载过程中想做的事
    },

    //`maxContentLength` 定义http返回内容的最大容量
    maxContentLength: 2000,

    //`validateStatus` 定义promise的resolve和reject。
    //http返回状态码，如果`validateStatus`返回true（或者设置成null/undefined），promise将会接受；其他的promise将会拒绝。
    validateStatus: function(status){
        return status >= 200 && stauts < 300;//默认
    },

    //`httpAgent` 和 `httpsAgent`当产生一个http或者https请求时分别定义一个自定义的代理，在nodejs中。
    //这个允许设置一些选选个，像是`keepAlive`--这个在默认中是没有开启的。
    httpAgent: new http.Agent({keepAlive:treu}),
    httpsAgent: new https.Agent({keepAlive:true}),

    //`proxy`定义服务器的主机名字和端口号。
    //`auth`表明HTTP基本认证应该跟`proxy`相连接，并且提供证书。
    //这个将设置一个'Proxy-Authorization'头(header)，覆盖原先自定义的。
    proxy:{
        host:127.0.0.1,
        port:9000,
        auth:{
            username:'cdd',
            password:'123456'
        }
    },

    //`cancelTaken` 定义一个取消，能够用来取消请求
    //（查看 下面的Cancellation 的详细部分）
    cancelToken: new CancelToken(function(cancel){
    })
}
```

## 返回响应概要 Response Schema

**一个请求的返回包含以下信息**

```js
{
  //`data`是服务器的提供的回复（相对于请求）
  data{},

  //`status`是服务器返回的http状态码
  status:200,


  //`statusText`是服务器返回的http状态信息
  statusText: 'ok',

  //`headers`是服务器返回中携带的headers
  headers:{},

  //`config`是对axios进行的设置，目的是为了请求（request）
  config:{}
}
```

使用then，你会接受打下面的信息

```js
axios.get('/user/12345')
  .then(function(response){
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
  });
```

使用catch时，或者传入一个reject callback作为then的第二个参数，那么返回的错误信息将能够被使用。

## 默认设置(Config Default)

你可以设置一个默认的设置，这设置将在所有的请求中有效。

### 全局默认设置 Global axios defaults

```js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type']='application/x-www-form-urlencoded';
```

### 实例中自定义默认值 Custom instance default

```js
//当创建一个实例时进行默认设置
var instance = axios.create({
    baseURL:'https://api.example.com'
});
//在实例创建之后改变默认值
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

## 设置优先级 Config order of precedence

设置(config)将按照优先顺序整合起来。首先的是在lib/defaults.js中定义的默认设置，其次是defaults实例属性的设置，最后是请求中config参数的设置。越往后面的等级越高，会覆盖前面的设置。看下面这个例子：

```js
//使用默认库的设置创建一个实例，
//这个实例中，使用的是默认库的timeout设置，默认值是0。
var instance = axios.create();
//覆盖默认库中timeout的默认值
//此时，所有的请求的timeout时间是2.5秒
instance.defaults.timeout = 2500;
//覆盖该次请求中timeout的值，这个值设置的时间更长一些
instance.get('/longRequest',{
    timeout:5000
});
```

## 拦截器 interceptors

你可以在请求或者返回被then或者catch处理之前对他们进行拦截。

```js
//添加一个请求拦截器
axios.interceptors.request.use(function(config){
    //在请求发送之前做一些事
    return config;
},function(error){
    //当出现请求错误是做一些事
    return Promise.reject(error);
});

//添加一个返回拦截器
axios.interceptors.response.use(function(response){
    //对返回的数据进行一些处理
    return response;
},function(error){
    //对返回的错误进行一些处理
    return Promise.reject(error);
});
```

如果你需要在稍后移除拦截器,你可以

    var myInterceptor = axios.interceptors.request.use(function(){/*...*/});
    axios.interceptors.rquest.eject(myInterceptor);

你可以在一个axios实例中使用拦截器

    var instance = axios.create();
    instance.interceptors.request.use(function(){/*...*/});

## 错误处理 Handling Errors

```js
axios.get('user/12345')
  .catch(function(error){
    if(error.response){
      //存在请求，但是服务器的返回一个状态码
      //他们都在2xx之外
      console.log(error.response.data);
      console.log(error.response.status);
      console.log(error.response.headers);
    }else{
      //一些错误是在设置请求时触发的
      console.log('Error',error.message);
    }
    console.log(error.config);
  });
```

**你可以使用validateStatus设置选项自定义HTTP状态码的错误范围**

```js
axios.get('user/12345',{
  validateStatus:function(status){
    return status < 500;//当返回码小于等于500时视为错误
  }
});
```

## 取消 Cancellation

你可以使用cancel token取消一个请求

axios的cancel token API是基于**cnacelable promises proposal**，其目前处于第一阶段。
你可以使用CancelToken.source工厂函数创建一个cancel token，如下：

```js
var CancelToken = axios.CancelToken;
var source = CancelToken.source();
axios.get('/user/12345', {
    cancelToken:source.toke
}).catch(function(thrown){
  if(axiso.isCancel(thrown)){
    console.log('Rquest canceled', thrown.message);
  }else{
    //handle error
  }
});
//取消请求(信息参数设可设置的)
source.cancel("操作被用户取消");
```

你可以给CancelToken构造函数传递一个executor function来创建一个cancel token:

```js
var CancelToken = axios.CancelToken;
var cancel;

axios.get('/user/12345', {
    cancelToken: new CancelToken(function executor(c){
        //这个executor 函数接受一个cancel function作为参数
        cancel = c;
    })
});
//取消请求
cancel();
```

注意：你可以使用同一个cancel token取消多个请求。

## 使用 application/x-www-form-urlencoded 格式化

默认情况下，axios串联js对象为JSON格式。为了发送application/x-wwww-form-urlencoded格式数据，你可以使用一下的设置。

### 浏览器 Browser

在浏览器中你可以如下使用URLSearchParams API:

```js
var params = new URLSearchParams();
params.append('param1','value1');
params.append('param2','value2');
axios.post('/foo',params);
```

注意：URLSearchParams不支持所有的浏览器，但是这里有个垫片
（poly fill）可用（确保垫片在浏览器全局环境中）

其他方法：你可以使用qs库来格式化数据。

    var qs = require('qs');
    axios.post('/foo', qs.stringify({'bar':123}));

### Node.js

在nodejs中，你可以如下使用querystring:

    var querystring = require('querystring');
    axios.post('http://something.com/', querystring.stringify({foo:'bar'}));

你同样可以使用qs库。

### promises

axios 基于原生的ES6 Promise 实现。如果环境不支持请使用垫片.

## TypeScript

axios 包含TypeScript定义

    import axios from 'axios'
    axios.get('/user?ID=12345')

---

本文完~