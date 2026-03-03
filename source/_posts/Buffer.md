---
title: Buffer(缓冲器)
date: 2025-10-03 10:34:23
wordcount: 200
categories: NodeJS学习笔记
category_bar: true
tags:
  - nodejs
---

# 1.概念

Buffer 是一个类似于数组的 对象，用于表示固定长度的字节序列。 Buffer 本质是一段内存空间，专门用来处理 二进制数据。

![image-20260214232753889](/img/node/image-20260214232753889.png)

# 2.特点

1.Buffer大小固定且无法调整

2.Buffer性能较好，可以直接对计算机内存进行操作

3.每个元素的大小为1字节（byte）

![image-20260214232939344](/img/node/image-20260214232939344.png)

# 3.使用

## 3-1.创建Buffer

Node.js中创建Buffer的方式主要如下几种：

1.Buffer.alloc

```js
//创建了一个长度为 10 字节的 Buffer，相当于申请了 10 字节的内存空间，每个字节的值为 0
let buf_1 = Buffer.alloc(10); // 结果为 <Buffer 00 00 00 00 00 00 00 00 00 00>
```

2.Buffer.allocUnsafe

```js
//创建了一个长度为 10 字节的 Buffer，buffer 中可能存在旧的数据, 可能会影响执行结果，所以叫 
unsafe
let buf_2 = Buffer.allocUnsafe(10);
```

3.Buffer.from

```js
//通过字符串创建 Buffer
let buf_3 = Buffer.from('hello');
//通过数组创建 Buffer
let buf_4 = Buffer.from([105, 108, 111, 118, 101, 121, 111, 117]);

```

## 3-2.Buffer与字符串的转化

可以借助  toString 方法将 Buffer 转为字符串

```js
let buf_4 = Buffer.from([105, 108, 111, 118, 101, 121, 111, 117]);
console.log(buf_4.toString())
```

> [!NOTE]
>
> toString默认是按照utf-8编码方式进行转换的。

## 3-3.Buffer的读写

Buffer 可以直接通过 [] 的方式对数据进行处理。

```js
//读取
console.log(buf_3[1]);
//修改
buf_3[1] = 97;
//查看字符串结果
console.log(buf_3.toString());
```

> [!CAUTION]
>
> 1. 如果修改的数值超过  255 ，则超过 8 位数据会被舍弃 。
> 2.  一个 utf-8 的字符 一般占 3 个字节。

<div style="border-left:4px solid #4CAF50;background:#E8F5E9;padding:12px 16px;margin:1em 0;">
    这里是内容
</div>
