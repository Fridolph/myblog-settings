---
title: 【Node】学习stream模块
date: 2018-11-15 22:17:52
tags:
  - node
  - stream
categories:
  - node.js
---

> 正好读到了相关的文章推送，正巧也把之前看到的和弄了一小半的资料整理一下。- - node的学习又再此开启

<!-- more -->

# stream的基本概念和常用API概述

让数据流动起来。数据从原来的source流向dest，要像水一样，慢慢的一点一点通过一个管道流过去。stream并不是node.js独有的概念，而是一个操作系统最基本的操作方式，只不过node.js有API支持这种操作方式。linux命令的`|`就是stream，因此所有server端语言都应该实现stream的API。

## 为何要使用stream

例子，在线播放视频。一点一点从服务端将视频流动到本地播放器，一边流动一边播放，直到播放完成。

```js
var http = require('http')
var fs = require('fs')
var path = require('path')
var server = http.createServer(function(req, res) {
  var fileName = path.resolve(__dirname, 'data.txt')
  fs.readFile(fileName, function(err, data) {
    res.end(data)
  })
})
server.listen(8000)
```

这段node.js代码跑起来会读取文件，语法上没问题，但如果data.txt文件非常大，在响应大量用户的并发请求时，程序可能会消耗大量的内存，这样很可能会造成用户连接缓慢的问题。而且，如果并发请求过大，服务器内存开销也很大。

要解决该问题很简单，用stream改造一下。即不是把全部文件读取了再返回，而是一边读取一边返回，一点点地把数据流动到客户端。

```js
var fs = require('fs')
var http = require('http')
var path = require('path')
var server = http.createServer(function(req, res) {
  var fileName = path.resolve(__dirname, 'data.txt')
  var stream = fs.createReadStream(fileName)
  stream.pipe(res)
})
server.listen(8000)
```

小结一下，之所以用stream，是因为一次性读取、操作大文件，内存和网络是吃不消的，因此要让数据流动起来，一点一点地进行操作。这符合分而治之的思想。

## stream流转的过程

从管道换水的例子可看出，stream包括source, dest还有中间的管道，下面将通过这三方面介绍stream的过程。其中比较关键的api有：

* data事件，用来监听stream数据的输入
* end事件，用来监听stream数据输入完成
* fs.createReadStream方法，返回一个文件读取的stream对象
* fs.createWriteStream方法， 返回一个文件读取的stream对象
* pipe方法，用来做数据流转

### source —— 从哪里来

stream常见的来源主要有三种：

* 从控制台输入
* http请求中的request
* 读取文件

运行如下代码，然后从控制台输入任何内容，都会被data事件监听到，process.stdin就是一个stream对象。`注意data就是stream用来监听数据传入的一个自定义函数`，后续会大量用到该方法。

```js
process.stdin.on('data', function(chunk) {
  console.log('stream by stdin', chunk.toString())
})
```

http请求中的request输入可以参考如下代码片段。即客户端发起http请求，服务端可以通过这种方式（用到了data事件监听）。这种http请求一般是一个post请求，上传数据。注意，end用来监听stream数据传输完毕，一般和data共用。

```js
req.on('data', function(chunk) {
  // 一点一点 接受内容
  data += chunk.toString()
})
res.on('end', function() {
  // end表示数据接受完成
})
```

读取文件用 fs.createReadStream(...) 可以返回一个读取文件的stream对象，该对象可以监听data和end事件

```js
var fs = require('fs')
var readStream = fs.createReadStream('./file1.txt')
var length = 0
readStream.on('data', function(chunk) {
  length += chunk.toString().length
})
readStream.on('end', function() {
  console.log(length)
})
```

### 管道

以上source三种代码示例有一个共同特点，就是对stream对象可以监听data和end事件。nodejs监听自定义事件要使用.on方法，例如 process.stdin.on('data', ...) ，能很直观地监听到stream数据的传入和结束。

### dest —— 到哪里去

stream常见的输出方式主要有三种：

* 输出到控制台
* http请求中的response
* 写入文件

如果让控制台输入这个source直接通过管道连接到控制台输入，即让数据从输入直接流向输出，代码如下：

```js
process.stdin.pipe(process.stdout)
```

