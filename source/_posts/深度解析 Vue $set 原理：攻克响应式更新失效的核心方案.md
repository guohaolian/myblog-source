---
title: Vue的$set
date: 2025-11-11 11:23:47
wordcount: 2777
categories: 学习笔记
category_bar: true
tags:
  - Vue
---
## 解析Vue$set 原理：攻克响应式更新失效的核心方案

在 Vue 开发过程中，不少开发者都遇到过这样的“坑”：明明修改了数据，视图却迟迟不更新。比如给响应式对象新增属性、直接修改数组索引后，页面没有同步刷新。这时候， `this.$set` （或 `Vue.set` ）就成了关键的解决方案。那么 `$set` 到底是如何工作的？它为何能让“失效”的响应式数据重新生效？本文将从 Vue 2 和 Vue 3 的底层实现差异出发，彻底拆解 `$set` 的核心原理与使用场景。

### 一、前置认知：为什么需要 $set？

要理解 `$set` 的原理，首先要明确它的“存在意义”——弥补 Vue 响应式系统的天然缺陷，而这个缺陷的根源在于不同版本的响应式实现机制：

#### 1. Vue 2 的核心痛点

Vue 2 的响应式基于 `Object.defineProperty` API实现，其核心是“劫持已声明属性的 getter/setter”。但这种实现存在两个致命局限：

- **无法监听对象新增属性** ：初始化时，Vue 只会对 `data` 中已声明的属性添加 getter/setter。后续通过 `this.obj.newKey = value` 新增的属性，没有被劫持，自然无法触发视图更新；

- **无法监听数组索引/长度变化** ：数组的 `arr[0] = 10` 、 `arr.length = 2` 等操作不会触发索引对应的 setter，Vue 2 只能通过重写 `push` / `splice` 等数组方法曲线兼容，但直接修改索引仍会失效。

#### 2. Vue 3 的兼容需求

Vue 3 基于 `Proxy` API 重构了响应式系统， `Proxy` 能拦截对象的所有操作（包括新增属性、数组索引修改），原本不需要 `$set` 就能实现全量响应式。但 `$set` 仍被保留作为 **兼容 API** ，用于处理“非响应式对象新增属性”等特殊场景，避免 Vue 2 项目迁移时的代码改造成本。

简单来说， `$set` 的核心作用是： **将数据（对象属性/数组元素）转化为响应式，并主动触发视图更新** 。

### 二、Vue 2 中 $set 的核心原理：补全响应式 + 触发更新

Vue 2 是 `$set` 发挥作用的主要场景，其原理可概括为“两步走”： **补全目标数据的响应式能力** + **主动触发依赖更新** 。下面分“对象”和“数组”两种场景详细拆解：

#### 1. 核心设计思路

Vue 2 中， `$set` 的本质是：针对不同数据类型（对象/数组），通过特定逻辑补全响应式劫持，再通过依赖管理器（Dep）通知视图更新。其核心流程如下：

```
判断目标类型（对象/数组）→ 补全响应式能力 → 执行数据修改 → 触发依赖更新 → 视图同步
```

#### 2. 场景 1：目标是数组

由于 Vue 2 无法监听数组索引修改， `$set` 采用了“曲线救国”的方案——复用已重写的数组方法 `splice` ：

- Vue 2 初始化时，会重写 `Array.prototype` 上的 `push` / `pop` / `splice` 等方法，让这些方法执行时能触发依赖更新；

- `$set` 对数组的处理逻辑： `Vue.set(arr, index, value)` 等价于 `arr.splice(index, 1, value)` ；

- 原理：通过 `splice` 方法修改数组元素，既实现了数据更新，又能触发 Vue 重写后的逻辑，进而通知视图刷新。

#### 3. 场景 2：目标是对象

针对对象， `$set` 会分“属性已存在”和“属性不存在”两种情况处理：

- **属性已存在** ：直接修改属性值。由于该属性已被 `Object.defineProperty` 劫持，修改会触发 setter，自动触发更新，无需额外操作；

