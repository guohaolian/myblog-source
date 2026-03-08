---
title: Vue 3 的 v-memo：高效的渲染优化利器
date: 2025-01-16 11:23:47
wordcount: 1380
categories: 学习笔记
category_bar: true
tags:
  - Vue
---
# Vue 3 的 v-memo：高效的渲染优化利器

在日常的Vue项目开发中，我们经常会遇到这样的情况：
父组件更新了，但子组件明明没有用到相关<span style="color: rgb(51, 145, 229); user-select: auto;">数据</span>，却也跟着无意义地重新渲染。这不仅浪费性能，还可能导致页面卡顿。

`Vue 3` 在 **3.2 版本** 引入了一个新的指令 —— **`v-memo`** ，它的作用就是帮助我们 **跳过不必要的渲染** ，让应用运行得更加丝滑。

本文将从概念、场景、示例、性能对比，再到和大家熟悉的 `v-if` 、 `v-show` 的比较，带你全面理解 `v-memo` 。

---

### 1. 什么是 `v-memo` 

- `v-memo` 是 `Vue 3.2` 新增的一个 **渲染优化指令** 。

- 它接收一个依赖数组作为参数。

- **当依赖数组中的值没有发生变化时，Vue 会直接复用上一次的虚拟 DOM，跳过这部分的渲染更新。**

基本语法：

```vue
<div v-memo="[dep1, dep2, dep3]">
  <!-- 只有 dep1 / dep2 / dep3 变化时才会重新渲染 -->
</div>
```

#### 和 `v-once` 的区别

- `v-once` ：只渲染一次，完全静态，后续不会更新。

- `v-memo` ：依赖没变就跳过更新，但依赖变了还是会更新。

---

### 2. 适用场景

1. **静态内容多、动态依赖少** 
   大部分内容不变，只有少数依赖可能发生变化。

2. **复杂子组件** 
   父组件频繁更新时，避免子组件跟着无意义刷新。

3. **长列表渲染** 
   大量数据时， `v-memo` 可以显著减少 DOM diff。

---

### 3. 基础示例

```vue
<template>
  <div>
    <button @click="count++">Count: {{ count }}</button>
    
    <!-- 只有 user.name 变化时才会重新渲染 -->
    <div v-memo="[user.name]">
      <p>{{ user.name }}</p>
      <p>当前时间：{{ new Date().toLocaleTimeString() }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const count = ref(0);
const user = ref({ name: "Alice" });
</script>
```

#### 效果

- 点击按钮 `count++` ：父组件更新，但 `user.name` 没变， `v-memo` 区域不会刷新。

- 修改 `user.name` ：依赖变化 → `v-memo` 区域正常更新。

---

### 4. 性能对比<span style="color: rgb(51, 145, 229); user-select: auto;">Demo</span>（1000 子组件）

我们再来看一个更典型的场景：渲染 **1000 个子组件** 。

```vue
<template>
  <div>
    <h2>v-memo 性能对比 Demo</h2>
    <button @click="count++">点我增加 count: {{ count }}</button>
    <button @click="toggleName">修改名字</button>

    <h3>❌ 没有 v-memo（每次都会更新）</h3>
    <div class="list">
      <ChildNoMemo v-for="i in 1000" :key="i" :index="i" :name="user.name" />
    </div>

    <h3>✅ 使用 v-memo（只有 name 改变才更新）</h3>
    <div class="list">
      <ChildWithMemo v-for="i in 1000" :key="i" :index="i" :name="user.name" />
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const count = ref(0);
const user = ref({ name: "Alice" });

function toggleName() {
  user.value.name = user.value.name === "Alice" ? "Bob" : "Alice";
}

// 子组件 A：不使用 v-memo
const ChildNoMemo = {
  props: ["index", "name"],
  template: `
    <div>
      <span>子项 {{ index }} - {{ name }} - {{ new Date().toLocaleTimeString() }}</span>
    </div>
  `
}

// 子组件 B：使用 v-memo
const ChildWithMemo = {
  props: ["index", "name"],
  template: `
    <div v-memo="[name]">
      <span>子项 {{ index }} - {{ name }} - {{ new Date().toLocaleTimeString() }}</span>
    </div>
  `
}
</script>

<style>
.list {
  max-height: 300px;
  overflow-y: auto;
  border: 1px solid #ddd;
  margin-bottom: 20px;
}
</style>
```

#### 效果对比

1. **点击「增加 count」** 

   - 无 `v-memo` ：1000 个子组件全部更新，时间刷新。

   - 有 `v-memo` ：没有子组件更新，时间保持不变。

2. **点击「修改名字」** 

   - `user.name` 改变时，有 `v-memo` 的子组件才会更新。

👉 在开发者工具里可以直观看到， `v-memo` 大幅减少了渲染次数。

---

### 5. 和 `v-if` 、 `v-show` 的区别

在 Vue 中，大家更熟悉的还是 **`v-if`** 和 **`v-show`** ，那它们和 `v-memo` 有什么不同呢？

| 特性 | `v-if`  | `v-show`  | `v-memo`  |
|:---:|:---:|:---:|:---:|
| 控制方式 | **条件渲染** ：决定是否创建/销毁 DOM | **条件展示** ：DOM 始终存在，只是切换 `display: none`  | **依赖缓存** ：DOM 始终存在，但依赖没变时跳过更新 |
| 性能消耗 | 创建/销毁 DOM 开销大 | 首次渲染开销大，后续切换性能好 | 渲染时 diff 开销小，适合避免无意义更新 |
| 使用场景 | 条件性显示/隐藏（较少切换） | 频繁切换显隐（如 tab 页） | 避免父组件更新导致的无意义刷新 |


#### 举例

- **`v-if`** ：用户是否已登录 → 渲染「登录框」或「用户中心」。

- **`v-show`** ：tab 切换 → DOM 保留，只切换显示状态。

- **`v-memo`** ：父组件频繁更新 → 子区域依赖没变时直接跳过更新。

👉 一句话总结：
**`if` 管生死， `show` 管显隐， `memo` 管性能。** 

---

### 6. 注意事项

1. `v-memo` 不是万能优化，应该只在性能瓶颈处精准使用。

2. 依赖数组要完整，确保包含所有决定渲染结果的变量。

3. 调试时可以在子组件 `mounted / updated` 里加 `console.log` ，观察更新次数。

---

### 7. 总结

- `v-memo` 是 `Vue 3.2` 引入的性能优化工具。

- 它通过缓存依赖，避免了无意义的渲染。

- 特别适合 **复杂子组件** 和 **长列表场景** 。

- 和 `v-if` 、 `v-show` 不同，它的关注点在于 **优化性能** 而不是 **控制渲染逻辑** 。

