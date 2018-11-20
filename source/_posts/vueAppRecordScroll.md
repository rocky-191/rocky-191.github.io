---
title: vue移动端项目记录滚动位置方案总结
date: 2018-11-20 21:53:37
tags:  
	- vue
categories: 'vue'
---

最近在用vue做移动端项目，列表展示页面有很多条数据，在滑动到下面的时候，点到对应详情页，此时后退，应该在之前滚动的位置，不能滚动到顶部，这就需要记录滚动位置。这是需求，说完了需求，就考虑怎么实现吧！   

<!--more--> 

## （1）利用vue内置组件keep-alive缓存页面
在路由文件router.js中设置缓存，方式如下
```
{
    path: '/home',
    name: 'home',
    component: resolve => require(['@/components/process/home'], resolve),
    meta: {
        keepAlive: true
    }
}
```
这样的话，后退的时候就不会刷新页面，完成第一步。
## （2）监听滚动条滚动位置
给滚动元素父级设置一个别名ref值，比如叫scroll吧。

```
let that=this;
this.$refs.scroll.addEventListener("scroll",that.recordScrollPosition,true);    //添加绑定事件
```
recordScrollPosition是一个方法，用于记录滚动位置。这里有一个需要注意的点，必须先把this对象存储一下，不然的话直接用this.recordScrollPosition是无法调用到recordScrollPosition，因为此时this指向的是当前dom节点，而不是vue实例，自然无法调用到methods里定义的方法，当时这个地方没注意，卡了好久。  
这个监听的动作要写在activated周期里，每次进入都要监听。

```
activated(){
    this.$refs.scroll.scrollTop = this.list_top;  
    let that=this;
    this.$refs.scroll.addEventListener("scroll",that.recordScrollPosition,true); 
  },
  deactivated(){
    let that=this;
    this.$refs.scroll.removeEventListener("scroll",that.recordScrollPosition,true); 
  }
```
当然最后应该销毁监听。
## （3）将滚动位置存储起来
这里存储滚动位置的话可以将其存到localStorage里，也可以将其存到vuex里，这里我为了方便就把它存到vuex里了。

```
recordScrollPosition(e) {
  this.$store.dispatch("setListTop",e.target.scrollTop);//实时存入到vuex中
}
```
## (4)vuex里设置

```
const state = {
    top:0 //滚动条初始位置
}
const mutations={
    setHomeListTop(state,top){
      state.top=top;
    }
}
const getters = {
    list_top(state){
      return state.top;
    }
}

const actions = {
  setListTop({commit,state},top){
    commit('setHomeListTop',top);
  }
}
```
到此大功告成，详细的代码我写了一个demo，放到了github上。[传送门](https://github.com/rocky-191/vueAppRecordScroll)