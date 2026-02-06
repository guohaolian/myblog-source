---
title: 修改MySQL的远程授权登录设置
date: 2026-01-19 10:34:23
wordcount: 300
categories: 教程
category_bar: true
tags:
  - 开发工具
---
# 1.登录远程服务器的数据库

`mysql -u root -p   ##以root登录数据库`

输入root的登录密码，成功后会看到以下信息：

![image-20260207004216868](/img/remotemysql/image-20260207004216868.png)

# 2.查看mysql服务当前的默认端口

`use mysql;   ##选择mysql数据库`

`select user,host from user;     ##查看用户访问端口`

![image-20260207004450265](/img/remotemysql/image-20260207004450265.png)

<!--**说明**：root用户默认的是localhost，说明只允许从本地登录mysql服务。而我们要从远程以root用户连接数据库，就必须修改host的值，改为**‘%’**：允许任何ip访问。-->

# 3.修改host允许任何ip访问

`update user set host = '%' where user = 'root';`

![image-20260207004649695](/img/remotemysql/image-20260207004649695.png)

看到以上信息说明修改成功！

这时再使用之前的命令：`select user,host from user;    ## 查看用户访问端口`

会看到：root用户的host已经修改为’%'！

![image-20260207004729999](/img/remotemysql/image-20260207004729999.png)

<!--修改完后需要刷新一下服务配置，不然修改不会生效，并且第4步会执行失败-->

接着在命令面板输入：

`mysql> FLUSH PRIVILEGES;    ## 刷新服务配置项`

显示**Query OK**,表示刷新完成。现在就可以配置用户权限

# 4.授权root用户进行远程登录

`mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root_pwd'; ## 授权root远程登录 后面的root_pwd代表登录密码`

输入完之后，看到**Query OK**，说明执行成功！

<!--**说明**：此命令可以授权任何在mysql数据库user表中的用户以远程登录的方式访问数据库，本例中以’root’作为举例，若想授权其他用户，只需修改’root’的值为指定用户即可，'root_pwd’为’root’用户对应的登录密码，可以修改为你想要授权用户的登录密码。-->