nodejs处理http请求时会用到req和res，其实这两者都是stream对象。其中，req是source，res是dest。所以stream方式读取文件然后直接返回http请求

```js
var stream = fs.createReadStream(fileName)
stream.pipe(res)
```

读取文件可以用stream，写入文件也可以用stream，其中 fs.createWriteStream(...) 会返回一个写入文件的stream对象，即dest。这段代码，就是将一个文件中的内容，一点一点地流动到另外的文件中，完成复制功能。

```js
var fs = require('fs')
var readStream = fs.createReadStream('./file1.txt')
var writeStrea = fs.createWriteStream('./file2.txt')
readStream.pipe(writeStream)
```

### stream常见使用场景

stream常见的使用场景是http请求和文件操作。 总结来看，http请求和文件操作都属于IO，即stream主要的应用场景是处理IO，这又回到了stream的本质——由于一次性IO操作过大，硬件开销太多，影响软件运行效率，因此将IO分批分段操作，让数据一点一点地流动起来，直到操作完成。

### 小结

本章主要介绍了stream的基本概念和常用API

* stream的基本概念，即 source -> 管道 -> dest
* 为何要用stream —— 一次性操作IO，内存和网络开销过大
* source pipe dest各部分常用API
* stream的常见应用场景——IO操作

---

## node.js实现http请求

```js
var http = require('http')
var fs = require('fs')
var path = require('path')

var server = http.createServer(function(req, res) {
  var fileName = path.resolve(__dirname, 'data.txt')
  fs.readFile(fileName, function(err, data) {
    res.end(data)
  })
})
server.listen(8000)
```

## get请求和response

通过req.method可获取请求方法

```js
var http = require('http')
var path = require('path')
var fs = require('fs')

var server = http.createServer(function(req, res) {
  var method = req.method
  if (method === 'GET') {
    var fileName = path.resolve(__dirname, 'data.txt')
    fs.readFile(fileName, function(err, data) {
      res.end(data)
    })
  }
})
server.listen(8000)
```

### response和stream

response常用的API有send、end等，如上面代码中的`res.end(data)`，但是response也是一个stream对象。大家再次回顾一开始的管道换水的图，以及source.pipe(dest)模型，response就是一个dest

```js
var http = require('http')
var path = require('path')
var fs = require('fs')

var server = http.createServer(function(req, res) {
  var method = req.method
  if (method === 'GET') {
    var fileName = path.resolve(__dirname, 'data.txt')
    var stream = fs.createReadStream(fileName)
    stream.pipe(res)
  }
})

server.listen(8000)
```

### 使用stream对性能的提升

略

###　实际应用

对response使用stream特性能提高性能。因此，在nodejs中如果要返回的数据是经过IO操作得来的，例如上面例子中读取文件内容，可以直接使用stream.pipe(res)这种方式，而不再使用res.end(data)了。

这种应用的实例很多，主要有两种场景：

* 使用node.js作为服务代理，即客户端通过node.js服务作为跳板去请求其他服务，返回请求的内容
* 使用node.js作为静态文件服务器，直接返回静态文件

### 总结

本节主要讲解了node.js如何处理http的get请求，以及如何对response使用stream特性，并做了压力测试证明可以提高性能。


## 在http post请求中使用stream

```js
var http = require('http')
var path = require('path')
var fs = require('fs')

var server = http.createServer(function(req, res) {
  var method = req.method
  if (method === 'POST') {
    req.on('data', function(chunk) {
      // 接受到部分数据
      console.log('chunk', chunk.toString().length)
    })
    req.on('end', function() {
      console.log('end')
      res.end('ok')
    })
  }
})
server.listen(8000)
```

post请求发送数据量若很大， res.on('data', ...) 要分多次才能把数据接受完毕

小结一下，request和response一样，本身也是一个stream对象，可以用stream的特性，那肯定也能提高性能。两者的区别在于，request是source类型，是stream的源头，而response是dest类型，是stream的目的地。

再举个例子，如果要把request请求的数据直接response，那么最快的方式就是res.pipe(res)

### 使用stream对性能的提升

