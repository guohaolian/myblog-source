---
title: 使用 Hexo+GitHub 搭建个人免费博客教程
date: 2026-01-17 15:30:23
wordcount: 467
excerpt: 作为一名喜欢技术的爱好者，平时喜欢把自己学习技术的心得或者一些踩坑、易错的过程记录下来，首选的是技术平台(博客)，下面教大家如何利用Github来搭建一个自己的个人博客平台。
categories: 教程
category_bar: true
tags:
  - blog
---

# 1.安装git

1.访问git官方地址，下载对应的安装包，进行安装（简单的点击下一步）。

2.安装好之后，鼠标右键可以看到：Git Bash Here，点击后打开了。

3.输入：git --version

![image-20260117144901894](/img/1.png)

出现这个说明git安装成功。

# 2.安装node.js

[Node.js安装及环境配置超详细教程【Windows系统】_windows 安装nodejs-CSDN博客](https://blog.csdn.net/Nicolecocol/article/details/136788200)

此教程里最后的npm源改为：

```
npm config set registry https://registry.npmmirror.com/
```

# 3.安装Hexo

打开终端输入：

```
npm install -g hexo-cli
```

创建博客：

```
hexo init myblog
cd myblog
npm install
```

构建静态页面，启动本地预览：

```
hexo g
hexo s
```

然后访问：

```
http://localhost:4000
```

# 4.新建Markdown文章

运行：

```
hexo new post "我的第一篇博客"
```

然后会生成：

```
source/_posts/我的第一篇博客.md
```

你用 VSCode 打开它，里面类似这样：（也可以用Typora等md软件打开编辑）

```
---
title: 我的第一篇博客
date: 2026-01-16
categories: 学习笔记
tags:
  - 深度学习
  - Python
---

这里开始写正文，支持 **Markdown**：

## 标题
- 列表
- **加粗**
- `代码块`
```

保存后：

```
hexo server
```

刷新网页就能看到你的文章

# 5.分类&标签管理

可以直接再文章里面指定：

```
categories: 分类name
tags:
  - 标签1
  - 标签2
```

Hexo会自动生成分类页面，标签页面，归档页面

# 6.部署到GitHub

## 1.创建GitHub仓库：

名字：你的用户名.github.io  因为这个仓库名称将作为你github博客的访问地址

![image-20260117151642558](/img/2.png)

## 2.安装部署插件

```
npm install hexo-deployer-git --save
```

## 3.修改_config.yml

找到最后面deploy:''替换为：

```
deploy:
  type: git
  repo: https://github.com/你的用户名/你的用户名.github.io.git
  branch: main
```

## 4.一键部署

```
hexo clean
hexo g
hexo deploy
```

然后访问：

https://你的用户名.github.io

# 7.修改配置

文件_config.yml中可以修改title，language等配置项

# 8.修改主题

可以去网上搜索自己喜欢的主题，我推荐Fluid主题，以这个主题为例：(详情可以浏览用户手册[开始使用 | Hexo Fluid 用户手册](https://hexo.fluid-dev.com/docs/start/#主题简介))

## 1.安装主题

```
npm install --save hexo-theme-fluid
```

然后在博客目录下创建 `_config.fluid.yml`，将主题的然后在博客目录下创建 `_config.fluid.yml`，将主题的 [_config.yml (opens new window)](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml)内容复制过去。内容复制过去。

## 2.指定主题

修改_config.yml文件：

```
theme: fluid  # 指定主题

language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改
```

其它配置自己根据用户手册自行设置
