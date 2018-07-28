---
title: 【web】cookie、session和token
date: 2018-07-28 12:18:12
tags:
  - cookie
  - session
  - token
categories:
  - web
---

![](http://blog.fueson.top/18-7-28/63404154.jpg)

<!-- more -->

## Cookie定义

Cookie 是一小段文本信息，伴随着用户请求和页面在 Web 服务器和浏览器之间传递。Cookie 包含每次用户访问站点时 Web 应用程序都可以读取的信息。

例如，如果在用户请求站点中的页面时应用程序发送给该用户的不仅仅是一个页面，还有一个包含日期和时间的 Cookie，用户的浏览器在获得页面的同时还获得了该 Cookie，并将它存储在用户硬盘上的某个文件夹中。

以后，如果该用户再次请求您站点中的页面，当该用户输入 URL 时，浏览器便会在本地硬盘上查找与该 URL 关联的 Cookie。如果该 Cookie 存在，浏览器便将该 Cookie 与页请求一起发送到您的站点。然后，应用程序便可以确定该用户上次访问站点的日期和时间。您可以使用这些信息向用户显示一条消息，也可以检查到期日期。

Cookie 与网站关联，而不是与特定的页面关联。因此，无论用户请求站点中的哪一个页面，浏览器和服务器都将交换 Cookie 信息。用户访问不同站点时，各个站点都可能会向用户的浏览器发送一个 Cookie；浏览器会分别存储所有 Cookie。

Cookie 帮助网站存储有关访问者的信息。一般来说，Cookie 是一种保持 Web 应用程序连续性（即执行状态管理）的方法。除短暂的实际交换信息的时间外，浏览器和 Web 服务器间都是断开连接的。对于用户向 Web 服务器发出的每个请求，Web 服务器都会单独处理。但是在很多情况下，Web 服务器在用户请求页时识别出用户会十分有用。例如，购物站点上的 Web 服务器跟踪每位购物者，这样站点就可以管理购物车和其他的用户特定信息。因此，Cookie 可以作为一种名片，提供相关的标识信息帮助应用程序确定如何继续执行。

使用 Cookie 能够达到多种目的，所有这些目的都是为了帮助网站记住用户。例如，一个实施民意测验的站点可以简单地将 Cookie 作为一个 Boolean 值，用它来指示用户的浏览器是否已参与了投票，这样用户便无法进行第二次投票。要求用户登录的站点则可以通过 Cookie 来记录用户已经登录，这样用户就不必每次都输入凭据。

分类：一般分为两种形式的Cookie：1.会话型的，2.持久性的。会话型的是浏览器的处理过程中保留的，是暂时性的，当浏览器关闭时则消除了！而持久性的是保存在客户端的硬盘上的，就像论坛的Cookie一样。

### 优缺点

优点：极高的扩展性和可用性

通过良好的编程，控制保存在cookie中的session对象的大小。
通过加密和安全传输技术（SSL），减少cookie被破解的可能性。
只在cookie中存放不敏感数据，即使被盗也不会有重大损失。
控制cookie的生命期，使之不会永远有效。偷盗者很可能拿到一个过期的cookie。

缺点：

Cookie数量和长度的限制。每个domain最多只能有20条cookie，每个cookie长度不能超过4KB，否则会被截掉。
安全性问题。如果cookie被人拦截了，那人就可以取得所有的session信息。即使加密也与事无补，因为拦截者并不需要知道cookie的意义，他只要原样转发cookie就可以达到目的了。
有些状态不可能保存在客户端。例如，为了防止重复提交表单，我们需要在服务器端保存一个计数器。如果我们把这个计数器保存在客户端，那么它起不到任何作用。



## Session定义

在Web开发中，服务器可以为每个用户浏览器创建一个会话对象(session对象)。注意：一个浏览器独占一个session对象（默认）。因此，在需要保存用户数据时，服务器可以把用户数据写到用户浏览器独占的session中，当用户使用浏览器访问时，可以从用户的session中取出该用户的数据。

注：新开浏览器窗口会生成新的Session，子标签页除外。子标签页公用父窗口的Session

### Session用途

1. 记录用户登录与行为数据. 考虑到这些数据用户修改随意性大，没必要直接存到数据库
2. 用户执行刷新时，可直接根据session打开上次访问网页的状态，优化体验
3. 通过session把用户行为联系起来，构建出完整模型，进行数据挖掘

> session其实就是会话变量的保存地，只要是能使用变量的地方，都能使用session变量。一般地session就是像一个临时容器，来存放临时东西。


### Session和Cookie的区别和联系

> 具体来说cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务端保存状态的方案。两者存储的都是用户相关行为信息

* cookie是把用户的数据写在本地浏览器上，其他网站也可以扫描使用该cookie，容易泄漏自己网站的用户隐私，且一般浏览器对单个网站站点有cookie数量与大小限制

* session是把用户数据写在用户独占的session上，存储在服务端，一般只将session的id存储在cookie中。但将数据存储在服务器，成本相对高些

* session是由服务端创建，开发人员可以在服务器上通过request对象拿到

* 一般情况，登录信息等重要信息存储在session中，其他信息存储在cookie中

由于HTTP协议是无状态协议，所以服务端需要记录用户状态时，就需要用某种机制（Session）来识别具体用户。服务端为特定用户创建特定 Session 用于标识这个用户。Session是保存在服务端的，有一个唯一标识。

在服务端保存Session方法很多，内存、数据库、文件都有，集群时也要考虑Session的转移，使用一些缓存服务如Memcached之类来存放。

那么服务端如何识别特定用户？ Cookie，每次HTTP请求时，客户端都会发送相应的Cookie信息到服务端。实际上大多数的应用都是用Cookie来实现Session跟踪。第一次创建Session时，服务端会在HTTP协议中告诉客户端，需要在Cookie里记录一个Session ID，以后每次请求都把这个会话ID发送到服务器，这样服务端就能识别客户端了。

* Session是在服务端保存的一个数据结构，用来跟踪用户的状态，该数据可以保存在集群、数据库、文件中
* Cookie是在客户端保存用户信息的一种机制，用来记录用户信息，也是Session的一种实现方式

### Session的实现原理

服务器会为每一个访问服务器的用户创建一个session对象，且把session对象的id保存在本地cookie上，只要用户再次访问服务器时，带着session id，服务器就会匹配用户在服务器上的session。根据session中的数据，还原用户上次的浏览状态或提供其他人性化服务。

### 浏览器禁用Cookie后如何实现Session

**URL地址重写**

原理是将用户session的id信息重写到url地址中。服务器能够解析重写后的url以获取sessionId。这样即时客户端不支持Cookie，也可以使用Session来记录用户状态。

### Session和Cookie有效时长

* session

服务器会把长时间没有活动的session从服务器内存中清除，此时session便失效。具体根据服务端设置

* cookie

主要内容包括：Key、value、过期时间、路径和域。路径与域一起构成cookie的作用范围，通过过期时间expires设置cookie的有效时长

> 若不设置过期时间，表示这个cookie的生命周期为浏览器的会话期间，关闭访问服务器的浏览器窗口，cookie就消失，一般称为会话cookie。若保存在内存中设置了过期时间，则cookie会存储在硬盘上直到超过有效时间


## Token

前端 http 里所说的 Token  是指 `访问资源的凭据`。

例如当调用 google api 需要带上有效的token来表明请求的合法性。 这个token是google给的，代表了有权访问api背后的资源。

* access token 调用api时携带的token

1. 首先需要向google api注册应用程序，注册完毕后拿到认证信息（credentials）包括id和secret
2. 接下来向google请求access token。如果想访问用户资源，这里会提醒用户授权
3. 授权完毕，google会返回access token，或者授权代码（authorization code）, 再通过代码取得access token
4. token获取到后，就能带上token访问api了

![access token流程](https://user-gold-cdn.xitu.io/2018/7/3/1646088e5837a9a2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在第3步通过code兑换access token的过程中，google不仅会返回access token，还会返回额外信息，这其中和之后更新相关的就是refresh token

一旦access token过期，就可以通过refresh token再次请求access token。当然这要根据请求方式和访问的资源类型而定，这又会引起两个问题：

1. 如果refresh token也过期了怎么办？需要用户重新登录授权
2. 为什么要区分refresh token和access token？如果合并成一个token然后把过期时间调整更长，且每次失效后用户重新登录授权就好？

## OAuth

从获取token到使用token访问接口。这其实是标准的OAuth2.0机制下访问api的流程。

### SSO Single Sign-On

单点登录（公司内部，一个用户登录，可访问所有系统）

SSO是一类解决方案的统称，而在具体实施，我们有两种策略可供选择：

1. SAML 2.0
2. OAuth 2.0

**Authentication VS Authorisation**

* Authentication 身份鉴别 认证
* Authorisation 授权

认证的作用在于认可你有权限访问系统，用于鉴别访问者是否是合法用户；而授权用于决定你有访问哪些资源的权限。作为系统设计者来说，这两者的差别是不同的工作职责。

Authorization Server/Identity Provider(IdP) VS Service Provider(SP)/Resource Server
把负责认证的服务称为 Authorization Server 或者 Identity Provider，以下简称 IdP；而负责提供资源（API调用）的服务称为  Resource Server 或者 Service Provider，以下简称 SP

### SMAL 2.0

![SMAL2.0流程图](https://user-gold-cdn.xitu.io/2018/7/3/16460893ef7a34a4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 还未登录的用户打开浏览器访问网站（SP）网站提供服务但是不负责用户认证
* SP向IdP发送一个SAML认证请求，同时SP向用户浏览器重定向到IdP
* IdP在验证完来自SAML的请求无误后，在浏览器中呈现登录表单让用户进行填写用户名和密码进行登录
* 一旦用户登录成功，IdP会生成一个包含用户信息（用户名和密码）的SAML token（SAML token 又称为 SAML Assertion，本质上是 XML 节点）IdP向SP返回token，并且将用户重定向到SP（token的返回是在重定向步骤中实现的）
* SP对拿到的token进行验证，并且解析用户信息。此时就能够根据这些信息允许用户访问我们网站的内容了

当用户在IdP登录成功后，IdP需要将用户再次重定向至SP站点，这一步有两个方法：

* HTTP重定向（不推荐，因无法携带更长的信息）
* HTTP POST请求，当用户登录后渲染表单，点击向SP提交POST

如果是应用是基于web，无问题。但若是Android和IOS问题就来了：

* 用户在iphone上打开应用，需要通过IdP认证
* 应用跳转safari登录认证完毕后，需要通过http post形式将token返回至手机应用

虽然post的url可以拉起应用，但无法解析post内容，也就无法获取SAML token

---

### OAuth 2.0

![OAutho2.0流程图](https://user-gold-cdn.xitu.io/2018/7/3/164608b5c33898d0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 用户通过客户端（也可以是浏览器或手机应用）想要访问SP上的资源，但是SP告诉用户需要认证，将用户重定向至IdP
* IdP向用户询问SP是否可以访问用户信息，若用户同意，IdP向客户端返回access code
* 客户端拿code向IdP换access token，并拿着access token向SP请求资源
* SP接受请求后拿着附带token向IdP验证用户身份

OAuth本意是一个应用允许另一个应用在用户授权的情况下访问自己的数据，OAuth设计本意更倾向于授权而非认证（当然授权用户信息就间接实现了认证）

### OpenID

* OpenID只用于身份验证，允许你以同一个账户在多个网站登录。它仅仅是为你的合法身份背书，当你以xx帐号登录某个站点后，该站点无权访问你在xxb上的数据
* OAuth用户授权，允许被授权方访问授权方的用户数据

### Refresh Token

为什么需要？

这样处理是为了职责分离：refresh token负责身份验证， access token负责资源请求。虽然两者都由IdP发出，但access token还要和SP进行数据交换，如果公用会有身份泄漏可能。

---

token其实是为OAuth服务的，它是访问数据的一把钥匙。

## JWT

JWT 也是token，它是访问资源的凭证。甚至你可以不需要向 Google 索要 access token，而是携带 JWT 作为 HTTP header 里的 bearer token 直接访问 API 也是可以的。

顾名思义，它是json结构的token，由三部分组成：

* header

用于描述元信息，例如产生signature的算法

```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

* payload

用于携带你希望向服务端传递的信息。即可以往里面添加字段，也可以塞入自定义字段

```json
{
  "userId": "b08f86af-35da-48f2-8fab-cef3904660bd"
}
```

* signature

创建签名需要分以下几个步骤

1. 需要从接口服务端拿到密钥 假设为 `secret`
2. 将header进行base64编码，假设为 `headerStr`
3. 将payload进行base64编码，假设为 `payloadStr`
4. 将headerStr和payloadStr用 `.` 字符串拼接，成为字符 `data`
5. 以data和secret作为参数，使用哈希算法计算出签名

下面是伪代码：

```js
// signature algorithm
data = base64URLEncoded(header) + '.' + base64URLEncoded(payload)
signature = Hash(data, secret)
```

假设我们的原始 JSON 结构是这样的：

```json
// Header
{
  "typ": "JWT",
  "alg": "HS256"
}
// Payload:
{
  "userId": "b08f86af-35da-48f2-8fab-cef3904660bd"
}
```

如果密钥是字符串secret的话，那么最终 JWT 的结果就是这样的

    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiJiMDhmODZhZi0zNWRhLTQ4ZjItOGZhYi1jZWYzOTA0NjYwYmQifQ.-xN_h82PHVTCMA9vdoHrcZxH-x5mb11y1537t3rGzcM

你可以在 [jwt.io](https://jwt.io/) 上验证这个结果

---

JWT的目的不是为了隐藏或者保密数据，而是为了确保数据确实来自被授权的人创建的（不被篡改）

用于接口调用

## 有状态的会话

因为HTTP是无状态的，所以客户端和服务端需要解决如何让之间的对话变得有状态。例如只有登录状态的用户才有权限去调用某些接口，那么在用户登录后，需要记住该用户是已经登录的状态。常见的方法是使用session机制。

![常见的session模型](https://user-gold-cdn.xitu.io/2018/7/3/164608d56ba8fc6e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 用户在浏览器登录后，服务端为用户生成唯一的session id，存储在服务端的存储服务（mysql redis）
* 该session id也返回给浏览器以SESSION_ID为key存储在cookie中
* 如果用户再次访问该站，cookie里的SESSION_ID会随着请求一同发往服务端
* 服务端通过判断SESSION_ID是否在redis判断用户是否处于登录状态