```js
var http = require('http')
var path = require('path')
var fs = require('fs')

var server = http.createServer(function(req, res) {
  var method = req.method
  if (method === 'POST') {
    var dataStr = ''
    req.on('data', function(chunk) {
      var chunkStr = chunk.toString()
      dataStr += chunkStr
    })
    res.on('end', function() {
      var fileName = path.resolve(__dirname, 'data.txt')
      fs.writeFile(fileName, dataStr)
      res.end('ok')
    })
  }
})

server.listen(8000)
```

用stream改良后如下：

```js
var server = http.createServer(function(req, res) {
  var method = req.method
  if (method === 'POST') {
    var dataStr = ''
    req.on('data', function(chunk) {
      var chunkStr = chunk.toString()
      dataStr += chunkStr
    })
    res.on('end', function() {
      var fileName = path.resolve(__dirname, 'data.txt')
      var writeStream = fs.createWriteStream(fileName)
      res.pipe(writeStream)
      req.on('end', function() {
        res.end('ok')
      })
    })
  }
})
```

### 实际应用

和get请求使用stream场景类似，post请求使用stream的场景，主要是用于将接受的数据直接进行IO操作，例如：

* 将接收的数据直接存储为文件
* 将接收的数据直接post给其他的web server

### 小结

介绍了stream在http请求中的应用和性能提升，IO操作不仅仅包括网络IO，还包括文件IO，下一节讲解stream在文件操作中的使用，以及性能提升。

---

## node.js读写文件

```js
var fs = require('fs')
var path = require('path')
var fileName = path.resolve(__dirname, 'data.txt')
// 读文件
fs.readFile(fileName, function(err, data) {
  if (err) return
  console.log(data.toString())
})
// 写文件
fs.writeFile(fileName, 'xxx', function(err) {
  if (err) return
  console.log('写入成功')
})
```

根据以上读写操作，可以做一个简单的文件拷贝程序，将data.txt中的内容拷贝到data-bak.txt 中

```js
var fs = require('fs')
var path = require('path')

// 读文件
var fileName1 = path.resolve(__dirname, 'data.txt')
fs.readFile(fileName1, function(err, data) {
  if (err) return
  var dataStr = data.toString()
  // 写入文件
  var fileName2 = path.resolve(__dirname, 'data-bak.txt')
  fs.writeFile(fileName2, dataStr, function(err) {
    if (err) return
    console.log('拷贝文件成功')
  })
})
```

### 使用stream读写文件

* 使用 fs.cretaeReadStream(filename) 来创建读取文件的stream对象
* 使用 fs.createWriteStream(filename) 来创建写入文件的stream对象

```js
var fs = require('fs')
var path = require('path')

var filename1 = path.resolve(__dirname, 'data.txt')
var filename2 = path.resolve(__dirname, 'data-bak.txt')
var readStream = fs.createReadStream(filename1)
var writeStream = fs.createWriteStream(filename2)
readStream.pipe(writeStream)
readStream.on('end', function() {
  console.log('拷贝完成')
})
```

### 使用stream带来的性能提升

略

### 应用场景

所有执行文件操作的场景，都应该尝试使用stream，例如文件的读写、拷贝、解压缩、格式转换等。除非是体积小且读写次数少，性能上可忽略。

---

原生的stream对“行”无能为力，它只是把文件当作一个数据流，简单粗暴地流动。很多文件格式都是分行的，例如csv文件、日志文件等

node.js提供了按行读取API——readline，它本质上也是stream，只不过是以“行”作为数据流动的单位

## readline的使用

相比于stream的data和end自定义事件，readline需要监听line和close两个自定义事件。readline的基本使用示例如下：

```js
var fs = require('fs')
var path = requrie('path')
var readline = require('readline')

var fileName = path.resolve(__dirname, 'readline-data.txt')
// 创建读取文件的stream对象
var readStream = fs.createReadStream(fileName)
// 创建readline对象
var rl = readline.createInterface({
  // 输入，依赖于stream对象
  input: readStream
})
// 监听逐行读取的内容
rl.on('line', function(lineData) {
  console.log('lineData: ', lineData)
})
// 监听读取完成
rl.on('close', function() {
  console.log('readline end')
})
```

以上代码，需要先根据文件名，创建读取文件的stream对象，然后传入并生成一个readline对象，然后通过line事件监听逐行读取，通过close事件监听读取完成。

### 应用场景

对于处理按行为单位的文件，如日志文件，使用readline是最佳选择。下面是一个实际例子，用来记录访问数：

