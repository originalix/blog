---
title: 在create-react-app中使用sass
date: 2018-04-04 22:00:13
tags: ["React.js", "前端", "Sass"]

---

> Sass（英文全称：Syntactically Awesome Stylesheets）是一个最初由Hampton Catlin设计并由Natalie Weizenbaum开发的层叠样式表语言。Sass是一个将脚本解析成CSS的脚本语言，即SassScript。Sass包括两套语法。最开始的语法叫做“缩进语法”，与Haml类似，使用缩进来区分代码块，并且用回车将不同规则分隔开。而较新的语法叫做“SCSS”，使用和CSS一样的块语法，即使用大括号将不同的规则分开，使用分号将具体的样式分开。通常情况下，这两套语法通过.sass和.scss两个文件扩展名区分开。相信每个前端开发者都对这种css预处理器有所耳闻。

不管你是刚使用Reactjs或者是Reactjs的老司机，你们一定都听说过`create-react-app`这个脚手架，而从`create-react-app`的官方文档中，我们可以看到他们暂时还不支持直接导入LESS或者Sass。但是通过一些配置，我们还是可以从官方脚手架中使用sass/scss/less的。

<!--more-->

首先我们先安装`node-sass`这个组件，我推荐我们国内的coder使用下面的命令行安装

```bash
npm install -g mirror-config-china --registry=http://registry.npm.taobao.org
npm install node-sass
```

然后在自己项目的`package.json`中，将以下行添加到scripts中:

```json
"build-css": "node-sass-chokidar src/ -o src/",
"watch-css": "npm run build-css && node-sass-chokidar src/ -o src/ --watch --recursive",
```

> Note:  在使用不同的预处理器时，请根据预处理的文档替换build-css和watch-css命令。

现在，您可以将src/App.css重命名为src/App.scss并运行npm run watch-css。watch-css将在src子目录中找到每个Sass文件，并在其旁边创建一个相应的CSS文件，在我们的例子中覆盖src/App.css。由于src/app.js仍然 improt src/App.css，所以样式同样成为您的应用程序的一部分。您现在可以编辑src/App.scss，同时会生成相应的src/App.css。

为了能一边编译sass，一边运行我们的前端项目，我们还需要`npm-run-all`这个工具，这是一个并行运行多个npm脚本的脚手架工具，安装方式也非常简单。

```bash
$ npm install npm-run-all --save-dev
# or
$ yarn add npm-run-all --dev
```

最后，在不使用`ejec`命令的情况下，更改`create-react-app`的`webpack`配置，我们使用`react-app-rewired`来处理，安装方式如下:

```bash
$ npm install react-app-rewired --save-dev
```

在完成这些步骤之后，我们修改`package.json`的`script`内容，让sass一边编译，一边跑着我们的前端项目，实现热更。

```json
"scripts": {
	"build-css": "node-sass src/ -o src/",
	"watch-css": "npm run build-css && node-sass src/ -o src/ --watch --recursive",
	"start-js": "node scripts/start.js",
	"start": "npm-run-all -p watch-css start-js",
	"build": "npm run build-css && node scripts/build.js",
	"test": "node scripts/test.js --env=jsdom"
},
```

`scripts`的命令如上所述，安装完之后，`npm start`就可以搞定sass的使用问题了。

现在运行run npm和npm run build同样构建了Sass文件。
