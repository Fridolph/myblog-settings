---
title: 【CLI】CLI开发尝试初体验
date: 2019-02-10 23:49:03
tags:
  - CLI
  - commander
  - chalk
  - inquirer
categories:
  - CLI
---

> 正好看到相关的内容推送，就学习学习，虽然都是用得别人的(vue-cli、create-react-app)，自己了解一下也不错，顺便弄个简单的todo-demo。

<!-- more -->

# CLI 定义

引用自百度百科

> 命令行界面（英语：command-line interface，缩写：CLI）是在图形用户界面得到普及之前使用最为广泛的用户界面，它通常不支持鼠标，用户通过键盘输入指令，计算机接收到指令后，予以执行。也有人称之为字符用户界面（CUI）。
> 通常认为，命令行界面（CLI）没有图形用户界面（GUI）那么方便用户操作。因为，命令行界面的软件通常需要用户记忆操作的命令，但是，由于其本身的特点，命令行界面要较图形用户界面节约计算机系统的资源。在熟记命令的前提下，使用命令行界面往往要较使用图形用户界面的操作速度要快。所以，图形用户界面的操作系统中，都保留着可选的命令行界面。

比如我们使用 @vue/cli 在命令行输入 `vue init helloworld` 出来一堆选项后就帮我们自动生成了项目


    command [subCommand] [options] [arguments]

- command：命令，比如 vue
- [subCommand]：子命令，比如 vue create
- [options]：选项，配置，同一个命令不同选项会有不一样的操作结果，比如 vue -h，vue -v
- [arguments]：参数，某些命令需要使用的值，比如 vue create myApp

选项与参数的区别：选项是命令内置实现，用户进行选择，参数一般是用户决定传入的值

接下来介绍下CLI开发常用到第三方框架

- commander 命令行开发工具
- chalk 命令行样式风格控制器
- inquirer 交互式命令行工具

## commander

### .parse(argv: string[])

解析执行传入的 argv 命令字符串，通常改命令字符串来自用户在命令行的输入，process.argv, commander 同时会默认创建一个 -h, --help 的选项

### .version(str, flags?)

设置版本信息，该方法会自动为命令注册一个 -V,

- --version 的 option
- str：版本号
- flags：指定的 option，默认为："-V, --version"

### .option(flags, description?, fn?, defaultValue?)

设置命令选项

- flags：选项标记名称，"-v, --version"
- description：选项使用说明
- fn：默认值，函数返回值为defaultValue，优先级高于defaultValue
- defaultValue：选项默认值，如果需要的话

选项属性

- flags 中的格式可以接收参数
- `-n, --name [val]`
- `-n, --name <val>` `[]` 可选  `<>` 必填
- 设置成功以后，会在命令对象下增加一个与全局的同名的属性

### .action(fn)

指定命令要执行的动作行为. 该函数执行过程会接收到至少一个参数, 如果命令中带有参数，则是对应的参数列表, 参数的最后一个永远都是 commander 实例

### .command(name, desc?, opts?)

子命令

- name：命令的名称，也可以接受值 'create [appName]'
- desc：简介
- opts：配置

### .description(str) 命令描述

### .alias(str)	设置命令别名

### .usage(str) 设置或获取当前命令的使用说明

## chalk

安装 `npm i chalk / yarn add chalk`

在Node中 `const chalk = require('chalk')`

得到一个 chalk 对象，通过这个对象，我们就可以给控制台中的文字加上各种样式了，就像css一样

### usage

`chalk.<style>[.<style>...](string, [string...])`

- Styles
  - Modifiers 文字修饰：
  - bold Colors 文字颜色：red、green、yellow、blue、cyan
  - Background colors 背景颜色：bgRed、bgGreen、bgYellow、bgBlue、bgCyan

## inquirer

交互式命令，提问用户，收集用户输入数据

安装 `npm i inquirer`

使用

```js
const inquirer = require('inquirer')
inquirer.prompt(questions).then(answers=>{
  // ...
})
```

### questions

- type：提问类型，input, confirm, list, rawlist, expand, checkbox, password, editor
- name：问题名称，供程序后续使用
- message：问题文字，给用户看的
- default：默认值
- choices：选项
- validate：输入验证
- filter：数据过滤

#### input

提出问题，用户输入答案。

可用选项：

- type
- name
- message[, default, filter, validate, transformer]

#### confirm

