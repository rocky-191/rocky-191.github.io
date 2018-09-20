---
title: 圆环周围均匀分布内容
date: 2018-05-18 17:20:31
categories: "css"  
tags:
	- css
	- js
---
逐渐迁移别的技术论坛博文到该博客，这是之前写的一篇文章。项目中用到了一个圆环周围均匀分布内容。直接上代码，毕竟对程序员来说talk is cheap,show me code.  
<!--more-->
  
		<html><head><meta http-equiv="Content-Type" content="text/html; charset=utf-8" /><meta http-equiv="X-UA-Compatible" content=”IE=Edge,chrome=1″ /><meta name="renderer" content="webkit"> 
    <title>无标题</title>
    <style type="text/css">
        .centerDiv{
            width:600px;
            height:600px;
            margin:0 auto;
            margin-top:200px;
            position:relative;
        }
        .dashedCircle{
            height:300px;
            width:300px;
            border-radius: 100%;
            border: 1px dashed #494949;
            margin: 0 auto;
        }
        .circle,.circleCenter{
            width:40px;
            height:40px;
            line-height:40px;
            text-align:center;
            color:#fff;
            background:#97e49a;
            border-radius: 100%;
            position: absolute;
        }

        .circleCenter{
            width:20px;
            height:20px;
            line-height:20px;
        }
    </style></head><body>
    <div class="centerDiv">
        <div class="dashedCircle"></div>
        <div class="circleCenter"></div>
        <div class="circle">7</div>
        <div class="circle">6</div>
        <div class="circle">5</div>
        <div class="circle">4</div>
        <div class="circle">3</div>
        <div class="circle">2</div>
        <div class="circle">1</div>
        <div class="circle">12</div>
        <div class="circle">11</div>
        <div class="circle">10</div>
        <div class="circle">9</div>
        <div class="circle">8</div>
    </div>
    <script type="text/javascript">
        window.onload=function(){
            var dx=280,
                dy=130,
                s=180,//半径
                x=Math.sin(0),
                y=Math.cos(0),
                dig=2*Math.PI/12;
            var circle=document.querySelectorAll(".circle");
            for(var i=0;i<12;i++){
                var x=Math.sin(i*dig);
                var y=Math.cos(i*dig);
                var topValue=Number(dy+y*s),
                    leftValue=Number(dx+x*s);       
                circle[i].style.top=topValue+"px";
                circle[i].style.left=leftValue+"px";
            }
            var circleCenterObj=document.querySelector(".circleCenter");
            circleCenterObj.style.top=Number(dy)+10;//10是中心小圆半径
            circleCenterObj.style.left=Number(dx)+10;
        }
    </script></body></html>