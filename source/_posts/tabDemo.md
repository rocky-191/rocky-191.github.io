---
title: tab切换巧妙布局
date: 2018-05-19 11:16:56
tags:  
	-css
	-js
categories: "html"
---
一个项目中用到了tab切换，无奈内容太多，不能按照往常的一般形式排列tab。UI设计了一种类似于蜂巢的tab切换。如图所示  
![示意图](tabDemo/tab.png)  
<!--more-->
代码如下  
```
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="content-type" content="text/html; charset=UTF-8">
        <title>蜂巢tab切换实例</title>
        <meta http-equiv="keywords" content="">
        <meta http-equiv="description" content="">
        <meta http-equiv="X-UA-Compatible" content=”IE=Edge,chrome=1″ />
        <meta name="renderer" content="webkit">
        <style type="text/css">
             .Ct{
                 width:1000px;
                 margin:0 auto;
             }
.ywPart1{
    width: 310px;
    margin: 0 auto;
    padding: 20px 10px 20px 30px;
}

/**蜂巢样式***/
.hex { 
    float: left; 
    margin-left: 2px; 
    margin-bottom: -18px;
    cursor: pointer; 
} 
.hex .hex_top { 
    width: 0; 
    border-bottom: 20px solid #f0f0f0; 
    border-left: 35px solid transparent; 
    border-right: 35px solid transparent;
    margin-top: -1px\9\0;
    margin-top: 0px\0; 
} 
.hex .hex_middle { 
    width: 70px; 
    height: 30px;
    line-height: 30px; 
    background: #f0f0f0;
    text-align: center;
    font-family: "宋体";
    color: #333333; 
} 
.hex .hex_bottom { 
    width: 0; 
    border-top: 20px solid #f0f0f0; 
    border-left: 35px solid transparent; 
    border-right: 35px solid transparent; 
    margin-top: -1px\9\0;
    margin-top: 0px\0; 
} 
.hex-row { 
    clear: left; 
} 
.hex-row.even { 
    margin-left: 38px; 
}
.hex-row.odd { 
    margin-right: 38px; 
}

.hex.hexActive .hex_top { 
    width: 0; 
    border-bottom: 20px solid #86ccac; 
    border-left: 35px solid transparent; 
    border-right: 35px solid transparent;
    margin-top: -1px\9\0;
    margin-top: 0px\0; 
} 
.hex.hexActive .hex_middle { 
    width: 70px; 
    height: 30px;
    line-height: 30px; 
    background: #86ccac;
    text-align: center;
    font-family: "宋体";
    color: #FFFFFF; 
} 
.hex.hexActive .hex_bottom { 
    width: 0; 
    border-top: 20px solid #86ccac; 
    border-left: 35px solid transparent; 
    border-right: 35px solid transparent;
    margin-top: -1px\9\0;
    margin-top: 0px\0;  
}
/**蜂巢样式***/
        </style>
    </head>
    <body>
        <div class="Ct">
            <div class="ywPart1" id="hexDiv">
                <!--
                    描述：蜂巢
                -->
                <div class="hex hexActive" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">即时通信</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">阅读</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">微博</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">导航</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex hex-row even" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">视频</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">音乐</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">应用商店</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex hex-row" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">游戏</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">支付</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">动漫</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">P2P业务</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex hex-row even" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">VOIP业务</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">彩信</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">浏览下载</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex hex-row" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">财经</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">安全杀毒</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">邮箱</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="hex" name="hexBlcok">
                    <div class="hex_top"></div>
                    <div class="hex_middle">其他</div>
                    <div class="hex_bottom"></div>
                </div>
                <div class="ClearFloat"></div>
            </div>
        </div>
    <script src="//cdn.bootcss.com/jquery/2.0.0/jquery.min.js"></script>
    <script>
        $("#hexDiv").find("div[name='hexBlcok']").on("click",function(){
            $(this).addClass("hexActive");
            $(this).siblings("div[name='hexBlcok']").removeClass("hexActive");
        });
    </script>
    </body>
</html>
```