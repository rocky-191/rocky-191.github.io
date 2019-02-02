---
title: webpack4.0配置记录（2）
date: 2019-02-02 10:40:56
tags:
	- webpack
categories: 'webpack'
---

接上一篇[webpack4.0配置记录(1)](https://rocky-191.github.io/2019/02/01/webpackConfig1/#more),继续记录学习webpack配置。

<!--more-->

## 定义环境变量

```
new Webpack.DefinePlugin({//用来定义全局环境变量
    DEV:JSON.stringify('dev'),
    FLAG:'true'
}),
```

## webpack简单优化
1. noParse
```
module:{
    noParse:'/jquery/',//不去解析设置的包所依赖的关系,如jquery
}
```

2. ignorePlugin
```
module:{
    noParse:'/jquery/',//不去解析设置的包所依赖的关系
    rules:[
        {
            test:/\.js$/,
            exclude:/node_modules/,
            include:path.resolve('src'),
            use:{
                loader:'babel-loader',
                options:{
                    presets:[
                        '@babel/preset-env',
                        '@babel/preset-react'
                    ]
                }
            }
        }
    ]
}
```
通过exclude排除和include包含某些模块  
另外也可以使用webpack自带的ignorePlugin插件排除某些包，减少体积。

```
new webpack.IgnorePlugin(/\.\/locale/,/moment/),
```
以上配置忽略了时间格式化moment.js中的语言包  
3. happypack多线程打包
```
let Happypack=require('happypack')
...
module.exports={
    module:{
        noParse:'/jquery/',//不去解析设置的包所依赖的关系
        rules:[
            {
                test:/\.js$/,
                exclude:/node_modules/,
                include:path.resolve('src'),
                use:'Happypack/loader?id=js'
                // use:{
                //     loader:'babel-loader',
                //     options:{
                //         presets:[
                //             '@babel/preset-env',
                //             '@babel/preset-react'
                //         ]
                //     }
                // }
            }
        ]
    },
    plugins:[
        new Happypack({
            id:'js',
            use:[
                {
                    loader:'babel-loader',
                    options:{
                        presets:[
                            '@babel/preset-env',
                            '@babel/preset-react'
                        ]
                    }
                }
            ]
        })
    ]
}
```
4. webpack内置功能  
  (1)tree-shaking  
  (2)scope-hosting  
  **这两项优化只在生产环境下有效**  
5. 抽离公共代码

```
module.exports={
    optimization:{
        splitChunks:{//分割代码块
            cacheGroups:{//缓冲组
                common:{
                    chunks:'initial',
                    minSize:0,//抽离模块最小粒度是0
                    minChunks:2//表示代码块用过2次以上就要抽离
                },
                vendor:{
                    priority:1,//相当于权重，先抽离第三方模块，如果不设置该属性，分割代码块将从上到下，无法抽离第三方模块。
                    test:/node_modules/,
                    chunks:'initial',
                    minSize:0,//抽离模块最小是0
                    minChunks:2//表示用过2次以上就要抽离
                }
            }
        }
    },
}
```
6. 文件热更新

```
devServer:{
    hot:true
},
plugins:[
    new webpack.NamedModulesPlugin(),//打印更新的模块路径
    new webpack.HotModuleReplacementPlugin()//热更新
]
```
7.可以使用dllPlugin动态链接库优化  
DllPlugin 和 DllReferencePlugin提供了以大幅度提高构建时间性能的方式拆分软件包的方法。原理是将特定的第三方NPM包模块提前构建，然后通过页面引入。这不仅能够使得vendor文件可以大幅度减小，同时，也极大的提高了构件速度。网上别的大神有一篇文章写的很详细，可以参考，[传送门](https://github.com/nicejade/vue-boilerplate-template/blob/master/build/webpack.dll.conf.js)。  

以上就是一些自己在学习webpack4.0配置过程中的一些学习记录，写出来和大家分享，如果有错误，还望告知。欢迎关注交流！不要忘了点个赞，谢谢！