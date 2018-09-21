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

# 获取字符串长度（汉字算两个字符，字母数字算一个）

```javascript
function getByteLen(val) {
  var len = 0;
  for (var i = 0; i < val.length; i++) {
    var a = val.charAt(i);
    if (a.match(/[^\x00-\xff]/ig) != null) {
      len += 2;
    }
    else {
      len += 1;
    }
  }
  return len;
}
```

# 获取字符串字节长度

```javascript
/**
 * 获取字符串字节长度
 * @param {String}
 * @returns {Boolean}
 */
function checkLength(v) {
  var realLength = 0;
  var len = v.length;
  for (var i = 0; i < len; i++) {
    var charCode = v.charCodeAt(i);
    if (charCode >= 0 && charCode <= 128) realLength += 1;
    else realLength += 2;
  }
  return realLength;
}
```

# 限制字数

```javascript
/*
  * 限制字数用, 一个汉字算一个字,两个英文/字母算一个字
  */
function getByteVal(val, max) {
  var returnValue = '';
  var byteValLen = 0;
  for (var i = 0; i < val.length; i++) {
      if (val[i].match(/[^\x00-\xff]/ig) != null)
      byteValLen += 1;
      else
      byteValLen += 0.5;
      if (byteValLen > max)
      break;
      returnValue += val[i];
  }
  return returnValue;
}
```

# 限制字符数

```javascript
/*
  * 限制字符数用, 一个汉字算两个字符,一个英文/字母算一个字符
  */
function getCharVal(val, max) {
  var returnValue = '';
  var byteValLen = 0;
  for (var i = 0; i < val.length; i++) {
      if (val[i].match(/[^\x00-\xff]/ig) != null)
      byteValLen += 2;
      else
      byteValLen += 1;
      if (byteValLen > max)
      break;
      returnValue += val[i];
  }
  return returnValue;
}
```

# 判断数组中是否有重复值

```javascript
// 判断数组中是否有重复值
function isRepeat(arr){
  var hash = {};
  for(var i in arr) {
    if(hash[arr[i]]){
       return true;
     }
     hash[arr[i]] = true;
  }
  return false;
}
```

# 截取指定字节的字符串

```javascript
/**
 * 截取指定字节的字符串
 * @param str 要截取的字符穿
 * @param len 要截取的长度，根据字节计算
 * @param suffix 截取前len个后，其余的字符的替换字符，一般用“…”
 * @returns {*}
 */
function cutString(str, len, suffix) {
  if (!str) return "";
  if (len <= 0) return "";
  if (!suffix) suffix = "";
  var templen = 0;
  for (var i = 0; i < str.length; i++) {
    if (str.charCodeAt(i) > 255) {
      templen += 2;
    } else {
      templen++
    }
    if (templen == len) {
      return str.substring(0, i + 1) + suffix;
    } else if (templen > len) {
      return str.substring(0, i) + suffix;
    }
  }
  return str;
}
```

# 判断是否是微信浏览器

```javascript
/**
 * 判断微信浏览器
 * @returns {Boolean}
 */
function isWeiXin() {
  var ua = window.navigator.userAgent.toLowerCase();
  if (ua.match(/MicroMessenger/i) == 'micromessenger') {
    return true;
  } else {
    return false;
  }
}
```

# 获取时间格式的实例

