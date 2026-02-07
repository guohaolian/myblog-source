---
title: 安装Maven并用IDEA使用Maven构建javaweb项目
date: 2025-12-19 10:34:23
wordcount: 500
categories: 教程
category_bar: true
tags:
  - 开发工具
---

# 1.Maven安装与配置

## 下载maven

在浏览器里打开maven下载页，选择对应的版本。这里以windows为例，下载最新的版本3.9.5。下载地址：https://maven.apache.org/download.cgi

![image-20260207223317939](/img/maven/image-20260207223317939.png)

解压 apache-maven-3.9.5.zip 既安装完成

> [!TIP]
>
> 建议解压缩到没有中文、特殊字符的路径下。

解压缩后的目录结构如下：

![image-20260207223636091](/img/maven/image-20260207223636091.png)

- bin目录 ： 存放的是可执行命令。mvn 命令重点关注。

- conf目录 ：存放Maven的配置文件。 settings.xml 配置文件后期需要修改。

- lib目录 ：存放Maven依赖的jar包。Maven也是使用java开发的，所以它也依赖其他的jar包。

## 配置环境变量

此电脑 右键 --> 高级系统设置 --> 高级 --> 环境变量

![image-20260207223836341](/img/maven/image-20260207223836341.png)

![image-20260207223846449](/img/maven/image-20260207223846449.png)

在系统变量处新建一个变量 MAVEN_HOME,变量值为你的Maven的安装目录

![image-20260207223903180](/img/maven/image-20260207223903180.png)

在 Path 中,点击新建，进行配置

![image-20260207223916950](/img/maven/image-20260207223916950.png)

打开命令提示符,输入‘mvn -version’,进行验证，出现如图所示表示安装成功

![image-20260207223933819](/img/maven/image-20260207223933819.png)

## 配置本地仓库

修改conf/settings.xml中的localRepository标签，指定一个目录作为本地仓库，用来存储jar包。

```xml
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
<localRepository>K:\Java\apache-maven-3.9.5\mvn_resp</localRepository>

```



## 配置阿里云私服

中央仓库在国外，所以下载jar包速度可能比较慢，而阿里公司提供了一个远程仓库，里面基本也都有开源项目的jar包。修改 conf/settings.xml 中的 标签，为其添加如下子标签：

```xml
 <mirror>
      <id>aliyunmaven</id>
      <mirrorOf>*</mirrorOf>
      <name>阿里云公共仓库</name>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
  </mirrors>

```

# 2.IDEA配置Maven环境

选择IDEA中File-->Settings

![image-20260207225645323](/img/maven/image-20260207225645323.png)

![image-20260207225659081](/img/maven/image-20260207225659081.png)

设置 IDEA 使用本地安装的 Maven，并修改配置文件路径

![image-20260207225745410](/img/maven/image-20260207225745410.png)

# 3.IDEA创建Maven的Javaweb项目

新建一个项目，选择Maven Archetype ，jdk这里我选择1.8

![image-20260207225912695](/img/maven/image-20260207225912695.png)

选择Archetype，下拉框选择maven-archetype-webapp

![image-20260207225930756](/img/maven/image-20260207225930756.png)

下拉选择Advanced Settings，

![image-20260207225950845](/img/maven/image-20260207225950845.png)

配置你的Groupld和ArtifactId（坐标），在这里解释一下这个配置

Groupld和ArtifactId统称为坐标：可以唯一标识一个项目

GroupId为项目的组织名称：一般结构为com.公司名，这里属于个人开发，就起名为com.zhang

ArtifactId:通常就是项目名，默认不需要更改

配置完后，点击creat

maven会为你下载一些必要的组件，需要等待几分钟

![image-20260207230040939](/img/maven/image-20260207230040939.png)

# 4.将Maven的Web工程部署到Tomcat中

配置Tomcat

![image-20260207230235610](/img/maven/image-20260207230235610.png)

![image-20260207230251082](/img/maven/image-20260207230251082.png)

点击debug启动，Tomcat启动成功！

![image-20260207230308792](/img/maven/image-20260207230308792.png)