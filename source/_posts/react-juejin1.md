---
title: 使用React构建精简版本掘金（二）
date: 2019-03-20 20:23:09
tags:
	- react
	- ant-design
categories: 'react'
---

![](https://user-gold-cdn.xitu.io/2019/3/20/1699a60f0e65a946?w=1088&h=931&f=png&s=150116)
咋一看，是不是感觉掘金改版了呢！如果你有这个错觉，那就说明我仿照的还算可以，我就当是对我的肯定吧，O(∩_∩)O~~

# 首页

## 顶部标签

![](https://user-gold-cdn.xitu.io/2019/3/20/1699a6698752f04f?w=1084&h=406&f=png&s=71112)
即我上面红框圈住的部分，这部分由于要做页面滚动的时候常驻顶部，个人为了简单省事，采用了ant-design中的Affix组件，另外导航组件我抽离了一个公用组件，从外部传入tags数组。

```
import { Affix} from 'antd';
...
this.state={
    tags:[
        {
            path:'recommended',
            text:'推荐'
        },
        {
            path:'following',
            text:'关注'
        }
    ]
}
...
<Affix offsetTop={this.state.top}>
    <div className="home-nav">
        <nav>
            <HomeNav tags={this.state.tags} match={match}/>
            <a href="/">标签管理</a>
        </nav>
    </div>
</Affix>
```
HomeNav组件如下

```
import React, { Component } from 'react';
import { NavLink } from 'react-router-dom'
...
class HomeNav extends Component {
    constructor(props) {
        super(props);
        this.state = {}
    }
    render() {
        return (
            <ul>
                {this.props.tags.map((item,index)=>{
                    return <li key={item.path}>
                        <NavLink to={`/timeline/${item.path}`}>{item.text}</NavLink>
                    </li>
                })}
            </ul>
        );
    }
}

export default HomeNav;
```

## 动作部分
![](https://user-gold-cdn.xitu.io/2019/3/20/1699a6d9318aae61?w=1105&h=196&f=png&s=26896)
该部分的实现方式可参考上面tag标签实现部分，基本类似。  
头像部分我使用了ant-design中Avatar，用来代表用户或事物，支持图片、图标或字符展示。

![](https://user-gold-cdn.xitu.io/2019/3/20/1699a704aea145bc?w=1103&h=450&f=png&s=75565)

## 文章列表部分
列表部分使用了ant-design中list组件

* 列表结构实现
```
import { List,Statistic,Icon,Popover } from 'antd';
...
const {data}=this.props;
...
<List
    itemLayout="horizontal"
    dataSource={data}
    renderItem={item => (
    <List.Item extra={item.articleImage ? <img width={80} alt="logo" src={item.articleImage} />:''} onClick={this.showArticleInfo.bind(this,item.id)}>
        添加内容
    </List.Item>
    )}
/>
```
实际中data应该是根据传入的userID和标签去数据库查询得到的真实数据，我这里的data就从上一级中进行了获取。每一个listItem上面绑定了点击事件，最终需要跳转到文章详情页面。

* 列表内部结构

![](https://user-gold-cdn.xitu.io/2019/3/20/1699a7ad410054ff?w=761&h=144&f=png&s=16284)
整体分成上中下三部分来实现布局,右侧的图片是在上一步listItem中配置extra属性可以实现。

```
<article>
    <section className="list-part1">
        <ul>
            <li className="item post">{item.articleType==='1'?'专栏':'小册'}</li>
            <li>{item.author}</li>
            {item.time?<li>{item.time}</li>:''}
            {item.tags.length!==0?<li>{item.tags.map((tag,index)=>{
                return <NavLink key={tag} to={`/tag/${tag}`}>{tag}</NavLink>
            })}</li>:''}
        </ul>
    </section>
    <section className="list-part2">
        <NavLink to={`/post/:articleId`}>{item.title}</NavLink>
    </section>
    <section className="list-part3">
        {item.articleType==='1'?
            <div>
                <Statistic value={item.starNum} prefix={<Icon type="like" theme="filled" style={{ fontSize: '14px'}} />} onClick={()=>this.props.editStar(item.id)} />
                <Statistic value={item.commentNum} prefix={<Icon type="message" theme="filled" style={{ fontSize: '14px'}} onClick={()=>this.props.lookComment(item.id)} />} />
                <Icon type="upload" style={{ fontSize: '16px',marginLeft:'10px',borderRight:'none'}} />
                <Icon type="star" theme="filled" style={{ fontSize: '16px'}} onClick={()=>this.props.collectArticle(item.id)} />
            </div>:
            <div className="xiaoce-action-row">
                <span className="link-btn buy">购买人数: {item.sellNums}</span>
                <span className="link-btn sale">特价: {item.price}元</span>
                <span className="link-btn share">
                    <Icon type="upload" style={{ fontSize: '16px',marginLeft:'10px',borderRight:'none'}} onClick={()=>this.props.shareArticle(item.id)} />
                    分享
                </span>
            </div>
        }
    </section>
</article>
```
掘金官网文章列表根据文章的类型是专栏还是小册显示不同的内容，即这里根据item的articleType进行了区分，这里用到了三目运算符做了显示控制渲染。  
**React中条件渲染的方式有以下几种，补充下知识点：**  
（1）if 语句  
（2）三目操作符  
（3）逻辑 && 操作符  
（4）switch.. case.. 语句  
（5）枚举   
（6）多层条件渲染  
（7）使用高阶组件  
详情可查阅该[文章](https://www.jianshu.com/p/fe5205b33707)

## 点击分享

* 安装qrcode.react插件

```
yarn add qrcode.react --save
```

* 引入使用

```
const QRCode = require('qrcode.react');
...
<QRCode value={this.props.value} />
```
value值是从上一级组件传入的值,该插件属性描述：
| prop          | type                     | default value |
| ------------- | ------------------------ | ------------- |
| value         | string                   | -             |
| renderAs      | string ('canvas' 'svg')  | 'canvas'      |
| size          | number                   | 128           |
| bgColor       | string (CSS color)       | "#FFFFFF"     |
| fgColor       | string (CSS color)       | "#000000"     |
| level         | string ('L' 'M' 'Q' 'H') | 'L'           |
| includeMargin | boolean                  | false         |


* 抽离分享组件

```
import React, { Component } from 'react';

class Qrcode extends Component {
    constructor(props) {
        super(props);
        this.state = {
            shareTypes:[
                {
                    image:'//b-gold-cdn.xitu.io/v3/static/img/weibo.8e2f5d6.svg',
                    text:'微博',
                    showQrcode:false,
                },
                {
                    image:'//b-gold-cdn.xitu.io/v3/static/img/wechat.844402c.svg',
                    text:'微信扫一扫',
                    showQrcode:true,
                }
            ]
        }
    }
    render() {
        const QRCode = require('qrcode.react');
        return (
            <ul>
                {this.state.shareTypes.map(item=>{
                    return <li key={item.text} style={{borderBottom:'1px solid rgba(217,222,224,.99)',padding:'10px',cursor:'pointer'}}>
                        <div>
                            <img alt={item.text} src={item.image} style={{width:'24px',height:'24px',marginRight:'5px'}} />
                            <label style={{color:'#8f969c'}}>{item.text}</label>
                        </div>
                        {item.showQrcode?<div style={{textAlign:'center'}}>
                            <QRCode value={this.props.value} />
                        </div>:''}
                    </li>
                })}
            </ul>
        );
    }
}

export default Qrcode;
```

* 分享组件使用
```
import Qrcode from '../../components/Qrcode';
...
<Popover content={<Qrcode value={window.location.href+'/'+item.id} />} placement="bottom" trigger="click">
    <Icon type="upload" style={{ fontSize: '16px',marginLeft:'10px',borderRight:'none'}} onClick={()=>this.props.shareArticle(item.id)} />
    分享
</Popover>
```
## 右侧卡片内容
这块内容采用了ant-design中的card组件，直接看代码吧。

```
import { Card } from 'antd';
...
<Card
    title="掘金优秀作者"
    style={{ width: '100%' }}
    hoverable={'true'}
    actions={[<NavLink to='/recommendation/authors/recommended' style={{color:'#007fff'}}>查看更多></NavLink>]}
    >
    <List
        itemLayout="horizontal"
        dataSource={this.props.goodAuthor}
        renderItem={item => (
        <List.Item onClick={()=>window.location.href='/user/'+item.id}>
            <List.Item.Meta
            avatar={<Avatar size={46} src={item.userImage} />}
            title={item.title}
            description={<div className="overflow-ellipsis">{item.desc}</div>}
            />
        </List.Item>
        )}
    />
</Card>
```
goodAuthor数据是从父级传入的数据哈。

![](https://user-gold-cdn.xitu.io/2019/3/20/1699afd7ca3b5310?w=524&h=1602&f=png&s=201017)
这里的内容结构类似，外层card组件，内置不同不同内容，就不重复了，想看具体实现的话请移步[github](https://github.com/rocky-191/react-juejin),不要忘了star哈。   

card内部使用list组件，竖着排列各自的内容，这一块总觉得可以抽成一个公用的card组件，然后填充不同内容，暂时就先这样吧，列入后期重构计划！这时就怀念vue中的slot方法了，可以方便的放置不同内容。jsx也有自己的方便之处吧，灵活的使用各种标签。

以上就是首页的一个简单说明了，页面大部分的链接router跳转功能还没有实现，后续陆续更新中。  

想看第一篇文章的朋友可以查看[使用React构建精简版本掘金（一）](https://rocky-191.github.io/2019/03/19/react-juejin/),上述所有详细代码都已经放到[github](https://github.com/rocky-191/react-juejin)了，欢迎浏览和star哈，都已经看到这里了，那就麻烦大家点个赞在走吧！

既然没有合适的坑，那就趁着闲暇，继续提升自己的能力吧！最后鼓励下自己：扛过了艰难的时光，回头看，那也就不是什么大不了的事情了！