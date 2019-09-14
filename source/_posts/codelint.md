---
title: vue项目整合Eslint和stylelint规范代码
date: 2019-09-14 10:34:03
categories: "vue"  
tags:
	- vue
	- js
---

## 前言

最近在搭建项目框架，想着上一个项目代码风格各异，就想着在新项目中引入Eslint来规范团队成员代码风格，保持统一，也方便大家维护代码，减少不必要的错误。前端应用愈加复杂，代码规范问题必须通过强制的方式保持统一。以下是团队逐渐摸索出的一些配置，各取所需。

## Eslint配置

在用vue-cli3搭建项目的过程中就会问你是否需要Eslint，选择就好来。如果没有选择后期又想加入eslint，可以手动安装Eslint的相关依赖。

### 安装
```
npm install eslint eslint-plugin-vue --save-dev
```
**需要注意**：Node.js (>=6.14), npm version 3+。

### 创建Eslint配置文件
在项目根目录下新建一个文件，名.eslintrc.js。下面是个人的一些配置，可以自行参考。

```
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true,
    es6: true
  },
  extends: ["eslint:recommended", "plugin:vue/essential", "@vue/prettier"],
  rules: {
    "generator-star-spacing": "off",
    "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
    "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off",
    "vue/no-parsing-error": [
      2,
      {
        "unexpected-solidus-in-tag": false
      }
    ]
  },
  parserOptions: {
    parser: "babel-eslint",
    ecmaVersion: 7,
    sourceType: "module",
    ecmaFeatures: {
      // 添加ES特性支持，使之能够识别ES6语法
      jsx: true
    }
  }
};

```
### 忽略Eslint校验
如果一些文件不需要Eslint的校验，可以配置一个.eslintignore，里面写上需要排除的文件。

```
/build/
/config/
/dist/
/*.js
/test/unit/coverage/
```

## stylelint配置
stylelint可以帮助我们规范化css的书写，风格统一，减少错误。
### 安装依赖

```
npm i -D stylelint stylelint-config-standard stylelint-webpack-plugin
```
### 配置
在项目根目录下新建配置文件.stylelintrc.js，相关配置如下：

```
module.exports = {
  extends: "stylelint-config-standard",
  rules: {
    "color-no-invalid-hex": true,
    "rule-empty-line-before": null,
    "color-hex-length": "long",
    "color-hex-case": "lower",
    "unit-whitelist": ["em", "rem", "%", "s", "px"],
    "declaration-colon-newline-after": null
  }
};

```

## 代码美化prettier配置

虽然借助 Eslint 来提高代码质量，但是却无法保证代码风格统一。一个统一的代码风格对于团队来说是很有价值的，所以为了达到目的，我们使用了 Prettier在保存和提交代码的时候，将代码修改成统一的风格。

### 安装依赖
```
npm i -D prettier @vue/eslint-config-prettier
```
### 配置
相关配置写在.eslintrc.js中

```
extends: ["eslint:recommended", "plugin:vue/essential", "@vue/prettier"]
```
我使用的是vscode编辑器，同时配置了vscode。

```
{
  "eslint.autoFixOnSave": true,
  "eslint.validate": [
    "javascript",
    {
      "language": "vue",
      "autoFix": true
    },
    "html",
    "vue"
  ],
  "editor.wordWrap": "wordWrapColumn",
  "editor.formatOnSave": true,
  "vetur.validation.template": false,
  "cSpell.ignoreWords": [
    "menu",
    "mixins"
  ]
}
```
推荐使用vscode的同学安装eslint和Prettier - Code formatter这两个插件，配合上面的配置，达到保存的时候自动格式化和校验的目的。

## 提交时校验

### 安装两个工具

* husky：一个方便用来处理 pre-commit 、 pre-push 等 githooks 的工具

* lint-staged：对 git 暂存区的代码，运行 linters 的工具


```
npm i lint-staged husky -D
```

### package.json增加配置

```
...
"husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,vue}": [
      "prettier --tab-width 2 --write",
      "vue-cli-service lint --fix",
      "git add"
    ]
  }
...
```
这样就可以实现在提交的时候校验，保证错误的代码无法提交。

到目前为止，项目中就整合进了Eslint校验，prettier美化代码，提交hooks代码检查。

## 参考

* [Eslint](https://cn.eslint.org/)
* [stylelint](https://stylelint.io/)
* [stylelint指南](https://cloud.tencent.com/developer/section/1489626)