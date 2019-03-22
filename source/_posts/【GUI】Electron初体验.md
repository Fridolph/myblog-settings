---
title: 【GUI】Electron初体验
date: 2019-02-11 23:49:03
tags:
  - GUI
  - electron
categories:
  - Electron
---

> hello-world工程师的日常任务~ 虽然说又开始折腾新东东了，当然这只是作个了解，毕竟前端要学的太多了，… 重心还是放在前端和Node上吧~ 于是跟着前人的路（别人的文章 + 个人心得 + 整理）就顺便写写博客来记录了

<!-- more -->

# GUI 定义

引用自百度百科

> 图形用户界面（Graphical User Interface，简称 GUI，又称图形用户接口）是指采用图形方式显示的计算机操作用户界面。
> 与早期计算机使用的命令行界面相比，图形界面对于用户来说在视觉上更易于接受。然而这界面若要通过在显示屏的特定位置，以”各种美观而不单调的视觉消息“提示用户”状态的改变“，势必得比简单的消息呈现花上更多的计算能力。

## Electron

文档相当详细 http://electronjs.org/docs

> Electron是由Github开发，用HTML，CSS和JavaScript来构建跨平台桌面应用程序的一个开源库。 Electron通过将Chromium和Node.js合并到同一个运行时环境中，并将其打包为Mac，Windows和Linux系统下的应用来实现这一目的。

小结下就是：

- 使用HTML、CSS、JavaScript来构建 UI、处理与用户的交互，同时不约而同的使用了开源浏览器 Chromium
- 使用 Node.js 来访问 浏览器 之外的内容，比如系统、文件、网络等等……

### 主进程与渲染进程

在 Electron 中，被 Electron 直接运行的脚本（package.json 中指定的 main 脚本）被称为主进程。
在 Electron 中用来展示界面的 web 页面都运行在一个独立的，属于它自己的渲染进程中。
我们可以通过主进程来创建 web 页面，但一个 web 页面被销毁的时候，对应的渲染进程也会被终止

- 主进程管理所有的 web 页面和它们对应的渲染进程
- 一个应用程序有且仅有一个主进程

在 Electron 中，Electron 同时为 主进程 与 渲染进程暴露了 Node.js 的所有接口，也就是说，我们可以在 Electron 的主进程 与 渲染进程 中使用 Node.js 的 API

### app 对象

该对象提供了一系列的事件用来控制整个应用程序的生命周期，从打开到关闭，如：

- ready
- window-all-closed
- quit

同时也提供了一些方法来管理应用程序的状态与行为，如：

- quit()
- relaunch()
- hide()
- show()

### BrowserWindow

类，创建和控制浏览器窗口

`new BrowserWindow( [options] )`

- options：窗口选项 https://electronjs.org/docs/api/browser-window

每一个 BrowserWindow 对象的实例都是一个独立的渲染进程，同时该对象也提供了各种用于操控的 API，包括：事件、属性、方法

#### 实例事件

- close
- closed
- focus
- blur
- show
- hide

#### 实例属性

- webContents：窗口包含的内容对象
- id：窗口的唯一 ID

#### 实例方法

- close()
- show()
- hide()
- maximize()
- unmaximize()
- setSize()
- getSize()
- setPosition()
- getPosition()
- setTitle()
- getTitle()
- loadFile() ： 加载页面（这就是我们要显示的内容了），页面地址使用相对路径，相对路径相对于应用程序根目录
- loadURL() : 使用 URL 协议加载文件，可以是 http 协议，也可以是 file 协议

---

## todolist 实战

上面说了些主要用到的，还是结合实例来说说吧

```bash
# todolist
|- layout # 用于存放前端相关文件
|- public # 项目用的静态资源文件
|- index.js # electron的主文件
|- package.json
```

安装依赖

`npm i electron electron-builder -D`

`npm i vue`

package.json

```js
{
  // ...省略
  "scripts": {
    "dev": ".\\node_modules\\.bin\\electron .",
    "build": ".\\node_modules\\.bin\\electron-builder -w"
  },
  // 项目运行依赖
  "dependencies": {
    "vue": "^2.5.17"
  },
  // 开发依赖
  "devDependencies": {
    "electron": "^2.0.8",
    "electron-builder": "^20.28.3"
  },
  "build": {
    "appId": "todolist-app",
    "productName": "todolist",
    "directories": {
      "output": "./dist"
    },
    "win": {
      "icon": "./source/logo.ico",
      "target": [
        "nsis",
        "zip"
      ]
    },
    "nsis": {
      "oneClick": false,
      "allowToChangeInstallationDirectory": true,
      "installerIcon": "./source/logo.ico",
      "installerHeader": "./source/header.bmp",
      "license": "./source/license.txt"
    }
  }
```

index.js

```js
const {app, BrowserWindow} = require('electron');

app.on('ready', () => {
  // 具体配置参考文档，这里只是完成一个简单的todolist 并打包
  const win = new BrowserWindow({
    width: 540,
    height: 720,
    frame: false,
    resizable: false
  });
  // 开发时可打开devtools
  // win.webContents.openDevTools();

  // 可理解为一个浏览器进程 读取了 index.html 资源, 这就是应用会呈现的页面
  win.loadFile('./layout/index.html')
});
```

这里来看看 js 的逻辑

```js
const {remote} = require('electron')

let _id = 0;

new Vue({
  el: '#root',
  data: {
      title: 'TODOS',
      type: 'all',    //all, done, undone
      newTodo: '',
      todos: []
  },
  computed: {
    showTodos() {
      return this.todos.filter(todo => {
        switch(this.type) {
          default:
          case 'all':
            return true;
          case 'done':
            return todo.done;
          case 'undone':
            return !todo.done;
        }
      });
    }
  },
  methods: {
    // 关闭应用
    closeApp() {
      // app对象只能通过主线程调用
      remote.app.exit();
    },

    // 最小化应用窗口
    miniApp() {
      // 通过remote下的一个方法来获取当前窗口对象（BrowserWindow）
      remote.getCurrentWindow().minimize();
    },

    // 添加任务
    addTodo() {
      this.todos.push({
        id: _id++,
        title: this.newTodo,
        done: false
      });
      this.newTodo = '';
    },

    // 任务状态切换
    toggle(todo) {
      todo.done = !todo.done;
    },

    // 移除任务
    remove(todo) {
      // 保留下来，todos中与传入的todo不一样的任务
      this.todos = this.todos.filter( item => item != todo );
    },

    // 更改查看的任务类型
    choose(type) {
      this.type = type;
    }
  }
});
```

最后执行 npm run build 来打包
