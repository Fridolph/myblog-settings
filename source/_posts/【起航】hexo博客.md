---
title: 【搬家】wordpress到hexo~ 都是泪
date: 2017-05-15 13:41:03
tags: 
  - 杂记
categories: 
  - 心情
---

> 个人博客什么的，15年建的wordpress小站在2017年4月被我搞崩了…… 文章也没有备份，加上阿里云到期，备案等各种问题，最后还是先转到github，一来速度不错，二来可以hit督促自己学习，三来用于备份，以后有时间和心情还是弄个云服务器玩玩吧。不过最重要的一点还是因为程序员就是爱折腾…… 好在之前写的文章也没什么营养，丢就丢了吧，希望放在这上面的能精简些

如果你对 hexo搭建博客感兴趣可以参考这篇：[20分钟教你使用hexo搭建github博客](http://www.jianshu.com/p/e99ed60390a8)，主题的话推荐下很好看的 [yilia](https://github.com/litten/hexo-theme-yilia)。
如果想和我使用的一样可以参考下我的仓库：[我的主题备份](https://github.com/Fridolph/Fridolph.github.io)

<!-- more -->

所以说，一切都是自作孽啊，博客重新运营吧，正好多说关闭了，评论模块还得折腾下。

## 其他叨唠

作为过来人的经验，我们应该做的第一件事是为博客备份，至于怎么做就不展开了，本地备份加上云盘备份一份最好。
虽然之前的文章都不要了，但貌似也没什么损失，确实文才不好也算了，原创质量都不高，也没法啊~~
重新开个坑，希望业余时间能多多充实下自我，多多学习，提高学习力与竞争力

## 开始写博客

这里就记些简单的操作吧（虽然以后会很常用）

### 发表一篇文章

```
hexo new "文章标题"
```

新建的文章会统一存放在 source -> _post 路径下，至于markdown的语法可以参考 [markdown语法](http://www.appinn.com/markdown/) 很简单，多写几次就会了。

### 本地环境

```
hexo s 
```

### 上线部署

然后在启动本地环境 localhost:4000 默认地址，点开可查看最近修改，之后没问题就可以部署了
_config.yml 已做好备份，可以直接参考我的设置，官方文档的说明也很详细

```
hexo generate
hexo deploy
```

那么直接开始吧！