```javascript
function getTimeFormat(time) {
  var date = new Date(parseInt(time) * 1000);
  var month, day, hour, min;
  parseInt(date.getMonth()) + 1 < 10 ? month = '0' + (parseInt(date.getMonth()) + 1) : month = parseInt(date.getMonth()) + 1;
  date.getDate() < 10 ? day = '0' + date.getDate() : day = date.getDate();
  date.getHours() < 10 ? hour = '0' + date.getHours() : hour = date.getHours();
  date.getMinutes() < 10 ? min = '0' + date.getMinutes() : min = date.getMinutes();
  return [month, day].join('-') + '  ' + hour + ':' + min
}

function getTimeFormatYMD(time) {
  var date = new Date(parseInt(time) * 1000);
  var year, month, day;
  year = date.getFullYear();
  parseInt(date.getMonth()) + 1 < 10 ? month = '0' + (parseInt(date.getMonth()) + 1) : month = parseInt(date.getMonth()) + 1;
  date.getDate() < 10 ? day = '0' + date.getDate() : day = date.getDate();
  return [year, month, day].join('-')
}

function getTimeFormatAll(time) {
  var date = new Date(parseInt(time) * 1000);
  var year, month, day, hour, min, second;
  year = date.getFullYear();
  parseInt(date.getMonth()) + 1 < 10 ? month = '0' + (parseInt(date.getMonth()) + 1) : month = parseInt(date.getMonth()) + 1;
  date.getDate() < 10 ? day = '0' + date.getDate() : day = date.getDate();
  date.getHours() < 10 ? hour = '0' + date.getHours() : hour = date.getHours();
  date.getMinutes() < 10 ? min = '0' + date.getMinutes() : min = date.getMinutes();
  date.getSeconds() < 10 ? second = '0' + date.getSeconds() : second = date.getSeconds();

  return [year, month, day].join('-') + '  ' + hour + ':' + min + ':' + second
}
```

# 对象的克隆和拷贝

```javascript
/**
 * 对象克隆&深拷贝
 * @param obj
 * @returns {{}}
 */
function cloneObj(obj) {
  var newO = {};
  if (obj instanceof Array) {
    newO = [];
  }
  for (var key in obj) {
    var val = obj[key];
    newO[key] = typeof val === 'object' ? arguments.callee(val) : val;
  }
  return newO;
};

//克隆拷贝增强版
/**
 * 对象克隆&深拷贝
 * @param obj
 * @returns {{}}
 */
function clone(obj) {
  // Handle the 3 simple types, and null or undefined
  if (null == obj || "object" != typeof obj) return obj;
  // Handle Date
  if (obj instanceof Date) {
    var copy = new Date();
    copy.setTime(obj.getTime());
    return copy;
  }
  // Handle Array
  if (obj instanceof Array) {
    var copy = [];
    for (var i = 0,
    len = obj.length; i < len; ++i) {
      copy[i] = clone(obj[i]);
    }
    return copy;
  }
  // Handle Object
  if (obj instanceof Object) {
    var copy = {};
    for (var attr in obj) {
      if (obj.hasOwnProperty(attr)) copy[attr] = clone(obj[attr]);
    }
    return copy;
  }
  throw new Error("Unable to copy obj! Its type isn't supported.");
}
// 测试用例：
// var origin = {
//   a: "text",
//   b: null,
//   c: undefined,
//   e: {
//     f: [1, 2]
//   }
// }
```

# 组织机构代码验证

```javascript
// 验证规则：
// 组织机构代码是每一个机关、社会团体、企事业单位在全国范围内唯一的、始终不变的法定代码标识。最新使用的组织机构代码在1997年颁布实施，由8位数字（或大写拉丁字母）本体代码和1位数字（或大写拉丁字母）校验码组成。本体代码采用系列（即分区段）顺序编码方法。校验码按下列公式计算：8 C9 = 11 - MOD(∑Ci * Wi，11)… (2) i = 1其中：MOD——表示求余函数；i——表示代码字符从左到右位置序号；Ci——表示第i位置上的代码字符的值，采用附录A“代码字符集”所列字符；C9——表示校验码；Wi——表示第i位置上的加权因子，其数值如下表：i 1 2 3 4 5 6 7 8 Wi 3 7 9 10 5 8 4 2当MOD函数值为1（即C9 = 10）时，校验码用字母X表示。
// 验证方法：
function checkOrgCodeValid(el) {
  var txtval = el.value;
  var values = txtval.split("-");
  var ws = [3, 7, 9, 10, 5, 8, 4, 2];
  var str = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  var reg = /^([0-9A-Z]){8}$/;
  if (!reg.test(values[0])) {
    return false
  }
  var sum = 0;
  for (var i = 0; i < 8; i++) {
    sum += str.indexOf(values[0].charAt(i)) * ws[i];
  }
  var C9 = 11 - (sum % 11);
  var YC9 = values[1] + '';
  if (C9 == 11) {
    C9 = '0';
  } else if (C9 == 10) {
    C9 = 'X';
  } else {
    C9 = C9 + '';
  }
  return YC9 == C9;
}

```

