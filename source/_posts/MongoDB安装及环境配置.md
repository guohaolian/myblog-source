---
title: Windows环境下MongoDB安装及环境配置
date: 2024-02-16 10:34:23
wordcount: 600
categories: 教程
category_bar: true
tags:
  - 开发工具
---
# 安装MongoDB

打开MongoDB官网的下载页面，地址为 https://www.mongodb.com/try/download/community。单击 Select Package 显示版本选择界面。在 Version 下拉列表中选择版本 8.0.4，在 Platform 下拉列表中选择 Windows X64 操作系统，安装包格式选择 msi，单击 Download 按钮，开始下载，如下图所示。

![image-20260206233724804](/img/mongdb/image-20260206233724804.png)

下载完成之后，双击 .msi 安装包文件开始安装。在安装过程中会弹出安装向导界面，指导使用者安装 MongoDB 以及 MongoDB 的可视化工具 MongoDB Compass。安装包运行后，界面如下图所示：

![image-20260206233752203](/img/mongdb/image-20260206233752203.png)

安装向导提示当前安装的 MongoDB 版本信息，如下图所示，单击 Next 按钮。

![image-20260206233809718](/img/mongdb/image-20260206233809718.png)

接受当前的终端用户安装协议，选中同意协议的复选框（I accept the terms in the License Agreement），单击 Next 按钮，如下图所示。

![image-20260206233827389](/img/mongdb/image-20260206233827389.png)

选择安装类型，MongoDB 支持完整安装和个性化安装。个性化安装方式支持选择所需要的安装组件，同时也支持自定义 MongoDB 的安装路径。官方推荐的是完整安装。对于初学者或对 MongoDB 使用不熟练的使用者，建议默认完整安装。此方式会将 MongoDB 安装在路径 C:\Program Files\MongoDB\Server\8.0 下。选择 Complete 安装类型后，单击 Next 按钮，如下图所示。

![image-20260206234058661](/img/mongdb/image-20260206234058661.png)

配置服务，从 4.0 版本开始，MongoDB 可以配置为一项 Windows 服务。在这个步骤中，可以选择将 MongoDB 作为 Windows 的一个系统服务，如下图所示。

![image-20260206234604014](/img/mongdb/image-20260206234604014.png)

在这个界面中，可以配置服务的名称，一般默认为 MongoDB，同时配置 MongoDB 的数据目录和日志目录。配置完成后，单击 Next 按钮。如果这里不将 MongoDB 配置为 Windows 的服务，可以手动启动，具体可以查看后面关于手动启动 MongoDB 实例的内容。

安装可视化工具 MongoDB Compass。这里为可选项，既可以安装，也可以不安装。这里选中进行安装，单击 Next 按钮进入下一步，如下图所示。

![image-20260206234642420](/img/mongdb/image-20260206234642420.png)

进入安装进程。上述所有配置完成，单击 Install 按钮进入安装进程，如下图所示：

![image-20260206234659484](/img/mongdb/image-20260206234659484.png)

等待安装进程进行。整个过程可能会持续较长时间，与所使用的计算机配置有关，不同配置耗费的时间有所不同，耐心等待即可，如下图所示。

![image-20260206234717637](/img/mongdb/image-20260206234717637.png)

安装进程结束，完成 MongoDB 和 MongoDB Compass 的安装。单击 Finish 按钮，退出安装进程，如下图所示。

![image-20260206234733567](/img/mongdb/image-20260206234733567.png)

如果将 MongoDB 作为服务安装，那么安装结束后，会自动开启服务。要查看 MongoDB 服务的状态，可以打开系统服务界面，按 Ctrl + Shift + Esc 组合键，打开任务管理器，切换到“服务”选项卡，或者直接通过 Windows 系统的搜索找到“服务”界面，在“服务”界面中找到 MongoDB 服务，查看服务是否正在运行。在服务名称上右击，可以选择开启或停止该服务，如下图所示。

![image-20260206234819093](/img/mongdb/image-20260206234819093.png)

或者打开浏览器访问 https://localhost:27017，如下图所示：

![image-20260206234839108](/img/mongdb/image-20260206234839108.png)

同样，也可以通过命令查看 MongoDB 服务的状态，开启或停止 MongoDB 服务。命令如下：

`

```
#查看MongoDB服务的状态
sc query MongoDB
#停止MongoDB服务
net stop MongoDB
#开启MongoDB服务
net start MongoDB
```

`

执行结果如下图所示：

![image-20260206235126005](/img/mongdb/image-20260206235126005.png)

如果不作为服务安装，那么需要手动启动 MongoDB 实例，这里需要用到 mongod 命令。该命令在 MongoDB 安装目录下的 bin 文件夹（D:\MongoDB\Server\8.0\bin）下。启动时，需要配置数据目录，例如 D:\MongoDB\data，如下图所示。

![image-20260206235201031](/img/mongdb/image-20260206235201031.png)

MongoDB 启动后，该数据目录会生成一些数据库文件，如下图所示：

![image-20260206235217879](/img/mongdb/image-20260206235217879.png)

在 bin 目录下，存在一些关键文件，如下图所示：

![image-20260206235240868](/img/mongdb/image-20260206235240868.png)

其中，mongod.exe 用来启动 MongoDB 服务，mongos.exe 用来管理分片集群。

# 配置环境变量

为了方便使用 mongod 以及其他命令，可以将 MongoDB 的安装路径加入环境变量。在配置了环境变量后，以上命令就可以省略路径，只保留命令本身的名称。打开环境变量配置界面，检查是否已自动创建环境变量，如果没有，则进行手动创建。

首先创建 MONGO_HOM E的系统变量，如下图所示：

![image-20260206235337935](/img/mongdb/image-20260206235337935.png)

然后编辑环境变量 Path，在最后加入一行内容，内容为：

`%MONGO_HOME%\bin`

最终结果如下图所示：

![image-20260206235425756](/img/mongdb/image-20260206235425756.png)

这样就完成了环境变量的配置。之后使用相关命令时，不需要再进入命令所在目录。
