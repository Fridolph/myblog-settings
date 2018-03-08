---
title: 【JS】编译一个函数，解析url字符串
date: 2017-11-28 10:49:03
tags: 
  - js
categories: 
  - js
---

> 这是最近看到的一道面试题，看似简单，其实有很多东西可深挖。正好写到博客里进行下总结

<!-- more -->

题目：编写一个函数解析 url 参数，尽可能全面正确的解析一个任意 url 的所有参数为 Object

    var url = 'http://www.domain.com/?user=anonymous&id=123&id=456&city=%E5%8C%97%E4%BA%AC&d&enabled';
    parseParam(url);

希望结果返回：

 {
user: 'anonymous',
// 重复出现的 key 要组装成数组，能被转成数字的就转成数字类型
id: [123, 456],
// 中文
city: '北京',
// 未指定值的 key 约定值为 true
enabled: true
}

---

分析下题目，编写函数，解析 url 参数，要求返回是对象

```js
// 大致思路有了，把框架勾勒出来
const parseUrl = url => {
  // 声明变量存储结果
  let result = {}
  // ...
  // 返回结果
  return result
}
```

query 值是 ?后面的所有字符串，不同参数间以&为分隔，根据题目可知 key 不一定都带上 value，我们所需的即是处理字符串，即以下方法：

String.prototype.substring(a) 拿到 a 以后（包含 a）的所有字符串
String.prototype.split('&') 将字符串按'&'分隔为数组
Array.prototype.map(item) 遍历数组，对每一项进行操作

...

思路有了，接下来尝试实现逻辑：

```js
const parUrl = url => {
  let result = {}
  let querystring = ''
  let queryArr = []
  let key
  let value
  // 若没有 '?' 即不进行处理
  if (url.indexOf('?') === -1) {
    return result
  }
  // 取到 ? 后的所有字符串
  querystring = url.substring(url.indexOf('?') + 1)
  queryArr = querystring.split('&')
  queryArr.map(item => {
    if (item.indexOf('=') === -1) {
      result[item] = true
    } else {
      let kv = item.split('=')
      key = kv[0]
      value = kv[1]
      result[key] = value
    }
  })
  return result
}
```

感觉挺好，不过对于要求还是少了些东西…… 这里直接提出来了，毕竟是看了答案之后：

1. 校验不够严谨，缺少对传入类型，是否为 url 的判断
2. 没有对传入值进行解码
3. 传入相同的 key，后面 key 的值会将前面的值覆盖掉

接下来，继续优化我们的代码：

```js
/**
 * 解析url，将之转换为对象，一个key有多个值时生成数组
 * @param {String} url 需要解析的url，按照application/x-www-form-urlencode编码
 * @return {Object} result 参数细节后的对象
 */
const parseUrl = url => {
  let result = {}
  if (typeof url !== 'string') return result
  // 判断是否为合法url可跳过
  if (url.indexOf('?') === -1) return reuslt
  let querystring = url.substring(url.indexOf('?') + 1)
  let queryArr = querystring.split('&')
  let key
  let value
  queryArr.map(item => {
    if (item.indexOf('=') === -1) {
      result[item] = true
    } else {
      let kv = item.split('=')
      let key = decodeURIComponent(kv[0])
      let value = decodeURIComponent(kv[1])
      // 如果是新key 直接添加
      if (!(key in result)) {
        result[key] = value
      }
      // 如果key已经出现一次以上，直接向数组添加value
      else if (isArray(result[key])) {
        result[key].push(value)
      }
      // 如果key第二次出现，将结果改为数组
      else {
        let arr = [result[key]]
        arr.push(value)
        result[key] = arr
      }
    }
  })

  function isArray(obj) {
    if (obj && typeof obj === 'object') {
      return Object.prototype.toString.call(obj) === '[object Array]'
    }
    return false
  }
}
const URL = 'http://www.domain.com/?user=anonymous&id=123&id=456&city=%E5%8C%97%E4%BA%AC&d&enabled'
parseUrl(URL)
/**
结果：
{
  user: 'anonymous',
  // 重复出现的 key 要组装成数组，能被转成数字的就转成数字类型
  id: [123, 456], 
  // 中文
  city: '北京', 
  // 未指定值的 key 约定值为 true
  enabled: true
}
*/
```

