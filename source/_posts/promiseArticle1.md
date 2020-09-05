---
title: Promise链式调用特性总结
date: 2020-09-05 11:51:45
tags:
	- promise
categories: 'js'
---

相信各位前端小伙伴对于Promise应该很熟悉了吧，日常工作中，100%会用到的一个东西，除非你还在用callback解决异步，那我就太佩服了。话不多说，进入正题。

提前声明一下，以下代码在node环境中实现，你可以创建一个文件，使用nodemon这个工具执行这个文件，就可以进行监听更新，真香。

首先你要创建一个promise
```
let p=new Promise((resolve,reject)=>{
  resolve('first resolve')
})
```
## 方式一、通过return传递结果
```
p.then(res=>{
  return res;
}).then(res=>{
  console.log(res)
})
```
控制台就会输出：first resolve

## 方式二、通过返回新的promise resolve结果
```
p.then(res=>{
  return res;
}).then(res=>{
  return new Promise((resolve,reject)=>{
    resolve('second resolve: '+res)
  })
}).then(res=>{
  console.log(res)
})
```
控制台就会输出：second resolve: first resolve  
如果在返回的promise里加一个异步比如settimeout呢，结果会是什么样？
```
p.then(res=>{
  return res;
}).then(res=>{
  return new Promise((resolve,reject)=>{
    // resolve('second resolve: '+res)
    setTimeout(()=>{
      resolve('second resolve: '+res)
    },2000)
  })
}).then(res=>{
  console.log(res)
})
```
控制台等待2s后输出：second resolve: first resolve

## 方式三、通过返回新的promise reject 原因
既然可以通过新的promise resolve，那么reject应该也可以。
```
p.then(res=>{
  return res;
}).then(res=>{
  return new Promise((resolve,reject)=>{
    setTimeout(()=>{
      reject('error')
    },2000)
  })
}).then(res=>{
  console.log(res)
},err=>{
  console.log('error: '+err)
})
```
控制台等待2s后输出：error: error
## 方式四、then函数走了失败回调继续走then
紧接着上一步，失败后，reject出原因，继续后面then
```
p.then(res=>{
  return res;
}).then(res=>{
  return new Promise((resolve,reject)=>{
    setTimeout(()=>{
      reject('error')
    },2000)
  })
}).then(res=>{
  console.log(res)
},err=>{
  console.log('error: '+err)
  // 默认 return undefined
}).then(res=>{
  console.log('second then success: '+res)
},err=>{
  console.log('second then error: '+err)
})
```
控制台会输出两行内容：error: error，second then success: undefined。这就表明在reject 后面继续then会执行下一步的resolve，如果上一步没有返回值，默认接收undefined。

## 方式五、then中使用throw new Error情况
如果在then中抛出异常呢，如何显示？
```
p.then(res=>{
  return res;
}).then(res=>{
  return new Promise((resolve,reject)=>{
    setTimeout(()=>{
      reject('error')
    },2000)
  })
}).then(res=>{
  console.log(res)
},err=>{
  console.log('error: '+err)
}).then(res=>{
  throw new Error('happend error')
}).then(res=>{
  console.log('third then success'+res)
},err=>{
  console.log('third then error '+err)
})
```
控制台会输出：  
error: error  
third then error Error: happend error  
这表明throw error抛出异常类似reject，会由下一步的then方法中的错误方法处理。

## 方式六、在promise中使用catch进行错误捕获
```
p.then(res=>{
  return res;
}).then(res=>{
  return new Promise((resolve,reject)=>{
    setTimeout(()=>{
      reject('error')
    },2000)
  })
}).then(res=>{
  console.log(res)
},err=>{
  console.log('error: '+err)
}).then(res=>{
  throw new Error('happend error')
}).then(res=>{
  console.log('third then success'+res)
}).catch(err=>{
  console.log('catched '+err)
})
```
控制台会输出：  
error: error  
catched Error: happend error  

如果在catch方法的前面then中有对上一步错误的处理办法会怎么样呢？
```
p.then(res=>{
  return res;
}).then(res=>{
  return new Promise((resolve,reject)=>{
    // resolve('second resolve: '+res)
    setTimeout(()=>{
      reject('error')
    },2000)
  })
}).then(res=>{
  console.log(res)
},err=>{
  console.log('error: '+err)
}).then(res=>{
  throw new Error('happend error')
}).then(res=>{
  console.log('third then success'+res)
},err=>{
  console.log('third then error '+ err)
}).catch(err=>{
  console.log('catched '+err)
})
```
控制台会输出：  
error: error  
third then error Error: happend error
这说明catch捕获，如果catch前面有error处理函数，catch不会捕获异常的。

如果在catch后面继续then呢？
```
p.then(res=>{
  return res;
}).then(res=>{
  return new Promise((resolve,reject)=>{
    // resolve('second resolve: '+res)
    setTimeout(()=>{
      reject('error')
    },2000)
  })
}).then(res=>{
  console.log(res)
},err=>{
  console.log('error: '+err)
}).then(res=>{
  throw new Error('happend error')
}).then(res=>{
  console.log('third then success'+res)
}).catch(err=>{
  console.log('catched '+err)
  return 'catched error'
}).then(res=>{
  console.log('catched then '+res)
})
```
控制台会输出：  
error: error  
catched Error: happend error  
catched then catched error  
这说明catch后面是可以继续调用then的，catch 在promise的源码里面其实也是一个then，catch遵循then的运行规则。

## 总结
promise链式调用，具体是失败还是成功，取决于以下情况：
### 成功的条件
- then return 一个普通的js 值
- then return 一个新的promise成功态的结果 resolve处理
### 失败的条件
- then return 一个新的promise失败态的原因 error
- then throw抛出异常

以上就是promise链式调用的一些实践总结，复习复习基础知识。欢迎大家交流。


**参考资料：**

- [promise+规范](https://promisesaplus.com/)