# 验证身份证号

```javascript
/**
 * 验证身份证号
 * @param el 号码输入input
 * @returns {boolean}
 */
function checkCardNo(el) {
  var txtval = el.value;
  var reg = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/;
  return reg.test(txtval)
}
```

# URL有效性验证

```javascript
/**
 * URL有效性校验
 * @param str_url
 * @returns {boolean}
 */
function isURL(str_url) { 
  // 验证url
  var strRegex = "^((https|http|ftp|rtsp|mms)?://)" + "?(([0-9a-z_!~*'().&=+$%-]+: )?[0-9a-z_!~*'().&=+$%-]+@)?" //           
  ftp的user@ + "(([0-9]{1,3}\.){3}[0-9]{1,3}" // IP形式的URL- 199.194.52.184
  + "|" // 允许IP和DOMAIN（域名）
  + "([0-9a-z_!~*'()-]+\.)*" // 域名- www.
  + "([0-9a-z][0-9a-z-]{0,61})?[0-9a-z]\." // 二级域名
  + "[a-z]{2,6})" // first level domain- .com or .museum
  + "(:[0-9]{1,4})?" // 端口- :80
  + "((/?)|" // a slash isn't required if there is no file name
  + "(/[0-9a-z_!~*'().;?:@&=+$,%#-]+)+/?)$";
  var re = new RegExp(strRegex);
  return re.test(str_url);
}
// 建议的正则
functionisURL(str) {
  return !! str.match(/(((^https?:(?:\/\/)?)(?:[-;:&=\+\$,\w]+@)?[A-Za-z0-9.-]+|(?:www.|[-;:&=\+\$,\w]+@)[A-Za-z0-9.-]+)((?:\/[\+~%\/.\w-_]*)?\??(?:[-\+=&;%@.\w_]*)#?(?:[\w]*))?)$/g);
}
```

# 自定义封装jsonp方法

```javascript
/**
 * 自定义封装jsonp方法
 * @param options
 */
jsonp = function(options) {
  options = options || {};
  if (!options.url || !options.callback) {
    throw new Error("参数不合法");
  }
  //创建 script 标签并加入到页面中
  var callbackName = ('jsonp_' + Math.random()).replace(".", "");
  var oHead = document.getElementsByTagName('head')[0];
  options.data[options.callback] = callbackName;
  var params = formatParams(options.data);
  var oS = document.createElement('script');
  oHead.appendChild(oS);
  //创建jsonp回调函数
  window[callbackName] = function(json) {
    oHead.removeChild(oS);
    clearTimeout(oS.timer);
    window[callbackName] = null;
    options.success && options.success(json);
  };
  //发送请求
  oS.src = options.url + '?' + params;
  //超时处理
  if (options.time) {
    oS.timer = setTimeout(function() {
      window[callbackName] = null;
      oHead.removeChild(oS);
      options.fail && options.fail({
        message: "超时"
      });
    },
    time);
  }
};
```

# 格式化参数

```javascript
/**
 * 格式化参数
 * @param data
 * @returns {string}
 */
formatParams = function(data) {
  var arr = [];
  for (var name in data) {
    arr.push(encodeURIComponent(name) + '=' + encodeURIComponent(data[name]));
  }
  return arr.join('&');
}
```

# cookie相关操作

