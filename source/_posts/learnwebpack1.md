---
title: 如何使用webpack打包一个库library
date: 2019-12-17 23:16:16
tags: 
	- webpack
categories: 'webpack'
---

日常我们开发了一个库之后，如何打包之后提供给别人使用呢？如果你不清楚，就继续看吧！！！
## 初始化库

```
mkdir library
cd library
npm init -y
```
经过以上步骤后会生成一个library文件夹，里面包含一个package.json文件。然后简单修改为如下所示：

```
{
  "name": "library",
  "version": "1.0.0",
  "description": "",
  "main": "./dist/library.js",
  "scripts": {
    "build": "webpack"
  },
  "keywords": [],
  "author": "rocky",
  "license": "MIT"
}
```
## 简单创建几个文件
在根目录下新建src文件夹，新建一个math.js和string.js。相关文件内容如下：

```
// math.js
export function add(a,b){
    return a+b;
}

export function minus(a,b){
    return a-b;
}

export function multiply(a,b){
    return a*b;
}

export function division(a,b){
    return a/b;
}
```

```
// string.js
export function join(a,b){
    return a+" "+b;
}
```
继续新建一个index.js

```
import * as math from "./math";
import * as string from "./string";

export default {math,string}
```
## 简单安装webpack依赖

```
npm install webpack webpack-cli --save
```
安装的同时，可以创建webpack配置文件webpack.config.js，如下配置：

```
const path = require("path");

module.exports={
    mode:"production",
    entry:"./src/index.js",
    output:{
        path:path.resolve(__dirname,"dist"),
        filename:"library.js",
        library:"library",// 在全局变量中增加一个library变量
        libraryTarget:"umd"
    }
}
```
安装成功后，执行打包命令

```
npm run build
```
之后会在根目录下生成一个dist文件夹，里面包含一个library.js。

## 如何使用呢？
如果别人要使用这个打包后的library.js的话，可能会有如下几种方式：

```
// es6方式
import library from "library"

// commonjs方式
const library=require("library")

// AMD方式
require(["library"],function(){})

// script标签引入
<script src="library.js"></script>
```
在dist文件夹里创建一个index.html，用script引入之前打包生成的library.js。浏览器打开index.html,在控制台中输入library,会得到如下所示的结果：![result](https://user-gold-cdn.xitu.io/2019/12/17/16f144b0f52c0126?w=1612&h=514&f=jpeg&s=97951)
一个简单的库便打包生成了。
注解：webpack中libraryTarget配置项可以设为umd,表示采用umd规范，如果设置为this,表示在this下挂载了一个library变量。更多用法可参考[webpack官网](https://webpack.js.org/configuration/output/#outputlibrarytarget)。

## 引入别的库用法
假设需要引入lodash.安装lodash

```
npm install lodash --save
```
修改之前创建的string.js

```
import _ from "lodash";

export function join(a,b){
    // return a+" "+b;
    return _.join([a,b]," ");
}
```
运行打包命令，发现打包出来的库体积变大了，因为我们引入了lodash,导致包变大。怎么办呢？修改webpack配置文件。
增加一个externals配置项：

```
const path =require("path");

module.exports={
    mode:"production",
    entry:"./src/index.js",
    externals:["lodash"],// 配置不打包文件
    output:{
        path:path.resolve(__dirname,"dist"),
        filename:"library.js",
        library:"library",
        libraryTarget:"umd"
    }
}
```
之后打包就会发现库的体积又变小了。

以上就是一个简单打包库的过程，打包完成后，就可以使用npm相关命令将库发布到npm仓库，发布成功后，就可以让别的小伙伴使用了。当然在实际情况中，打包一个库的话，需要做的还有很多，比如tree-shaking,优化方面的东西，我也正在逐渐学习中！
## 参考资料

* [webpack output libraryTarget](https://webpack.js.org/configuration/output/#outputlibrarytarget)
* [webpack external](https://webpack.js.org/configuration/externals/#externals)