---
title: webpack打包性能优化之路
date: 2019-01-12 17:24:46
tags:
	- webpack
	- vue
categories: 'webpack'
---

性能优化的路没有穷尽，只有更快。打开页面越快越好，点击响应越快越好。在当今这个以快为主的时代，快才是王道。闲话扯完，说正事！！！

<!--more-->

该优化方案以最近做的一个hybrid webapp为实例演示。
## 路由懒加载
（1）vue-router文件中的router使用懒加载方式。如下图所示

![](https://user-gold-cdn.xitu.io/2019/1/12/168411d45d93032f?w=1676&h=430&f=png&s=65531)

（2）在vue文件中，也采用类似方式引入其他vue组件

```
const showImage = () => import('@/components/common/showImage');
```
这个优化的方式在vue官网也有介绍，[传送门](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)
## 启用gzip压缩和关闭sourcemap
所有现代浏览器都支持 gzip 压缩并会为所有 HTTP 请求自动协商此类压缩。启用 gzip 压缩可大幅缩减所传输的响应的大小（最多可缩减90%），从而显著缩短下载相应资源所需的时间、减少客户端的流量消耗并加快网页的首次呈现速度。 如下图所示

![](https://user-gold-cdn.xitu.io/2019/1/12/16841251a560cae3?w=1332&h=750&f=png&s=156357)
如果你使用的是vue-cli2生成的项目的话，在config文件夹下的index.js中可以找到这段代码。记得开启gzip压缩前要安装一个插件，如途中注释掉的一段代码所示。
## 生产环境去掉console代码，减少代码体积，使用uglifyjs压缩代码

![](https://user-gold-cdn.xitu.io/2019/1/12/16841297bfb5d2d3?w=1510&h=1266&f=png&s=288261)
## 图片优化
对于网页来说，在所下载的字节数中，图片往往会占很大比例。因此，优化图片通常可以卓有成效地减少字节数和改进性能：浏览器需要下载的字节数越少，对客户端带宽的争用就越少，浏览器下载内容并在屏幕上呈现内容的速度就越快。  
尽量减少图片的使用，或者使用css3来代替图片效果。如果不行的话，小图片通过一定的工具合成雪碧图或者转成base64。
## 引用的库尽量按需加载。
(1)像一般的ui库element，vant等库都提供来按需加载的方式，避免全部引入，加大项目体积。
(2)以cdn方式载入需要的库，也可以减少打包后的体积。
在index.html文件中

引入mintui
```
<!-- 引入样式 -->
<link rel="stylesheet" href="https://unpkg.com/mint-ui/lib/style.css">
<!-- 引入组件库 -->
<script src="https://unpkg.com/mint-ui/lib/index.js"></script>
```
引入vue

```
<!-- 开发环境使用此方案-->
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
<!-- 生产环境使用此方案 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.min.js"></script>
```
以这种外链方式引入mint-ui和vue后，需要做些别的配置  
（1）在入口文件main.js 中就不需要引入vue和mintui了  
（2）在build\webpack.base.conf.js中添加如下配置，意为打包的时候不打包vue和mint-ui。

```
externals:{
  "mint-ui":"mintui",
  "vue":"Vue"
},
```
## 使用DllReferencePlugin
将平时不经常变动的文件抽离出来，统一打包，这样也可以减少后续打包的时间。

* 在build文件夹中新建一个webpack.dll.conf.js.
```
const path = require('path')
const webpack = require('webpack')
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin')
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
  mode: process.env.NODE_ENV === 'production' ? 'production' : 'development',
  entry: {
    vendor: [
        //根据实际情况添加
      'axios',
      'vue/dist/vue.min.js',
      'vue-router',
      'vuex',
      'mint-ui'
    ]
  },
  output: {
    path: path.resolve(__dirname, '../static/js'),
    filename: '[name].dll.js',
    library: '[name]_library'
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules\/(?!(autotrack|dom-utils))/
      }
    ]
  },
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        cache: true,
        parallel: true,
        sourceMap: false // set to true if you want JS source maps
      }),
      // Compress extracted CSS. We are using this plugin so that possible
      // duplicated CSS from different components can be deduped.
      new OptimizeCSSAssetsPlugin({})
    ]
  },
  plugins: [
    /*
      @desc: https://webpack.js.org/plugins/module-concatenation-plugin/
      "作用域提升(scope hoisting)",使代码体积更小[函数申明会产生大量代码](#webpack3)
    */
    new webpack.optimize.ModuleConcatenationPlugin(),
    new webpack.DllPlugin({
      path: path.join(__dirname, '.', '[name]-manifest.json'),
      name: '[name]_library'
    })
  ]
}

```

* 在package.json中增加配置
```
"scripts": {
    "build:dll": "webpack -p --progress --config build/webpack.dll.conf.js"
  }
```
执行npm run build:dll命令就可以在根目录下生成vendor-manifest.json,static/js下生成vendor.dll.js

* 在webpack.base.conf.js中增加如下
```
const manifest = require('../vendor-manifest.json')

....

plugins: [
   //把dll的vendor-manifest.json引用到需要的预编译的依赖
   new webpack.DllReferencePlugin({
     manifest
   })
]
```

* 在index.html底部添加
```
<script src="./static/js/vendor.dll.js"></script>
```
目前在项目中做的优化就是这些，还是那句话，性能优化的路没有穷尽，只有更快。

## 参考文章

（1）https://blog.csdn.net/blueberry_liang/article/details/80320607  
（2）https://developers.google.com/speed/docs/insights/rules?csw=1  
（3）https://www.jeffjade.com/2017/03/11/120-how-to-write-vue-better/