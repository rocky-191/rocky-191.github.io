---
title: 前端vue项目实现mock数据方式
date: 2020-05-05 15:35:38
tags: 
	- vue
categories: "vue"
---

前后端分离开发已成大势所趋，基本上大部分公司的开发模式都是如此，那如何自己本地实现一个数据mock呢？当然你也可以使用在线的工具，比如[easymock](https://www.easy-mock.com/login)也可以实现mock数据，但是如果追求稳定性，还是本地搭一套环境吧。下面我介绍的是使用了vue-cli本身自带的功能实现mock 数据。

<!--more-->

## 初始化项目

（1）使用vue-cli初始化

```vue
vue create mock-demo
```

全部采用默认即可

（2）创建配置文件

在项目根目录下创建vue.config.js配置文件。内容如下：

```javascript
const bodyParser = require("body-parser");
const isProduction = process.env.NODE_ENV === "production";

let feMock;
if (!isProduction) {
  feMock = require("./mockApi");
}
module.exports = {
  publicPath: isProduction ? "././" : "",
  pages: {
    index: {
      entry: "src/main.js",
      template: "public/index.html",
      filename: "index.html"
    }
  },
  devServer: {
    before: app => {
        // 关键代码
      app.use(bodyParser.json());
      app.use(bodyParser.urlencoded({ extended: true }));
      if (!isProduction) {
        feMock(app);
      }
    }
  }
};

```

这里主要是利用了webpack-dev-server实现的mock功能，为了实现接口请求，当然需要提前安装axios，body-parser。

```
npm i axios body-parser -S
```

## 编写mock的API

（1）在项目根目录新建文件夹mockApi，新建index.js

```js
const feMock = app => {
  app.get("/mock/api/news", function(req, res) {
    res.json({
      name: "tom"
    });
  });
};
module.exports=feMock;
```

（2）在src目录下新建一个api文件夹，新建文件index.js

```javascript
const prefix = "/mock";
export default {
  methods: {
    _testMock() {
      return this.$http.get(`${prefix}/api/news`);
    }
  }
};
```

这里的请求路径一定要和上一步mock数据的路径保持一致。等后端写好接口之后，统一修改此文件里的prefix接口即可。

## 组件使用

（1）在App.vue里使用

```vue
<script>
import api from "./api";
export default {
  name: "App",
  mixins: [api],
  mounted() {
    this.testMock();
    this.testMock1();
    this.testPostMock();
  },
  methods: {
    testMock() {
      this._testMock()
        .then(res => {
          console.log(res);
        })
        .catch(err => {
          console.log(err);
        });
    }
  }
};
</script>
```

正常启动项目后，在浏览器network里就可以看到请求了，初始功能实现。

## 优化

（1）如果项目中多个人写多个mock文件怎么整呢？

（2）能不能直接扫描特定目录加载mock文件呢？是否可以有一个统一的对外入口，每个人只需要写自己的mock文件，不用修改出口？

### 解决办法

（1）在mockApi文件夹下新建一个mockList文件夹，之后所有人的mock接口写在这里。示例如下：

在mockList中新建一个test.js

```javascript
function testMock(app) {
  app.get("/mock/api/news", function(req, res) {
    res.json({
      name: "tom"
    });
  });
}

function testPostMock(app) {
  app.post("/mock/api/news", function(req, res) {
    console.log(req.body);
    setTimeout(function() {
      res.json({
        code: 0,
        data: "success",
        desc: ""
      });
    }, 500);
  });
}
module.exports = [testMock, testPostMock];

```

在mockList中新建一个test1.js

```javascript
function testMock1(app) {
  app.get("/mock/api/news1", function(req, res) {
    res.json({
      name: "jack"
    });
  });
}
module.exports = [testMock1];
```

(2)修改mockApi/index.js文件

```javascript
const fs = require("fs");
const path = __dirname;
const files = fs.readdirSync(path + "/mockList");
const mockList = [];
files.forEach(function(filename) {
  let model = require(path + "/mockList/" + filename);
  mockList.push(...model);
});

function handleMock(app) {
  mockList.forEach(func => {
    func(app);
  });
}

module.exports = handleMock;

```
引入fs,自动读取设定目录下的文件，这样配置好之后，其他人只管写自己的mock接口，不需要去修改这个对外的文件，这样就完美解决了。

示例代码目录结构如下：

![content](vue-mock-demo\content.png)