提出选择，用户选择 Y or N

可用选项：

- type
- name
- message
- [default] default如果提供，必须是 boolean 类型

#### list

用于单选

可用选项：

- type
- name
- message
- choices[, default, filter]

choices为一个数组，数组中可以是简单的字符串，也可以是一个包含了name和value属性的对象，默认选中项为数组中某条数据的下标

#### rawlist

用于单选. 可用选项：

- type
- name
- message
- choices[, default, filter]

choices为一个数组，数组中可以是简单的字符串，也可以是一个包含了name和value属性的对象，默认选中项为数组中某条数据的下标

#### checkbox

用于多选，可用选项：

- type
- name
- message
- choices[, filter, validate, default]

choices 为一个对象数组，对象中 checked 属性 为 true 的表示选中项

### 方法

- validate方法

对用户输入或选择的内容进行验证，返回boolean值，确定提问是否继续
可以返回字符串作为验证失败的提示

- filter方法

对用户输入或选择的内容进行过滤
接受一个参数：用户输入或选择的内容
返回的值将作为过滤后的值

## 练习demo

commander & chalk

```js
/**
 * ls
 *  输出当前运行命令所在的目录下的文件和文件夹
 * ls -p d:\
 *  我们还可以指定要显示的目录
 */

// 加载commander模块
const commander = require('commander');
// 加载fs模块
const fs = require('fs');
// 美化
const chalk = require('chalk');

// 文字的修饰：斜体，加粗重windows下（cmd）下支持不是特别的好
// console.log(chalk.italic('Hello world!') + ' miaov');

// console.log(chalk.bold.red.bgGreen('Hello world!') + ' miaov');

// process.exit();

// 设置当前命令工具的版本
commander.version('v1.0.0', '-v, --version');

// 设置命令选项
commander.option('-p, --path [path]', '设置要显示的目录', __dirname);

//commander.path = undefined;
// console.log(commander.path);
// process.exit();

// 以列表的形式显示，如果选项不接收用户输入的值，那么这个选项将以boolean的形式提供给后面命令使用
commander.option('-l, --list', '以列表的形式显示');

// 实现命令的具体逻辑
commander.action( () => {
    // console.log(commander)
    // return;

    // option中的变量会挂在到当前commander对象的同名属性下
    // console.log(commander.list);

    try {
        const files = fs.readdirSync( commander.path );
        //console.log( files );

        if ( commander.list ) {
            // 如果commander.list为true，以列表的形式显示

            // 通过map生成要显示的数据
            let output = files.map( file => {

                // 文件的扩展信息（除了文件内容以外的信息）
                let stat = fs.statSync( commander.path + '/' + file );
                // console.log(stat.isDirectory());
                // 根据isDirectory()显示不同的文件类型
                return stat.isDirectory() ? chalk.red(`[目录]   ${file}\r\n`) : `[文件]   ${file}\r\n`;

                // return `[${type}] ${file}\r\n`;

            } ).join('');

            console.log(output);
        } else {
            console.log( files );
        }

    } catch(e) {
        // 开发过程中，可以把错误打印出来，实际发布以后应该屏蔽错误信息
        console.log(e);
    }
} );

commander.parse( process.argv );
```

inquirer

```js
const inquirer = require('inquirer');

// 提问用户，与用户进行命令行的交互
// prompt数组中存放一个指定格式的对象，我们称为question对象
inquirer.prompt([

    {
        type: 'input',
        name: 'username',
        message: '请输入你的应用名称',
        default: 'app',
        // 对用户输入的数据或选择的数据进行验证
        validate(val) {
            if (val.trim() === '') {
                return '应用名称不能为空';
            }
            return true;
        },
        // 对用户输入的数据或选择的数据进行过滤
        filter(val) {
            return val.toLowerCase();
        }
    },

    {
        type: 'confirm',
        name: 'useES6',
        message: '是否启用ES6支持',
        default: true
    },
    {
        type: 'list',
        name: 'framework',
        message: '请选择框架：',
        choices: ['Vue', 'React', 'Angular'],
        default: 1
    },

    {
        type: 'checkbox',
        name: 'tools',
        message: '开发工具',
        choices: [
            {
                name: '使用ESLint',
                value: 'eslint',
                checked: true
            },
            {
                name: '使用mocha单元测试',
                value: 'mocha'
            }
        ]
    }

]).then(answers=>{
    console.log(answers);
})
```
