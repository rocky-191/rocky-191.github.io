---
title: vue项目开发记录随笔
date: 2018-07-12 11:20:13
tags:  
	- vue
categories: "vue"
---

最近都在做vue相关的项目，在公司推行前后端分离，重构以前的项目，真的好忙，一个项目接着一个，爬不完的坑，不说了，说多了都是眼泪。开始正文了！！！  

<!--more-->

* Vue 路由切换时页面内容没有重新加载  
  问题原因：在组件mounted钩子中调用的刷新页面内容，但测试发现这个钩子没有被调用。后来发现App.vue中使用了<keep-alive>：
  keep-alive是Vue的内置组件，能在组件切换过程中将状态保留在内存中，防止重复渲染DOM。这就是问题所在了。

解决办法：

使用Vue组件切换过程钩子activated(keep-alive组件激活时调用)，而不是挂载钩子mounted：

```

<script>
export default {
  activated: function() {
    this.getData()
  }
}
</script>
```
参考网址[关于keep-alive组件的钩子：https://cn.vuejs.org/v2/api/#activated](https://cn.vuejs.org/v2/api/#activated)

* 文件导出方式  
  项目中涉及到文件导出，分xml和excel导出。不同的文件导出格式不同，需要根据文件类型判断导出格式。

```
exportAllData(val){
    //全部导出
    if(!val){
      this.exportFile(this.exportAllType);
    }
},
exportFile(exportType){
    let url='';//接口地址
    this.$axios.get(url,{responseType: 'arraybuffer'}).then(res => {
      this.download(res.data,exportType);
    },res => {
      this.$Message.error('导出失败');
    });
},
download (data,exportType) {
    if (!data) {
      return
    }
    let exportGs='';
    if(exportType==='excel'){
      exportGs='application/vnd.ms-excel';
    }else if(exportType==='xml'){
      exportGs='text/xml';
    }
    let url = window.URL.createObjectURL(new Blob([data],{type: exportGs}));
    let link = document.createElement('a')
    link.style.display = 'none'
    link.href = url;
    link.setAttribute('download', '文件');
    document.body.appendChild(link)
    link.click();
 }
```

* 在vue多层次组件监听动作和属性的时候可以使用如下方式进行监听
```
v-bind="$attrs" v-on="$listeners"
```
Vue 2.4 版本提供了这种方法，将父组件中不被认为 props特性绑定的属性传入子组件中，通常配合 interitAttrs 选项一起使用。之所以要提到这两个属性，是因为两者的出现使得组件之间跨组件的通信在不依赖 vuex 和事件总线的情况下变得简洁，业务清晰。

![官网API](vueDevDiary1\imageAPI.png)
比如组件A=>B组件=>C组件等，这种多层级组件，A组件向C组件传递数据或者C组件的事件要触发A组件中的事件的话，就可以在B组件中写成
```
<template>
 <div>
    <span>{{child1}}<span>
 <!-- C组件中能直接触发test的原因在于 B组件调用C组件时 使用 v-on 绑定了$listeners 属性 -->
 <!-- 通过v-bind 绑定$attrs属性，C组件可以直接获取到A组件中传递下来的props（除了B组件中props声明的） -->
 <c v-bind="$attrs" v-on="$listeners"></c>
 </div>
</template>
<script>
 import c from './c.vue';
 export default {
 props: ['child1'],
 data () {
 return {};
 },
 inheritAttrs: false,
 components: { c },
 mounted () {
    this.$emit('test1');
 }
 };
</script>
```
C组件
```
<template>
 <div>
    <span>{{child2}}<span>
 </div>
</template>
<script>
 export default {
 props: [child2'],
 data () {
 return {};
 },
 inheritAttrs: false,
 mounted () {
    this.$emit('test2');
 }
 };
</script>
```
A组件：
```
<template>
 <div id="app">
    <b :child1="child1" :child2="child2" @test1="test1" @test2="test2"></b>
 </div>
</template>
<script>
 import b from './b.vue';
 export default {
 data () {
 return {
     child1:'hello child1',
     child2:'hello child2'
 };
 },
 components: { b },
 methods: {
 test1 () {
 console.log('test1');
 },
 test2 () {
 console.log('test2');
 }
 }
 };
</script>
```
记录日常开发中用到的一些知识点，权当一次总结吧。