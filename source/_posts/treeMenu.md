---
title: 多层级组织结构树
date: 2018-07-13 10:59:36
tags:  
	- vue
	- css
categories: "vue"
---

最近公司使用vue开发手机端项目，有个需求是实现派发树，网上找了半天没找到合适的。决定自己撸一个试试，要求多层级，层级不定而且。  

<!--more-->

# 正文
treeMenu.vue文件
```
<template>
  <div>
    <ul class="distributionTree">
      <li v-for="(item,index) in data" :key="index">
        <div class="treeContent0">
          <span v-if="mode==item.orgType || mode=='2'">
            <span v-if="assignType=='1'">
              <span class="checkbox" :class="{'checkbox_active':item.checkboxActive}" @click="item.checkboxActive=!item.checkboxActive"></span>
            </span>
            <span v-else-if="assignType=='0'">
              <span class="radio" :class="{'radio_active':item.radioActive}" @click="radionToggle(item,data)"></span>
            </span>
          </span>
          <label>{{item.text}}</label>
          <span class="nextSet" :class="{'nextSet_active':item.nextSetActive}" v-if="item.orgType!='0'" @click="toggle(item, index)"></span>
        </div>
        <tree-menu v-on="$listeners" :data="item.org" :assignType="assignType" :mode="mode" v-if="item.nextSetActive"></tree-menu>
      </li>
    </ul>
  </div>
</template>
```
data就是派发树的内容，我们项目实际需要判断是否显示单选钮和多选钮，通过orgType 和mode判断，orgType 是每一条数据里的一项，mode是上一层传过来的数据信息。然后通过assignType判断是单选钮还是多选钮，当然assignType也是上一层调用树组件传过来的数据啦。  
其中用到了一个知识点：v-on="$listeners"，用来监听多层级树中的事件。之前写的一篇文章有介绍该内容[vue项目技术随笔](https://rocky-191.github.io/2018/07/12/vueDevDiary1/#more)，欢迎阅读。  
一些相关的methods如下:  

```
methods:{
  toggle (item, index) {
    item.nextSetActive=!item.nextSetActive;
    if (item.org==null ||item.org.length==0) {
      Indicator.open('加载中...');
      this.$emit('getsubmenu', item,()=>{
        Indicator.close();
      });
    }
  },
  radionToggle(item,data){
    data.forEach((e,index)=>{
      e.radioActive=false;
    });
    item.radioActive=!item.radioActive;
  }
}
```
该移动端项目中使用了mintui作为ui框架，使用了其中的加载效果，点击展现下一层级的时候出现加载中，提升用户体验。toggle作用展示和隐藏下一层级，如果下一层级没有数据，需要触发动作去取数据，呈现加载中，取到数据后，回调函数，让加载中提示效果消失。  
radionToggle方法就是实现单选钮的选中效果了，很简单。看一下最终的效果呈现：
![](https://user-gold-cdn.xitu.io/2018/7/13/164918db32f78538?w=750&h=1333&f=png&s=108657)
具体的css就靠大家自己发挥了，不同的设计不同的css，感谢大家的支持。