---
title: 使用React构建精简版本掘金（三）
date: 2019-03-20 20:23:09
tags:
	- react
	- ant-design
categories: 'react'
---

抽了两天工作中闲暇时间，文章详情页终于写完了，先上图哈，截图少了下面一部分哈，见谅！


![](https://user-gold-cdn.xitu.io/2019/3/22/169a455683f49328?w=1228&h=931&f=png&s=181334)
本篇文章为使用React构建精简版本掘金系列第三篇，想看前两篇的话，请查看[第一篇](https://rocky-191.github.io/2019/03/19/react-juejin/#more),[第二篇](https://rocky-191.github.io/2019/03/20/react-juejin1/#more)。  
**整理一下详情页用到的一些知识点：**

* redux存储、取值
* React-router路由跳转传值
* 父子组件传值

# 实现过程

## 左侧部分
主要包含点赞数量显示、评论数量、收藏按钮，分享链接等。由于要常驻在左侧，且页面滚动过程中位置不变，故使用fixed定位方式，并且使用ant-design中的Avatar,Badge,Icon,Popover组件。

```
import { Avatar,Badge,Icon,Popover } from 'antd';
...
<div style={articleSuspendedPanel}>
    <Badge count={this.props.starCount} overflowCount={this.state.overflowCount} style={badgeSy}>
        <div style={panelCircleSy}>
            <Icon type="like" theme="filled" style={IconSy}/>
        </div>
    </Badge>
    <Badge count={this.props.commentsCount} overflowCount={this.state.overflowCount} style={badgeSy}>
        <div style={panelCircleSy}>
            <Icon type="message" theme="filled" style={IconSy}/>
        </div>
    </Badge>
    <div style={panelCircleSy}>
        <Icon type="star" theme="filled" style={IconSy}/>
    </div>
    <label>分享</label>
    <Avatar style={avatarSy1}>
        <img alt="" src="//b-gold-cdn.xitu.io/v3/static/img/weibo.2076a57.svg" />
    </Avatar>
    <Avatar style={avatarSy1}>
        <img alt="" src="//b-gold-cdn.xitu.io/v3/static/img/qq.0834411.svg" />
    </Avatar>
    <Popover content={<QRCode value={window.location.href+'/post/'+this.props.wxShareAddr} />} placement="bottom">
        <Avatar style={avatarSy1}>
            <img alt="" src="//b-gold-cdn.xitu.io/v3/static/img/wechat.63e1ce0.svg" />
        </Avatar>
    </Popover>
</div>
```
render中的代码结构如上，starCount和commentsCount数据是从父级组件传入的，微信分享扫一扫利用了QRCode组件，关于该组件的介绍可查看上一篇文章，有详细的说明。

## 中间部分（重点）
仔细看的话，其实中间部分也可以分为顶部的作者介绍，包括文章编辑时间，阅读量，登录用户是否关注了作者。第二部分才是真正的文章详情。第三部分有关于该文章的评论区域，最下面是一系列的相关推荐文章。接下来分别介绍下：
### 顶部

```
<List
    itemLayout="horizontal"
    dataSource={this.state.articleInfo.articleList}
    renderItem={item => (
    <List.Item actions={[item.isFocus?<button className='focusedBtn'>已关注</button>:<button className='focusBtn'>关注</button>]}>
        <List.Item.Meta
            avatar={<Avatar src={item.authorImage} />}
            title={item.author}
            description={<div><span>{item.editDate}</span><span style={{marginLeft:'10px'}}>阅读{item.readNum}</span></div>}
        />
    </List.Item>
    )}
/>
```
很简单的一部分，应该能看明白。
### 中间部分
中间部分我这里就简单的用section包裹内容了，实际开发中个人觉得应该需要读取对应各层标题的样式、正文的样式，分段落显示，需要去分类型解析各种内容，包括图片、超链接等内容。
### 评论区域

![](https://user-gold-cdn.xitu.io/2019/3/22/169a461c9eebe893?w=720&h=175&f=png&s=9981)
布局问题就不说了，重点说一下输入框评论功能实现，其实类似大家常说的todolist功能。输入内容，点击评论按钮或者直接回车，评论内容添加到评论列表，输入框内容清空。

```
<div style={{marginLeft:'10px',flex:'1'}}>
    <div>
        <Input value={this.state.value} placeholder="输入评论..." onFocus={this.handleFocus.bind(this)} onChange={this.handleChange.bind(this)} onPressEnter={this.handlePressEnter.bind(this)}/>
    </div>
    {
        this.state.showIconAndBtn?
            <div style={{display:'flex',justifyContent:'space-between',marginTop:'10px'}}>
                <span style={{color:'#027fff',cursor: 'pointer',display:'flex',alignItems:'center'}} onClick={info}>
                    <img alt='' src='//b-gold-cdn.xitu.io/v3/static/img/emoji.5594dbb.svg' />
                    表情
                </span>
                <p>
                    <label style={{color:'#c2c2c2',marginRight:'8px'}}>Enter</label>
                    <Button type="primary" onClick={this.handlePressEnter.bind(this)}>评论</Button>
                </p>
            </div>:''
    }
</div>
```
showIconAndBtn是控制输入框下面那一行是否显示的，初始进入页面的时候是只能看到输入框，下面的表情和评论按钮只能等输入框获取焦点后才能显示。 输入框上绑定了三个事件，分别用来处理获取焦点事件、输入内容改变事件、按下回车事件。

* 获取焦点事件

```
handleFocus=()=>{
    this.setState({
        showIconAndBtn:true
    })
}
```

* 输入内容改变事件

```
handleChange=(e)=>{
    this.setState({
        value:e.target.value
    });
}
```

* 回车事件（后面的评论按钮功能相同）

```
handlePressEnter=()=>{
    if(this.state.value!==''){
        this.props.submitComment(this.state.value);//调用父级组件并传值
        this.setState({
            value:'',
            showIconAndBtn:false
        })
    }else{
        message.warning('还未填写评论哦！');
    }
}
```
这里有用到子组件向父组件传值的方法，简单介绍下：  
**父向子传值**
> 父向子传值起始比较简单，重点要考虑向下传值是否会使得父组件中的状态过于繁杂，是否会影响页面性能。  

假设存在一个组件ComponentA，在父组件中调用ComponentA，并且传递参数data

```
...
this.state={
    text:'hello world'
}
...
<ComponentA data={this.state.text} /
```
在组件ComponentA中就可以通过this.props.data获取到父组件传入的data值了。  
**子向父传值**
>由子组件的事件触发，在触发的函数体中调用父组件传入的方法，将子组件里的值传入即可。  

假设存在父组件ComponentParent，子组件ComponentChild,父组件中调用ComponentChild

父组件：
```
...
handleChange=(val)=>{
    console.log(`信息:`+val);//子组件传入的数据
}
...
<ComponentChild change={this.handleChange.bind(this)} />
```
子组件：

```
//假设有个方法叫handleClick
handleClick=()=>{
    this.props.change('hello');
}
```
在子组件中执行handleClick方法的时候就会触发父组件中的方法handleChange，并在控制台输出hello。

**兄弟组件传值**
>可以将数据提升到共同的父组件中，进行传值，之后在利用父子组件传值即可。

**多层级组件或者称为不相邻组件传值**
>可以利用redux管理全局状态，之后在任何地方都可以取到对应的数据。

这种方式的使用方式在[第一篇文章](https://rocky-191.github.io/2019/03/19/react-juejin/#more)做了说明，可以浏览哦！

**待实现功能：**
- [ ] 表情组件

### 评论列表区域
个人实现的这块区域和掘金有少许不同。

```
<List
    itemLayout="vertical"
    size="large"
    dataSource={this.props.commentList}
    renderItem={item => (
    <List.Item
        key={item.userId}
        actions={[<span>{moment().subtract(item.editDate, 'days').fromNow()}</span>,<IconText type="like-o" text={item.starNum} />, <IconText type="message" text={item.commentNum} />]}
    >
        <List.Item.Meta
        avatar={<a href={'/user/'+item.userId}><Avatar src={item.userImage} /></a>}
        title={<div>{item.authorName}</div>}
        description={item.userDesc}
        />
        {item.commentText}
    </List.Item>
    )}
/>
```
采用了ant-design中的List组件进行显示  
**待实现**
- [ ] 嵌套评论列表

### 相关推荐列表
该部分结构采用了首页列表组件，只需按照文章类型，推荐不同内容即可，这里不再赘述。

## 右侧部分
右侧整体结构和首页右侧内容类似，分6块：

* 关于作者

```
<Card
    title="关于作者"
    style={{ width: '100%' }}
    hoverable={'true'}
    headStyle={{fontSize:'14px',color:'#333'}}
    bodyStyle={{padding:'0 16px'}}
    >
    <List
        itemLayout="vertical"
        dataSource={this.props.author}
        renderItem={item => (
        <List.Item onClick={()=>window.location.href='/user/'+item.id}>
            <List.Item.Meta
            avatar={<Avatar size={46} src={item.authorImage} />}
            title={item.author}
            />
            {item.isGroup?<div>
                <Avatar style={{ backgroundColor: '#e1efff',color:'#7bb9ff' }} icon="user" />
                <label style={{color:'#000',marginLeft:'10px',fontSize:'16px'}}>联合编辑</label>
            </div>:''}
            <div style={{marginTop:'10px'}}>
                <Avatar style={{ backgroundColor: '#e1efff' }}>
                    <Icon type="like" theme="filled" style={{color:'#7bb9ff'}} />
                </Avatar>
                <label style={{color:'#000',marginLeft:'10px',fontSize:'16px'}}>获得赞数{item.allStarNum}</label>
            </div>
            <div style={{marginTop:'10px'}}>
                <Avatar style={{ backgroundColor: '#e1efff' }}>
                    <Icon type="eye" theme="filled" style={{color:'#7bb9ff'}} />
                </Avatar>
                <label style={{color:'#000',marginLeft:'10px',fontSize:'16px'}}>获得阅读数{item.allReadNum<99999?item.allReadNum:'99999+'}</label>
            </div>
        </List.Item>
        )}
    />
</Card>
```
* 感兴趣的小册推荐

```
<Card
    title="你可能感兴趣的小册"
    style={{ width: '100%',marginTop:'20px' }}
    hoverable={'true'}
    headStyle={{fontSize:'14px',color:'#333'}}
    bodyStyle={{padding:'0 16px'}}
    >
    <List
        itemLayout="horizontal"
        dataSource={this.props.recommendBooks}
        className="bookCard"
        renderItem={item => (
        <List.Item onClick={()=>window.location.href='/book/'+item.id}>
            <List.Item.Meta
            avatar={<img alt='' src={item.bookImage} />}
            title={item.title}
            description={<p className="book-desc"><span>{item.sellNum+'人已购买'}</span><span className="try-read">试读</span></p>}
            />
        </List.Item>
        )}
    />
</Card>
```
这一块单独抽一个组件，别的地方也可能会用到，后期只需传入小册内容即可。目前首页和文章详情页用的都是这个组件。
* 掘金客户端下载二维码

```
<Card style={{ width: '100%',marginTop:'20px' }} hoverable='true' className="download-card" bodyStyle={{padding:'15px'}}>
    <NavLink to='/app'>
        <img alt='qrcode' src='//b-gold-cdn.xitu.io/v3/static/img/timeline.e011f09.png' />
        <div>
            <div className="headline">下载掘金客户端</div>
            <div className="desc">一个帮助开发者成长的社区</div>
        </div>
    </NavLink>
</Card>
```
* 掘金微信群

```
<Card
    style={{ width: '100%',marginTop:'20px' }}
    hoverable={'true'}
    bodyStyle={{padding:'0'}}
>
    <img alt='' src='//b-gold-cdn.xitu.io/v3/static/img/backend.ba44b94.png' style={{height:'200px',width:'100%'}} />
</Card>
```
* 相关文章列表

```
<Card
    title="相关文章"
    headStyle={{fontSize:'14px',color:'#333'}}
    style={{ width: '100%',marginTop:'20px'}}
    hoverable={'true'}
    bodyStyle={{padding:'0 16px'}}
    >
    <List
        itemLayout="vertical"
        dataSource={this.props.relationArticles}
        split={false}
        renderItem={item => (
        <List.Item onClick={()=>window.location.href='/post/'+item.id}>
            <div style={{color:'#333',fontSize:'16px'}}>{item.title}</div>
            <div style={{marginTop:'10px',color:'#b2bac2',fontSize:'12px'}}>
                <Icon type="like" theme="filled" style={{marginRight:'3px'}} />{item.starNum}
                <Icon type="message" theme="filled" style={{marginLeft:'15px',marginRight:'3px'}} />{item.commentNum}
            </div>
        </List.Item>)}
    />
</Card>
```
* 文章目录

```
<div className='text-catalogue'>
    <Timeline>
        {this.props.data.map((item,index)=>{
            return <Timeline.Item color='#000' key={item.text} dot={<span className='catalogue-circle'></span>}>{(item.children && item.children.length !== 0)?<div><a href={`#heading-${index}`}>{item.text}</a><div className='secondCatalogue'><Catalogue data={item.children} /></div></div>:<a href={`#heading-${index}`}>{item.text}</a>}</Timeline.Item>
        })}
    </Timeline>
</div>
```
用到了ant-design中的Timeline组件，这个单独抽离成一个组件，整个系统中都可以复用，这里用到了递归组件实现目录嵌套的功能。

## 路由跳转传参
从首页文章列表进入文章详情页的时候需要传递一些参数，比如文章的id值

### 方法一（本文采用了该方法）
* 第一步
  在路由文件中设置

```
<Route path='/post/:articleId' component={Post}/>
```

* 第二步
  文章列表点击事件中

```
showArticleInfo=(id)=>{
    console.log(`文章id值：${id}`);
    window.location.href='/post/'+id;
}
```

* 第三步
  在进入文章详情页的时候就可以获取到文章id了。

```
componentDidMount(){
    const {match}=this.props;
    console.log(`文章id：${match.params.articleId}`);
}
```
### 方法二

* 第一步：定义路由

```
<Route path='/post' component={Post} />
```

* 第二步：传递方式

```
let data = {id:3,name:sam};
let path = {
  pathname:'/post',
  state:data,
}
```
链接跳转

```
<Link to={path}>详情</Link>
```

* 第三步：获取

```
let data = this.props.location.state;
let {id,name} = data;
```

截止到目前为止，首页和文章详情页的结构和基本功能就算完成了，细节后续在优化，剩余部分陆续更新中。   

上述详细代码请见[github](https://github.com/rocky-191/react-juejin),不要忘了**star**和**点赞**哦，多谢！