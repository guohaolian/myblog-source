---
title: Promise的状态吸收
date: 2025-12-08 10:34:23
wordcount: 604
categories: 学习笔记
category_bar: true
tags:
  - Javascript
---
# Promise的状态吸收

先先讨论一个点,**then里面的回调**什么时候加入到微队列中?

> [!TIP]
>
> 1. 当promise的状态变为已完成时,将所有等待队列的回调函数加入微队列
>
> 2. 执行到then时,如果promise的状态是已完成,直接将回调函数加入微队列,否则加入等待队列**

代码举例说明:

这两段代码的执行结果分别是什么？

```js
new Promise((resolve, reject) => {
  resolve(2)
  new Promise((resolve, reject) => {
    resolve(5)
  }).then(res=>console.log(res))
}).then(res=>console.log(res))


```

```js
new Promise(( resolve, reject ) => {
  setTimeout(() => {
    resolve(2)
    new Promise(( resolve, reject ) => {
      resolve(5)
    }).then(res => console.log(res))
  })
}).then(res => console.log(res))

```

第一段的结果是 5 2

第二段的结果是 2 5

1.对于第一段代码,resolve函数执行之后,promise的状态就为已完成,两个promise都是这样,后面执行到then时,直接将回调函数加入微队列执行
2.对于第二段, 当底部的then执行时, promise的状态还是pending,将回调函数加入等待队列,等定时器执行时,先执行resolve(2),此时将等待队列的回调函数加入微队列,然后再执行第二个promise

## 状态吸收

先看一段测试代码

```js
const p1 = new Promise((resolve, reject)=>{
  console.log(1)
  resolve()
}).then((res)=>{
    console.log(2)
    return Promise.resolve()
}).then(()=>{
    console.log(3)
})

const p2 = new Promise((resolve, reject)=>{
  console.log(4)
  resolve()
}).then((res)=>{
    console.log(5)
}).then(()=>{
  console.log(6)
}).then(()=>{
  console.log(7)
}).then(()=>{
  console.log(8)
})

```

先说结果：1 4 2 5 6 7 3 8

解释:

在状态吸收时,会经历两个过程

> [!IMPORTANT]
>
> 准备 吸收
> 解释不清楚,直接看下面微队列的变化

一：

```js
执行栈: 1 4 
微队列: 2 5
当前输出结果: 无

```

二：

```js
执行栈: 2 5
微队列: 准备 6
当前输出结果: 1 4

```

三：

```js
执行栈: 准备 6
微队列: 吸收 7
当前输出结果: 1 4 2 5

```

四：

```js
执行栈: 吸收 7
微队列: 3 8
当前输出结果: 1 4 2 5 6

```

五：

```js
执行栈: 3 8
微队列: 无
当前输出结果: 1 4 2 5 6 7

```

六：

```js
执行栈: 无
微队列: 无
当前输出结果: 1 4 2 5 6 7 3 8

```

即,`(res)=>{ console.log(2) return Promise.resolve() }` 这个回调函数返回的是一个promise,那么`then()`返回的promise就会去吸收这个promise的状态,但是这个吸收不是一步就完成的,要经过`准备,吸收`这两个阶段