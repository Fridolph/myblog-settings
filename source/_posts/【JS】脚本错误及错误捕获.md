---
title: 【JS】脚本错误及错误捕获
date: 2018-07-14 20:52:15
tags: 
  - JS
  - Error
categories: 
  - JS
---

> 从脚本错误开始，了解脚本为什么会报错以及抛出错误，最后介绍了用window.onerror、try catch捕获错误。

<!-- more -->

## 什么是脚本错误

如果您以前使用过JavaScript onerror事件做过任何工作，您可能会遇到以下情况：

"Script error."

“脚本错误”是当错误源自从不同来源（不同域，端口或协议）提供的JavaScript文件时，浏览器发送到onerror回调的内容。这很痛苦，因为即使出现错误，您也不知道错误是什么，也不知道它来自哪个代码。这就是window.onerror的全部目的 - 深入了解应用程序中未捕获的错误。

## 产生原因：跨源脚本

为了更好地了解正在发生的事情，请考虑以下示例HTML文档，假设从http://example.com/test提供：

```html
<!doctype html>
<html>
<head>
  <title>example.com/test</title>
</head>
<body>
  <script src="http://another-domain.com/app.js"></script>
  <script>
    window.onerror = function (message, url, line, column, error) {
      console.log(message, url, line, column, error);
    }
    foo(); // call function declared in app.js
  </script>
</body>
</html>
```

这是http://another-domain.com/app.js的内容。它声明了一个函数foo，其调用将始终抛出ReferenceError。

在浏览器中加载此文档并执行JavaScript时，会将以下内容输出到控制台（通过window.onerror回调记录）：

"Script error.", "", 0, 0, undefined

这不是JavaScript错误 - 出于安全原因，浏览器故意隐藏源自不同来源的脚本文件的错误。这是为了避免脚本无意中将潜在敏感信息泄露给它无法控制的onerror回调。出于这个原因，浏览器只允许window.onerror洞察来自同一域的错误。我们所知道的是发生了一个错误 - 没有别的！

## 但这并不坏

尽管浏览器有良好的意图，但是有一些很好的理由可以让您深入了解从不同来源提供的脚本引发的错误：

1. 您的应用程序JavaScript文件是从不同的主机名提供的，例如static.sentry.io/app.js
2. 您正在使用社区CDN提供的库，例如cdnjs或Google的托管库
3. 您正在使用商业第三方JavaScript库，该库仅由外部服务器提供

但不要担心 - 深入了解这些文件所提供的JavaScript错误只需要进行一些简单的调整。

## 修复：CORS attributes & headers

为了了解源自不同来源的脚本引发的JavaScript异常，您必须做两件事:

1. 添加 crossorigin="anonymous" 脚本属性

`<script src="http://another-domain.com/app.js" crossorigin="anonymous"></script>`

这告诉浏览器应该“anonymous”(匿名)获取目标文件。这意味着在请求此文件时，浏览器不会将任何潜在的用户识别信息（如cookie或HTTP凭据）传输到服务器。

2. 添加Cross Origin HTTP header

`Access-Control-Allow-Origin: *`

CORS是“跨源资源共享”的缩写，它是一组API（主要是HTTP标头），它们规定了文件应该如何从源头下载和提供。

通过设置Access-Control-Allow-Origin：*，服务器向浏览器指示任何源可以获取此文件。或者，您可以将其限制为您控制的已知来源，例如

`Access-Control-Allow-Origin: https://www.example.com`

注意：大多数社区CDN正确设置了Access-Control-Allow-Origin标头。

```bash
$ curl --head https://ajax.googleapis.com/ajax/libs/jquery/2.2.0/jquery.js | \
  grep -i "access-control-allow-origin"

Access-Control-Allow-Origin: *
```

完成这两个步骤后，此脚本触发的任何错误都将向window.onerror报告，就像任何常规的同域脚本一样。因此，代替“脚本错误”，从一开始的onerror示例将产生：

`"ReferenceError: bar is not defined", "http://another-domain.com/app.js", 2, 1, [Object Error]`

## 另一种解决方案：try / catch

有时，我们并不总是能够调整Web应用程序正在使用的脚本的HTTP头。在这些情况下，有一种替代方法：使用try / catch。

再次考虑原始示例，这次使用try / catch：

```html
<!-- note: crossorigin="anonymous" intentionally absent -->
<script src="http://another-domain.com/app.js"></script>
<script>
  window.onerror = function (message, url, line, column, error) {
    console.log(message, url, line, column, error);
  }

  try {
    foo(); // call function declared in app.js
  } catch (e) {
    console.log(e);
    throw e; // intentionally re-throw (caught by window.onerror)
  }
</script>
```

对于后代，some-domain.com/app.js再次看起来像这样：

```js
// another-domain.com/app.js
function foo() {
  bar(); // ReferenceError: bar is not a function
}
```

运行示例HTML将向控制台输出以下2个条目：

