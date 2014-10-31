# 前端架构分析说明文档

前海人寿官方商城项目技术选型主要侧重于三个方面：

1. 前端数据展现和后端逻辑分离，后端数据以数据API的方式提供
2. 开发过程中的自动化构建，例如开发和产品环境分离，CSS 的预编译等
3. 系统的高性能和可扩展性

## 技术架构和技术栈

基于以上侧重点，官方商城项目前端技术架构总结如下：

_TODO: 说明图_

### Express

> 标签: 前后端分离，自动化构建，高性能和可扩展性

Express 是托管在 Github 上基于 NodeJS 的 热门的开源 MVC 框架。依托长时间的开源社区积累，Express 已经成为 NodeJS 生态圈中非常成熟的开发框架。基于 NodeJS 平台在处理大量并发请求的优势，Express 框架的性能表现不俗，而且因为使用的是 JavaScript 语言，所以前后端代码复用性很强。

Express 在此架构中起到的是与其他系统 API 连接的作用，所以从前端架构角度来看，它是其中一个部分。这样一来，后端开发者（例如 Java 应用开发者）只用专注在**业务逻辑**和**数据API**。

相关文档链接：

* 官网 http://expressjs.com/
* 开源中国社区 http://www.oschina.net/p/expressjs
* 淘宝前后端分离分析系列，共6篇 http://ued.taobao.org/blog/2014/04/full-stack-development-with-nodejs/

### 包管理工具 Bower 和 NPM

> 标签: 自动化构建，可扩展性

后端 Java 应用程序都会有成熟的依赖包管理，随着 NodeJS 的成熟和流行，前端 UI 依赖包可通过Bower 来管理，而后端的依赖包（NodeJS包）可以通过 NPM 来管理。

相关文档链接：

* Bower 官网 http://bower.io/
* NPM 官网 https://www.npmjs.org/
* 一个比较详细的 Bower 使用介绍 http://blog.fens.me/nodejs-bower-intro/

### Swig 模板

> 标签: 前后端分离，自动化构建

因为需要考虑到复用之前代码，目前系统中推荐使用 swig 作为页面模板，它带来了以下好处：

* HTML 代码可以直接进行复用
* 支持强大的模板继承和重载功能，有利于灵活地组织页面模板

相关文档链接：

* Swig on Github https://github.com/paularmstrong/swig/
* Swig 例子 https://github.com/paularmstrong/swig/tree/master/examples

### Stylus 预编译语言

> 标签: 自动化构建

Stylus 是目前在 NodeJS 平台上最好的 CSS 预编译语言，支持的功能包括：变量定义、Mixin、导入（`import`）、依赖（`require`）、CSS 按层级编写。

同时，Stylus 的代码容错性很强，编写起来速度很快，质量很高。在配合第三方帮助包使用时，扩展性表现不错。目前使用到的第三方帮助包有：

* nib 推荐使用一个常用的 Mixin 的集合，比如引入后可以不加浏览器前缀（比如`-webkit-..`）的情况下使用 CSS3 的属性 https://visionmedia.github.io/nib/
* jeet 推荐使用一个通过属性定义而非类名（`class 属性为 span5`）的方式来使用 Grid 栅格系统 http://jeet.gs/
* rupture 推荐使用 CSS3 响应式布局帮助类库 https://github.com/jenius/rupture

相关文档链接：

* 官网 https://learnboost.github.io/stylus/
* 在线实验 https://learnboost.github.io/stylus/try.html

### Grunt 任务管理

> 标签: 自动化构建

前端需要处理的任务愈来愈多，包括CSS的预编译、资源的拷贝、模板的预编译、开发环境和产品环境区分、打包上传、监听文件修改、运行测试等等，从而需要一种统一的方式来管理这些任务。Grunt 是目前用来管理和运行任务的工具中最好的一个，现有的任务包在 http://npmjs.org 上数不甚数。

使用 Grunt 可以帮助我们自由组合任务，从而来完善前端代码开发过程中的自动化。

相关文档链接：

* Grunt 官网 http://www.gruntjs.org/
* 现有的 Grunt Plugins 列表 http://www.gruntjs.org/plugins.html

### Livereload 刷新

> 标签：自动化构建

