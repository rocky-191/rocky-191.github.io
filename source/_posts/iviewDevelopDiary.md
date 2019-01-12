---
title: vue+iview项目开发实践问题总结（一）
date: 2018-06-08 18:03:51
tags: 
	- vue
	- iview
categories: "vue"
---

记录一次项目中使用vue+iview开发的问题，权当一次总结吧！  

<!--more-->  

* 1.实际使用中需要监听对象变化或者对象数组中某一属性是否发生变化

```

data(){
    return {
        associationNewForm:{...}
    }
},
watch:{
      // ['associationNewForm.ruleName'](){
      //监听某一属性发生变化
      //   this.$emit('datachange',this.associationNewForm);
      // }
      associationNewForm:{
        handler:function(val, oldVal){
          this.$emit('datachange',val);//通知父组件数据变化
        },
        deep:true
      }
    },
configStepData:{
//监听对象数组属性变化，需要深度监听
handler: function (newVal) {
    let dataObj={
      flag:'zgjconfigStepData',
      data:this.configStepData
    };
    this.$emit('getSubAlarmConditionData',dataObj);
},
deep: true    //深度监听
}
```


* 2.iview中表格渲染行的事件阻止冒泡方式

```
on: {
click: e => {
  e.stopPropagation();
  this.removeRow(params);
}}
```


* 3.iview中modal点击确定的操作时，但是modal中有表单校验，这个时候就需要通过loading动态控制
```
ok () {
    this.$Message.info('异步验证数据');
    setTimeout(() => {
        this.loading = false;
        this.$nextTick(() => {
            this.loading = true;
        });
    }, 2000);
}
```
iview的github上有对应的issue有对应问题，[相关issue](https://github.com/iview/iview/issues/597#issuecomment-292422473)


* 4.iview中时间选择组件如果需要禁用的话，单独添加disabled不起作用，需要添加readonly


* 5.iview中table列开启了ellipsis鼠标滑过显示tooltip,网上找到一种方法，显示内容

```
{
    title: '属性值',
    key: 'attrValue',
    render: (h, params) => {
        return h('div', [
          h('span', {
              style: {
                  display: 'inline-block',
                  width: '100%',
                  overflow: 'hidden',
                  textOverflow: 'ellipsis',
                  whiteSpace: 'nowrap'
              },
              domProps: {
                  title: params.row.attrValue
              }
          }, params.row.attrValue)
      ]);

    }
  }
```
在iview的github上有对应[issue](https://github.com/iview/iview/issues/1667)，但是没有得到解决,上面好多人都在问，稍微吐槽下，感觉处理问题不如element-ui框架团队处理的快，个人更喜好element。

自己鼓捣出了一种方式

```
{
            title: '属性值',
            key: 'attrValue',
            ellipsis:true,
            render: (h, params) => {
                return h('div', [
                  // h('span', {
                  //     style: {
                  //         display: 'inline-block',
                  //         width: '100%',
                  //         overflow: 'hidden',
                  //         textOverflow: 'ellipsis',
                  //         whiteSpace: 'nowrap'
                  //     },
                  //     domProps: {
                  //         title: params.row.attrValue
                  //     }
                  // }, params.row.attrValue)
                  h(
                    'Tooltip',
                  {
                    props: { content: params.row.attrValue, placement: 'top-start', transfer: true },
                    style:{
                      width:'100%'
                    }
                  },
                  [
                    h('span', {
                      style: {
                          display: 'inline-block',
                          width: '100%',
                          overflow: 'hidden',
                          textOverflow: 'ellipsis',
                          whiteSpace: 'nowrap'
                      },
                      on: {
                        click: () => {
                          this.toUpdate(params.index);
                        }
                      }
                    },params.row.attrValue)
                  ]
                )
              ]);

            }
          }
```

不过需要配合样式才能正常实现

```
.ivu-tooltip-inner{
    white-space:normal;
  }
  .ivu-tooltip-rel{
    display: block;
  }
```
初次使用iview，还在不断摸索中，后续如果有问题的话，会持续更新。 
