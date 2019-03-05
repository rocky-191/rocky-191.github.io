---
title: mvvm简易版本实现
date: 2019-03-05 21:49:00
tags:
	- vue
categories: 'vue'
---

用了两年左右的vue，虽然看过vue的源码，推荐黄轶大佬的vue源码分析，相当到位。从头梳理了vue的实现过程。周末又看了一个公开课的vue源码分析，想着自己是不是也可以写一个来实现，说干就干，开始coding！  
目前最新版本的vue内部依然使用了Object.defineProperty()来实现对数据属性的劫持，进而达到监听数据变动的效果。  

* 需要数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者。
* 需要指令解析器Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数。
* 一个Watcher，作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图。
* mvvm入口函数，整合以上三者，实现数据响应。  
  相信看过vue官网的小伙伴们一定看过下面这张图吧，解释了vue是如何实现响应式的数据绑定。

![](https://user-gold-cdn.xitu.io/2019/3/5/1694e017cb783666?w=1278&h=746&f=png&s=86912)
## Observer类的实现
主要利用了Object.defineProperty()这个方法，对数据进行遍历，给每一个对象都添加了getter()和setter().主要代码如下：

```
class Observer{
    constructor(data){
        this.data=data;
        this.traverse(data);
    }
    traverse(data) {
        var self = this;
        Object.keys(data).forEach(function(key) {
            self.convert(key, data[key]);
        });
    }
    convert(key,val){
        this.defineReactive(this.data, key, val);
    }

    defineReactive(data, key, val) {
        var dep = new Dep();
        var childObj = observe(val);

        Object.defineProperty(data, key, {
            enuselfrable: true, // 可枚举
            configurable: false, // 不能再define
            get(){
                if (Dep.target) {
                    dep.depend();
                }
                return val;
            },
            set(newVal) {
                if (newVal === val) {
                    return;
                }
                val = newVal;
                // 新的值是object的话，进行监听
                childObj = observe(newVal);
                // 通知订阅者
                dep.notify();
            }
        });
    }
}

function observe(value, vm) {
    if (!value || typeof value !== 'object') {
        return;
    }
    return new Observer(value);
}
```
经过以上的方法，我们就劫持到了数据属性。
## Compile类的实现
主要用来解析各种指令，比如v-modal，v-on:click等指令。然后将模版中的变量替换成数据，渲染view，将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据发生变动，收到通知，更新视图。

```
class Compile{
    constructor(el,vm){
        this.$vm = vm;
        this.$el = this.isElementNode(el) ? el : document.querySelector(el);
        if (this.$el) {
            this.$fragment = this.node2Fragment(this.$el);
            this.init();
            this.$el.appendChild(this.$fragment);
        }
    }
    node2Fragment(el){
        var fragment = document.createDocumentFragment(),
            child;

        // 将原生节点拷贝到fragment
        while (child = el.firstChild) {
            fragment.appendChild(child);
        }

        return fragment;
    }

    init(){
        this.compileElement(this.$fragment);
    }

    compileElement(el){
        var childNodes = el.childNodes,
            self = this;

        [].slice.call(childNodes).forEach(function(node) {
            var text = node.textContent;
            var reg = /\{\{(.*)\}\}/;

            if (self.isElementNode(node)) {
                self.compile(node);

            } else if (self.isTextNode(node) && reg.test(text)) {
                self.compileText(node, RegExp.$1);
            }

            if (node.childNodes && node.childNodes.length) {
                self.compileElement(node);
            }
        });
    }

    compile(node){
        var nodeAttrs = node.attributes,
            self = this;

        [].slice.call(nodeAttrs).forEach(function(attr) {
            var attrName = attr.name;
            if (self.isDirective(attrName)) {
                var exp = attr.value;
                var dir = attrName.substring(2);
                // 事件指令
                if (self.isEventDirective(dir)) {
                    compileUtil.eventHandler(node, self.$vm, exp, dir);
                    // 普通指令
                } else {
                    compileUtil[dir] && compileUtil[dir](node, self.$vm, exp);
                }

                node.removeAttribute(attrName);
            }
        });
    }

    compileText(node, exp){
        compileUtil.text(node, this.$vm, exp);
    }

    isDirective(attr){
        return attr.indexOf('v-') == 0;
    }

    isEventDirective(dir){
        return dir.indexOf('on') === 0;
    }

    isElementNode(node){
        return node.nodeType == 1;
    }

    isTextNode(node){
        return node.nodeType == 3;
    }
}

// 指令处理集合
var compileUtil = {
    text: function(node, vm, exp) {
        this.bind(node, vm, exp, 'text');
    },

    html: function(node, vm, exp) {
        this.bind(node, vm, exp, 'html');
    },

    model: function(node, vm, exp) {
        this.bind(node, vm, exp, 'model');

        var self = this,
            val = this._getVMVal(vm, exp);
        node.addEventListener('input', function(e) {
            var newValue = e.target.value;
            if (val === newValue) {
                return;
            }

            self._setVMVal(vm, exp, newValue);
            val = newValue;
        });
    },

    class: function(node, vm, exp) {
        this.bind(node, vm, exp, 'class');
    },

    bind: function(node, vm, exp, dir) {
        var updaterFn = updater[dir + 'Updater'];

        updaterFn && updaterFn(node, this._getVMVal(vm, exp));

        new Watcher(vm, exp, function(value, oldValue) {
            updaterFn && updaterFn(node, value, oldValue);
        });
    },

    // 事件处理
    eventHandler: function(node, vm, exp, dir) {
        var eventType = dir.split(':')[1],
            fn = vm.$options.methods && vm.$options.methods[exp];

        if (eventType && fn) {
            node.addEventListener(eventType, fn.bind(vm), false);
        }
    },

    _getVMVal: function(vm, exp) {
        var val = vm;
        exp = exp.split('.');
        exp.forEach(function(k) {
            val = val[k];
        });
        return val;
    },

    _setVMVal: function(vm, exp, value) {
        var val = vm;
        exp = exp.split('.');
        exp.forEach(function(k, i) {
            // 非最后一个key，更新val的值
            if (i < exp.length - 1) {
                val = val[k];
            } else {
                val[k] = value;
            }
        });
    }
};


var updater = {
    textUpdater: function(node, value) {
        node.textContent = typeof value == 'undefined' ? '' : value;
    },

    htmlUpdater: function(node, value) {
        node.innerHTML = typeof value == 'undefined' ? '' : value;
    },

    classUpdater: function(node, value, oldValue) {
        var className = node.className;
        className = className.replace(oldValue, '').replace(/\s$/, '');

        var space = className && String(value) ? ' ' : '';

        node.className = className + space + value;
    },

    modelUpdater: function(node, value, oldValue) {
        node.value = typeof value == 'undefined' ? '' : value;
    }
};
```
## Watcher类的实现
作为链接的桥梁，链接了compile和observer。添加订阅者，当检测到属性发生变化，接收到dep.notify()的通知的时候，就执行自身的update()方法
```
class Watcher{
    constructor(vm, expOrFn, cb){
        this.cb = cb;
        this.vm = vm;
        this.expOrFn = expOrFn;
        this.depIds = {};

        if (typeof expOrFn === 'function') {
            this.getter = expOrFn;
        } else {
            this.getter = this.parseGetter(expOrFn);
        }

        this.value = this.get();
    }
    update(){
        this.run();
    }
    run(){
        var value = this.get();
        var oldVal = this.value;
        if (value !== oldVal) {
            this.value = value;
            this.cb.call(this.vm, value, oldVal);
        }
    }
    addDep(dep){
        if (!this.depIds.hasOwnProperty(dep.id)) {
            dep.addSub(this);
            this.depIds[dep.id] = dep;
        }
    }
    get() {
        Dep.target = this;
        var value = this.getter.call(this.vm, this.vm);
        Dep.target = null;
        return value;
    }

    parseGetter(exp){
        if (/[^\w.$]/.test(exp)) return;

        var exps = exp.split('.');

        return function(obj) {
            for (var i = 0, len = exps.length; i < len; i++) {
                if (!obj) return;
                obj = obj[exps[i]];
            }
            return obj;
        }
    }
}
```
## mvvm实现
MVVM作为数据绑定的入口，整合Observer、Compile和Watcher三者，通过Observer来监听自己的model数据变化，通过Compile来解析编译模板指令，最终利用Watcher搭起Observer和Compile之间的通信桥梁，达到数据变化，触发视图更新；视图交互变化(input) 触发数据model变更的双向绑定效果。

```
class Mvvm{
    constructor(options){
        this.$options=options;
        this.data=this._data = this.$options.data;
        console.log(this.$options)
        var self = this;
        // 数据代理,实现响应，vue3会改写，使用proxy代理方式
        Object.keys(this.data).forEach(function(key) {
            self.defineReactive(key);
        });
        this.initComputed();

        new Observer(this.data, this);

        this.$compile = new Compile(this.$options.el || document.body, this)
    }
    defineReactive(key){
        var self=this;
        Object.defineProperty(this,key,{
            configurable:false,
            enuselfrable:true,
            get(){
                return self.data[key];
            },
            set(newValue){
                self.data[key]=newValue;
            }
        })
    }
}
```
基本上就大功告成了，大部分代码都是参考了vue源码的实现，学着读源码吧，体会vue设计的优雅。顺便推荐一个github读源码的chrome插件：octotree.本文完整代码请查看[github](https://github.com/rocky-191/mvvm)  

本文同步发表于掘金/segmentfault

**顺便说一句，最近开始找工作了，坐标北京，如果各位大佬有机会，望推荐下哈，在此先行谢过！**