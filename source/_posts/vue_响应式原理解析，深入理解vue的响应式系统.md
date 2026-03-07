---
title: 响应式原理
date: 2025-06-28 11:23:47
wordcount: 1789
categories: 学习笔记
category_bar: true
tags:
  - Vue
---
# vue:响应式原理解析，深入理解vue的响应式系统

## 一、文章秒读

vue的响应式系统核心有两个，简单描述就是：

**1.在数据变化时重新render依赖相关函数（组件）。** 

**2.在vue2和vue3中分别使用Object.defineProperty和Proxy进行对象属性的读写。** 

数据变化时：

![](/img/0_1.jpeg)

##  **二、什么是响应式** 

**当依赖数据发生改变时，与之关联的数据或计算结果能够自动更新就是响应式。** 

我们用代码可以更直观理解：

```javascript
let A0=1;
let A1=2;
let A2;
function update() {
  A2 = A0 + A1
}
whenDepsChange(update);
```

在代码中

首先，我们定义了一个update()函数来作为我们需要在数据变化时进行的操作。这个操作会更改程序的某个状态，也就是说这是个副作用函数，简称为 **作用** (effect)。

其次，我们还需要定义一个whenDepsChange()函数，让它能够在数据变化时执行update()函数。

这个 whenDepsChange() 函数有如下的任务：

1. 当一个变量被读取时进行追踪。例如我们执行了表达式 A0+A1 的计算，则 `A0` 和 `A1` 都被读取到了。在这里，A0 和 A1 被视为这个作用的 **依赖 (dependency)** ，因为它们的值被用来执行前面提到的作用。这次作用也可以被称作它的依赖的一个 **订阅者** (subscriber)

2. 如果一个变量在当前运行的副作用中被读取了，就将该副作用设为此变量的一个订阅者。例如由于 `A0` 和 `A1` 在 `update()` 执行时被访问到了，则 `update()` 需要在第一次调用之后成为 `A0` 和 `A1` 的订阅者。

3. 探测一个变量的变化。例如当我们给 `A0` 赋了一个新的值后，应该通知其所有订阅了的副作用重新执行。

综上所述，我们可以简单的将响应式过程理解为：

依赖数据变化=>触发对该变量追踪的函数（监听变化）=>触发副作用函数（触发更新）

## 三、Vue是如何实现响应式的

### 1.数据的get和set

在标准的JavaScript中，直接追踪局部变量（如函数内的变量）的读写操作是不可能的，因为语言本身没有提供这样的机制。但是，对于对象的属性，JavaScript提供了可以利用的特性来间接实现这种追踪。

**1. Object.defineProperty** 
Object.defineProperty允许你定义或修改对象上的一个属性，并且可以指定该属性的访问器方法（getter和setter）。当属性被读取或设置时，相应的getter或setter将被调用。

示例：

```javascript
    <script>
      const obj = {};
      // 定义属性'value'，包含getter和setter
      Object.defineProperty(obj, "value", {
        get() {
          console.log("get value");
          return this._value;
        },
        set(newValue) {
          console.log("set value");
          this._value = newValue;
        },
        // 可以通过这个属性来控制属性的可枚举性
        configurable: true,
        // 可以通过这个属性来控制属性的可写性
        enumerable: true,
      });
 
      obj.value = 5;
      console.log(obj.value); // get value ,set value, 5
    </script>
```

**2.Proxy** 
Proxy对象允许你拦截并自定义对象的基本操作，包括属性访问和修改。这使得你可以创建一个代理对象，当访问或修改目标对象的属性时，会触发预定义的行为。

示例：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>基础 ref（）</title>
  </head>
  <body>
    <button id="updateButton">点击+1</button>
    <div id="message"></div>
    <script>
      const createRef = (initialValue) => {
        return new Proxy(
          { value: initialValue },
          {
            get(target, key) {
              return target[key];
            },
            set(target, key, value) {
              target[key] = value;
              if (key === "value") {
                updateDisplay(); // 当.value被设置时，更新DOM
              }
              return true;
            },
          }
        );
      };
 
      // 初始化ref
      const numberRef = createRef(0);
 
      // 更新DOM的函数
      const updateDisplay = () => {
        document.getElementById("message").innerText = numberRef.value;
      };
 
      // 绑定按钮点击事件
      document.getElementById("updateButton").addEventListener("click", () => {
        numberRef.value++; // 点击按钮时计数器加一
      });
 
      // 初始显示
      updateDisplay();
    </script>
  </body>
</html>
```

在vue2中，出于支持旧版本浏览器的限制，使用了Object.defineProperty

在vue3中，则使用了功能更为强大的Proxy

### 2.vue实现响应式的过程

下文中讲到的 **视图更新** ：在vue中，模板会编译为渲染函数，也就是我们响应式中的副作用函数。当数据变化时就会重新执行依赖相关的渲染函数，实现视图的更新。

**1.Vue 2 的响应式系统** 
在 Vue 2 中，响应式系统基于 Object.defineProperty 来实现。对于每个响应式数据对象，Vue 都会递归遍历其所有属性，并使用 Object.defineProperty 将它们转换为 getter/setter 形式。当属性被访问时，getter 方法会被调用；当属性被修改时，setter 方法会被调用。这些方法内部会记录依赖关系，并在数据变化时通知观察者更新视图。

**数据观测** ：当 Vue 实例创建时，它会遍历 data 对象的所有属性，并使用 Object.defineProperty 将每个属性转换为响应式的。这个过程由 Observer 类完成。

**依赖收集** ：当模板渲染或计算属性计算时，Vue 会追踪哪些数据被访问了。这通过 Dep 类和 Watcher 类完成。Watcher 会在读取数据时将自身添加到数据的依赖列表中。

**数据变更通知** ：当数据被修改时，对应的 Watcher 会收到通知，并触发视图更新。

**2.Vue 3 的响应式系统** 
Vue 3 引入了新的响应式 API，使用 Proxy 替代了 Object.defineProperty。这提供了更高效、更简洁的解决方案，同时也更好地支持了现代浏览器（ES6）。

**数据包装** ：在 Vue 3 中，响应式数据不再是直接修改的原生对象，而是通过 reactive 函数包装后的代理对象。这个代理对象使用 Proxy 创建，可以拦截所有的读取和写入操作。

**读取操作的追踪** ：当访问响应式数据的属性时，Proxy 的 get 方法会被调用，Vue 的响应式系统会记录下这次读取操作，并将其与当前的副作用函数（effect）关联起来。

**写入操作的追踪** ：当修改响应式数据的属性时，Proxy 的 set 方法会被调用，Vue 的响应式系统会检查哪些副作用函数依赖于这个属性，并将它们标记为需要更新。

**触发更新** ：当执行到被标记为需要更新的副作用函数时，Vue 的调度器会确保它们重新执行，从而触发视图的更新。这个过程通常是异步的，以提高性能。