- **属性不存在** ：这是 `$set` 的核心场景。需要先通过 `Object.defineProperty` 为新增属性添加 getter/setter（补全响应式能力），再修改属性值，最后触发更新。

#### 4. 关键细节： `__ob__` 属性的作用

Vue 2 中，所有响应式对象都会被添加一个隐藏的 `__ob__` 属性（指向 `Observer` 实例），它是 `$set` 实现的核心“桥梁”：

- 标记对象为“响应式对象”， `$set` 会通过 `target.__ob__` 判断是否需要补全响应式；

- 持有 `dep` 实例（依赖管理器），用于后续触发依赖更新；

- 负责递归劫持对象的子属性，实现深层响应式。

#### 5. Vue 2 $set 简化版实现代码

下面通过简化代码还原 Vue 2 中 `$set` 的核心逻辑，帮助理解底层实现：

```javascript
// Vue 2 中 $set 的核心逻辑简化
function $set(target, key, value) {
  // 场景 1：目标是数组（通过 splice 触发更新）
  if (Array.isArray(target) && typeof key === 'number') {
    // 处理索引超出数组长度的情况（如给 arr[5] 赋值，arr 原长度为 3）
    target.length = Math.max(target.length, key);
    // 调用 Vue 重写后的 splice 方法，自动触发响应式更新
    target.splice(key, 1, value);
    return value;
  }

  // 场景 2：属性已存在于对象中（直接修改，触发 setter）
  if (key in target && !(key in Object.prototype)) {
    target[key] = value;
    return value;
  }

  // 场景 3：对象新增属性（补全响应式）
  const ob = target.__ob__; // 获取响应式对象的 Observer 实例

  // 非响应式对象（如普通字面量对象），直接赋值即可
  if (!ob) {
    target[key] = value;
    return value;
  }

  // 为新增属性添加 getter/setter，补全响应式劫持
  Object.defineProperty(target, key, {
    enumerable: true, // 确保属性可枚举（不影响遍历）
    configurable: true, // 确保属性可删除
    get() {
      return value;
    },
    set(newVal) {
      if (newVal !== value) {
        value = newVal;
        ob.dep.notify(); // 属性修改时，触发依赖更新
      }
    }
  });

  // 主动通知依赖更新，确保视图同步
  ob.dep.notify();
  return value;
}
```

从代码中可以清晰看到， `$set` 针对不同场景做了精准处理：数组复用 `splice` 触发更新，对象新增属性补全 getter/setter，最终都通过 `ob.dep.notify()` 通知视图刷新。

### 三、Vue 3 中 $set 的原理：兼容为主，本质是直接赋值

Vue 3 的响应式系统基于 `Proxy` 实现， `Proxy` 能拦截对象的所有操作（包括新增属性、数组索引修改、删除属性等），原本不需要 `$set` 就能实现全量响应式。因此 Vue 3 中的 `$set` 更多是 **兼容 API** ，核心逻辑大幅简化：

#### 1. 核心原理

Vue 3 中， `$set` 的本质就是“直接赋值”——因为 `Proxy` 会自动拦截 `target[key] = value` 操作，无论目标是对象还是数组，都能自动触发响应式更新。 `$set` 仅作为语法糖保留，避免 Vue 2 项目迁移时的代码改造。

#### 2. Vue 3 $set 简化版实现代码

```javascript
// Vue 3 中 $set 的核心逻辑（兼容为主）
function $set(target, key, value) {
  // 数组：直接修改索引，Proxy 自动拦截 set 操作
  if (Array.isArray(target)) {
    target[key] = value;
    return value;
  }

  // 对象：直接添加/修改属性，Proxy 自动拦截 set 操作
  target[key] = value;
  return value;
}
```

可以看到，Vue 3 中的 `$set` 没有复杂的响应式补全逻辑，只是简单的赋值操作，完全依赖 `Proxy` 的拦截能力实现响应式更新。

