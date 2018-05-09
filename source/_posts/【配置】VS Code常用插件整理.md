---
title: 【配置】VS Code常用插件整理
date: 2017-02-20 09:51:45
tags: 
  - vscode
categories: 
  - 配置
---

Visual Studio Code是个牛逼的编辑器，启动非常快，我已经抛弃WebStorm Sublime了。
支持各种语言，相比其他IDE，轻量级完全可配置还集成Git感觉非常的适合前端开发。 所以我仔细研究了一下文档未来可能会作为主力工具使用。

虽然习惯了N久的快捷键，没法，习惯什么的是吧，习惯下就好了所以 [官方快捷键大全](https://code.visualstudio.com/docs/customization/keybindings) ，看看这个就好，好在现在快捷键熟悉得差不多了，不会比sublime慢多少

<!-- more -->

# 前言

而插件和调试上来说，简直不要太爽，这里整理和备份下常用的插件，方便每次换电脑时操作，虽然心里都记得清楚了… 换两次电脑，搭两次环境简直不要太爽…

2017年7月10号 …网上找到了 [VSCode拓展插件推荐](https://github.com/shenzekun/shenzekun.github.io/issues/1)（HTML、Node、Vue、React开发均适用）于是参考这个就好了~

好不容易码了这么多字，这篇继续更新完吧

---

## 常用插件

### Auto Close Tag

自动添加HTML / XML关闭标签（必备）

### Auto Rename Tag

自动重命名配对的HTML / XML标签(必备)





### Beautify

格式化javascript，JSON，CSS，Sass，和HTML



### Bootstrap 4 & Font awesome snippets

包含Bootstrap 4＆Font awesome 的代码片段



### Bracket Pair Colorizer

颜色识别匹配括号



### Class autocomplete for HTML

智能提示HTML class =“”属性（必备）

C### ode Runner

非常强大的一款插件，能够运行多种语言的代码片段或代码文件：C，C ++，Java，JavaScript，PHP，Python，Perl，Ruby，Go等等，安装完成后，右上角出现：

点击这个按钮就可以运行你的文件了（必备）



### css peek

能够查看CSS ID和类的字符串作为HTML文件中相应的CSS定义（必备）

使用方法：将光标放在class里面的属性，右击


### Dash

查文档必备，搭配 dash（不过似乎只有 mac 版） :grin:

快捷键 ctrl + h 它根据你当前选中的语言查找 dash 里面的文档



### Debugger for Chrome

让 vscode 映射 chrome 的 debug功能，使静态页面都可以用 vscode 来打断点调试

简单使用：戳我

### Document This

添加注释块



设置：

 "docthis.includeAuthorTag": true,//出现@Author
 "docthis.includeDescriptionTag": true,//出现@Description
 "docthis.authorName": "shenzekun",//作者名字
快捷键： 按两次Ctrl+alt+d

### ESLint

EsLint可以帮助我们检查Javascript编程时的语法错误。比如：在Javascript应用中，你很难找到你漏泄的变量或者方法。EsLint能够帮助我们分析JS代码，找到bug并确保一定程度的JS语法书写的正确性。

配置：戳我

### Font-awesome codes for html

用于 html 的Font-awesome代码片段

### filesize

在底部状态栏显示当前文件大小，点击后还可以看到详细创建、修改时间



### Git History

以图表的形式查看git日志


使用 command+shift+p（Ctrl+shift+p） 输入git log就可以看到了

### Git Lens

git 日志插件



### HTML CSS Support

在 html 标签上写class 智能提示当前项目所支持的样式（必备）

### HTML Snippets

html 代码片段（必备）

### htmlhint

html代码检测



### htmltagwrap

可以在选中HTML标签中外面套一层标签



使用：选择一大段代码，然后按“Alt + W”

### Indenticator

突出目前的缩进深度



### IntelliSense for CSS class names

智能提示 css 的 class 名

### JavaScript (ES6) code snippets

es6代码片段（必备）



### JavaScript Snippet Pack

js代码片段（必备）

### jQuery Code Snippets

jQuery 代码片段

### Live Sass Compiler

实时编译 sass ,不过需要配置，附上我的配置

"liveSassCompile.settings.formats":[
        // You can add more
        {
            "format": "compressed",//压缩
            "extensionName": ".min.css",//编译后缀名
            "savePath": "./css"//编译保存的路径
        }
    ],
使用



### markdownlint

markdown 语法检查

### Node.js Modules Intellisense

可以在导入语句中自动完成JavaScript / TypeScript模块。



### npm Intellisense

在导入语句中自动填充npm模块,跟Node.js Modules Intellisense差不多

### open in browser

当前的 html 文件用浏览器打开，类似 webstorm 的那四个小浏览器图标功能，前提条件html 文件必须保存

快捷键alt+b

### Output Colorizer

输出提示的文字颜色有一些变化，方便获取关键信息


### Path Intellisense
路径自动补全（必备）

### Prettier
格式化JavaScript / TypeScript / CSS 。

### Project Manager
工程项目过多时，shift+cmd+p(shift+ctrl+p) 然后输入project，第一次选择edit Project编辑自己的工程项目，之后就可以直接选择open打开你的项目

### Sass
写 sass 必备

### vscode-faker
生成假数据，地址，电话，图片等等


打开方式shift+cmd+p(shift+ctrl+p)) 然后输入faker 就可以选择了

### Quokka.js
实时观看 javascript 的变量的变化



使用：先shift+cmd+p （ctrl+shift+p）输入 quokka 选择 new javascript 就行了 :grinning:

### Regex Previewer
测试正则的插件



### TSLint
检查typescript编程时的语法错误语法

### TypeScript Importer
自动搜索工作区文件中的TypeScript定义，并将所有已知符号作为完成项，以允许代码完成。



### vscode-icons
目录树图标

r### eact
React-Native/React/Redux snippets for es6/es7
react代码片段，下载人数超多 :wink:



### react-beautify
格式化 javascript, JSX, typescript, TSX 文件

### vue
vetur
语法高亮、智能感知

V### ueHelper
vue代码片段

### Vue TypeScript Snippets
vue的 typescript 代码片段

### Vue 2 Snippets
vue 2代码片段

### One Dark Pro
这个也好看 :smile:


大家还有什么好的插件推荐吗 