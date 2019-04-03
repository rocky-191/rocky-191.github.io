---
title: 使用React构建精简版本掘金（四）
date: 2019-04-03 19:33:30
tags:
	- react
	- ant-design
categories: 'react'
---

到目前为止，首页、文章详情页、动态页、话题页及小册页面内容完成了，看一下效果图哈，数据不全，见谅哈！

<!--more-->

![](https://user-gold-cdn.xitu.io/2019/4/3/169e244fd9cfe8e9?w=1909&h=898&f=gif&s=1444510)

# 实现过程
## 动态页
该页面从布局来说分左右两部分，左边有分为输入框和已发表内容部分。  

* 输入框部分

![](https://user-gold-cdn.xitu.io/2019/4/3/169e24face2af6b6?w=766&h=201&f=png&s=13507)
我这里采用了ant-design中的input作为输入框，而掘金是采用了可编辑的div来实现输入内容,通过设置contenteditable="true"实现，感兴趣的小伙伴可查阅相关资料。  
发布按钮的disabled状态将根据输入框是否有值来决定。功能类似todolist添加功能，可参考文章详情页面的发布评论功能。

```
<div style={{background:'#fff',padding:'15px',position:'relative'}}>
    <TextArea placeholder="告诉你个小秘密，发沸点时添加话题会被更多小伙伴看见呦~" style={{paddingBottom:'30px'}} autosize={{ minRows: 3}} onChange={this.handelInputChange.bind(this)}/>
    {this.state.selectedTopic!==''?<span style={{position:'absolute',bottom:'60px',left:'28px',padding:'0 12px',border:'1px solid #007fff',userSelect:'none',height:'22px',borderRadius:'14px',color:'#007fff'}}>{this.state.selectedTopic}</span>:''}
    <div style={{marginTop:'5px',display:'flex',justifyContent:'space-between'}}>
        <ul style={sy1}>
            <li style={sy2} onClick={this.handleBtn.bind(this,'exp')}>
                <img alt='' src='//b-gold-cdn.xitu.io/v3/static/img/emoji.5594dbb.svg' />表情
            </li>
            <li style={sy2} onClick={this.handleBtn.bind(this,'pic')}>
                <img alt='' src='//b-gold-cdn.xitu.io/v3/static/img/active_file.d265f4e.svg' />图片
            </li>
            <li style={sy2} onClick={this.handleBtn.bind(this,'link')}>
                <img alt='' src='//b-gold-cdn.xitu.io/v3/static/img/active_link.b1a6832.svg' />链接
            </li>
            <Popover placement="bottom" content={<TopicList handleTopic={this.handleTopic.bind(this)} />} trigger="click">
                <li style={sy2} onClick={this.handleBtn.bind(this,'topic')}>
                    <img alt='' src='//b-gold-cdn.xitu.io/v3/static/img/topic.6a87bb7.svg' />话题
                </li>
            </Popover>
        </ul>
        <Button type="primary" disabled={this.state.disabledBtn} onClick={this.handlePressEnter.bind(this)}>发布</Button>
    </div>
</div>
```
输入框内容发生变化事件

```
handelInputChange=(e)=>{
    if(e.target.value.trim()!==''){
        this.setState({
            disabledBtn:false,
            inputValue:e.target.value
        })
    }else{
        this.setState({
            disabledBtn:true
        })
    }
}
```
发布按钮事件

```
handlePressEnter=()=>{
    if(this.state.inputValue!==''){
        console.log('发布沸点')
    }else{
        message.error('请输入沸点内容');
    }
}
```

* 添加话题

![](https://user-gold-cdn.xitu.io/2019/4/3/169e2683b3cbf40b?w=1080&h=693&f=gif&s=329742)
如图所示，一个搜索框加一个话题列表。
dom结构如下：
```
<div style={{background:'#fff'}}>
    <Search
        placeholder="搜索话题"
        onSearch={value => this.handleSearch(value)}
        style={{ width: '100%' }}
    />
    <List
        itemLayout="horizontal"
        dataSource={this.state.topicListData}
        style={{height:'300px',overflow:'auto'}}
        renderItem={item => (
        <List.Item style={{cursor:'pointer'}} onClick={this.handleClick.bind(this,item.title)}>
            <List.Item.Meta
            avatar={<Avatar size={42} shape="square" src={item.img} />}
            title={item.title}
            description={<div>{item.followers}关注 {item.num}沸点</div>}
            />
        </List.Item>
        )}
    />
</div>
```
搜索框回车搜索事件：

```
handleSearch=(value)=>{
    const list=[...this.state.storeListData];
    if(value!==''){
        const searchList=list.filter(item=>item.title.indexOf(value)>-1);
        this.setState({
            topicListData:searchList
        })
    }else{
        this.setState({
            topicListData:list
        })
    }
}
```
话题列表点击事件：

```
handleClick=(val)=>{
    this.props.handleTopic(val);
}
```
这里向父组件传递了一个val代表当前点击话题，关于父子组件传值的相关说明前面文章做了介绍，这里不再赘述。

* 已发表的话题列表  
  使用ant-design的list组件实现
```
render() {
    const IconText = ({ type, text,tag }) => (
        <span onClick={this.handleClick.bind(this,tag)}>
            <Icon type={type} style={{ marginRight: 8 }} />
            {text}
        </span>
    );
    const PopoverContent=(id)=>{
        return <p style={{cursor:'pointer'}} onClick={this.handleReport.bind(this,id)}>举报</p>
    }
    return (
        <div>
            <List
                itemLayout="vertical"
                size="large"
                dataSource={this.state.listData}
                renderItem={item => (
                <List.Item
                    key={item.author}
                    actions={
                        [
                            <IconText type="like" text={item.likeNum===0?'赞':item.likeNum} tag='like' />,
                            <IconText type="message" text={item.commentNum===0?'评论':item.commentNum} tag='comment' />,
                            <IconText type="share-alt" text="分享" tag='share' />
                        ]
                    }
                    extra={
                        <div>
                            {!item.isFollowed && <Button style={{borderColor:'#6cbd45',color:'#6cbd45'}} onClick={()=>this.handleFollow(item.id)}>{item.isFollowed?'已关注':'关注'}</Button>}
                            <Popover placement="bottom" content={<PopoverContent author={item.author} />} trigger="click">
                                <span style={{cursor:'pointer',margin:'10px'}}>...</span>
                            </Popover>
                        </div>
                    }
                >
                    <List.Item.Meta
                    // avatar={<Avatar size={45} src={item.avatar} />}
                    avatar={<Popover placement="top" content={<PersonalPop info={item} handleFollow={(id)=>this.handleFollow(id)} />}>
                                <Avatar size={45} src={item.avatar} />
                            </Popover>}
                    title={<Popover placement="top" content={<PersonalPop info={item} handleFollow={(id)=>this.handleFollow(id)} />}>
                                <span style={{cursor:'pointer'}}>{item.author}</span>
                            </Popover>}
                    description={<div><span>{item.description}</span><span style={{margin:'0 5px'}}>·</span><span>{timeUtil.getTimeAgo(item.editTime)}</span></div>}
                    />
                    {item.content}
                </List.Item>
                )}
            />
        </div>
    );
}
```
用户头像鼠标滑过会出现详情，使用了popover组件，具体内容是单独抽离的一个PersonalPop组件。每一项根据isFollowed值判断是否显示关注按钮，如果没有关注，点击可关注。

* 右侧内容  
  右侧内容统一使用card组件实现，利用Redux获取登录用户信息。

```
render() {
    const { Meta } = Card;
    return (
        <div className='dynamicSide'>
            <Card style={{ width: '100%' }} className='card1'>
                <Meta
                    avatar={<Avatar size={62} src={this.props.userImage} />}
                    title={this.props.userId}
                    description={this.props.userDesc}
                    />
                <div>
                    <ul>
                        <li>
                            <p className='liTitle'>沸点</p>
                            <p className='liNum'>{this.state.userInfo.topNum}</p>
                        </li>
                        <li>
                            <p className='liTitle'>关注</p>
                            <p className='liNum'>{this.state.userInfo.following}</p>
                        </li>
                        <li>
                            <p className='liTitle'>关注者</p>
                            <p className='liNum'>{this.state.userInfo.follower}</p>
                        </li>
                    </ul>
                </div>
            </Card>
            <Card
                title="你可能感兴趣的人"
                style={{ width: '100%' ,marginTop:'10px'}}
                className='card2'
                >
                <List
                    itemLayout="horizontal"
                    dataSource={this.state.interestList}
                    renderItem={item => (
                    <List.Item actions={[<Button>关注</Button>]}>
                        <List.Item.Meta
                            avatar={<Avatar size={40} src={item.userImage} />}
                            title={item.user}
                            description={item.desc}
                        />
                    </List.Item>
                    )}
                />
                <div className='user-recommend-footer' onClick={this.changeInterestList.bind(this)}>
                    <Icon type="sync" />换一批
                </div>
            </Card>
            <div style={{marginTop:'10px'}}>
                <TopicCard title='关注的话题' link='/topics' list={this.state.attentionTopic} />
            </div>
            <div style={{marginTop:'10px'}}>
                <TopicCard title='更多话题' link='/topics' list={this.state.allTopic} />
            </div>
        </div>
    );
}
```
鉴于关注的话题和更多话题内容结构类似，故抽离为一个公用组件，且为函数式组件。

```
function TopicCard({ title,list,link }){
    return (
        <Card
            title={title}
            extra={<a href={link} style={{color:'#007fff'}}>全部></a>}
            style={{ width: '100%' }}
            className='topicCard'
            >
            <List
                itemLayout="horizontal"
                dataSource={list}
                renderItem={item => (
                <List.Item>
                    <List.Item.Meta
                        avatar={<Avatar shape="square" size={40} src={item.userImage} />}
                        title={item.title}
                        description={<div style={{color:'#8a9aa9'}}><span>{item.followNum}关注</span>·<span>{item.hotNews}沸点</span></div>}
                    />
                </List.Item>
                )}
            />
        </Card>
    );
}
```
>React组件可分为函数组件(Functional Component )和类组件(Class Component)，划分依据是根据组件的定义方式。函数组件使用函数定义组件，类组件使用ES6 class定义组件。  

函数组件的写法要比类组件简洁，不过类组件比函数组件功能更强大。类组件可以维护自身的状态变量，即组件的state，类组件还有不同的生命周期方法，可以让我们能够在组件的不同阶段（挂载、更新、卸载）对组件做更多的控制，进行不同的操作。但函数组件的使用可以从思想上让你在设计组件时进行更多思考，更加关注逻辑控制和显示的分离，设计出更加合理的组件结构。实际操作中，当一个组件不需要管理自身状态时，可以把它设计成函数组件，当你有足够的理由发现它需要“升级”为类组件时，再把它改造为类组件。因为函数组件“升级”为类组件是有一定成本的，这样就会要求你做这个改造前更认真地思考其合理性，而不是仅仅为了一时的方便就使用类组件。

## 话题页

![](https://user-gold-cdn.xitu.io/2019/4/3/169e286e44164261?w=1043&h=712&f=png&s=157895)
同样适用函数式组件

```
function TopicItem({item,showCount}){
    const {id,topicName,topicImage,topicCount,followedNum,topicNum,isFollowed}=item;
    return (
        <div className='topicItem'>
            <Badge count={showCount?topicCount:0} overflowCount={999}>
                <Avatar shape="square" size={72} src={topicImage} />
            </Badge>
            <div style={{marginLeft:'15px'}}>
                <div>
                    <NavLink to={`/topic/${id}`}>{topicName}</NavLink>
                </div>
                <div style={{color:'#8a9aa9',marginTop:'5px'}}>
                    <span>{followedNum}关注</span>·
                    <span>{topicNum}沸点</span>
                </div>
                <div className={isFollowed?'hasFollowed':'noFollow'}>
                    {isFollowed?'已关注':'+关注'}
                </div>
            </div>
        </div>
    );
}
```
关注的话题数据其实是全部话题的子集，在父页面调用的时候根据数据中的isFollowed属性进行一次筛选。

```
componentDidMount(){
    const list=[...this.state.allTopicList];
    const followedTopicList=list.filter(item=>item.isFollowed);
    this.setState({
        followedTopic:[...followedTopicList]
    })
}
```

## 小册页

![](https://user-gold-cdn.xitu.io/2019/4/3/169e28b8116fa193?w=1061&h=587&f=png&s=119560)

* 头部的导航

```
function TopNav({tags,changeLink}){
    return (
        <ul>
            {tags.map((item,index)=>{
                return <li key={item.path} onClick={()=>changeLink(item.path)}>
                    <NavLink to={`/books${item.path}`}>{item.text}</NavLink>
                </li>
            })}
        </ul>
    )
}
```

* 小册列表  
  使用ant-design中的list组件进行基本布局

```
render() {
    return (
        <div>
            <List
                size="large"
                bordered
                dataSource={this.state.bookList}
                renderItem={item => (
                    <List.Item className='bookList' onClick={()=>this.showInfo(item.bookId)}>
                        <img alt='books' src={item.img} />
                        <div>
                            <div style={{color:'#000',fontSize:'20px',fontWeight:400}}>
                                {item.isPresell && <span className="presale">预售</span> }
                                <span className='bookName'>{item.name}</span>
                            </div>
                            <div className='bookDesc'>{item.desc}</div>
                            <div className='bookAuthor'>
                                <NavLink to={`/user/:${item.userId}`}>
                                    <Avatar size={26} src={item.userImage} />
                                    <span style={{color:'#000',marginLeft:'5px'}}>{item.author}</span>
                                </NavLink>
                                <span style={{color:'#71777c',margin:'0 10px',whiteSpace:'nowrap',overflow:'hidden'}}>{item.selfDesc}</span>
                            </div>
                            <div className='other'>
                                {item.isBuy?<span className='bought'>已购买</span>:<span className='price'>￥{item.price}</span>}
                                <span className='message'>{item.chapterNum}小节</span>
                                {item.isBuy && <span className='message'>阅读时长{item.readTime}分</span>}
                                <span className='message'>{item.purchaseNum}购买</span>
                            </div>
                        </div>
                    </List.Item>
                )}
            />
        </div>
    );
}
```
会根据属性是否预售isPresell和是否已经购买isBuy来判断显示不同内容。  

## 相关文章

* [使用React构建精简版本掘金（一）](<https://rocky-191.github.io/2019/03/19/react-juejin/>)
* [使用React构建精简版本掘金（二）](<https://rocky-191.github.io/2019/03/20/react-juejin1/>)
* [使用React构建精简版本掘金（三）](<https://rocky-191.github.io/2019/03/20/react-juejin3/>)

相关详细代码可查看[github](https://github.com/rocky-191/react-juejin),不要忘了star哦！工作中主要是以vue作为主要技术栈，这是第一次使用React+React-router+Redux来构建项目，不足之处还请大家多多包涵。  

**金三已过，银四会是什么样子呢？**