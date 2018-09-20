---
title: jquery旋转动画
date: 2018-09-20 17:00:07
tags: 
	- jquery
	- css
categories: 'jquery'
---

最近一个项目中用到一个小动画，特此记录下。  

<!--more-->  
![](https://user-gold-cdn.xitu.io/2018/9/20/165f5e8089e1d73b?w=1150&h=690&f=gif&s=413807)
代码如下：

```
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="content-type" content="text/html; charset=UTF-8">
	<meta http-equiv="X-UA-Compatible" content=”IE=Edge,chrome=1″ />
	<title>jquery小动画</title>
	<style type="text/css">
		*{
			margin: 0;
			padding: 0;
		}
		body{
			background: #134f6d;
		}
		.tyCon {
		  position: relative;
		  width: 401px;
		  height: 401px;
		  margin: 0 auto;
		  margin-top: 300px;
		}
		.tyCon .ty {
		  width: 401px;
		  height: 142px;
		  border: 1px solid #fff;
		  border-radius: 90%;
		  opacity: 0;
		  position: absolute;
		  top: 0;
		  left: 0;
		  -webkit-transform-origin: 50% 50%;
		  -ms-transform-origin: 50% 50%;
		  transform-origin: 50% 50%;
		}
	</style>
</head>
<body>
	<div class="tyCon js-tyCon"></div>
	<script src="https://cdn.bootcss.com/jquery/3.3.0/jquery.min.js"></script>
	<script type="text/javascript">
		window.onload=function(){
			var tyHtml='';
		    var xzDeg=0;
		    for(var i=0;i<25;i++){
		    	tyHtml+='<div class="ty"></div>';
		    }
		    $(".js-tyCon").append(tyHtml);
		    
			var n=0;
			var interval=setInterval(xz,100);
		    function xz(){
		    	if(n==25){
		    		clearInterval(interval);
		    		return
		    	}
		    	xzDeg+=7.2;
		    	$(".js-tyCon .ty").eq(n).css({"opacity":"0.55","transform":"rotate("+xzDeg+"deg)"});
		    	n++
		    }
		}
	</script>
</body>
</html>
```
使用setInterval循环旋转椭圆。