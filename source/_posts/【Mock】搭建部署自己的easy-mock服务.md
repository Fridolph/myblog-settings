---
title: 【Mock】搭建部署自己的easy-mock服务
date: 2018-01-04 21:52:15
tags:
  - mock
categories:
  - 环境
---

> Easy Mock 是一个可视化，并且能快速生成 模拟数据 的持久化服务。 真是超级好用啊，顺便还学习了mock.js语法，对于mock数据友好高效，在官网用了很久发现人数越来越多，偶尔会出现卡顿，遂决定搭建自己的easy-mock服务

更多参考[官方文档](https://www.easy-mock.com/)

<!-- more -->

## 特性

* 支持接口代理
* 支持快捷键操作
* 支持协同编辑
* 支持团队项目
* 支持 RESTful
* 支持 Swagger | OpenAPI Specification (1.2 & 2.0 & 3.0)
* 基于 Swagger 快速创建项目
* 支持显示接口入参与返回值
* 支持显示实体类
* 支持灵活性与扩展性更高的响应式数据开发
* 支持自定义响应配置（例：status/headers/cookies）
* 支持 Mock.js 语法
* 支持 restc 方式的接口预览

## 准备工作

准备一台服务器和域名，做好相关备案工作。这些就省略了，接下来安装

    $ git clone https://github.com/easy-mock/easy-mock.git
    $ cd easy-mock
    $ export NODE_ENV=production # 很重要
    $ npm install
    $ npm run build


接下来找到 config/default.json，或者创建一个 config/local.json 文件，将如下需要替换的字段换成自己的配置即可。

这里有踩坑，如果要以production方式成功部署，需要将 default.json 拷贝一份，命名为 `production.json`,

先说下，大部分保持默认配置就好，具体的可根据实际服务器情况做相应修改

```js
{
  "port": 7300, // 默认是7300端口，根据实际情况修改
  "host": "0.0.0.0", // 我改成了localhost 应该没影响
  "pageSize": 30, // 一页显示多少条接口列表， 可改成 20
  "proxy": false,
  "db": "mongodb://localhost/easy-mock", // 添加下mongodb的端口
  "unsplashClientId": "",
  "redis": {
    "port": 6379,
    "host": "localhost"
  },
  "blackList": {
    "projects": [], // projectId，例："5a4495e16ef711102113e500"
    "ips": [] // ip，例："127.0.0.1"
  },
  "rateLimit": { // https://github.com/koajs/ratelimit
    "max": 1000,
    "duration": 1000
  },
  "jwt": {
    "expire": "14 days",
    "secret": "shared-secret"
  },
  "upload": {
    "types": [".jpg", ".jpeg", ".png", ".gif", ".json", ".yml", ".yaml"],
    "size": 5242880,
    "dir": "../public/upload",
    "expire": {
      "types": [".json", ".yml", ".yaml"],
      "day": -1
    }
  },
  "fe": {
    "copyright": "",
    "storageNamespace": "easy-mock_",
    "timeout": 25000,
    "publicPath": "/dist/"
  }
}
```

### mongodb配置

既然提到了mongoDB和redis这里做个备份，一起记录下吧。

网上下载好最新的 mongoDB 和 redis传到 服务器，我放到了 /usr/local/mongodb  /usr/local/redis

启动mongo 只需进去 mongodb/bin 中 ～ 可以写个 mongodb.conf 也可以直接带上参数命令行的形式执行

我们先将mongodb添加到环境变量中，这里就可以直接输命令行启动了：

    cd ~
    export PATH=${PATH}:/usr/local/mongodb/bin

如果不能运行可以 写到 根目录到 .bash_profile下 source ./bash_profile后 在尝试

然后启动 mongodb

### redis安装

将下载好的文件解压

    tar zxvf redis.tar.gz

然后进入我们解压好的redis文件夹中， make一下 然后进入src后再 `make install`进行redis的安装

安装完后，和上面mongodb差不多，需要进行后端启动，这样不用每次启动服务器后再启动redis

a)创建bin和redis.conf文件

复制代码代码如下:

    mkdir -p/usr/local/redis/bin
    mkdir -p/usr/local/redis/ect

b)执行Linux文件移动命令：

复制代码代码如下:

    mv /lamp/redis-3.0.7/redis.conf /usr/local/redis/etc
    cd /lamp/redis-3.0.7/src
    mv mkreleasdhdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-server /usr/local/redis/bin

这里执行 ./redis-serve 就可以启动redis了，不过我们换成后台启动的方式

c)首先编辑conf文件，将daemonize属性改为yes（表明需要在后台运行）

    cd etc/
    Vi redis.conf

d)再次启动redis服务，并指定启动服务配置文件

    redis-server /usr/local/redis/etc/redis.conf

更多参考 https://www.cnblogs.com/rainowl-ymj/p/7403638.html

---

于是继续我们的easy-mock部署

接上面，easy-mock需要以 production的方式安装和run build，这里也是群里大佬告诉我的，这样的上产环境部署才不会出现各种bug。

服务，数据库各种没问题后，自己配置nginx吧，由于有隐私内容这里就省略了

回到 easy-mock根目录下

    NODE_ENV=production pm2 start app.js --name=easy-mock
    pm2 list

看看我们的easy-mock服务有没有启动好吧～～

![](http://blog.fueson.top/article/img/easy.jpg)

---

等项目完工了，回头再添加一点自己对mock.js 和 自行mock 请求的一些笔记和理解吧
