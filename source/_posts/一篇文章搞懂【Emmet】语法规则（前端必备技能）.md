---
title: Emmet常用语法
date: 2026-01-12 10:34:23
wordcount: 1052
categories: 学习笔记
category_bar: true
tags:
  - Javascript
---
# 【Emmet】语法规则

前端人员编写网页代码时可以依靠一些编辑器的语法提示加快编写速度。大多数编辑器也提供emmet插件来 **更快** 的编写HTML和css代码。emmet的语法规则比较简单易理解可以极大的提高编码速度，基本上是前端开发人员必备的一项技能了。下面简单介绍下常用的语法规则和效果。

**以VScode编辑器为例** 

## 1.初始化HTML结构

新建一个html结构后可以<span style="color: rgb(51, 145, 229); user-select: auto;">使用</span> `!` + `tab` 建初始化HTML结构

1. 输入：

![在这里插入图片描述](/img/emmet/0_1.jpeg)

1. 按tab或回车后：

   ![在这里插入图片描述](/img/emmet/0_2.jpeg)

## 2.id（#）和class（.）

生成一个带有id或者类名的emmet语法：

1. id： `div#main` + `tab`

2. class: `div.head` + `tab` ；多个类可以继续追加 `div.head.box` + `tab`

3. 组合： `div#main2.body` + `tab` 和 `div#main2.body.box` + `tab`

**注意** ： **输入完一行要紧接着按 `tab` ！！！！** 

输入

```javascript
div#main
div.head
div.head.box
div#main2.body
div#main2.body.box
```

![在这里插入图片描述](/img/emmet/0_3.jpeg)

每一行后面紧接着按 `tab` 后：

![在这里插入图片描述](/img/emmet/0_4.jpeg)

## 3.子节点（>）、上级节点（^)、兄弟结点（+）

#### 子节点（>）

使用 `>` 连接两个标签名，后者会加在前者的内部（即成为前者的子元素）

例如输入如下：

```
div>p 
div>p>span
```

每一行按 `tab` 会生成：

![在这里插入图片描述](/img/emmet/0_5.jpeg)

#### 上级节点（^）

与 `>` 符号常常同时使用；在 `^` 符号后的元素名会与前一各元素的父级元素同级。 `^` 可以连续使用，表示上升多级。

具体看<span style="color: rgb(51, 145, 229); user-select: auto;">实例</span>：

```
    div>p^a

    div>ul>li^p
    
    div>ul>li^^p
```

每行会生成：

![在这里插入图片描述](/img/emmet/0_6.jpeg)

#### 兄弟结点（+）

被 `+` 符号相连的元素最后生成同级元素。

输入例子：

```
    div+p

    div+a+p
```

结果：

![在这里插入图片描述](/img/emmet/0_7.jpeg)

## 4.重复（*）

有时候需要生成多个同样的标签，可以直接yong `*` 生成而不用一个个复制。

输入例子：

```
    h2*5
    
    ul>li*6
```

结果：

![在这里插入图片描述](/img/emmet/0_8.jpeg)

## 5.属性（[attr]）

与id和class比较类似，是为元素添加其它任意属性的。

例子：

```
    div[name='main']
    
    a[href="www.baidu.com" name="baidu"]
```

结果：

![在这里插入图片描述](/img/emmet/0_9.jpeg)

## 6.编号（$）

动态生成的序号， `$` 代表一位数字，后面跟的 `*` 和数字代表从1递增到紧跟的数字

例子：

```
ul>li.item$*6
```

结果：

![在这里插入图片描述](/img/emmet/0_10.jpeg)

**补充** ：

- 可以设置多位数字（一个 `$` 代表一位数字，就可以连写多个会在前面补0）

例子：

```
ul>li.item$$$*6
```

结果：

![在这里插入图片描述](/img/emmet/0_11.jpeg)

- 可以在 `$` 后加 `@-` 实现倒序

例子：

```
ul>li.item$@-*6
```

结果：

![在这里插入图片描述](/img/emmet/0_12.jpeg)

- 在 `$` 后面添加 `@N` 改变编号的起始基数

例子：(代表从3基数开始，生成共5个)

```
ul>li.item$@3*5
```

结果：

![在这里插入图片描述](/img/emmet/0_13.jpeg)

## 7.文本（{}）

元素后面使用 `{}` 符号可以在元素内部加入 `{}` 内容。

例子：

```
    p{一段文本}

    a[href="www.baidu.com"]{百度}
```

结果：

![在这里插入图片描述](/img/emmet/0_14.jpeg)

## 8.分组（()）

可以组合代码块，写较长的emmet语法时用来分隔。

```
div>(ul>li>a[href="www.baidu.com"])+p
```

结果：

![在这里插入图片描述](/img/emmet/0_15.jpeg)

## 综合：

这里有三个综合的题目，大家只看emmet语法看看能不能猜到会生成什么结构呢？

1. ```
   div#main>ul>li.item$*6
   ```

![在这里插入图片描述](/img/emmet/0_16.jpeg)

1. ```
   (div#head>ul>li>a[href="www.baidu.com"])+(div#main>p*6{$})+div#footer{版权}
   ```

![在这里插入图片描述](/img/emmet/0_17.jpeg)

1. ```
   div#main.main.box>(ul>(li>a[href="www.baidu.com"]{百度})*6)+p*5
   ```

![在这里插入图片描述](/img/emmet/0_18.jpeg)

**虽然很简单但是不要忘记多多练习嗷！！！** 