```js
var num = 0
// 监听逐行读取的内容
rl.on('line', function(lineData) {
  if (lineData.indexOf('2018-10-30 14:00') >= 0 && lineData.indexOf('user.html') >= 0) {
    num++
  }
})
// 监听读取完成
rl.on('close', function() {
  console.log('num: ', num)
})
```

最后，将这个示例所用的代码贴到下面，供学习参考：

```js
var fs = require('fs')
var path = require('path')
var memeye = require('memeye')
var readline = require('readline')

memeye()

function doReadLine() {
  var fileName = path.resolve(__dirname, 'readline-data.txt')
  var readStream = fs.createReadStream(fileName)
  var rl = readline.createInterface({
    input: readStream
  })

  var num = 0
  rl.on('line', function(lineData) {
    if (lineData.indexOf('2018-10-30 14:00') >= 0 && lineData.indexOf('user.html') >= 0) {
      num++
    }
  })
  // 监听读取完成
  rl.on('close', function() {
    console.log('num: ', num)
  })
}

setTimeout(doReadLine, 5000)
```

---

stream就是数据一点一点地流动起来，那么每次流动的数据是什么呢？

```js
var readStream = fs.createReadStream('./file.txt')
readStream.on('data', function(chunk) {
  // ...
})
```

## 二进制

冯诺依曼结构，其核心内容之一就是：计算机使用二进制形式存储计算。

计算机内存由若干个存储单元组成，每个存储单元只能存储0或1（因为内存是硬件，计算机硬件本质上就是一个一个的电子元件，只能识别充电和放电的状态，充电代表1，放电代表0），即二进制单元（bit）。但是这一个单元所能存储的信息太少，因此约定将8个二进制单元为一个基本存储单元，叫做字节（byte）。一个字节所能存储的最大整数就是(2^8 = 256)，也正好是16^2，因此也常常使用两位的16进制数代表一个字节。例如css常见的颜色值就是6位16进制数字，它占用3个字节的空间。

二进制是计算机最底层的数据格式，也是一种通用格式。计算机中的任何数据格式，字符串、数字、视频、音频、程序、网络包等，在最底层都是用二进制来进行存储的。这些高级格式和二进制之间，都可通过固定的编码格式进行相互转换。例如，C语言中int32类型的十进制数，就占用32bit即4byte。总之，计算机底层存储的数据都是二进制格式，各种高级类型都有对应的编码规则，和二进制进行相互转化。

## nodejs表示二进制

Buffer就是nodejs中二进制的表述形式。

```js
var str = '学习nodejs stream'
var buf = Buffer.from(str, 'utf-8')
```

以上代码，先通过 Buffer.from 将一段字符串转化为二进制形式，其中utf-8是一个编码规则。二进制打印出来之后是一个类数组的对象，每个元素都是两位的16进制数字，即代表一个byte，打印出来的buf一共有20byte。即根据utf-8的编码规则，这段字符串需要20byte进行存储，最后再通过utf-8规则将二进制转换为字符串并打印出来

### 流动的数据是二进制格式

```js
var readStrem = fs.createReadStream('./file.txt')
readStream.on('data', function(chunk) {
  console.log(chunk instanceof Buffer)
  console.log('chunk: ', chunk)
})
```

可以看到stream中流动的数据就是Buffer类型。因此，在使用stream chunk时，需要将这些二进制数据转换为相应的格式。例如之前讲解post请求是，从request中接收数据就是这样。再回归一下之前的代码，就能明白了。

```js
var dataStr = ''
req.on('data', function(chunk) {
  // 将二进制数据先转化为字符串
  var chunkStr = chunk.toString()
  dataStr += chunkStr
})
```

stream中为何要“流动”二进制格式的数据呢？

为了优化IO操作。无论是文件IO还是网络IO，其中包含的数据格式是未知的，如字符串、音频、视频、网络包等。即便这些字符串，其编码规则也是未知的，如ASC编码、utf-8编码。再这么多未可知的情况下，只能是以不变应万变，直接用最通过的二进制格式，谁都能认识，且二进制格式进行流动和传输，效率是最高的。

### Buffer带来的性能提升

略

## 总结

* 二进制和字节的基本认识
* node.js中Buffer表示二进制
* stream中的chunk是二进制格式，以及和字符串格式的转换
* 二进制格式在http请求中的性能提升

