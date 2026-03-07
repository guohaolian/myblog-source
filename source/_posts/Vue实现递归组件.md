---
title: 递归组件
date: 2026-01-28 11:23:47
wordcount: 789
categories: 学习笔记
category_bar: true
tags:
  - Vue
---

# Vue实现递归组件

## Vue实现递归组件

---

## 前言

在我们开发过程中，为了提高开发效率，降低开发难度，我们会直接使用组件库来实现，同时也衍生出了很多优秀的组件库，如：饿了么、蚂蚁、Iview、vant等等。但是有时这些组件库提供给我们的组件不满足我们的需求或者定制组件时成本太高，那么我们就要手动实现了。

## 一、递归组件是什么？

字面理解为层层递进最后归并到一起，它的特点就是层级分明。
例如饿了么组件库的树组件就是一个递归。

![在这里插入图片描述](/img/digui/0_1.png)

## 二、Vue实现递归的核心思路

1、循环出一级类别
2、判断如果有多级，再调用自身。

## 三、代码示例

### 1.父级

> 代码如下（示例）：

```c
<template>
  <div>
    <!-- 递归组件 -->
    <Recursion :list="list" />  list为获取数据，传入子页面
  </div>
</template>

<script>
import Recursion from "./recursion.vue";

export default {
  name: "index-Recursion",
  components: {
    Recursion,
  },
};
</script>

```

### 2.子级

代码如下（示例）：

```c
<template>
  <div>
    <div class="item">
      <div>
          <ul>
            <li v-for="(l, id) of list" :key="id">
              {{ l.name }}
              <ul style="padding-left: 20px" v-if="l.chidren"> // 核心代码1
                <li>
                    <index-chird :list="l.chidren" /> // 核心代码2
                </li>
              </ul>
            </li>
          </ul>
        </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "index-chird", // 自身组件
  props: {
    list: Array,
  },
  data() {
    return {
      list: [],
    };
  },

  watch: {
    list(newData) {
      this.list = newData;
    },
  },
  
};
</script>

<style  scoped>
.item {
  margin: 0 auto;
}
</style>
```

此处使用监听器监听数据变化，如果正常发请求传递数据不需要监听，如果报出没有拿到数据的错误可使用监听器。

### 3、实现效果

![在这里插入图片描述](/img/digui/0_2.png)

## 四、总结

很简单的一个demo，重点是我们是否了解Vue每个组件定义的name的真正用途是什么。每个组件的name值其实也是为了帮助我们实现递归的。

代码逻辑也很简单，重点在我的子组件。但父组件传过来的树形数据结构到子组件后，我们需要拿到数据并做遍历，然后再下一行加入核心逻辑：if 发现我们有子数据，那么我们直接调用自身组件，也就是直接使用name值做组件声明。最关键的是要把子数据结构再传入我们自身组件，那么我们就成功的实现了数据的层层遍历。

当然，这块儿的子数据结构字段我这里叫chirden，一般企业开发是后台给我们的，他们也可以叫A，叫B，我们需要根据自己的数据字段情况，去做相应的修改。

以上就是vue实现的简单的递归组件。欢迎大家提出更好的方案与建议~