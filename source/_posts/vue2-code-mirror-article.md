---
title: vue2项目引入代码编辑器插件小记
date: 2023-07-29 09:34:03
tags:  
	- vue
categories: "vue"
---

背景

为什么要用vue2呢？都2023年了，还在用vue2？没办法，公司老项目需要维护迭代，一直也没有升级vue3，只能在上面继续维护了。最近项目里需要用到一个代码编辑器，可以实现输入js、json、sql等，并且需要格式校验，当格式不正确的时候并给出错误提示。

实现过程

之前在使用react的过程中，使用了react-codemirror，所以应该也会有vue版本的code-mirror，google一下，果然不出所料，然而网上的文章千奇百怪，有的时候按照文章步骤，总会出现各种各样的问题，在整合多方资料，自己不断尝试，终于搞定了！！！特此记录，以防后来掉坑！

安装依赖

因为是vue2项目，不能使用最新的codemirror6版本，需要使用codemirror v5版本，安装如下依赖

yarn add codemirror@5
yarn add vue-codemirror@4.0.6

使用方法

创建一个vue组件

<template>
  <codemirror v-model="code" :options="cmOptions"></codemirror>
</template>

<script>
import  { codemirror } from 'vue-codemirror'
import 'codemirror/lib/codemirror.css'
// language js
import 'codemirror/mode/javascript/javascript.js'
// theme css
import 'codemirror/theme/base16-dark.css'

export const jsonData = {
  name:'tome',
  age:16
}
export default {
  name:'code-display',
  components:{
    codemirror
  },
  data() {
    return {
      code: JSON.stringify(jsonData,null,2),
      cmOptions: {
        // codemirror options
        // tabSize: 2,
        // mode: 'text/javascript',
        mode:'application/json',
        gutters: ['CodeMirror-lint-markers',"CodeMirror-linenumbers","CodeMirror-foldgutter"],
        theme: 'base16-dark',
        lineNumbers: true,
        line: true,
        lineWrapping: true,
        // more codemirror options, 更多 codemirror 的高级配置...
      }
    };
  },
  methods: {
  }
};
</script>

此时就可以正常输入json、js等代码，但是我们的需求需要校验代码的格式,v5 版本给提供一个lint配置，相关介绍可查看lint，此处只演示json格式校验，为了实现json格式校验，需要增加一个配置，打开lint

cmOptions: {
        mode:'application/json',
        gutters: ['CodeMirror-lint-markers',"CodeMirror-linenumbers","CodeMirror-foldgutter"],
        theme: 'base16-dark',
        lineNumbers: true,
        line: true,
        lint:true, // 格式校验开关
        lineWrapping: true,
      }

按照网上的文章，打开这个开关后即可，在做的过程中发现，单纯打开这个开关，并不能实现格式校验，console控制台还会报错：
![error](vue2-code-mirror-article\error.png)


根据报错内容，查看源码，发现在源码中会判断window.jsonlint是否存在
![resource-code](vue2-code-mirror-article\resource-code-jsonlint.png)


查找了好多资料后，需要安装如下几个npm包，安装jshint是为了后面校验js格式（按需安装）

yarn add jsonlint file system
yarn add jshint

代码增加如下

import jsonlint from 'jsonlint';
import { JSHINT } from 'jshint';

import 'codemirror/addon/lint/lint'
import 'codemirror/addon/lint/lint.css'
import 'codemirror/addon/lint/json-lint'
import 'codemirror/addon/lint/javascript-lint'

window.JSHINT = JSHINT;
window.jsonlint = jsonlint;

完整代码

<template>
  <codemirror v-model="code" :options="cmOptions"></codemirror>
</template>

<script>
import  { codemirror } from 'vue-codemirror'
import jsonlint from 'jsonlint';
import { JSHINT } from 'jshint';
import 'codemirror/lib/codemirror.css'
import 'codemirror/addon/lint/lint'
import 'codemirror/addon/lint/lint.css'
import 'codemirror/addon/lint/json-lint'
import 'codemirror/addon/lint/javascript-lint'
// language js
import 'codemirror/mode/javascript/javascript.js'
// theme css
import 'codemirror/theme/base16-dark.css'

window.JSHINT = JSHINT;
window.jsonlint = jsonlint;

export const jsonData = {
  name:'tome',
  age:16
}
export default {
  name:'code-display',
  components:{
    codemirror
  },
  data() {
    return {
      code: JSON.stringify(jsonData,null,2),
      cmOptions: {
        // codemirror options
        // tabSize: 2,
        // mode: 'text/javascript',
        mode:'application/json',
        gutters: ['CodeMirror-lint-markers',"CodeMirror-linenumbers","CodeMirror-foldgutter"],
        theme: 'base16-dark',
        lineNumbers: true,
        line: true,
        lint:true,
        lineWrapping: true,
        // more codemirror options, 更多 codemirror 的高级配置...
      }
    };
  },
  methods: {
  }
};
</script>
完整的package.json
{
  "name": "test-vue2",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint"
  },
  "dependencies": {
    "@codemirror/lang-html": "^6.4.5",
    "@codemirror/lang-javascript": "^6.1.9",
    "@codemirror/lang-json": "^6.0.1",
    "@codemirror/theme-one-dark": "^6.1.2",
    "codemirror": "5",
    "core-js": "^3.6.5",
    "file": "^0.2.2",
    "jshint": "^2.13.6",
    "jsonlint": "^1.6.3",
    "system": "^2.0.1",
    "vue": "^2.6.11",
    "vue-codemirror": "4.0.6"
  },
  "devDependencies": {
    "@vue/cli-plugin-babel": "~4.5.7",
    "@vue/cli-plugin-eslint": "~4.5.7",
    "@vue/cli-service": "~4.5.7",
    "babel-eslint": "^10.1.0",
    "eslint": "^6.7.2",
    "eslint-plugin-vue": "^6.2.2",
    "vue-template-compiler": "^2.6.11"
  },
  "eslintConfig": {
    "root": true,
    "env": {
      "node": true
    },
    "extends": [
      "plugin:vue/essential",
      "eslint:recommended"
    ],
    "parserOptions": {
      "parser": "babel-eslint"
    },
    "rules": {}
  },
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not dead"
  ]
}

当前格式是json格式，就可以输入json内容了，错误的时候会给出提示。

json格式错误如下：
![json-error-show](vue2-code-mirror-article\json-error-show.png)
json格式正确如下：
![json-right-show](vue2-code-mirror-article\json-right-show.png)


如果想输入其它格式内容，比如输入js代码，需要将配置项中的mode改为对应语言格式text/javascript，并引入对应语言校验的包才行。

参考资料

● [codemirror v5 官网](https://codemirror.net/5/index.html)
● https://github.surmon.me/vue-codemirror
● https://github.com/surmon-china/vue-codemirror/tree/v4.0.6
● https://github.com/scniro/react-codemirror2/issues/21