Livereload 顾名思义就是用来刷新页面的。当监听到配置好的文件有修改时，浏览器会自动的进行刷新。Livereload 只在**开发模式**下启用。

相关文档链接：

* 官网 http://livereload.com/
* on Github https://github.com/livereload
* 如何和 Express 做集成 https://github.com/intesso/connect-livereload#connectexpress-example

### 开发模式和产品模式

> 标签: 自动化构建，高性能和可扩展性

使用 `npm start` 可以开启开发模式，启用开发模式下的功能。同时，也可通过使用 `grunt build` 就可以把现有系统整理为产品模式，然后使用 `NODE_ENV=production PORT=80 node bin/www`。具体内容请参考下文“开发模式下运行”和“产品模式下运行”。

### 前端自动化构建任务说明

#### grunt clean

使用的是 [grunt-contrib-clean](https://github.com/gruntjs/grunt-contrib-clean) 插件，在此架构中的目的是在每次构建之前，清空之前生成的过期资源，保证新的运行环境是干净的。

目前这个任务会清理 `dev`，`public` 和临时文件夹 `.temp`。

#### grunt concat

使用的是 [grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat) 插件，用于合并生成产品模式下的 JS（`.min.js`） 和 CSS（`.min.css`） 文件。

```
css: {
  files: {
    '.temp/styles/index.min.css': '.temp/styles/**/*.css'
  }
}
```

指的是 .temp/styles/ 文件夹下的所有 CSS 文件合并成一个文件叫做  `index.min.css`。

#### grunt cssmin

使用的是 [grunt-contrib-cssmin](https://github.com/gruntjs/grunt-contrib-cssmin) 插件，用于最小化（minify）CSS 文件。一般用于 `grunt concat` 之后，这样合并之后的 CSS 代码就可以压缩到最小。

#### grunt uglify

使用的是 [grunt-contrib-uglify](https://github.com/gruntjs/grunt-contrib-uglify) 插件，用于最小化（minify）和代码优化 JS 文件。一般用于 `grunt concat` 之后，这样合并之后的 JS 代码进行变量优化，从而文件可以被压缩到最小。

#### grunt stylus

使用的是 [grunt-contrib-stylus](https://github.com/gruntjs/grunt-contrib-stylus) 插件，用于把 Stylus 文件编译为 CSS 文件。如果开启了文件监听 `grunt watch` ，此任务在文件修改的时候会自动更新为 CSS。

```
use: [
  require('rupture'),
  require('jeet'),
  require('nib')
]
```

代码指的是，在编译 Stylus 为 CSS 代码时，会自动引入第三方的 `rupture` 和 `jeet`  hunli的依赖包。

#### grunt watch

使用的是 [grunt-contrib-watch](https://github.com/gruntjs/grunt-contrib-watch) 插件，用于开启文件监听，目前开始监听 styles/scripts/views 三个文件夹下的文件。开发模式下，当文件修改时，会触发 CSS 的重新编译和其他资源文件的准备。

#### grunt nodemon

使用的是 [grunt-nodemon](https://github.com/ChrisWren/grunt-nodemon) 插件。在默认情况下，NodeJS 的运行程序不支持动态的加载，所以在开发模式下需要 nodemon 来帮助我们在文件修改后，自动地重新启动服务。

#### grunt concurrent

使用的是 [grunt-concurrent](https://github.com/sindresorhus/grunt-concurrent) 插件。由于 Grunt 是穿行运行任务，当上一个任务结束之后才会运行下一个，所以当遇到需要运行两个需要持续运行的任务，比如 `watch` 和 `nodemon` 时，就需要用到 `concurrent` 任务。

运行时的任务的命令行输出会统一显示在 `concurrent` 任务的命令行输出中。

## 安装步骤

### 准备开发环境

安装基础的 [Git](http://git-scm.com/book/zh/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git) 和 [NodeJS](http://nodejs.org/)，NodeJS的版本选择 **0.10.20** 以上。安装之后可以确认一下命令可用。

```bash
$ git
$ node
$ npm
```

安装项目搭建需要的基础工具包

```bash
$ npm install -g grunt-cli
$ npm install -g bower
```

### 下载代码库

```bash
$ git clone git@git.oschina.net:winsonwq/qhlife-web.git
```

下载完成之后进入项目目录

```bash
$ cd qhlife-web
```

### 安装依赖包

安装前端 bower 依赖包。

```bash
$ bower install
```

安装 npm 依赖包。

```bash
$ npm install
```

### 开发模式下运行

使用以下命令进入开发模式。

```bash
$ npm start
// or
$ NODE_ENV=development grunt dev
```

开发模式所需的资源文件都会位于项目根目录的 `dev` 临时文件夹中，进入开发模式后会开启以下功能：

* Stylus 到 CSS 代码的预编译，当修改Stylus代码时，会自动编译 CSS 代码
* Livereload 功能，当 Stylus 文件、模板文件、JavaScript 文件修改时，浏览器会自动刷新显示适配最终效果

### 产品模式下运行

先试用以下命令生成产品环境所需资源，所需的资源都会位于项目根目录下的 `public` 文件夹中。

```bash
$ grunt build
```

其中资源会包括：

* 压缩之后的JavaScript 和 CSS，比如 `index.min.js` 和 `index.min.css`
* 二次优化 JavaScript 代码，使之更精简，使用的是 [UglifyJS](https://github.com/mishoo/UglifyJS2)
* 系统所需的第三方库，比如 [jQuery](https://jquery.com/) 或者 [purecss](http://purecss.io/)
* 系统所需的图片或者字体资源

产品环境资源准备完成之后，运行以下命令运行产品代码。

```bash
$ NODE_ENV=production PORT=80 node bin/www
```

## 架构验收

前端技术架构已经搭建起来，已经有两个关键点做了技术验证：

* 现有商城 Profile 页面的个人信息页面实现（数据是模拟的）。已有的 Profile 页面使用静态 HTML 实现，数据请求用的是基础的 Ajax 请求。目前使用新的结构，新的开发方式快速实现了页面和样式，还做了数据演示的优化，放弃了之前延迟加载的方式，目前速度更快了。

## 项目文件结构说明
<pre>
.
+-- .temp -> 临时文件夹，用来存放编译，连接，压缩css和js文件过程中，产生的中间文件
+-- bin
|	+-- www -> 项目启动脚本，由nodeJs创建监听进程
+-- config -> 配置文件夹，内存放不同环境配置
|	+-- config.development.js -> 开发环境的配置信息
|	+-- config.global.js -> 共享的配置信息
|	+-- config.test.js -> 测试环境的配置信息
|	+-- index.js -> 根据不同参数，加载不同的配置信息
+-- dev -> 存放编译后的静态图片和文件（js，css），开发环境使用
+-- images -> 存放图片
+-- lib -> 存放第三方的js资源(第三方js文件一般用bower管理，这里存放需要对源文件进行自定义修改的js文件)
|	+-- cas.js -> cas验证模块
+-- models -> 存放mvc中的模型，存放业务逻辑
|	+-- cas
|	|	+-- index.js -> cas验证的业务逻辑
|	+-- policy
|	|	+-- index.js -> 获取保单信息业务逻辑
|	+-- profile
|	|	+-- index.js -> 获取用户信息业务逻辑
|	+-- index.js 
+-- node_modules -> nodeJs模块
+-- routes -> 路由信息
|	+-- cas -> cas相关的路由信息以及对应的挑战逻辑
|	+-- main -> 主页
|	+-- product 
|	|	+-- index.js -> 产品相关的路由信息
|	|	+-- middleware.js -> 利用中间件实现后端数据的验证
|	+-- profile -> 用户信息相关的路由
|	+-- index.js
+-- scripts -> 存放js文件
+-- styles -> 存放styl文件
+-- tasks -> 存放grunt任务配置信息 
|	+-- clean.js -> 存放clean任务的配置信息
|	+-- concat.js -> 存放concat任务的配置信息
|	+-- ... 
+-- test -> 测试文件夹
|	+-- routes -> 测试路由逻辑
|	|	+-- home_spec.js -> mocha测试js逻辑
+-- vendors -> bower将下载的第三方库放在此文件夹中
+-- views -> 存放所有的html模板
+-- .bowerrc -> bower的配置文件
+-- .gitignore -> git忽略配置文件
+-- app.js -> express网站配置文件
+-- bower.json -> bower的依赖管理配置
+-- gruntfile.js -> grunt的配置文件
+-- package.json -> npm的配置文件
+-- README.md -> readme的markdown文件
</pre>


