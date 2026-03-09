---
title: vue项目，如何修改Element-Plus等UI组件库的样式，三种方式搞定！！！
date: 2026-03-02 11:23:47
wordcount: 520
categories: 学习笔记
category_bar: true
tags:
  - CSS
  - Vue
---
# vue项目，如何修改Element-Plus等UI组件库的样式，三种方式搞定！！！

## 前言

我们在学习和<span style="color: rgb(51, 145, 229); user-select: auto;">使用</span>组件库构建页面的时候，时常会遇到这样的问题。
即，尽管组件库已经提供了较多的功能，来帮助我们构建自定义的效果，但有时仍不能使我们满意。
这个时候我们就不得不修改UI库的样式，来达到想要的状态。
以Element-Plus为例，在<span style="color: rgb(51, 145, 229); user-select: auto;">Vue3</span>中，主要有三种方式可以实现自定义第三方组件库的样式。

## 项目背景

例如，我希望调节Element-Plus中的Autocomplete（自动补全输入框） 组件的输入框宽度，但是查看文档，却发现官方并没有提供这样的接口。
这个时候，我就不得不手动的查看组件的HTML实现，并且对样式进行调整。
首先F12来查看其HTML源码，如下所示。

![源码](/img/1234.png)

这时候我们可以快速发现需要调整样式的div层，然后在Vue项目中通过三种方式进行调整。

## 实现方式

### 全局样式

默认的 `<style>` 标签中的样式就是全局样式，这意味着，其中的任何样式都会对整个项目生效，因此需要谨慎使用。

```html
<style>
.el-input__wrapper {
    width: 600px
}
</style>
```

### 全局选择器 `:global(）` 

全局选择器的效果和全局样式基本一致，但是它可以被用在 `<style scoped>` 中，这样你的组件中既能够定义非全局的样式，又能定义全局样式。

```html
<style scoped>
:global(.el-input__wrapper) {
    width: 600px
}
</style>
```

### 深度选择器（推荐）

深度选择器可以用于定义子组件的专属样式，不易发生冲突。因而相对于前两种定义全局样式的方式更加合适。

```html
<style scoped>
:deep(.el-input__wrapper) {
    width: 600px
}
</style>
```