### 四、$set 的使用场景与注意事项

#### 1. 必须使用 $set 的场景（重点针对 Vue 2）

- **给响应式对象新增属性** ：

  ```javascript
  // Vue 2 中，直接新增属性无响应式
  this.user = { name: '张三' };
  this.user.age = 20; // 视图不更新
  this.$set(this.user, 'age', 20); // 视图更新，age 成为响应式属性
  ```

- **直接修改数组索引/超出原长度添加元素** ：

  ```javascript
  // Vue 2 中，直接修改索引无响应式
  this.arr = [1, 2, 3];
  this.arr[0] = 10; // 视图不更新
  this.$set(this.arr, 0, 10); // 视图更新
  this.$set(this.arr, 3, 4); // 新增索引 3，视图更新
  ```

#### 2. 无需使用 $set 的场景

- Vue 2 中，修改对象已存在的响应式属性（直接触发 setter）；

- Vue 2 中，数组使用 `push` / `pop` / `splice` 等 Vue 重写的方法（本身会触发更新）；

- Vue 3 中，任何对象/数组的属性修改（ `Proxy` 自动拦截，直接赋值即可）；

- 给非响应式对象添加属性（无需响应式，直接赋值即可）。

#### 3. 关键注意事项

- **Vue 2 中不能给根对象新增属性** ： `$set` 无法给 Vue 实例（ `this` ）或 `data` 根对象（如 `this.$set(this, 'newKey', 1)` ）新增属性，这类操作会直接失效；

- **Vue 2 中仅对响应式对象有效** ：若目标是普通对象（非响应式，无 `__ob__` 属性）， `$set` 仅会执行赋值，不会使其成为响应式；

- **Vue 3 建议直接赋值** ：Vue 3 中 `$set` 是兼容 API，性能与直接赋值无差异，建议优先使用 `target[key] = value` ，减少不必要的 API 依赖。

### 五、$set 与其他更新方式的对比

为了更清晰理解 `$set` 的优势，我们对比 Vue 2 中其他常见的“更新触发”方式：

| 操作方式 | 能否触发响应式更新（Vue 2） | 核心原理 |
|:---:|:---:|:---:|
| `this.obj.newKey = value`  | 否 | 新增属性未被 `Object.defineProperty` 劫持 |
| `this.arr[0] = value`  | 否 | 直接修改索引未触发重写的数组方法 |
| `this.arr.splice(index, 1, value)`  | 是 | 调用重写后的 `splice` 方法，触发依赖更新 |
| `this.$set(target, key, value)`  | 是 | 补全响应式能力 + 主动触发依赖更新 |
| `this.$forceUpdate()`  | 是 | 强制触发组件重新渲染（不推荐，性能开销大） |


可以看到， `$set` 是最精准、最高效的解决方案，既补全了响应式能力，又避免了 `$forceUpdate()` 带来的不必要性能损耗。

### 六、总结

`$set` 的核心价值是“解决 Vue 2 响应式系统的局限性”，其原理可概括为：

- **Vue 2** ：针对对象，通过 `Object.defineProperty` 为新增属性补全 getter/setter；针对数组，复用重写的 `splice` 方法，最终都通过 `Dep.notify()` 触发视图更新；

- **Vue 3** ：仅作为兼容 API，本质是直接赋值，依赖 `Proxy` 的拦截能力实现响应式更新。

理解 `$set` 的原理后，在开发中遇到“数据变视图不变”的问题时，就能快速定位根源：是否是对象新增属性、数组索引修改导致的响应式失效？此时 `$set` 就是最直接的解决方案。而在 Vue 3 项目中，虽然 `$set` 仍可用，但更推荐直接赋值，遵循 `Proxy` 带来的更简洁的响应式开发体验。

掌握 `$set` 不仅能帮你避开开发中的“坑”，更能让你深入理解 Vue 响应式系统的底层逻辑，在面对复杂数据更新场景时更加游刃有余。