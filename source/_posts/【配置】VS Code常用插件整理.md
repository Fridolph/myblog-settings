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

> 2018年4月更新，转自掘金，作者王仕军。大部分转自这里，不过也有我自己插件，并一起整理进来。

---

[能让你开发效率翻倍的 VSCode 插件配置（上）](https://juejin.im/post/5a08d1d6f265da430f31950e)
[能让你开发效率翻倍的 VSCode 插件配置（中）](https://juejin.im/post/5ad13d8a6fb9a028ce7c0721)

时刻关注下篇~

---

## 用户设置备份

```json
{
  "workbench.iconTheme": "vscode-icons",
  "workbench.colorTheme": "One Monokai",
  "editor.wordWrap": "on",
  "editor.wordWrapColumn": 120,
  "terminal.integrated.shell.windows": "C:\\WINDOWS\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "editor.tabSize": 2,
  "vetur.format.defaultFormatter.html": "js-beautify-html",
  "eslint.autoFixOnSave": true,
  "files.autoSave": "afterDelay",
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "html",
    {
      "language": "vue",
      "autoFix": true
    }
  ],
  "eslint.options": {
    "plugins": [
      "html"
    ]
  },
  "extensions.autoUpdate": true,
  "prettier.singleQuote": true,
  "prettier.semi": true,
  "prettier.eslintIntegration": true,
  "files.eol": "\n",
  "explorer.confirmDragAndDrop": false,
  "files.autoSaveDelay": 5000,
  "fileheader.tpl": "/*\n * @Author: fridolph\n*  @Date: {createTime}\n * @Last Modified by: fridolph\n*  @Last Modified time: {updateTime}\n */\n",
  "fileheader.Author": "fridolph",
  "fileheader.LastModifiedBy": "fridolph",
  "editor.fontSize": 14,
  "editor.lineHeight": 22,
  "editor.lineNumbers": "on",
  "editor.quickSuggestions": {
    "other": true,
    "comments": true,
    "strings": true
  },
  "files.trimFinalNewlines": true,
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
}
```

## VS code插件整理

### 风格

* one monokai 因为我喜欢，我放在第一了
* vscode-icons
* output colorizer 让log文件变得五颜六色


### 风格检查 格式化

* vetur 用vue的必装插件
* prettier 代码格式化，这两个选一个就好
* beautify 代码格式化你值得拥有
* better comments 为注释增加几种风格的标注
* editorconfig for vscode 支持 .editorconfig
* ESlint 代码检查工具不解释了
* html snippets
* javascript code snippets

---

typescript的单独拿出来了，用都可以装

* TSlint 用typescript的可以装
* TypeScript Hero
* Typescript Extension Pack
* typescript importer
* typescript react code snippets
* typescript toolbox
* vue typescript snippets

### 代码片段

* html snippets
* snippets 自己搜啦
* file peek 可以看到对应文件的代码片段
* vscode-js-console-utils 为变量增加console，调试经常用

### 自动补全

* Auto Close Tag，适用于 JSX、Vue、HTML，在打开标签并且键入 `</` 的时候，能自动补全要闭合的标签；
* Auto Rename Tag，适用于 JSX、Vue、HTML，在修改标签名时，能在你修改开始（结束）标签的时候修改对应的结束（开始）标签，帮你减少 50% 的击键；
* Path Intellisense，文件路径补全，在你用任何方式引入文件系统中的路径时提供智能提示和自动完成；
* NPM Intellisense，NPM 依赖补全，在你引入任何 node_modules 里面的依赖包时提供智能提示和自动完成；
* IntelliSense for CSS class names，CSS 类名补全，会自动扫描整个项目里面的 CSS 类名并在你输入类名时做智能提示；
* Emmet，以前叫做 Zen Coding，我发现后，也是爱不释手，可以把类 CSS 选择符的字符串展开成 HTML 标签，VSCode 已经内置，官方介绍文档参见，你需要做的就是熟悉他的语法，并勤加练习；

### 编码效率

* Document This 能够一键给代码中的类、函数加上注释，支持函数声明、函数表达式、箭头函数等
* Embrace 快速的在选中代码两边添加各种引号、括号，不用来回移动光标

|Command|Description|Notation|Suggested Key|
|-------|-----------|--------|-------------|
|extension.embraceParenthesis|Surround with Parenthesis|( )|ctrl+k,e|
|extension.embraceSquareBrackets|	Surround with Square Brackets|	[ ]	|ctrl+k,s	|
|extension.embraceCurlyBrackets|	Surround with Curly Brackets|	{ }	||ctrl+k,l	|
|extension.embraceAngleBrackets|	Surround with Angle Brackets|	< >	|ctrl+k,a	|
|extension.embraceSingleQuotes|	Surround with Single Quotes|	' '	|ctrl+k,q	|
|extension.embraceDoubleQuotes|	Surround with Double Quotes|	" "	|ctrl+k,o|

* Bracket Pair Colorizer 更方便看括号，作用域等
* ECMAScript Quoets Transformer 方便在字符串和变量混搭模式（String Concat）的代码和字符串模板（Template Literals）模式间来回转换，省去手动的移动光标、修改引号等操作
* Code Spell Checker 检查英语拼写是否合法. 选装… 一堆报错
* Code Runner 名副其实的代码运行插件，支持数十种语言，在不离开代码编辑器的前提下通过命令面板可直接执行代码，并查看输出

其他的配置：关于行末的空格、文件末尾的空行，以前需要使用插件来实现，现在直接修改 VSCode 内置配置即可实现：

福利啊，不然每次eslint一堆报错

```json
{
    "files.trimTrailingWhitespace": true,
    "files.insertFinalNewline": true,
    "files.trimFinalNewlines": true
}
```

* path autocomplete 路径补全


### 功能增强

* Color Highlight，识别代码中的颜色，包括各种颜色格式；
* Bracket Pair Colorizer，识别代码中的各种括号，并且标记上不同的颜色，方便你扫视到匹配的括号，在括号使用非常多的情况下能环节眼部压力，编辑器快捷键固然好用，但是在临近嵌套多的情况下却有些力不从心；
* Project Manager，项目管理，让我们方便的在命令面板中切换项目文件夹，当然，你也可以直接打开包含多个项目的父级文件夹，但这样可能会让 VSCode 变慢；
* Settings Sync 基于 Gist 实现 VSCode 用户配置、快捷键配置、已安装插件列表等的备份和恢复功能，配置过程有详细精确的操作步骤文档。生成的备份 Gist 默认是私密的，如果你想设置为共享的，也可以一键切换。
* Git Lens 把 VSCode 结合 Git 的使用体验优化到了极致，能让我们在不离开编辑器，不执行任何命令的情况下知晓光标所在位置代码的修改时间、作者信息等。
* Code Outline 能在单独窗口中列出当源代码中的各种符号，比如变量名、类名、方法名等，并支持快速跳转，有点类似于 Vim 里面的 ctags，翻看老代码、开源项目代码时非常有用。

### 辅助工具

* can i use 浏览器兼容性检查
* Debugger for Chrome 允许像chrome一样在vs code里调试
* partial diff 同名，可以查看文件间的差异
* todo highlight 待做项高亮，且可查看统计
* Version lens 查看package依赖，且帮更新，需要自行npm install
* VSCode Map Preview 地图坐标信息格式文件，就可就行预览
* vscode-fileheader 自行生成file header
