---
title: Windows环境下Node.js安装及环境配置
date: 2024-03-16 10:34:23
wordcount: 600
categories: 教程
category_bar: true
tags:
  - 开发工具
---
# 1.下载node.js

下载地址：[Node.js — Download Node.js®](https://nodejs.org/en/download)

![image-20260206222904416](/img/nodejs/image-20260206222904416.png)

# 2.Node.js安装

![image-20260206223055194](/img/nodejs/image-20260206223055194.png)

勾选“同意安装许可”，接着到下一步。

![image-20260206223135194](/img/nodejs/image-20260206223135194.png)

选定好一个目录安装即可。

![image-20260206223420989](/img/nodejs/image-20260206223420989.png)

选择默认，点击Next按钮。

![image-20260206223522017](/img/nodejs/image-20260206223522017.png)

点击Install，进一步等待安装即可。

![image-20260206223544609](/img/nodejs/image-20260206223544609.png)

![image-20260206223628832](/img/nodejs/image-20260206223628832.png)

按下win+R，打开命令提示符，查看是否安装完成，我们来验证一下，输入node -v和npm -v命令来验证即可。

![image-20260206223659998](/img/nodejs/image-20260206223659998.png)

# 3.Node.js环境配置

安装完之后的node.js目录下的文件如下所示。

![image-20260206223819497](/img/nodejs/image-20260206223819497.png)

接下来我们创建两个空的文件夹，分别是node_cache还有node_global文件夹。

![image-20260206223838063](/img/nodejs/image-20260206223838063.png)

在cmd中执行如下命令，配置缓存目录和全局目录。

`npm config set prefix "D:\nodejs\node_global"`

`npm config set prefix "D:\nodejs\node_cache"`

如果出现标红报错，是由于权限的原因，右击Nodejs文件夹->属性->安全，点击编辑，将所有权限都 ✔即可。或者以 管理员身份运行 cmd。

![image-20260206224531671](/img/nodejs/image-20260206224531671.png)

以记事本格式打开.npmrc文件，把下面两行命令放进去。

![image-20260206224607862](/img/nodejs/image-20260206224607862.png)

![image-20260206224641014](/img/nodejs/image-20260206224641014.png)

 接着打开系统环境变量，进行配置，配置截图如下。

![image-20260206224704728](/img/nodejs/image-20260206224704728.png)

然后把用户变量下的Path路径中npm修改为D:\nodejs\node_global

![image-20260206224731096](/img/nodejs/image-20260206224731096.png)

 也就是把下面的“C:\User\AppData\Roaming\npm”修改为“D:\nodejs\node_global”，即可，最后修改之后点击确定即可。

![image-20260206224803425](/img/nodejs/image-20260206224803425.png)

# 4.更换npm源为淘宝镜像

npm 默认的 registry ,也就是下载 npm 包时是从国外的服务器下载，国内很慢，一般都会指向淘宝https://registry.npmmirror.com/

在cmd里输入`npm config set registry https://registry.npmmirror.com/`

检查配置是否成功：`npm config get registry`

配置完成后，安装个module测试下，输入npm install express -g,进行模块的全局安装

安装好之后可以看到目录如下。

![image-20260206225437908](/img/nodejs/image-20260206225437908.png)