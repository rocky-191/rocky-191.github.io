---
title: taro多端实践初探
date: 2019-03-07 22:56:27
tags:
	- react
	- taro
categories: "react"
---

历史的发展，小程序风行一时，安卓/ios/H5/微信小程序/支付宝小程序/头条小程序，产品经理让你适配这么多，你的心情如何呢？然而总会有人给咱们造出合适的工具，解放生产力，一次编码，多端运行。开始探索之旅吧！

<!--more-->

## taro安装
安装 Taro 开发工具 @tarojs/cli  
使用 npm 或者 yarn 全局安装，或者直接使用npx

```
$ npm install -g @tarojs/cli
$ yarn global add @tarojs/cli
```
## 使用命令创建模版

```
$ taro init multiportApp
```

![](https://user-gold-cdn.xitu.io/2019/3/7/16958317533d8d4b?w=1572&h=742&f=png&s=113477)
按照自己情况选择安装即可
## 启动
进入对应目录，执行命令启动。

```
npm run dev:h5
```
会出现启动成功的界面，如下

![](https://user-gold-cdn.xitu.io/2019/3/7/1695833e27156fa9?w=1998&h=1246&f=png&s=237512)
自动就会打开浏览器，出现hello world界面，表示项目启动成功了！
## todolist功能实现
### 添加数据
在pages/index/index.js文件中添加如下
```
constructor(props){
    super(props);
    this.state={
      val:'',
      todos:[
        {
          title:'吃饭',
          done:false
        },
        {
          title:'睡觉',
          done:false
        },
        {
          title:'coding',
          done:false
        }
      ]
    }
  }
```
### 渲染数据

```
render () {
    return (
      <View className='index'>
        <Text>Hello world!</Text>
        {
          this.state.todos.map((item,index)=>{
            return <View key={index}>{item.title}</View>
          })
        }
      </View>
    )
  }
```
列表渲染搞定。
![](https://user-gold-cdn.xitu.io/2019/3/7/169584f746205544?w=890&h=1524&f=png&s=48060)
### 添加输入框和按钮
引入组件
```
import { View, Text,Input,Button } from '@tarojs/components'
```
render修改

```
render () {
    return (
      <View className='index'>
        <Text>Hello world!</Text>
        <Input value={this.state.val} onInput={this.handleInput}></Input>
        <Button onClick={this.handleClick}>添加</Button>
        {
          this.state.todos.map((item,index)=>{
            return <View key={index}>{item.title}</View>
          })
        }
      </View>
    )
  }
```
添加方法

```
handleInput=(e)=>{
    this.setState({
      val:e.detail.value
    })
}

handleClick=()=>{
    this.setState({
      todos:[...this.state.todos,{title:this.state.val,done:false}],
      val:''
    })
}
```
一个简单的todolist就算完成了，界面有点丑，继续优化！
![](https://user-gold-cdn.xitu.io/2019/3/7/169585c9775a2c21?w=431&h=737&f=gif&s=40747)
## 优化，引入UI库
### 安装taro-ui
[官网](https://taro-ui.aotu.io/#/docs/quickstart)
```
npm install --save taro-ui
```
### 简单配置
由于引用 `node_modules` 的模块，默认不会编译，所以需要额外给 H5 配置 `esnextModules`，在 taro 项目的 `config/index.js` 中新增如下配置项：

```
h5: {
  esnextModules: ['taro-ui']
}
```
### 全局引入
在app.scss中引入

```
@import 'taro-ui/dist/style/index.scss'
```
在index.js中引入

```
import { AtList, AtListItem } from "taro-ui"
```
修改render

```
render () {
    return (
      <View className='index'>
        <Text>Hello world!</Text>
        <Input value={this.state.val} onInput={this.handleInput}></Input>
        <Button onClick={this.handleClick}>添加</Button>
        <AtList>
          {
            this.state.todos.map((item,index)=>{
              return <AtListItem key={index} title={item.title}></AtListItem>
            })
          }
        </AtList>
      </View>
    )
  }
```

![](https://user-gold-cdn.xitu.io/2019/3/7/16958920d3f9deba?w=860&h=776&f=png&s=32921)

### 添加滑块开关，改变item状态

```
<AtListItem key={index} title={item.title} className={{'done':item.done}} isSwitch switchIsCheck={item.done} onSwitchChange={(e)=>this.handleChange(e,index)}></AtListItem>
```
增加一个isSwitch,switch切换事件，class。  
增加事件

```
handleChange=(e,i)=>{
    console.log(e,i);
    const todos=[...this.state.todos];
    todos[i].done=e.detail.value;
    this.setState({
      todos
    })
  }
```
在同级目录下index.scss中增加样式

```
.done{
  color: red;
  text-decoration: line-through;
}
```
### 效果
h5效果
![](https://user-gold-cdn.xitu.io/2019/3/7/169589ea2f51be88?w=854&h=1286&f=png&s=50703)
微信小程序中的效果

![](https://user-gold-cdn.xitu.io/2019/3/7/16958a1601ff0e3d?w=780&h=1364&f=png&s=61825)
这就是这个框架的威力，感谢taro开发团队。)

**最后在说一句，正在找工作，坐标北京，各位大佬有合适的机会推荐下哈！**