---

## stream常用类型总结

再次回顾这张图 source通过一个管道流向了dest，如下图：

这里的source可能是http请求中的request，也可能是读取文件的stream对象，也可能是process.stdin；这里的dest可能是请求中的response，也可能是写入文件的stream对象，也可能是process.stdout；这里的管道就是pipe函数。

先不管pipe函数。source和dest完全就是两个不同的类型，一个是读取数据的，叫做readable stream，一个是写入数据的，叫作Writeable stream。除了这两种类型之外，还有一种类型叫作duplex stream（双工流），即有读取又有写入能力。示例代码如下：

```js
var fs = require('fs')
var zlib = require('zlib')
var readStream = fs.createReadStream('./file.txt')
var writeStream = fs.createWriteStream('./file.txt')
readStream.pipe(zlib.createGzip()).pipe(writeStream)
```

### readable stream

http请求中的request和读取文件的stream对象都是readable stream。它有两种常用操作方式，第一种是直接将数据pipe到一个Writeable stream

```js
var fileName = path.resolve(__dirname, 'data.txt')
var stream = fs.createReadStream(fileName)
//
stream.pipe(res)
```

第二种是通过监听on end自定义事件来获取数据再手动处理，例如之前讲解post请求时的代码示例

```js
var dataStr = ''
req.on('data', function(chunk) {
  // 接收到数据先存储起来
  var chunkStr = chunk.toString()
  dataStr += chunkStr
})
req.on('end',function() {
  // 接收完数据将数据写入文件
  var fileName = path.resolve(__dirname, 'post.txt')
  fs.writeFile(fileName, dataStr)
  res.end('OK')
})
```

以上说的两个例子，都是已经分装好的readable stream，那么它本来的面目是怎样的？如下代码：

```js
var Readable = require('stream').Readable
// 构造一个readable stream并往里添数据
var rs = new Readable
rs.push('learn')
rs.push('nodejs')
rs.push('stream')
rs.push(null)
// pipe到一个Writeable stream
rs.pipe(process.stdout)
```

从上代码可看出，nodejs提供了readable stream的构造函数，可以new出一个新的readable stream对象。然后通过push函数往里灌入完成，即可输入了。最后pipe到了一个Writeable stream

### Writeable stream

根据之前的分析，http请求中的response和写入文件的stream对象都是Writeable stream，它可以作为参数传入pipe函数，以读取上游的数据。例如之前讲解文件操作时拷贝文件的代码示例。

```js
var fileName = path.resolve(__dirname, 'post.txt')
var writeStream = fs.createWriteStream(fileName)
res.pipe(writeStream)
```

以上代码中 writeStream 是已经封装好了的 Writeable stream ，下面再来看看它的真实面目。

```js
var Writeable = require('stream').Writeable

var ws = Writeable()
ws._write = function(chunk, enc, next) {
  // 输出流动的数据
  console.log(chunk.toString())
  // 继续监听下一次输出
  next()
}
// 作为参数传递到pipe函数中
process.stdin.pipe(ws)
```

根据以上代码得知，nodejs提供了Writeable构造函数可以new一个新的Writeable stream。通过实现它的_write方法即可监听到每次流动的数据，运行next()可继续监听.

### 再谈pipe

之前一直是用`source.pipe(dest)`这种模式来用pipe的，其实pipe可以链式调用。例如上文演示的duplex stream示例代码`readStream.pipe(zlib.createGzip()).pipe(writeStream)`，还有之前讲解文件操作最后列举的gulp配置文件~

之前讲解`source.pipe(dest)`模式是为了方便理解和使用，现在我们更新一个更严谨的pipe用法：

* 调用pipe的对象必须是readable stream或者duplex stream，即具有读取数据的功能，如req.pipe(...)
* 传入pipe的参数必须是writeable stream或者duplex stream, 即具有写入数据的功能，如req.pipe(res)
* pipe支持链式调用

更新了pipe的最新规则，再来看就不会有困惑了。

### 小结

这里主要讲解了stream的常用类型和pipe函数的规则：

* stream的常见类型：readable stream和writeable stream
* readable stream的本质和用法
* writeable stream的本质和用法
* pipe的新规则
