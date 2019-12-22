---
title: 通过bundler学习webpack模块依赖分析
date: 2019-12-22 13:36:54
tags: 
	- webpack
categories: 'webpack'
---

你知道webpack是如何分析模块各个依赖关系的？是如何将ES6代码编译成浏览器可执行代码的吗？
## 项目初始化

* 创建文件夹

```
mkdir bundler
cd bundler
```

* 创建相关文件
  在bundler中创建src文件夹，在src文件夹新建index.js,message.js,word.js。文件内容如下：

```
// word.js
export const word="word";

// message.js
import {word} from "./word.js";
const message=`hello ${word}`;
export default message;

// index.js
import message from "./message.js";
console.log(message);
```
如果想直接在浏览器中运行index.js的话，当然是不能的，浏览器无法识别es6的语法，以前我们都是通过类似webpack的打包工具将es6代码转换成es5的代码，然后直接在浏览器中运行。
## 入口文件依赖分析
在项目根目录下新建一个bundler文件，实现打包过程。其实所谓的webpack编译打包就是通过一些特定的方法函数将源代码转换成浏览器可识别的代码

* 定义一个模块分析函数
```
const moduleAnalyser=(filename)=>{

}
moduleAnalyser("./src/index.js");// 入口函数
```

* 读取文件内容
  这里使用了node中的一个核心模块fs。

```
const fs=require("fs");

const moduleAnalyser=(filename)=>{
    const content=fs.readFileSync(filename,"utf-8");// 读取文件内容
    console.log(content);
}
moduleAnalyser("./src/index.js");// 入口函数
```
在终端中执行node命令

