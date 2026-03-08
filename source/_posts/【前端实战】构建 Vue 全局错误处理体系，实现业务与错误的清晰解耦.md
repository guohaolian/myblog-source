---
title: Vue全局错误处理体系
date: 2025-11-11 11:23:47
wordcount: 1380
categories: 学习笔记
category_bar: true
tags:
  - Vue
---
# 构建 Vue 全局错误处理体系

### 一、为什么要做全局错误处理？

#### 1、将业务逻辑与错误处理解耦

在业务模块中，我们真正关心的是数据是否可用，以及页面状态如何变化，并不关心网络异常的类型、提示和跳转。所以需要将错误<span style="color: rgb(51, 145, 229); user-select: auto;">策略</span>抽离到全局层，让业务代码只专注于处理业务，全局错误处理层专注于处理各类错误，解耦后业务层和全局错误层都更加纯粹，也更有利于长期维护和拓展。

#### 2、为监控和埋点提供统一入口

项目上线后，对于错误信息除了要建立临时应对和处理机制外，还需要定时收集和上报，给错误分级，还要收集用户的环境信息，这样才能给开发者提供准确的数据信息，从而针对性的修复 bug 以及性能优化。



### 二、Vue 中的基础全局错误处理方式

#### 1、Vue 中全局错误处理写法

在 Vue 3 中，官方提供了一个明确的入口：app.config.errorHandler。在 main.js 中，添加如下代码即可：

```javascript
const app = createApp(App)
 
app.config.errorHandler = (err, instance, info) => {
  console.error(err)
}
```

#### 2、它会捕获哪些错误？

app.config.errorHandler 只会捕获 Vue 运行时上下文中的错误，包括：

- 组件 setup / render 中的同步错误

- 生命周期钩子中的错误

- 模板渲染期间的错误

- watch / computed 中抛出的错误

- 被 Vue 追踪的 Promise 链中的错误

#### 3、它不会捕获哪些错误？

不在上述范围内，脱离了 Vue 的响应式调度体系的错误均不会被捕获，比如：

```javascript
setTimeout(() => {
  throw new Error('timeout error') // 不会捕获
})
 
 
fetch('/api').then(() => {
  throw new Error('fetch error') // 不会捕获
})
```

所以Promise的 reject 并不会天然进全局错误处理，后面进阶方案里会<span style="color: rgb(51, 145, 229); user-select: auto;">解决</span>这个问题。

#### 4、errorHandler 的参数含义

Vue 3 中的定义如下：

```javascript
app.config.errorHandler = (
  err: unknown,
  instance: ComponentPublicInstance | null,
  info: string
) => void
```

三个参数的具体意义为：

```javascript
app.config.errorHandler = (err, instance, info) => {
  console.log(err)      // 实际抛出的错误对象
  console.log(instance) // 出错的组件实例（可能为 null）
  console.log(info)     // 错误来源描述（字符串）
}
```



### 三、全局错误处理的进阶设计

#### 1、定义“可识别的业务错误”

真正的工程实践中，我们关心错误是为了解决错误，所以需要对业务错误进行鉴别分类。

首先需要定义可识别的业务错误基类及其派生类，比如：

```javascript
export class BusinessError extends Error {
  constructor(message, code) {
    super(message)
    this.code = code
    this.isBusinessError = true
  }
}
 
 
export class AuthError extends BusinessError {
  constructor(message = '登录已失效') {
    super(message, 'AUTH_ERROR')
  }
}
 
export class PermissionError extends BusinessError {
  constructor(message = '没有操作权限') {
    super(message, 'PERMISSION_ERROR')
  }
}
```

在具体的业务代码中，遇到错误时，就使用对应的错误类实例化并抛出，app.config.errorHandler 就会捕获到这个错误实例，比如：

```javascript
if (!token) {
  throw new AuthError()
}
```

#### 2、在 errorHandler 中做真正的“分类处理”

现在，在定义了可识别的业务错误之后，全局错误处理的优势就体现出来了，此时业务错误类型可控，有基础应对手段，并且还有错误上报策略 reportError 以应对突发情况：

```javascript
app.config.errorHandler = (err, instance, info) => {
  if (err instanceof BusinessError) {
    handleBusinessError(err)
    return
  }
 
  handleUnknownError(err, instance, info)
}
```

```javascript
export function handleBusinessError(err) {
  ElMessage.warning(err.message)
 
  if (err.code === 'AUTH_ERROR') {
    router.push('/login')
  }
}
 
export function handleUnknownError(err, instance, info) {
  ElMessage.error('系统异常，请稍后再试')
 
  reportError({
    err,
    component: instance?.type?.name,
    info,
  })
}
 
export function reportError({ err, component, info }) {
  const payload = {
    message: err.message,
    stack: err.stack,
    component,
    info,
    ua: navigator.userAgent,
    time: Date.now(),
  }
 
  fetch('/error/report', {
    method: 'POST',
    body: JSON.stringify(payload),
  })
}
```

reportError 方法收集了错误的类型、信息、位置、发生时间，客户端的类型、操作系统、浏览器版本等信息，集中上报等待解决。

#### 3、补齐 Promise reject 的捕获能力

前面说过，errorHandler 不会自动捕获所有 Promise 的 reject，工程中常见解决方案是在请求层统一转抛错误，这就回到了文章开头时我们提到的在网络层面实现数据与状态解耦的 Axios 错误处理封装方案，由于那篇博文已经详细介绍过了，这里只给个简要的例子：

```javascript
axios.interceptors.response.use(
  res => res,
  err => {
    throw err // 重新抛给 Vue
  }
)
```

这样就能保证所有异常最终都会汇聚到一个出口。

#### 4、错误处理的策略化封装

看到这个词有些粉丝可能会有印象，以前的博文也提到过策略表模式，在全局错误处理中依然好用，这样就不用在 errorHandler 里写一堆 if else了，更容易拓展和维护，比如：

```javascript
app.config.errorHandler = (err) => {
  const handler = errorStrategyMap[err.code] || errorStrategyMap.default
  handler(err)
}
```

```javascript
const errorStrategyMap = {
  AUTH_EXPIRED: (err) => {
    ElMessage.error(err.message)
    router.push('/login')
  },
  default: (err) => {
    ElMessage.error('系统异常')
    reportError(err)
  }
}
```



### 四、结语

通过进阶的全局错误处理设计，将业务逻辑与错误处理解耦，不仅能让页面代码更加清晰简洁，还能实现错误的分级处理，从而显著提升项目在生产环境中的可维护性和长期稳定性。

只有锻炼思维才能可持续地解决问题，只有思维才是真正值得学习和分享的核心要素。