```bash
=> ReferenceError: bar is not defined
    at foo (http://another-domain.com/b.js:2:3)
    at http://example.com/test/:15:3

=> "Script error.", "", 0, 0, undefined
```

第一个控制台语句 - 来自try / catch - 设法获取一个错误对象，包括类型，消息和堆栈跟踪，包括文件名和行号。来自window.onerror的第二个控制台语句再一次只能输出“脚本错误”。

现在，这是否意味着您需要尝试/捕获所有代码？可能不是 - 如果您可以轻松更改HTML并在CDN上指定CORS标题，则最好这样做并坚持使用window.onerror。但是，如果您不控制这些资源，使用try / catch包装第三方代码是一种可靠的（虽然是乏味的）方式，可以深入了解跨源脚本引发的错误。

注意：默认情况下，Raven.js - Sentry的JavaScript SDK - 仔细设备内置方法，尝试自动将代码包装在try / catch块中。它这样做是为了尝试捕获所有脚本中的错误消息和堆栈跟踪，无论它们来自哪个来源。如果可能，仍建议设置CORS属性和标头。

## window.onerror

onerror是一个特殊的浏览器事件，只要抛出未捕获的JavaScript错误就会触发该事件。这是记录客户端错误并将其报告给服务器的最简单方法之一。它也是Sentry的客户端JavaScript集成（raven-js）工作的主要机制之一。

你通过向window.onerror分配一个函数来监听onerror事件：

```js
window.onerror = function (msg, url, lineNo, columnNo, error) {
  // ... handle error ...
  return false;
}
```

抛出错误时，以下参数将传递给函数：

* msg - 与错误相关的消息，例如“未捕获的ReferenceError：foo未定义”
* url - 与错误关联的脚本或文档的URL，例如“/dist/app.js”
* lineNo - 行号（如果有）
* columnNo - 列号（如果可用）
* error - 与此错误关联的Error对象（如果可用）

前四个参数告诉你错误发生在哪个脚本，行和列中。最后一个参数Error对象可能是最有价值的。让我们来了解原因。

### Error对象和error.stack

乍一看，Error对象不是很特别。它包含3个标准化属性：message，fileName和lineNumber。已通过window.onerror提供给你的冗余值。

有价值的部分是非标准属性：Error.prototype.stack。此堆栈属性告诉你错误发生时程序的每个帧的源位置。错误堆栈跟踪可能是调试的关键部分。尽管不标准，但每个现代浏览器都提供此属性。

以下是Chrome 46中Error对象的堆栈属性的示例：

```js
"Error: foobar\n    at new bar (<anonymous>:241:11)\n    at foo (<anonymous>:245:5)\n    at <anonymous>:250:5\n    at <anonymous>:251:3\n    at <anonymous>:267:4\n    at callFunction (<anonymous>:229:33)\n    at <anonymous>:239:23\n    at <anonymous>:240:3\n    at Object.InjectedScript._evaluateOn (<anonymous>:875:140)\n    at Object.InjectedScript._evaluateAndWrap (<anonymous>:808:34)"
```

难以阅读，对吗？stack属性实际上只是一个未格式化的字符串。这是格式化的样子：

```js
"Error: foobar
    at new bar (<anonymous>:241:11)
    at foo (<anonymous>:245:5)
    at <anonymous>:250:5
    at <anonymous>:251:3
    at <anonymous>:267:4
    at callFunction (<anonymous>:229:33)
    at <anonymous>:239:23
    at <anonymous>:240:3
    at Object.InjectedScript._evaluateOn (<anonymous>:875:140)
    at Object.InjectedScript._evaluateAndWrap (<anonymous>:808:34)"
```

一旦格式化，就很容易看出堆栈属性如何在帮助调试错误时起到关键作用。只有一个障碍：堆栈属性是非标准的，它的实现在浏览器之间有所不同。例如，这是来自Internet Explorer 11的相同堆栈跟踪：

```js
Error: foobar
   at bar (Unknown script code:2:5)
   at foo (Unknown script code:6:5)
   at Anonymous function (Unknown script code:11:5)
   at Anonymous function (Unknown script code:10:2)
   at Anonymous function (Unknown script code:1:73)
```

不仅每个帧的格式不同，帧也具有较少的细节。例如，Chrome会识别出已使用新关键字，并且对eval调用有更深入的了解。这只是IE 11与Chrome的比较 - 其他类似的浏览器具有不同的格式和细节。

幸运的是，有一些工具可以规范化堆栈属性，使其在浏览器中保持一致。例如，raven-js使用TraceKit来规范化错误字符串。还有stacktrace.js和其他一些项目。

### 浏览器兼容性

window.onerror已经在浏览器中出现了一段时间 - 你会在浏览器中找到它与IE6和Firefox 2一样古老的版本。

问题是每个浏览器都以不同的方式实现window.onerror。特别是，将多少个参数发送给onerror侦听器，以及这些参数的结构。

这是一个在大多数浏览器中将参数传递给onerror的表：