```
node bundler.js
```
就会输出index.js的文件内容
![](https://user-gold-cdn.xitu.io/2019/12/22/16f2b501f84a10a6?w=1164&h=306&f=jpeg&s=36680)

* 解析文件依赖  

（1）执行npm init -y初始化

（2）安装一个babel模块

```
npm install @babel/parser --save
```
（3）使用parser

```
const fs=require("fs");
const parser=require("@babel/parser");

const moduleAnalyser=(filename)=>{
    const content=fs.readFileSync(filename,"utf-8");// 读取文件内容
    console.log(parser.parse(content,{
        sourceType:"module"
    }));
}

moduleAnalyser("./src/index.js");// 入口函数
```
再次执行node命令，node bundler.js，查看文件内容,输出的就是常说的AST,描述了文件的相关依赖关系。
![](https://user-gold-cdn.xitu.io/2019/12/22/16f2b6380de1e667?w=1584&h=982&f=jpeg&s=133569)

修改一下bundler

```
...
const ast=parser.parse(content,{
        sourceType:"module"
    })
    console.log(ast.program.body);
...
```
执行node bundler.js命令就会得到如下输出内容![](https://user-gold-cdn.xitu.io/2019/12/22/16f2b6aced3c5014?w=1616&h=1284&f=jpeg&s=183744)
输出的就是文件相关依赖，type为ImportDeclaration表示是引入声明，type为ExpressionStatement表示是表达式。接下来要做的就是遍历body的内容得到依赖关系。  
（4）安装模块

```
npm install @babel/traverse --save
```
（5）使用traverse

```
...
traverse(ast,{
        ImportDeclaration({node}){
            console.log(node)// 查看node内容
        }
    })
...
```
继续改写bundler.js

```
const dependencies=[];
    traverse(ast,{
        ImportDeclaration({node}){
            dependencies.push(node.source.value);
        }
    })
    console.log(dependencies)// 得到依赖数组
```
继续改写bundler.js

```
const dependencies={};// 变成对象，key是依赖路径，value是相对依赖路径。便于之后使用
    traverse(ast,{
        ImportDeclaration({node}){
            const dirname=path.dirname(filename);//filename对应的文件夹路径
            const newFile="./"+path.join(dirname,node.source.value);
            dependencies[node.source.value]=newFile;
        }
    })
```
（6）安装babel/core转换代码

```
npm install @babel/core @babel/preset-env --save
```
（7）转换代码

```
const { code }=babel.transformFromAst(ast,null,{
        presets:["@babel/preset-env"]
    })//转换ast
    console.log(code);
```
执行node bundler.js命令就会得到如下输出内容
![](https://user-gold-cdn.xitu.io/2019/12/22/16f2b8bbd776fa20?w=2076&h=444&f=jpeg&s=84412)

入口文件的依赖分析就完成了。完整代码如下：

```
const fs=require("fs");
const path=require("path");
const babel=require("@babel/core");
const parser=require("@babel/parser");
const traverse=require("@babel/traverse").default;// 默认es module导出

const moduleAnalyser=(filename)=>{
    const content=fs.readFileSync(filename,"utf-8");// 读取文件内容
    const ast=parser.parse(content,{
        sourceType:"module"
    })
    const dependencies={};
    traverse(ast,{
        ImportDeclaration({node}){
            const dirname=path.dirname(filename);//filename对应的文件夹路径
            const newFile="./"+path.join(dirname,node.source.value);
            dependencies[node.source.value]=newFile;
        }
    })
    const { code } = babel.transformFromAst(ast,null,{
        presets:["@babel/preset-env"]
    })//转换ast
    return {
        filename,
        dependencies,
        code
    }
}

const moduleInfo=moduleAnalyser("./src/index.js");// 入口函数
console.log(moduleInfo);
```
## 构建依赖图谱
一个项目不可能只有一个文件，这就需要我们分析整个项目的依赖关系，即生成得到依赖图谱。

* 定义生成依赖图谱方法

```
const makeDependenciesGraph=(entry)=>{
    const entryModule=moduleAnalyser(entry);
    console.log(entryModule);
}
```

* 从入口开始，循环递归分析依赖关系

```
const makeDependenciesGraph=(entry)=>{
    const entryModule=moduleAnalyser(entry);
    const graphArray=[entryModule];
    for(let i=0;i<graphArray.length;i++){
        const item=graphArray[i];
        const { dependencies } = item; // 解构出依赖
        if(dependencies){
            for(let j in dependencies){
                // 递归分析依赖,放入依赖图谱数组
                graphArray.push(moduleAnalyser(dependencies[j]))
            }
        }
    }
    console.log(graphArray);
}
```
执行node bundler.js命令就会得到如下输出内容![](https://user-gold-cdn.xitu.io/2019/12/22/16f2be107bc7d1cc?w=2336&h=828&f=jpeg&s=310778)

* 生成依赖图谱graph对象，完整代码如下：

```
const makeDependenciesGraph=(entry)=>{
    const entryModule=moduleAnalyser(entry);
    const graphArray=[entryModule];
    for(let i=0;i<graphArray.length;i++){
        const item=graphArray[i];
        const { dependencies } = item; // 解构出依赖
        if(dependencies){
            for(let j in dependencies){
                // 递归分析依赖,放入依赖图谱数组
                graphArray.push(moduleAnalyser(dependencies[j]))
            }
        }
    }
    // 转换为对象 便于使用
    const graph={}
    graphArray.forEach(item=>{
        graph[item.filename]={
            dependencies:item.dependencies,
            code:item.code
        }
    });
    return graph;
}
```
## 生成浏览器可识别代码

```
const generateCode=(entry)=>{
    const graph=JSON.stringify(makeDependenciesGraph(entry));
    return `
        (function(graph){
            function require(module){
                function localRequire(relativePath){
                    return require(graph[module].dependencies[relativePath])
                }
                var exports={};
                (function(require,exports,code){
                    eval(code)
                })(localRequire,exports,graph[module].code);
                return exports;
            };
            require('${entry}');
        })(${graph})
    `;
}
```
执行node bundler.js命令就会得到如下输出内容![](https://user-gold-cdn.xitu.io/2019/12/22/16f2c0a097ac14e4?w=2336&h=1010&f=jpeg&s=357701)
将输入内容拷贝后到浏览器console中执行，便会得到代码的正常输出![](https://user-gold-cdn.xitu.io/2019/12/22/16f2c0c48d6fb103?w=3346&h=684&f=jpeg&s=412233)

bundler文件完整代码：

```
const fs=require("fs");
const path=require("path");
const babel=require("@babel/core");
const parser=require("@babel/parser");
const traverse=require("@babel/traverse").default;// 默认es module导出

const moduleAnalyser=(filename)=>{
    const content=fs.readFileSync(filename,"utf-8");// 读取文件内容
    const ast=parser.parse(content,{
        sourceType:"module"
    })
    const dependencies={};
    traverse(ast,{
        ImportDeclaration({node}){
            const dirname=path.dirname(filename);//filename对应的文件夹路径
            const newFile="./"+path.join(dirname,node.source.value);
            dependencies[node.source.value]=newFile;
        }
    })
    const { code }=babel.transformFromAst(ast,null,{
        presets:["@babel/preset-env"]
    })//转换ast
    return {
        filename,
        dependencies,
        code
    }
}

const makeDependenciesGraph=(entry)=>{
    const entryModule=moduleAnalyser(entry);
    const graphArray=[entryModule];
    for(let i=0;i<graphArray.length;i++){
        const item=graphArray[i];
        const { dependencies } = item; // 解构出依赖
        if(dependencies){
            for(let j in dependencies){
                // 递归分析依赖,放入依赖图谱数组
                graphArray.push(moduleAnalyser(dependencies[j]))
            }
        }
    }
    // 转换为对象 便于使用
    const graph={}
    graphArray.forEach(item=>{
        graph[item.filename]={
            dependencies:item.dependencies,
            code:item.code
        }
    });
    return graph;
}

const generateCode=(entry)=>{
    const graph=JSON.stringify(makeDependenciesGraph(entry));
    return `
        (function(graph){
            function require(module){
                function localRequire(relativePath){
                    return require(graph[module].dependencies[relativePath])
                }
                var exports={};
                (function(require,exports,code){
                    eval(code)
                })(localRequire,exports,graph[module].code);
                return exports;
            };
            require('${entry}');
        })(${graph})
    `;
}

const code=generateCode("./src/index.js");// 入口函数
console.log(code);
```
以上就是一个webpack代码转换编译的整个过程。继续学习中！
## 参考资料

* [babel中parser模块文档](https://www.babeljs.cn/docs/babel-parser)
* [babel中core模块文档](https://www.babeljs.cn/docs/babel-core)