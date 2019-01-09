---
title: 【开源】2种方式同步Fork的项目[转载]
date: 2017-02-15 8:36:06
tags:
  - 同步
  - fork
categories:
  - 开源
---

> 因为这个操作很重要，且老容易忘记、出错，就特意从网上转载了几篇并整理了下然后顺便码点字增加一篇产出…

![github上 2种方式同步Fork的项目](http://blog.fueson.top/19-1-9/16197721.jpg)

<!-- more -->

## github 网站上操作

1. 打开自己的仓库，进入 code 下面

2. 点击 new pull request 创建

![1](http://blog.fueson.top/19-1-9/63215848.jpg)

3. 选择 base fork

4. 选择 head fork

5. 点击 Create pull request，并填写创建信息

![2](http://blog.fueson.top/19-1-9/82262467.jpg)

![3](http://blog.fueson.top/19-1-9/60670988.jpg)

6. 点击 Merge pull request 合并从源 fork 来的代码

![4](http://blog.fueson.top/19-1-9/67996731.jpg)

7. 完成

---

## 用 git 命令操作

1. 用 git remote 查看远程主机状态

```bash
git remote -v
git remote add upstream git@github.com:xxx/xxx.git
git fetch upstream
git merge upstream/master
git push
```

![5](http://blog.fueson.top/19-1-9/237949.jpg)

![6](http://blog.fueson.top/19-1-9/3020691.jpg)

![7](http://blog.fueson.top/19-1-9/62969692.jpg)
