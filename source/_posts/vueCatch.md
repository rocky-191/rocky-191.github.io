---
title: vue移动端项目缓存问题实践
date: 2018-10-28 18:04:08
tags: 
	- vue
categories: "vue"
---

最近在做一个vue移动端项目，被缓存问题搞得头都大了，积累了一些经验，特此记录总结下，权当是最近项目问题的一个回顾吧！    

<!--more-->

先描述下问题场景：A页面->B页面->C页面。假设A页面是列表页面，B页面是列表详情页面，C页面是操作改变B页面的一些东西，进行提交类似的操作。A页面进入B页面，应该根据不同的列表item显示不一样的详情，从B进入C，也应该根据item的标识比如ID展示不一样的内容，在C页面操作后，返回B页面，B页面数据发生变化。这个时候会有两种情况：  
* C页面操作数据后返回B页面，B页面对应数据应该发生变化。
* C页面直接点击左上角箭头返回，B页面对应数据不应该发生变化。继续返回A列表页面，换一条数据，继续进入B页面，B页面展示不同内容，进入C页面，C页面刷新展示不同内容  

另一种情况发生在写邮件的页面中，添加收件人，选人之后，继续添加，之前添加的联系人应该存在。但是从写邮件页面返回邮件列表再次进入写邮件页面，之前添加过的联系人数据就不应该存在了，这里就涉及到如何处理缓存，何时使用缓存，何时清除缓存的问题了。  

目前项目整体结构如下：  

```
<template>
  <div id="app">
    <keep-alive>
      <router-view v-if="$route.meta.keepAlive"></router-view>
    </keep-alive>
    <router-view v-if="!$route.meta.keepAlive"></router-view>
  </div>
</template>
```
虽然官方提供了include，exclude，可以让我们决定哪些组件使用缓存，哪些不使用缓存，但是并没有解决我们想动态使用缓存的目的，目前我的项目使用了如下两种方式处理缓存：
## 方式一  ，使用是否使用缓存标识
在路由文件router.js里给每个路由添加meta信息，标识是否使用缓存。

```
meta: {
    isUseCache: false,//不使用缓存
    keepAlive: true
}
```
使用方式：    
A->B，B不能缓存;B->A,A缓存。
* （1）A页面：
```
beforeRouteLeave(to, from, next) {
  // 设置下一个路由的 meta
  if(to.path=='/B'){
    to.meta.isUseCache = false;
  }
  next();
},
activated(){
    if(!this.$route.meta.isUseCache){
        this.getData();
    }
}  
```
* (2) B页面
```
beforeRouteLeave(to, from, next) {
  // 设置下一个路由的 meta
  if(to.path=='/A'){
    to.meta.isUseCache = true;
  }
  next();
},
activated(){
    if(!this.$route.meta.isUseCache){
        this.getData();
    }
}  
```

## 方式二，强制清除缓存。
这种方式是从网上找的一种方式，使用了vue内部组件<keep-alive></keep-alive>之后，不在支持动态销毁组件，缓存一直存在，只能从源头上下手，清掉缓存。

```
export const removeCatch = {
  beforeRouteLeave:function(to, from, next){
    if (from && from.meta.rank && to.meta.rank && from.meta.rank>to.meta.rank)
      {//此处判断是如果返回上一层，你可以根据自己的业务更改此处的判断逻辑，酌情决定是否摧毁本层缓存。
          if (this.$vnode && this.$vnode.data.keepAlive)
          {
              if (this.$vnode.parent && this.$vnode.parent.componentInstance && this.$vnode.parent.componentInstance.cache)
              {
                  if (this.$vnode.componentOptions)
                  {
                      var key = this.$vnode.key == null
                                  ? this.$vnode.componentOptions.Ctor.cid + (this.$vnode.componentOptions.tag ? `::${this.$vnode.componentOptions.tag}` : '')
                                  : this.$vnode.key;
                      var cache = this.$vnode.parent.componentInstance.cache;
                      var keys  = this.$vnode.parent.componentInstance.keys;
                      if (cache[key])
                      {
                          if (keys.length) {
                              var index = keys.indexOf(key);
                              if (index > -1) {
                                  keys.splice(index, 1);
                              }
                          }
                          delete cache[key];
                      }
                  }
              }
          }
          this.$destroy();
      }
      next();
  }
};

```
在需要清掉缓存的页面混合引入该js即可。
[原文链接](http://wanyaxing.com/blog/20180723114341.html)  

## 结语
移动端的缓存真是麻烦啊，前进后退，什么时候使用缓存，什么时候不使用缓存，都需要经过仔细的处理，不然就会有想不到的问题。不过经过这次项目，也积累了一定的经验。如果有大佬有别的更好的解决办法，还请告知，多谢！还是那句话，有问题就去解决，不要害怕问题，解决了问题，你就会成长！ 
