---
date: "2018-06-25 18:04:09"
title: 译：openfaas-workshop-Lab1
---

今天大多数公司在开发应用程序并将其部署在服务器上的时候，无论是选择公有云还是私有的数据中心，都需要提前了解究竟需要多少台服务器、多大容量的存储和数据库的功能等。并需要部署运行应用程序和依赖的软件到基础设施之上。假设我们不想在这些细节上花费精力，是否有一种简单的架构模型能够满足我们这种想法？这个答案已经存在，这就是今天软件架构世界中新鲜但是很热门的一个话题——Serverless（无服务器）架构。

目前已经有一批优秀的serverless架构开源项目，OpenFaas就是其中的佼佼者。奈何其中的中文资料比较少，我也是边学边翻译，希望能够抛砖引玉，助力serverless的发展。

这是一个自学研讨会，学习如何构建、部署和运行OpenFaas 函数。

# Lab1 - OpenFaas的准备工作

OpenFaas可以在Docker Swarm和Kubernetes的过几个主要平台之上运行。在此教程里，我们将会在的您本地电脑使用Docker Swarm来入门。

## 预备条件

### Docker

Mac

- [Docker CE for Mac Edge Edition](https://store.docker.com/editions/community/docker-ce-desktop-mac)

Windows

- 仅针对windows10 专业版或企业版
- 安装[Docker CE for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)
- 安装[Git Bash](https://git-scm.com/downloads)

> 备注：所有步骤中请使用Git Bash：不要尝试使用WSL或Bash for Windows。

Linux - Ubuntu 或 Debian

- Docker CE for Linux

> 你可以从[Docker Store](https://store.docker.com/)中安装Docker CE

### 设置一个单节点的Docker Swarm

OpenFaas在Docker Swarm和Kubernetes上工作。因为Docker Swarm很容易设置，所以在此Workshop中我们使用Docker Swarm。在文档中有他们两个的指南。

在你的笔记本或虚拟机中设置一个单节点的Docker Swarm：

```
$ docker swarm init
```

> 如果运行此命令出错，加上 --advertise-addr 你的IP  参数。

### Docker Hub

注册一个Docker Hub账号。Docker Hub允许你在互联网中发布自己的Docker镜像来用于多节点集群或社区共享。在Workshop中我们使用Docker Hub发布函数。

你可以在这里注册：[Docker Hub](https://hub.docker.com/)

> 备注：Docker Hub也可以设置为自动构建镜像。



打开一个终端或者Git Bash窗口，然后使用上面注册的用户名登陆Docker Hub。

```
$ docker login
```

### OpenFaas CLI

你可以在mac上使用brew或者在Linu和mac上使用一个集成脚本来安装OpenFaas CLI。

在Mac或Linux上终端中输入：

```
$ curl -sL cli.openfaas.com | sudo sh
```

对于windows平台，从[releases page](https://github.com/openfaas/faas-cli)中下载最新的的faas-cli.exe。你可以把它放在一个local文件夹或者在C:\Windows\路径中，这样它就可以在命令行中使用。

> 如果你是一个高级Windows用户，把CLI放在你自定义的文件夹中，然后把此文件夹添加到环境变量。

我们将会使用faas-创建新函数的脚手架，build，deploy和invoke函数。你可以从faas-cli —help中找到这些命令。

测试faas-cli

打开一个终端或Git Bash窗口，然后输入：

```
$ faas-cli help
$ faas-cli version
```

### 部署OpenFaas

发布OpenFaas的说明文档修改了很多次，因为我们努力使他简单。接下来将会在60秒左右的时间使得OpenFaas部署起来。

- 首先clone项目

```
git clone https://github.com/openfaas/faas
```

- 然后使用git检出到最新版本

```
$ cd faas && \
  git checkout master
```

> 备注：你也可以在[project release page](https://github.com/openfaas/faas/releases)中找到最新导入release版本。

- 现在使用Docker Swarm部署stack

```
$  ./deploy_stack.sh
```

你现在应该已经把OpenFaas部署了。

如果你现在在一个共享WIFI连接中，它将会需要几分钟时间拉取镜像并启动。

在此屏幕上检查服务是否显示为1/1:

```
$ docker service ls
```

如果你期间有遇到任何问题，请查阅Docker Swarm的 [部署指南](https://github.com/openfaas/faas/blob/master/guide/deployment_swarm.md)。

现在进入[Lab 2](https://github.com/openfaas/workshop/blob/master/lab2.md)。

# 未完待续