```javascript
//写cookies
setCookie = function(name, value, time) {
  var strsec = getsec(time);
  var exp = new Date();
  exp.setTime(exp.getTime() + strsec * 1);
  document.cookie = name + "=" + escape(value) + ";expires=" + exp.toGMTString();
}
//cookie操作辅助函数
getsec = function(str) {
  var str1 = str.substring(1, str.length) * 1;
  var str2 = str.substring(0, 1);
  if (str2 == "s") {
    return str1 * 1000;
  } else if (str2 == "h") {
    return str1 * 60 * 60 * 1000;
  } else if (str2 == "d") {
    return str1 * 24 * 60 * 60 * 1000;
  }
}
//读取cookies
getCookie = function(name) {
  var arr, reg = new RegExp("(^| )" + name + "=([^;]*)(;|$)");
  if (arr = document.cookie.match(reg)) return (arr[2]);
  else return null;
}

//删除cookies
delCookie = function(name) {
  var exp = new Date();
  exp.setTime(exp.getTime() - 1);
  var cval = getCookie(name);
  if (cval != null) document.cookie = name + "=" + cval + ";expires=" + exp.toGMTString();
}
```

# 生成随机字符串

```javascript
/**
 * 生成随机字符串(可指定长度)
 * @param len
 * @returns {string}
 */
randomString = function(len) {
  len = len || 8;
  var $chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678';
  /****默认去掉了容易混淆的字符oOLl,9gq,Vv,Uu,I1****/
  var maxPos = $chars.length;
  var pwd = '';
  for (var i = 0; i < len; i++) {
    pwd += $chars.charAt(Math.floor(Math.random() * maxPos));
  }
  return pwd;
}
```

# 判断浏览器

```javascript
function parseUA() {
  var u = navigator.userAgent;
  var u2 = navigator.userAgent.toLowerCase();
  return { //移动终端浏览器版本信息
    trident: u.indexOf('Trident') > -1,
    //IE内核
    presto: u.indexOf('Presto') > -1,
    //opera内核
    webKit: u.indexOf('AppleWebKit') > -1,
    //苹果、谷歌内核
    gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') == -1,
    //火狐内核
    mobile: !!u.match(/AppleWebKit.*Mobile.*/),
    //是否为移动终端
    ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/),
    //ios终端
    android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1,
    //android终端或uc浏览器
    iPhone: u.indexOf('iPhone') > -1,
    //是否为iPhone或者QQHD浏览器
    iPad: u.indexOf('iPad') > -1,
    //是否iPad
    webApp: u.indexOf('Safari') == -1,
    //是否web应该程序，没有头部与底部
    iosv: u.substr(u.indexOf('iPhone OS') + 9, 3),
    weixin: u2.match(/MicroMessenger/i) == "micromessenger",
    ali: u.indexOf('AliApp') > -1,
  };
}
// var ua = parseUA();
// if (!ua.mobile) {
//   location.href = './pc.html';
// }
```

# 获取url后面的参数方法

```javascript
function GetRequest() {
  var url = location.search; //获取url中"?"符后的字串
  var theRequest = new Object();
  if (url.indexOf("?") != -1) {
    var str = url.substr(1);
    strs = str.split("&");
    for (var i = 0; i < strs.length; i++) {
      theRequest[strs[i].split("=")[0]] = (strs[i].split("=")[1]);
    }
  }
  return theRequest;
}
```

# 动态加载js

```javascript
function loadScript(url, callback) {
  var script = document.createElement("script");
  script.type = "text/";
  if (typeof(callback) != "undefined") {
    if (script.readyState) {
      script.onreadystatechange = function() {
        if (script.readyState == "loaded" || script.readyState == "complete") {
          script.onreadystatechange = null;
          callback();
        }
      };
    } else {
      script.onload = function() {
        callback();
      };
    }
  }
  script.src = url;
  document.body.appendChild(script);
}
```

# 生成随机颜色

```javascript
function getRandomColor () {
  const rgb = []
  for (let i = 0 ; i < 3; ++i){
    let color = Math.floor(Math.random() * 256).toString(16)
    color = color.length == 1 ? '0' + color : color
    rgb.push(color)
  }
  return '#' + rgb.join('')
}
```

最近几个项目用到就这么多，还会陆续添加的！！！  

[参考资料](https://juejin.im/post/5b62d02ee51d453467552dc9)