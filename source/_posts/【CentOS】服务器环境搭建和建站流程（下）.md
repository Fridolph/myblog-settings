---
title: 【CentOS】服务器环境搭建和建站流程（下）
date: 2019-3-1 21:47:09
tags:
  - centos
  - git
  - nvm
categories:
  - 环境配置
---

> 一朋友（不是无中生友啊 ...）看了我的网站，问道：“怎么建网站的? 巴拉巴拉……” 嘛，我肯定不能回答忘了是吧？ 善良的我主动提出帮助朋友搞定这一套并帮他把网站上线，于是…… 多了台服务器玩，那么温故知新把这套流程记下来

<!-- more -->

我们的目标是要能够运行网站，域名备案那套让朋友去弄了，那么我们要做的就是环境搭建和相关配置了，我们把目标立一下吧

1. 安装Git
2. 安装Node (通过nvm)
3. 安装各类数据库 mysql mongodb redis
4. 进程守护 pm2
5. 安装和配置nginx
6. 测试能访问网站（阶段性 大功告成）

知道了要做什么，接下来就开干吧！

## 安装依赖库和编译工具

为了后续安装能正常进行，我们先来安装一些相关依赖库和编译工具

```bash
# 依赖库
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
# 编译工具
yum install gcc perl-ExtUtils-MakeMaker
```

## 安装Git

> 注： 可先输入 `git --version` 若没有命令的话可用 `yum install git`来安装 (这种方式安装的git可能不是最新版本，所以我们用另一种方式来)

### 下载 git

选一个目录，用来放下载下来的安装包，这里将安装包放在 /usr/local/src 目录里

`cd /usr/local/src` 到官网找一个新版稳定的源码包下载到 /usr/local/src 文件夹里

`wget https://www.kernel.org/pub/software/scm/git/git-2.10.0.tar.gz`

### 解压和编译

```bash
# 解压下载的源码包
tar -zvxf git-2.10.0.tar.gz

# 解压后进入 git-2.10.0 文件夹
cd git-2.10.0

# 执行编译
make all prefix=/usr/local/git

# 编译完成后, 安装到 /usr/local/git 目录下
make install prefix=/usr/local/git
```

### 配置环境变量

```bash
# 将 git 目录加入 PATH
# 将原来的 PATH 指向目录修改为现在的目录
echo 'export PATH=$PATH:/usr/local/git/bin' >> /etc/bashrc

# 生效环境变量
source /etc/bashrc

# 此时我们能查看 git 版本号，说明我们已经安装成功了
git --version
```

## Node.js包管理工具 nvm

### 安装nvm

```bash
# 1. 安装nvm
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
```

会输出如下：

```bash
=> Downloading nvm as script to '/root/.nvm'

=> Appending nvm source string to /root/.bashrc
=> Appending bash_completion source string to /root/.bashrc
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

提示信息可以看出，设置了环境变量， 需要刷新环境变量

```bash
source /root/.bashrc
# 验证环境变量是否生效

echo $NVM_DIR
# 输出了/root/.nvm说明已经OK
# 验证nvm安装是否成功

nvm --version
#输出版本号说明nvm安装Ok
```

### 使用nvm安装nodejs

由于网络问题，请设置国内源 指定 nvm 的镜像需要在环境配置中增加 NVM_NODEJS_ORG_MIRROR

在/root/.bashrc中增加以下内容

vim ~/.bashrc

```bash
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
```


```bash
# 刷新环境变量
source /root/.bashrc

# 查看nodejs可用版本
nvm ls-remote

# 安装nodejs - 8版本的稳定版
nvm install v8.15.1

# 查看已经安装的所有版本
nvm ls

# 使用某个版本
nvm use v8.15.1

# 查看node npm版本
node -v
npm -v

# 若能正确输入版本号，则环境搭建成功，可以开始后面的了~
```

## 安装各类数据库 mysql mongodb redis

略

## 守程守护 pm2

略

## 安装和配置nginx

略

## 测试能访问网站（阶段性 大功告成）

略

---

由于 - - 临时出点状况，服务器暂访问不上 … 额，这些就放后面再补完了

（待填完坑...）
