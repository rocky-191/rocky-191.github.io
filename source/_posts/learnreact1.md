---
title: 假期无聊，不如写写简单版React
date: 2020-01-29 19:51:56
tags: 
	- react
categories: 'react'
---

各位小伙伴，春节假期已过大半，这个春节应该是历年来过的最安静的一个春节了吧，天天家里蹲，吃了睡，睡了吃，没想到不出门竟然成了对社会最大的贡献。废话不多说，开始说正题。  

相信很多人都用过react开发项目，也有很多人好奇页面中明明没有直接用到React，但是页面却必须引入React，否则就会报错。初学者一定有这个疑问，不要着急，今天我就为你解答这个疑问。

## JSX
### jsx是个啥？
jsx其实就是个语法糖，用写html的方式来写js，一个很像xml的js扩展。React使用jsx来替代常规的js。[在线体验](https://reactjs.org/)
### 为什么要引入jsx？
写js不香吗？为什么要引入一个新概念jsx来迷惑大家？概括来讲jsx来讲有以下几个好处：

* 提升开发效率。使用jsx来写模版速度嗖嗖的。
* 提升执行效率。jsx编译为js代码后进行来很多优化，执⾏更快。
* 类型安全。在编译过程中就能发现错误。

尝试过上面的在线体验后，就会发现实际上babel-loader会把jsx预编译为React.createElement(xxx)。  
**jsx预处理前：**  

![](https://user-gold-cdn.xitu.io/2020/1/29/16ff0e6f63326709?w=1174&h=758&f=jpeg&s=94971)

**jsx预处理后**

![](https://user-gold-cdn.xitu.io/2020/1/29/16ff0e8c4a5fe52c?w=1168&h=698&f=jpeg&s=102427)
通过以上两张图片对比可以解答文章开头的疑问，为什么React没有使用，却必须引入？其实并不是没有使用，只不过没有直接使用，需要经过babel转化。另外可以看出React几个核心API：React.createElement, React.Component, ReactDom.render。

接下来我们就自己动手来实现这三个API吧！
## 核心API实现（简版）
### createElement和Component
作用：将传入的节点转化成vdom。  
（1）创建./simple-react/component.js。实现class组件必备条件。

```
export class Component{
    static isReactComponent={};
    constructor(props){
        this.props=props;
        this.state={}
    }
}
```
（2）创建./simple-react/index.js文件

```
import {Component} from "./component"

function createElement(type,props,...children){
    props.children=children;
    // console.log(type);
    // 判断组件类型
    let vtype;
    if(typeof type==="string"){
        // 原生标签
        vtype=1;
    }else if(typeof type === "function"){
        // 类组件，函数式组件
        vtype=type.isReactComponent ? 3 : 2;
    }
    return {
        vtype,
        type,
        props
    }
}

const React={
    createElement,
    Component
}

export default React;
```
上面我只是简单使用了1、2、3来分别标示来标签类型，你也可以使用别的方式来处理。createElement被调用时会传入标签类型type，标签属性props及若⼲子元素children。
### render
作用：渲染vdom，挂载到真实dom树上。  
创建./simple-react/ReactDOM.js文件，包含render函数。

```
function render(vnode,container){
    // vnode->node
    mount(vnode,container); // 待实现

}

const ReactDOM={
    render
}

export default ReactDOM;
```
这样就实现了代码中的常见的ReactDOM.render(jsx,container)。接下来重点实现mount函数来处理vnode挂载。  
创建./simple-react/virtual-dom.js文件。

```
export function mount(vnode,container){
    const {vtype}=vnode;
    if(!vtype){
        // 纯文本节点
        mountText(vnode,container);
    }
    if(vtype===1){
        // 原生节点
        mountHtml(vnode,container);
    }
    if(vtype===2){
        // 创建函数式节点
        mountFunc(vnode,container)
    }
    if(vtype===3){
        // 创建class类型组件
        mountClass(vnode,container);
    }
}

function mountText(vnode,container){
    let textNode=document.createTextNode(vnode);
    container.appendChild(textNode)
}

function mountHtml(vnode,container){
    const {type,props}=vnode;
    let htmlNode=document.createElement(type);
    const {children,...rest}=props;
    Object.keys(rest).map(item=> {
        if(item==="className"){
            htmlNode.setAttribute("class",rest[item]);
        }
        if(item.slice(0,2)==="on"){
            // 简单处理click事件，实际情况很复杂，需要考虑多种情况
            htmlNode.addEventListener("click",rest[item])
        }
    })
    children.map(item=>{
        if(Array.isArray(item)){
            item.map(i=>mount(i,htmlNode))
        }else{
            mount(item,htmlNode)
        }
    })
    container.appendChild(htmlNode)
}

function mountFunc(vnode,container){
    const {type,props}=vnode;
    let node=type(props);
    mount(node,container);
}

function mountClass(vnode,container){
    const {type,props}=vnode;
    let cmp=new type(props);
    let node=cmp.render();
    mount(node,container);
}
```
mount函数创建好之后，完整的ReactDOM文件就可以改写为如下：

```
import {mount} from "./virtual-dom";

function render(vnode,container){
    // vnode->node
    mount(vnode,container)

}

const ReactDOM={
    render
}

export default ReactDOM;
```
相关文件创建好之后，在由creat-react-app创建好的项目中改写index.js文件，引入自己写好的简版react文件，进行测试。

```
import React from "./simple-react";
import ReactDOM from "./simple-react/ReactDOM";
import './index.css'
function Funcomp(props){
    return (
        <div className="border">{props.name}</div>
    )
}

class ClassComp extends React.Component {
    constructor(props) {
        super(props);
        this.state = {  }
    }
    handleClick=()=>{
        alert("hello")
    }
    render() { 
        return (
            <div className="border">
                <h1>{this.props.name}</h1>
                {
                    [1,2,3].map(item=>
                    (
                        <h1 key={item}>{item}</h1>
                    )
                )
                }
                <button onClick={this.handleClick}>点我</button>
            </div>
        );
    }
}

let jsx=(
    <div>
        <div className="border">我是内容</div>
        <Funcomp name="我是函数组件内容" />
        <ClassComp name="我是class组件内容" />
    </div>
)
ReactDOM.render(jsx,document.getElementById("root"))
```
实际页面效果如果和下面截图一样，代表着简版react大功告成。

![](https://user-gold-cdn.xitu.io/2020/1/29/16ff10a387a870e6?w=3348&h=848&f=jpeg&s=174595)

## 总结

* webpack+babel编译时，替换jsx为React.createElement(type,props,...children)。
* 所有React.createElement()执⾏结束后得到⼀个JS对象即vdom，一个能够完整描述dom结构的对象。
* ReactDOM.render(vdom,container)可以将vdom转换为真实dom并添加container中。

当然在实际react处理中，要处理的情况远比上面写的复杂多得多，dom的更新、替换、删除要经过diff的过程，打补丁，更新布丁等过程，v16.8之后的fiber更是类似于时间分片的方式，将任务拆分，将高优先级任务优先执行，提高页面渲染的流畅度等等，有很多需要我们掌握的东西。  

**应了那句话：路漫漫其修远兮，吾将上下而求索。**

欢迎点赞、留言、交流。

## 参考资料

* [React中文网](https://react.docschina.org/)
* [React源码](https://github.com/facebook/react)