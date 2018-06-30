---
title: 【JS】学习正则表达式
date: 2018-06-29 17:49:03
tags:
  - js
  - 正则表达式
categories:
  - js
---

> 最近因工作用到了，顺便查看了一些关于正则的博客，确实非常使用。学好正则看来是以后的一个必选项，趁着最近时间挺充裕的，于是系统的入门了一下，也算是能手写几个简单的正则来匹配规则了。

<!-- more -->

![学习正则表达式](http://blog.fueson.top/18-6-28/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F.png?imageView2/0/interlace/1/q/70|watermark/2/text/ZnJpZG9scGg=/font/5a6L5L2T/fontsize/240/fill/IzAwMDAwMA==/dissolve/100/gravity/SouthEast/dx/10/dy/10|imageslim)

### 子字符串匹配和替换

如果只想知道某个字符串是否包含在一个更大的字符串中，下面的 String.prototype 方法就可以实现：

```js
const input = 'As I was going to Saint Ives'
input.startsWith('As') // true
input.endsWith('Ives') // true
input.startsWith('going', 9) // true - 从下标9开始数
input.endsWith('going', 14) // true - 将下标14当作字符串结尾
input.includes('going') // true
input.includes('going', 10) // false - 从下标10开始
input.indexOf('going') // 9
input.indexOf('going', 10) // -1

// 若想进一步操作，如替换掉刚才匹配的字符串，可以使用String.prototype.replace
const output = input.replace('going', 'walking')
```

### 构造正则表达式

在 JS 中，正则可以通过 RegExp 类来表示：

```js
const re1 = /going/ // 可以搜索 'going'的正则表达式
const re2 = new RegExp('going') // 使用对象构造器的等价形式
```

### 使用正则进行搜索

```js
const input = 'As I was going to Saint Ives'
const re = /\w{3,}/gi

// 从字符串(input)开始
input.match(re) // ['was', 'going', 'Saint', 'Ives']
input.search(re) // 5

// 从正则表达式开始(re)
re.test(input) // true - input至少包含一个三个字母的单词
re.exec(input) // ['was'] (第一个匹配)
re.exec(input) // ['going'] (exec会记住它所在的位置)
re.exec(input) // ['Saint']
re.exec(input) // ['Ives']
re.exec(input) // null - 匹配完毕

// 注意，所有这些方法都可以直接使用字面量语法
input.match(/\w{3,}/gi)
input.search(/\w{3,}/gi)
;/\w{3,}/gi.test(input)
;/\w{3,}/gi.exec(input)
```

### 使用正则表达式进行替换

```js
const input = 'As I was going to Saint Ives'
const output = input.replace(/\w{3,}/gi, '****')
```

### 匹配 HTML

```js
const html1 = `
  HTML width <a href="/one">one link</a>, and some JavaScript.
  <script src="strff.js"></script>
`
const matches = html.match(/<area|<a|<link|<script|<source/gi)
```

要了解的是，现在来说正则表达式不能解析 HTML。为解决这个问题，需引入一个解析器：

```js
const html2 = '<br> [!CDATA[<br>]]'
const matches = html.match(/<br>/gi) // ['<br>', '<br>']
```

### 字符集

字符集提供了一种简洁的方式，来表达单个字符的分支。如果想在一个字符串中查找所有的数字，可以使用分支：

**具名字符集**

\d [0-9]
\D [^0-9]
\s [\t\v\n \r] 包含制表符、空格和垂直制表符
\S [^\t\v\n \r]
\w [a-zA-Z_] 破折号和句号没有被包含进来，所以它不能用于域名和 CSS 类
\W [^a-za-z_]

### 重复

**重复修饰符**

`{n}` 精确 n 次
/\d{5}/ 匹配 5 位数字

`{n,}` 至少 n 次
/\d{5,}/ 匹配 5 位或 5 位以上数字

`{n, m}` 最少 n 次，最多 m 次
/\d{2,5}/ 匹配 2 到 5 位数字

`?` 0 或 1 次，等价于{0,1}
/[a-z]\d?/i 匹配跟随了 0 个或 1 个数字的字符

`*` 0 次或多次
/[a-z]\d\*/i 匹配跟随了 0 个或多个数字的字母

`+` 1 次或多次
/[a-z]\d+/i 匹配了至少跟随了 1 个数字的字母

### 句点元字符和转义

在正则中，句点是一个特殊的字符，表示“匹配任何内容”（除了新的一行）。通常，这个匹配一切的元字符用来消费哪些输入中并不关心的内容。

### 分组

到目前为止，所构造的正则能够识别单个字符（重复允许多次匹配，但这依旧是一个单字符匹配）。而分组则允许构造子表达式，它可以被当作一个独立单元来使用。

除了创建子表达式，分组还可以帮助“捕获”分组结果，以便后续使用。
“捕获”结果是默认功能，不过也有办法创建“非捕获组”，这也是接下来要学习的内容。

分组是使用圆括号来指定的，非捕获组看起来像 `(?:<subexpression>)`，其中<subexpression>是需要匹配的内容

看组例子，假设现在要匹配的后缀为.com .org .edu 的域名

```js
const text = 'Visit oreilly.com today!'
const match = text.match(/[a-z]+(?:\.com|\.org|\.edu)/i)
```

分组的另一个好处是可以在分组时使用重复。一般情况下，重复仅被用在重复元字符前面的单个字符上。分组则允许将其用在一整个字符串上。有一个常见的例子是，如果想匹配 URL,以及那些以http://或https://（独立于协议的URL）开始的URL，可以在分组上使用代表匹配0个或1个`?`的重复元字符：

```js
const html = `
  <link rel="stylesheet" href="http://insecure.com/stuff.css">
  <link rel="stylesheet" href="https://secure.com/securestuff.css">
  <link rel="stylehseet" href="//anything.comflexible.css">
`
const matches = html.match(/(?:https?)?\/\/[a-z][a-z0-9-]+[a-z0-9]+/gi)
```

以一个非捕获组开始 (?:https?)? ，注意这里有两个匹配 0 或 1 个的重复元字符

第一个表示 https 的 s 是可选的（一般情况下重复元字符只作用于它左边最近的字符）第二个指向它左边的整个组（整体来看，它会匹配空字符串：没有 https、http 或者 https）

继续执行，匹配了两个斜杠 \/\/ (必须对斜杠进行转义)
然后得到了一个复杂的字符类。

> 需要记住一点，使用正则时并不需要一次做完所有事情。事实上，每当浏览网站时，可以先找出所有 URL 或疑似 URL 的东西，然后做二次分析，筛选出那些非法或不完整的 URL 等。 但，为防止注入攻击而检查用户输入等情况就要让正则滴水不漏

---

### 懒惰匹配、贪婪匹配

例：html 文本，想将其中的`<i>`标签替换成`<source>`标签，下面是第一次尝试：

```js
var input = `
  Regex pros know the difference between
  <i>greedy</i> and <i>lazy</i> matching.
`
input.replace(/<i>(.*)<\/i>/gi, '<strong>$1</strong>')
// Regex pros know the difference between
// <strong>greedy</i> and <i>lazy</strong> matching.
```

我们认识一下正则表达式引擎的工作方式，它只有在找到符合要求的匹配后，才会消费输入并且继续运行。默认情况下，它是通过贪婪模式来实现的，它会找到第一个`<i>`，然后，在找到`</i>`并且确定在这个`</i>`之后不存在同样的`</i>`，查找都不会停止。因为这里有两个`</i>`，所以正则会匹配到第二个`</i>`，而非第一个。

这里可以使用重复元字符 \* 将其转换成懒惰匹配来解决，在后面添加一个问号即可：

```js
var input = `
  Regex pros know the difference between
  <i>greedy</i> and <i>lazy</i> matching.
`
input.replace(/<i>(.*?)<\/i>/gi, '<strong>$1</strong>')
```

与之前相比，该正则除了在\*后面加了一个问号，其他完全一样。

所有的重复元字符： _ + ？ {n} {n, } {n, m}都可以在后面跟随一个问号将它变成懒惰的（虽然在实践中，通常只把它和_ + 一起使用过）

### 反向引用

先由简单例子入手，假设想匹配符合 XYXY 格式的乐队名字，所以当希望匹配（PJJP、GOOG、ANNA）这些乐队名时，反向引用就可以登场了。正则表达式中的每个组（包括子组）都被分配了一个数字，从左到右依次是 1，2，3...可以通过在反斜杠后加一个数字的方式来引用特定的组。

```js
const promo = 'Opening for XAAX is the dynamic GOOG! At the box office now !'
const bands = promo.match(/(?:[A-Z])(?:[A-Z])/g)
// 使用重音符，是因为我们将单引号和双引号都用过了
const html = `
  <img alt="A 'simple' example.">
  <img alt="Don't abuse it!">
`
const matches = html.match(/<img alt=(?:['"]).*?/g)
```

### 替换组

分组带来的好处是，可以利用它做一些更加复杂的替换，继续看 HTML 的例子，加入想要去掉一个`<a>`标签中除了 href 以外的内容

```js
let html = `<a class="nope" href="/yep">Yep</a>`
let str = html.replace(/<a .*?(href=".*?").*?/, '<a $1>')
// str -> "<a href="/yep">>Yep</a>"
```

所有的组都被分配了一个从 1 开始的数字，这个正则表达式中，通过\1 来引用第一个组；而在替换字符串上，用的是$1。注意，在这个表达式中使用懒惰量词是为了防止它在匹配时跨域多个`<a>`标签。不过，如果`<a>`标签的 href 属性使用的是单引号而非双号，也会匹配失败。

下面来扩展这个例子，希望保持 class 和 href，依旧删除其他元素：

```js
let html = `<a class="yep" href="/yep" id="nope">Yep</a>`
let str = html.replace(/<a .*?(class=".*?").*?(href=".*?").*?>/, '<a $2 $1>')
```

注意在这个表达式中，将 class 和 href 的顺序颠倒了，使得 href 始终先出现，这个表达啊是的问题在于 class 和 href 始终要保持相同的顺序，并且，一旦`<a>`标签中的属性使用了单引号，就会匹配失败

除了$1,$2,$3...这些组引用，还有$` 匹配项之前的所有内容， $& 匹配目标本身， $'匹配项之后的所有内容，如果想使用一个美元符号，可以使用$$:

```js
const input = 'One two three'
input.replace(/two/, '($`)') // "One (One ) three"
input.replace(/\w+/g, '($&)') // "(One) (two) (three)"
input.replace(/two/, "($')") // "One ( three) three"
input.replace(/two/, '($$)') // "One ($) three"
```

### 函数替换

这是正则最棒的一个特性之一，因为它允许将一个非常复杂的正则表达式拆分成一些简单的表达式。

再来看一个实际修改 HTML 的例子：希望保留 class, id, href 属性并删除其他内容

```js
const html = `
  <a class="foo" href="/foo" id="foo">Foo</a>
  <A href='/foo' Class="foo">Foo</a>
  <a href="/foo">Foo</a>
  <a onclick="javascript:alert('foo!')" href="/foo">Foo</a>
`
```

用正则来实现感觉很麻烦，因为变化太多！确实，不过之前说过，并不一定要一次到位。可以通过将表达式拆分成两个，从而大大减少变化的数量：一个用于识别`<a>`标签，而另一个用于将`<a>`替换成期望的内容。

```js
function sanitizeATag(aTag) {
  // 获取标签
  const parts = aTag.match(/<a\s+(.*?)>(.*?)<\/a>/i)
  // parts[1]是<a>标签中间的属性
  // parts[2]是<a>和</a>中间的内容
  const attributes = parts[1]
    // 接下来将其分割成独立的属性
    .split(/\s+/)

  return `<a ${attributes
    .filter(attr => /^(?:class|id|href)[\s=]/i.test(attr))
    .join(' ')}>
    ${parts[2]}
  </a>`
}
```

该函数比想象中要长，不过为了更加清晰，可以将它分成不同的部分。注意即使在这个函数中，依旧使用了多个正则表达式：一个用来匹配`<a>`，一个用来切割（使用一个正则表达十来识别一个或多个空格字符）字符串，还有一个用来过滤期望的属性。如果只用一个正则表达式来完成这些工作将会非常复杂。

接下来：在一个包含很多`<a>`的 HTML 块中使用 sanitizeATag 函数，编写一个只匹配`<a>`的正则表达式就很简单了：

    html.match(/<a .*?>(.*?)<\/a>/ig);

在匹配时，可以将函数当作一个替换参数传给 String.prototype.replace。目前为止，只 ongoing 过字符串作为替换参数。而使用函数则允许对每一个替换执行一个特定的操作。

```js
html.replace(/<a .*?>(.*?)<\/a>/gi, function(m, g1, offset) {
  console.log(`<a> tag found at ${offset}. contents: ${g1}</a>`)
})
/*undefined
  undefined
  undefined
  undefined
*/
// <a> tag found at 3. contents: Foo</a>
// <a> tag found at 49. contents: Foo</a>
// <a> tag found at 86. contents: Foo</a>
// <a> tag found at 111. contents: Foo</a>
```

传给 String.prototype.replace 的函数会按顺序接收以下参数：

* 整个匹配的字符串(等价于$&)
* 匹配上的组(如果存在)，有多少个组，这种参数就会有多少个
* 原始字符串中的匹配偏移量(一个数字)
* 原始字符串(很少使用到)

该函数的返回值就是用来替换正则表达式的字符串。在上例中，没有指定返回值，所以默认返回 undefined。它会被转换成字符串后当作替换字符串使用。上例的重点就是强调这种工作机制，而非真实的转换，所以这里并没有返回最终结果。现在来回顾一下这个例子，有了能够清理单个`<a>`标签的函数，以及在 HTML 中查找`<a>`标签的方法，所以可以将它们结合起来使用：

```js
html.replace(/<a .*?<\/a>/gi, sanitizeATag)
```

当需要从一个大字符串中匹配小字符串，并且还要对小字符串做额外处理时，都可以通过向`String.prototype.replace`中传入函数来解决这个问题！

### 锚点

通常，我们会关心一个字符串的开始和结束，或者整个字符串（而不只是一部分），这时`锚点`就派上用场了。有两种锚点：分别是用于匹配行开始的`^`，以及用于匹配行结束的`$`

```js
const input = 'It was the best of times, it was the worst of times'
const begin = input.match(/^\w+/g) // "It"
const end = input.match(/\w+$/g) // "times"
const everything = input.match(/^.*$/g)
// "It was the best of times, it was the worst of times"
const nomatch1 = input.match(/^best/gi)
const nomatch2 = input.match(/worst$/gi)
```

关于锚点，一般情况下，它匹配的是整个 字符串的开始和末尾，即使字符串中有换行。如果想把某个字符串当作多行字符串（以换行符分隔）来处理，就需要用到 m(多行选项)：

```js
const input = 'One line\nTwo lines\nThree lines\nFour'
const begin = input.match(/^\w+/gm)
const end = input.match(/\w+$/gm)
```

### 单词边界匹配

正则中一个经常被忽视，但却非常有用的特性。类似开始锚点和行末锚点，单词边界匹配的是\b，取反是\B，它不消费输入内容。单词边界界定为一个\w 匹配之前或之后紧挨着一个\W（非单词字符），或字符串的开始或结尾。来看个例子：

```js
const inputs = [
  'yinlinshengxiao@gmail.com',
  'yinlinshengxiao@gmail.com is my email',
  'my email is yinlinshengxiao@gmail.com',
  'use yinlinshengxiao@gmail.com, my email',
  'my email: yinlinshengxiao@gmail.com.'
]
let str = `
  虽然有多种不同情况，这些邮箱有一个共同点：它们都处在单词边界。
  单词边界标记的另一个好处是，因为它们不消费输入，所以不用担心“将它们放回”到替换字符串中：
`
const emailMatcher = /\b[a-z][a-z0-9._-]*@[a-z][a-z0-9_-]+\.[a-z]+(?:\.[a-z]+)?\b/gi
inputs.map(s => s.replace(emailMatcher, '<a href="mailto:$&">$&</a>'))
```

当需要搜索以外的单词开始、结束或包含其他单词的文本时，使用单词边界也非常方便。例如：/\bcount/ 会找到 count countdown， 但不会找到 discount recount 等。而 /\bcount\B/只能找到 countdown，/\Bcount\b/会找到 discount 和 recount，而 /\Bcount\B/只能找到 accountable

### 向前查找

与锚点和单词边界元字符一样，它不消费输入。然而，不同于锚点和单词边界的是，它们是通用的，可以匹配任何子表达式却不消费它。事实上，正如单词边界元字符，向前查找的这种不消费的特性，解决了有时候不得不进行“原封不动”的替换问题。只要有内容重复，向前查找就是必须的，而且他们可以简化某些特定类型的匹配。

例子：验证密码是否符合预设规则

```js
function validPassword(p) {
  return (
    /[A-Z]/.test(p) &&
    /[0-9]/.test(p) &&
    /[a-z]/.test(p) &&
    !/[^a-zA-Z0-9]/.test(p)
  )
}
// 假设想将它们组合成一个正则表达式：
function validPassword(p) {
  return /[A-Z].*[0-9][a-z]/.test(p)
}
```

该表达式对顺序有要求，不仅要求大写字母出现在数字之前，数字出现在两个小写字母前，而且没有对非法字符做任何校验。实际上也没有有个好办法可以实现它，因为字符在正则表达式运行时就被消费了。

`向前查找`通过不消费输入来解决这个问题，本质上每个向前查找都是一个不消费输入的独立正则表达式，在 JS 中，向前查找是这样的 `(?=<subexpression>)`, 还有一个“否定向前查找” `:(?!<subexpression>)` 只会匹配不存在于子表达式中的内容。下面继续来重写上面的验证密码：

```js
function validPassword(p) {
  return /(?=.*[A-Z])(?=.*[0-9])(?=.*[a-z])(?!.*[^a-zA-Z0-9])/.test(p)
}
```

该例子展示了向前检查（以及否定向前检查）的一个重要场景。

### 动态构造正则表达式

这里提倡优先使用正则表达式字面语法而非构造器，因为不用对反斜杠进行转义。需要使用构造器的地方是动态构造。例如，想在一个字符串中匹配一个包含多个用户名的数组，但却没有办法将这些用户名整合在一个正则表达式字面量中。此时正则构造器就有用了，它可以通过字符串来构造正则表达式：

```js
const users = ['mary', 'nick', 'arthur', 'sam', 'yevtte']
const text =
  'User @arthur started the backup and 15:15, ' +
  'and @nick and @yvette restored it at 18:35.'
const userRegexp = new RegExp(`@(?:${users.join('|')})\\b`, 'g')
text.match(userRegexp)
```

与该例正则等价的字面量是： `/@(?:mary|nick|arthur|sam|yevtte)\b/g`
需要注意的是：必须在 b 单词边界元字符之前使用双反斜杠

## 总结

这一篇已经涉及了正则表达式中的主要知识点，但对于正则中包含的技术、例子和其固有的复杂性，也只是浅尝辄止。想要深入学习正则表达式，需要更多的练习与理解，那么以后在工作与学习中多多运用进去吧。
