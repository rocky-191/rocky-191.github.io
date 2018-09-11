---
title: '常用函数集合整理'
date: 2018-09-11 10:52:41
tags:	
	- js
categories: 'js'
---

收集整理日常工作中会用到的一些常用函数

<!--more-->

# 函数防抖

```javascript
/**
 * @desc 函数防抖
 * @param func 函数
 * @param wait 延迟执行毫秒数
 * @param immediate true 表立即执行，false 表非立即执行
 */
function debounce(func,wait,immediate) {
    var timeout;

    return function () {
        var context = this;
        var args = arguments;

        if (timeout) clearTimeout(timeout);
        if (immediate) {
            var callNow = !timeout;
            timeout = setTimeout(function(){
                timeout = null;
            }, wait)
            if (callNow) func.apply(context, args)
        }
        else {
            timeout = setTimeout(function(){
                func.apply(context, args)
            }, wait);
        }
    }
}
```

# 时间戳转化成时间格式

```javascript
add0(m) {
    return m < 10 ? '0' + m : m;
},
    formatDate(myTime) {
        let time = new Date(myTime);
        var year = time.getFullYear();
        var month = time.getMonth() + 1;
        var date = time.getDate();
        var hours = time.getHours();
        var minutes = time.getMinutes();
        var seconds = time.getSeconds();
        return year + '-' + this.add0(month) + '-' + this.add0(date) + ' ' + this.add0(hours) + ':' + this.add0(minutes); // + ':' + this.add0(seconds)
    }
```

# 时间转成时间戳

```javascript
var date = new Date('2014-04-23 18:55:49:123');
// 有三种方式获取
var time1 = date.getTime();
var time2 = date.valueOf();
var time3 = Date.parse(date);
console.log(time1);//1398250549123
console.log(time2);//1398250549123
console.log(time3);//1398250549000
```

## 备注

以上三种获取方式的区别：

第一、第二种：会精确到毫秒

第三种：只能精确到秒，毫秒用000替代

以上三个输出结果可观察其区别

注意：获取到的时间戳除以1000就可获得Unix时间戳，就可传值给后台得到。

# 固定范围随机数生成

```javascript
getRandom(x, y) {
    let random = Math.round(Math.random() * (y - x) + x);
    return random;
}
```

# 函数节流

```javascript
/**
 * @desc 函数节流
 * @param func 函数
 * @param wait 延迟执行毫秒数
 * @param type 1 表时间戳版，2 表定时器版
 */
function throttle(func, wait ,type) {
    if(type===1){
        var previous = 0;
    }else if(type===2){
        var timeout;
    }
    return function() {
        var context = this;
        var args = arguments;
        if(type===1){
            var now = Date.now();
            if (now - previous > wait) {
                func.apply(context, args);
                previous = now;
            }
        }else if(type===2){
            if (!timeout) {
                timeout = setTimeout(function(){
                    timeout = null;
                    func.apply(context, args)
                }, wait)
            }
        }
    }
}
```

# 获取当前可视范围的高度

```javascript
getClientHeight() {
    var clientHeight = 0;
    if (document.body.clientHeight && document.documentElement.clientHeight) {
        clientHeight = Math.min(document.body.clientHeight, document.documentElement.clientHeight);
    } else {
        clientHeight = Math.max(document.body.clientHeight, document.documentElement.clientHeight);
    }
    return clientHeight;
}
```

# 获取滚动条距离顶部高度

```javascript
getScrollTop() {
    var scrollTop = 0;
    if (document.documentElement && document.documentElement.scrollTop) {
        scrollTop = document.documentElement.scrollTop;
    } else if (document.body) {
        scrollTop = document.body.scrollTop;
    }
    return scrollTop;
}
```

# 获取文档完整高度

```javascript
getScrollHeight() {
    return Math.max(document.body.scrollHeight, document.documentElement.scrollHeight);
}
```

最近几个项目用到就这么多，还会陆续添加的！！！