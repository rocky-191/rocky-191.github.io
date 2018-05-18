---
title: css 钟表
date: 2018-05-17 17:25:06  
categories: "css"
tags:  
	- css  
	- js
---
第一篇博文，就把以前的文章发上来试试吧。直接上代码。 
<!--more-->
  
		<!DOCTYPE html><html>
    <head>
        <meta charset="UTF-8">
        <title>钟表</title>
        <style type="text/css">
            body {
              background-color:#00A2D4;
            }

            .clock {
                width: 200px;
                height: 200px;
                background: -webkit-radial-gradient(#3b3b3b, #000);
                background: radial-gradient(#2E3F50, #0E1B29);
                box-shadow: inset 0px 0px 30px #131313, 0px 2px 18px rgba(0,0,0,0.5);
                border: 6px solid #172839;
                border-radius: 106px;
                margin: auto;
                position: absolute;
                top: 0; bottom: 0; left: 0; right: 0;
            }

            .hour-hand {
                width: 4px;
                height: 55px;
                background: #fff;
                box-shadow: 0px 0px 7px #000;
                position: absolute;
                top: 45px;
                left: 98px;
            }

            .minute-hand {
                width: 4px;
                height: 80px;
                background: #fff;
                box-shadow: 0px 0px 4px #000;
                position: absolute;
                top: 20px;
                left: 98px;
            }

            .second-hand {
                width: 2px;
                height: 80px;
                background: #bbb;
                box-shadow: 0px 0px 7px #000;
                position: absolute;
                top: 20px;
                left: 99px;
            }

            .pin {
                width: 10px;
                height: 10px;
                background: #222;
                border-radius: 10px;
                margin: auto;
                position: absolute;
                top: 0; bottom: 0; left: 0; right: 0;
            }

            .hour-hand,
            .minute-hand,
            .second-hand {
                -webkit-transform-origin: 50% 100%;
                -moz-transform-origin: 50% 100%;
                -o-transform-origin: 50% 100%;
                -ms-transform-origin: 50% 100%;
                transform-origin: 50% 100%;
            }
            .circle{
                width:20px;
                height:20px;
                line-height:20px;
                text-align:center;
                color:#fff;
                position: absolute;
            }
        </style>
    </head>
    <body>
        <div class="clock">
            <div class="minute-hand"></div>
            <div class="hour-hand"></div>
            <div class="second-hand"></div>
            <div class="pin"></div>
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
            <div class="circle">7</div>
        </div>
        <div class="time"></div>
        <script type="text/javascript">
            window.onload=function(){
                setInterval(function(){
                    var dt = new Date();
                    var sec_deg = dt.getSeconds() * (360/60);
                    var min_deg = dt.getMinutes() * (360/60);
                    var hr_deg = dt.getHours() * (360/12) + dt.getMinutes() * (360/60/12);          
                    document.querySelector(".clock .second-hand").style.cssText='-webkit-transform:rotate(' + sec_deg + 'deg)','-moz-transform:rotate(' + sec_deg + 'deg)', '-o-transform:rotate(' + sec_deg + 'deg)', '-ms-transform:rotate(' + sec_deg + 'deg)', 'transform:rotate(' + sec_deg  + 'deg)';
                    document.querySelector('.clock .minute-hand').style.cssText='-webkit-transform:rotate(' + min_deg + 'deg)', '-moz-transform:rotate(' + min_deg + 'deg)', '-o-transform:rotate(' + min_deg + 'deg)', '-ms-transform:rotate(' + min_deg + 'deg)', 'transform:rotate(' + min_deg + 'deg)';
                    document.querySelector('.clock .hour-hand').style.cssText='-webkit-transform:rotate(' + hr_deg + 'deg)', '-moz-transform:rotate(' + hr_deg + 'deg)', '-o-transform:rotate(' + hr_deg + 'deg)', '-ms-transform:rotate(' + hr_deg + 'deg)', 'transform:rotate(' + hr_deg + 'deg)';
                  }, 1000);
                var dx=90,
                    dy=90,
                    s=87,//半径
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
            }
        </script>
    </body>
</html>