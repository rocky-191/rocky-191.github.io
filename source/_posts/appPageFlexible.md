---
title: webapp字号大小跟随系统字号大小缩放
date: 2018-12-25 18:17:27
tags:
	- css
	- Html5
categories: "css"
---

最近做了一个webapp项目，混合式开发，外部原生，内部webview嵌套H5页面。前端方面采用了vue开发，适配采用的是flexible+rem做的适配。本来一切都很好，可是吧，领导说客户有的年纪大 ，看不清字体，希望网页字体可以跟随系统字号大小变化。当时心里真是...，然无奈只能想办法解决问题，网上搜罗一圈都是禁止内部跟随系统字号变化，看来只能自己搞了。  

<!--more-->

## 第一种方案

最简单的让原生进行操作，内部不做控制，外部放大，里面自己适应。但是有问题，文本字体可以放大，有的输入框和输入框的内容却没有放大，故淘汰该方案。
## 第二种方案
外部原生webview让里面的放大缩小不跟随系统变化，内部自己控制。和安卓同事商量后，他去获取系统放大的参数，然后将参数传递给内部webapp，内部来自定义控制缩放。  
代码如下：
```
setScaleFont(){
      let fontScale=1;
      let scaleFontSize;
      let initFontSize;
      fontScale=parseFloat(window._nativeMe.getFontScale());
      console.log(`缩放比例：${fontScale}`);
      let docHtml=document.getElementsByTagName("html")[0];
      initFontSize=this.getStyle(docHtml,'fontSize').split('px')[0];
      scaleFontSize=fontScale*initFontSize;//1-1.4等比缩放
      docHtml.style.fontSize=scaleFontSize +'px';
    },
    getStyle(obj, name){
      if(window.currentStyle){
        return obj.currentStyle[name];
      }
      else{
        return getComputedStyle(obj, false)[name];
      }
    }
```
先获取到初始的缩放比例，然后根据安卓原生传入的缩放比例改写html标签上的fontsize大小，由于采用了rem适配，自会根据根元素大小进行适配。这种方案必须确保先让flexible的适配先执行，然后判断是否是安卓，如果是安卓就执行setScaleFont方法才有效，否则会被flexible里面的方法覆盖掉，造成页面先放大后缩小或者先缩小后放大的现象。

![](https://user-gold-cdn.xitu.io/2018/12/25/167e4d972720a7e9?w=865&h=419&f=png&s=67996)
如上图，我是注释掉了这段代码，不然就会产生上述放大缩小的现象。

## 结论
该种方法也只能在安卓上有效，苹果由于安全权限的问题无法获取系统字体的缩放比例，故无法调整，如果有大神知道在苹果上如何操作或者有别的更好办法，请告知，不胜感激。