|Browser|Message|URL|lineNo|colNo|errorObj|
|:-----:|:-----:|:-:|:----:|:---:|:------:|
|Firefox|✓|✓|✓|✓|✓|
|Chrome|✓|✓|✓|✓|✓|
|Edge|✓|✓|✓|✓|✓|
|IE 11|✓|✓|✓|✓|✓|
|IE 10|✓|✓|✓|✓|
|IE 9, 8|✓|✓|✓|
|Safari 10 and up|✓|✓|✓|✓|✓|
|Safari 9|✓|✓|✓|✓|
|Android Browser 4.4|✓|✓|✓|✓|

Internet Explorer 8,9和10对onerror的支持有限，这可能不足为奇。但是你可能会惊讶于Safari只在Safari 10中添加了对错误对象的支持（2016年发布）。此外，仍旧使用现有Android浏览器（现已替换为Chrome Mobile）的旧手机版仍然在那里，并且不会传递错误对象。

没有错误对象，就没有堆栈跟踪属性。这意味着这些浏览器无法从错误捕获的错误中检索有价值的堆栈信息。

### 使用try / catch 兼容 window.onerror

但是有一种解决方法 - 你可以将应用程序中的代码包装在try / catch中并自己捕获错误。这个错误对象将在每个现代浏览器中包含我们令人垂涎的堆栈属性。

考虑以下帮助器方法invoke，它使用参数数组调用对象上的函数：

```js
function invoke(obj, method, args) {
    return obj[method].apply(this, args);
}

invoke(Math, 'max', [1, 2]); // returns 2
```

这里再次调用，这次包装在try / catch中，以捕获任何抛出的错误：

```js
function invoke(obj, method, args) {
  try {
    return obj[method].apply(this, args);
  } catch (e) {
    captureError(e); // report the error
    throw e; // re-throw the error
  }
}

invoke(Math, 'highest', [1, 2]); // throws error, no method Math.highest
```

当然，在任何地方手动执行此操作非常麻烦。你可以通过创建通用包装器实用程序函数来使其更容易：

```js
function wrapErrors(fn) {
  // don't wrap function more than once
  if (!fn.__wrapped__) {
    fn.__wrapped__ = function () {
      try {
        return fn.apply(this, arguments);
      } catch (e) {
        captureError(e); // report the error
        throw e; // re-throw the error
      }
    };
  }

  return fn.__wrapped__;
}

var invoke = wrapErrors(function(obj, method, args) {
  return obj[method].apply(this, args);
});

invoke(Math, 'highest', [1, 2]); // no method Math.highest
```

因为JavaScript是单线程的，所以你不需要在任何地方使用wrap  - 只是在每个新堆栈的开头。

这意味着你需要包装函数声明：

* 在你的应用程序开始时（例如，如果你使用jQuery，请在$（document）.ready中）
* 在事件处理程序中，例如addEventListener或$ .fn.click
* 基于计时器的回调，例如setTimeout或requestAnimationFrame

示例：

```js
$(wrapErrors(function () { // application start
  doSynchronousStuff1(); // doesn't need to be wrapped

  setTimeout(wrapErrors(function () {
    doSynchronousStuff2(); // doesn't need to be wrapped
  });

  $('.foo').click(wrapErrors(function () {
    doSynchronousStuff3(); // doesn't need to be wrapped
  });
}));
```

如果这看起来像是很多工作，请不要担心！大多数错误报告库都具有增强内置函数（如addEventListener和setTimeout）的机制，因此你不必每次都调用包装实用程序。是的，raven-js也这样做。

### 将错误传输到你的服务器

好的，所以你已经完成了你的工作 - 你已经插入window.onerror，并且你还在try / catch中包装函数，以便捕获尽可能多的错误信息。

最后一步是：将错误信息传输到你的服务器。为了实现这一点，你需要设置某种报告Web服务，该服务将通过HTTP接受你的错误数据，将其记录到文件和/或将其存储在数据库中。

如果此Web服务与Web应用程序位于同一个域中，则可以使用XMLHttpRequest轻松实现。在下面的示例中，我们使用jQuery的AJAX函数将数据传输到我们的服务器：

```js
function captureError(ex) {
  var errorData = {
    name: ex.name, // e.g. ReferenceError
    message: ex.line, // e.g. x is undefined
    url: document.location.href,
    stack: ex.stack // stacktrace string; remember, different per-browser!
  };

  $.post('/logger/js/', {
    data: errorData
  });
}
```

请注意，如果必须跨源传输错误，则报告端点需要支持CORS（跨源资源共享）。

## 参考文章

* [什么是脚本错误](https://blog.sentry.io/2016/05/17/what-is-script-error)
* [window.onerror捕获错误](https://blog.sentry.io/2016/01/04/client-javascript-reporting-window-onerror.html)

## 总结

如果你已经做到这一点，那么你现在可以拥有所需的所有工具来推送自己的基本错误报告库并将其与你的应用程序集成：

* window.onerror的工作原理以及它支持的浏览器
* 如何使用try / catch来捕获缺少window.onerror的堆栈跟踪
* 将错误数据传输到你的服务器
