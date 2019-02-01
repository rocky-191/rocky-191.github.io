---
title: webpack4.0配置记录（1）
date: 2019-02-01 17:25:47
tags:
	- webpack
categories: 'webpack'
---

趁着假期闲暇，练习下webpack4.0的一些配置。    



<!--more-->

## webpack4优化压缩js和css方式
```
let UglifyJsPlugin = require("uglifyjs-webpack-plugin");
let OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");
optimization: {//优化项
    minimizer: [
        new UglifyJsPlugin({
            cache: true,
            parallel: true,//并发打包
            sourceMap: true // set to true if you want JS source maps
        }),//开发环境下不压缩js，想启用压缩功能，需要把mode切换为production
        new OptimizeCSSAssetsPlugin({})
    ]
},
```
**注意：若想优化生效，必须将mode改为production模式**  
详情见[npm官网](https://www.npmjs.com/package/mini-css-extract-plugin)  

**expose-loader 暴露全局loader，称为内联loader。到目前为止，有内联loader，普通normal loader，前置loader (pre loader)，后置loader （post loader）**  
## 在项目中引入jquery类似模块方式
webpack.config.js配置

```
let webpack =require('webpack');
plugins:[//存放webpack插件
    new webpack.ProvidePlugin({//在每个模块中注入$
        '$':'jquery'
    })
],
```
## webpack引入基层模块方式
1. expose-loader暴露到全局window上
2. providePlugin给每个模块提供$
3. cdn方式引入不打包，webpack需要配置externals

## 打包文件分类

```
new MiniCssExtractPlugin({
    filename:'css/main.css'
}),
```
将css打包在css文件夹中  

```
{
    test:/(.png|.jpg)$/,
    use:{
        loader:'url-loader',
        options:{
            limit:50*1024,
            outputPath:'images/',
            //publicPath:''
        }
    }
}
```

图片打包路径前配置publicPath即可。

## 生成source-map便于调试，几种不同选项  
(1)增加源码映射文件，便于调试。标示报错文件行和列，大而全文件
```
devtool:'source-map'
```
(2)不会产生单独文件，但是可以显示行和列
```
devtool:'eval-source-map'
```
(3)不会产生列，但是是一个单独的映射文件，用于调试
```
devtool:'cheap-module-source-map'
```
(4)不会产生文件，集成在打包后的文件中，也不会产生列
```
devtool:'cheap-module-eval-source-map'
```

## 监听文件变动，实时打包

```
watch:true,
watchOptions:{//监听选项
    poll:1000,//每秒问我1000次，是否打包
    aggregateTimeout:500,//防抖
    ignored:/node_modules///不需要监控的文件
},
```

## webpack插件应用
1. cleanWebpackPlugin(需要安装依赖模块)
```
new CleanWebpackPlugin('./dist')//先清空dist目录下的文件在打包
```

2. copyWebpackPlugin(需要安装依赖模块)
```
new CopyWebpackPlugin([
    {
        from:'./doc',
        to:'./dist'
    }//可以写多个，拷贝多个目录文件
])
```

3. bannerPlugin(内置插件)
```
//添加版权注释信息
new Webpack.BannerPlugin('make by mgl 2019-2-1')
```
运行打包命令后，可在打包文件中看到注释信息

```
npm run dev
```

## webpack中devServer几种配置
(1)单纯配置跨域代理方式
```
proxy:{
     '/api':{
         target:'http://localhost:3000',
         pathRewrite:{'/api':''}
     }
}
```
(2)前端单纯mock数据
```
before(app){
    app.get('/user',(req,res)=>{
        res.json({name:'mgl-before'});
    })
}
```
(3)有服务端，不用代理来处理，在服务端中启动webpack，用服务端端口
```
//express

let express = require('express');
let webpack=require('webpack');
//引入中间件
let middle=require('webpack-dev-middleware');

let config=require('./webpack.config.js');

let compiler=webpack(config);//webpack处理返回结果

let app=express();
app.use(middle(compiler));

app.get('/user',(req,res)=>{
    res.json({name:'mgl'});
})

app.listen(3000)
```
## resolve属性配置

```
resolve:{//解析第三方模块
    modules:[path.resolve('node_modules')],
    extensions:['.js','.css','.vue','.json']//指定解析后缀名称，从左向右
    // mainFields:['style','main']//指定引入模块的先后顺序
    // mainFiles:[],//指定入口文件的名字，默认是index.js
    // alias:{//配置别名
    //     bootstrap:'bootstrap/dist/css/bootstrap.css'
    // }
},
```
陆续更新中，欢迎关注！