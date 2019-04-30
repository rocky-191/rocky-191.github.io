---
title: 使用React构建精简版本掘金（五）
date: 2019-04-30 21:25:18
tags:
	- react
	- ant-design
categories: 'react'
---

距离上篇文章已经过去了大半个月，本来打算只更新代码，最后还是决定把做的过程中遇到的问题记录出来，说不定就可以帮助一些同学，也算是幸事，如果没有，那就当作自己梳理知识点吧！ 

该篇文章主要讲述以下知识点：

* 如何mock数据
* React组件中修改Redux中的数据

# mock数据
有三种方式：

* 利用node搭服务，mock数据
* 利用现成网上mock服务，比如[easy mock](https://www.easy-mock.com/login)
* 本地json数据模拟请求

介绍下本地数据模拟请求，利用axios请求数据：

```
import axios from 'axios';

const BaseUrl='./mock/homeData/';
const getHome=()=>{
    return axios.get(BaseUrl+'home.json').then(res=>{
        return res.data
    })
}

const getArticleList=()=>{
    return axios.get(BaseUrl+'articleList.json').then(res=>{
        return res.data
    })
}

export { getHome,getArticleList }
```
本地json数据存放位置如下
![](https://user-gold-cdn.xitu.io/2019/4/30/16a6e5b54df5f8d1?w=672&h=632&f=png&s=61735)

# React组件中修改Redux中的数据

定义好state，reducer
```
//userReducer.js

// 1.定义默认数据
let initialState = {
    userId:'',
    userName:'',//实际项目与此不同
    userImage: '',
    userDesc:''
}

// action creators
export const actions = {
    login: (userInfo) => {
        return {type: 'CHANGE_USER',userName: userInfo.username,userId:userInfo.userId}
    },
    logout:()=>{
        return {type:'LOGOUT'}
    }
};

// 2.Reducer
const userReducer = (state = initialState, action) => {
    switch (action.type) {
        case 'CHANGE_USERIMAGE':
            return { ...state, userImage: action.userImage }
        case 'CHANGE_USERID':
            return { ...state,userId:action.userId}
        case 'CHANGE_USER':
            return { ...state,userName:action.userName,userId:action.userId}
        case 'LOGOUT':
            return {...state,userName:'',userId:''}
        default:
            return state
    }
}
// 3.导出
export default userReducer;
```
在组件中操作修改state

```
...
this.props.login({username:username,userId:userId});//修改存储在redux中的用户信息
...
const mapStateToProps = (state, ownProps) => {
  return {
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return bindActionCreators({
    login: authActions.login
  },dispatch);
}

Login=connect(mapStateToProps,mapDispatchToProps)(Login)

export default Login;
```

Redux通过reducer解析action。reducer是一个普通的js函数，接受action作为参数，然后返回一个新的state。

**三大原则**

* 唯一数据源
* 保持state状态只读
* state的改变必须通过纯函数完成

## Redux
Redux应用只维护一个全局的状态对象，存在Redux的store中。程序任何时候都不能直接去修改状态state，如果需要修改state，必须发送一个action，通过action去描述如何修改state。action描述了改变state的意图，真正对state作出修改的是reducer。reducer必须是**纯函数**，所以reducer在收到action后，不能直接去修改原来的状态state，而是应该创建一个新的状态对象返回。  
> 纯函数需要满足两个条件：  
> （1）对于同样的参数值，函数返回结果总是相同的，就是函数结果不依赖任何在程序执行过程中可能改变的变量。  
> （2）函数的执行过程不会产生比的副作用，例如会修改外部对象，有时会去修改页面标题，这就是副作用。

数据流向的一个简单示意图，来自[Flux官网](https://facebook.github.io/flux/docs/in-depth-overview.html#content)
![](https://user-gold-cdn.xitu.io/2019/4/30/16a6b751c735f491?w=1300&h=393&f=png&s=39726)

可以看出Redux应用的主要组成有action，reducer和store。
### action
action是Redux中的信息载体，是store的唯一来源。把action发给store必须通过store中的dispach方法。其实action就是一个普通的js对象，但是每个action必须有一个type属性用来描述action类型，要做什么操作。除了type属性歪，action的结构完全由你决定，不过应该保证可以清晰描述使用场景。

```
function logout(info){
    return {
        type:'LOGOUT',
        info
    }
}
```
### reducer
reducer根据action做出响应，决定如何修改应用状态state。其实在写reducer之前就应该设计好state。

state：
```
// 1.定义默认数据
let initialState = {
    userId:'',
    userName:'',//实际项目与此不同
    userImage: '',
    userDesc:''
}
```
reducer：

```
// 2.Reducer
const userReducer = (state = initialState, action) => {
    switch (action.type) {
        case 'CHANGE_USERIMAGE':
            return { ...state, userImage: action.userImage }
        case 'CHANGE_USERID':
            return { ...state,userId:action.userId}
        case 'CHANGE_USER':
            return { ...state,userName:action.userName,userId:action.userId}
        case 'LOGOUT':
            return {...state,userName:'',userId:''}
        default:
            return state
    }
}
```
> 这里使用了ES6的扩展运算符（...）创建新的state对象，避免直接修改之前的state对象。

在实际项目中，可能会有很多reducer，就需要把reducer拆分保存在单独的文件中。Redux提供了一个combineReducers函数，用来合并reducer。

```
import {combineReducers} from 'redux';

import pageHeaderReducer from './pageHeader.js';
import userReducer from './userReducer';

const appReducer = combineReducers({
    pageHeaderReducer,
    userReducer,
});
export default appReducer;
```
### store
store是Redux中的一个对象，作为action和reducer之间的一个桥梁。  
作用：

* 保存应用状态
* 通过方法获取应用状态
* 通过dispatch(action)发送更新状态的指令
* 通过方法subscribe(listener)注册监听函数、监听状态的改变。

总结下Redux中的数据流过程：

* （1）通过调用store.dispatch(action)。一个action描述了“发生了什么”的对象以及可能会携带一些参数。store.dispatch(action)可以在应用中任何位置调用。
* （2）Redux的store调用reducer函数。store传递两个参数给reducer，分别是当前应用的状态和action，reducer必须是纯函数。
* （3）根reducer可以把多个子reducer合并在一起，返回组成最终的应用状态。利用combineReducers进行组合。
* （4）Redux的store保存根reducer返回的完整应用状态，这整个流程走完，应用状态才完成更新。

## 在React中使用Redux
基础安装使用方法已经在[第一篇](https://rocky-191.github.io/2019/03/19/react-juejin/)中做了介绍，此处重点介绍下connect中的mapStateToProps和mapDispatchToProps。
### mapStateToProps
mapStateToProps是一个函数，从名字上看，该函数作用就是把state转换成props。state就是Redux store中保存的应用状态，会作为参数传递给mapStateToProps，props就是被连接展示组件的props。

```
const mapStateToProps = (state) => {
    return {
        userName:state.userReducer.userName,
        userImage:state.userReducer.userImage,
        userId:state.userReducer.userId
    }
}
```
当store中的state改变后，mapStateToProps就会重新执行，重新计算传递给展示组件的props，从而触发组件的重新渲染。
> **注意**：store中的state改变一定会导致mapStateToProps重新执行，但却不一定会触发组件渲染render方法重新执行。如果mapStateToProps新返回的对象和之前的对象浅比较相等，组件的shouldComponentUpdate方法就会返回false，组件的render方法就不会被再次触发，这也是一个重要优化吧！

### mapDispatchToProps
mapDispatchToProps接收store.dispatch方法作为参数，返回展示组件用来修改state的函数。

```
const mapDispatchToProps = (dispatch, ownProps) => {
  return {
      getOrder: (data) => dispatch(actionCreator(data))
  }
}
```
另外一种写法，利用bindActionCreators

```
import { bindActionCreators } from "redux";

const mapDispatchToProps = (dispatch, ownProps) => {
  return bindActionCreators({
    getOrder: actionCreator.getOrder
  },dispatch);
}
```
bindActionCreators作用是将单个或多个ActionCreator转化为dispatch(action)的函数集合形式。个人感觉会内部自动注入dispatch，不用我们手动去dispatch。之后通过connect连接即可
```
const mapStateToProps = (state, ownProps) => {
  return {

  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return bindActionCreators({
    login: authActions.login
  },dispatch);
}

Login=connect(mapStateToProps,mapDispatchToProps)(Login)
```
> **注意**connect函数参数必须第一个为mapStateToProps，第二个必须为mapDispatchToProps，可以不写mapDispatchToProps，但是如果需要mapDispatchToProps却不能不写mapStateToProps，否则会报错，个人第一次使用就犯了这个错误😓

啰哩啰嗦的说了这么多，码字不易，点赞再走哈。完整项目代码在[github](https://github.com/rocky-191/react-juejin),欢迎点个star，不胜感激👏👏👏