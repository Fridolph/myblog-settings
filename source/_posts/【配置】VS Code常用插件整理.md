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

而插件和调试上来说，简直不要太爽，这里整理和备份下常用的插件，方便每次换电脑时操作，虽然心里都记得清楚了… 换两次电脑，搭两次环境简直不要太爽…


2017年7月10号 …网上找到了 [VSCode拓展插件推荐](https://github.com/varHarrie/Dawn-Blossoms/issues/10)（HTML、Node、Vue、React开发均适用）于是参考这个就好了~  

好不容易码了这么多字，这篇继续更新完吧

---

## 通用配置插件

`One Monokai`
  其实自带的Monokai就挺好，不过写ES6箭头函数参数没颜色不爽，这个和Monokai很相近，细节高亮处理更好，个人推荐

`Material Icon Theme`
  文件图标主题，这个青菜萝卜，选自己看得顺眼的就好

`Path Intellisense`
  自动路劲补全，默认不带这个功能的，必装

`Path Autocomplete`
  和上面个功能差不多，上面个在windows下有时出点小bug，这个当备用了

`Npm Intellisense`
  require 时的包提示（最新版的vscode已经集成此功能有就可以丢了）

`Document this`
  JS的注释模板 （注意：最新版的vscode已经原生支持）

`filesize`
  在底部状态栏显示当前文件大小，点击后还可以看到详细创建、修改时间，效果如下，相信你会喜欢的

`Version Lens`
  package.json文件显示模块当前版本和最新版本

## 项目及规范相关

`fileheader`
  顶部注释模板，可定义作者、时间等信息，并会自动更新最后修改时间

`ESlint`
  与VSCode相容性好，接管原生 js 提示，可以自定制提示规则。

`Project Manager`
  在多个项目之前快速切换的工具

`Easy Less`
  保存less后自动编译成css。同名的还有Easy Sass 自动编译，考拉什么的可以丢啦（命令行什么的虽然高大上但其实还是折腾啊）

`Better Comments`
  编写更加人性化的注释，具体是怎么试了你就知道了

## 前端开发通用插件

`HTML Snippets  `
  超级实用且初级的 H5代码片段以及提示

`HTML CSS Support`
  让 html 标签上写class 智能提示当前项目所支持的样式

`Debugger for Chrome`
  让 vscode 映射 chrome 的 debug功能，静态页面都可以用 vscode 来打断点调试

`jQuery Code Snippets`
  jquery用得多就上唄，不解释

`Can I Use`
  前端都懂的吧？ 写高级特性先查查，免得返工。升级了，现在鼠标指到该属性就可显示兼容了，非常nice

`HTML Less Class Completion`
  写多了就喜欢这些偷懒的插件，其实想变厉害就应该手敲（~误）

`Open in browswer`
  打开浏览器，可以设置多个浏览器很不错

`View in Browser`
  功能同上，备用~ open的好处是可以配置多个浏览器（但windows下有时会崩~~ 待修复吧）

`beautify css/sass/scss/less`
  代码格式化，和formatter什么的自己看着选吧，可惜还未兼容stylus，话说stylus这写着就像格式化过的。。哈哈~~

`Vue 2 Snippets`
  不解释了，写vue可以上

`React-Redux ES6 Snippets`
  有点恐怖… 说真的，效率是快了… 为了装逼我选择放弃


## 辅助工具类

`Auto Close Tag`
  自动关闭 html 标签

`Auto Rename Tag`
  修改 html 标签，自动帮你完成尾部闭合标签的同步修改

`Code Runner`
  运行选中代码段（支持Node）

`Code Spellchecker`
  单词拼写检查（对于英语渣来说很需要啊）

`CodeBing`
  在VSCode中弹出浏览器并搜索，可编辑搜索引擎（节约了点开浏览器的步骤省下几十秒呢）

`Color Highlight`
  颜色值在代码中高亮显示（我没装这个，新版自带了吧）

`Color Picker`
  拾色器（不过我一直用的qq截图里的颜色…… 哈哈哈）

`Emoji`
  在代码中输入emoji

`File Peek`
  根据路径字符串，快速定位到文件

`Code Outline`
  展示代码结构树

`Output Colorizer`
  彩色输出信息

`Partial Diff`
  对比两段代码或文件


至于其他工具类，需要什么搜什么吧，完！