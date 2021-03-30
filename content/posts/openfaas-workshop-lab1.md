---
title: "Openfaas Workshop Lab1"
date: 2021-03-30T09:54:25+08:00
draft: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
disableHLJS: true
disableShare: true
disableHLJS: false
---

# 实验 1 - OpenFaaS 准备

<img src="https://github.com/openfaas/media/raw/master/OpenFaaS_Magnet_3_1_png.png" width="500px"></img>

OpenFaaS 需要一个[Kubernetes](https://kubernetes.io) 集群来操作。您可以使用单节点集群或多节点集群，无论是在笔记本电脑上还是在云中。

任何 OpenFaaS 函数的基本基元都是 Docker 镜像，它是使用`faas-cli`工具链构建的。

## 先决条件：

让我们安装 Docker，OpenFaaS CLI 并设置 Kubernetes。

### Docker

For Mac

- [Docker CE for Mac Edge Edition](https://store.docker.com/editions/community/docker-ce-desktop-mac)

For Windows

- 仅支持 Windows 10 Pro 或 Enterprise
- 安装 [Docker CE for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)

> 请确保通过使用 Windows 任务栏通知区域中的 Docker 菜单来使用**Linux**容器 Docker 守护进程。

- 安装 [Git Bash](https://git-scm.com/downloads)

当您安装 git bash 时，请选择以下选项：“安装 UNIX 命令”和“使用真实类型的字体”。

> 注意：所有步骤请使用*Git Bash*：不要尝试使用*PowerShell*，*WSL*或*Bash for Windows*。

Linux - Ubuntu 或 Debian

- Docker CE for Linux

> 您可以从[Docker Store](https://store.docker.com)上安装 Docker CE。

注意：作为最后的手段，如果您的电脑不兼容，您可以在 https://labs.play-with-docker.com/ 上运行研讨会

### OpenFaaS CLI

您可以使用官方 bash 脚本安装 OpenFaaS CLI，“brew”也可以使用，但可能会落后一两个版本。

使用 MacOS 或 Linux 在终端中运行以下操作：

```sh
$ curl -sLSf https://cli.openfaas.com | sudo sh
```

对于 Windows，在*Git Bash*中运行：

```sh
$ curl -sLSf https://cli.openfaas.com | sh
```

> 如果遇到任何问题，您可以从[发布页面](https://github.com/openfaas/faas-cli/releases)手动下载最新的`faas-cli.exe`。您可以将其放在本地目录或`C:\Windows\`路径中，以便从命令提示符中获得。

我们将使用`faas-cli`来构建新函数、构建、部署和调用函数。您可以使用`faas-cli--help`找到可用的 cli 命令。

测试 faas-cli。打开终端或 Git Bash 窗口并键入：

```sh
$ faas-cli help
$ faas-cli version
```

## 配置 Docker Hub

注册一个 Docker Hub 帐户。[Docker Hub](https://hub.docker.com) 允许您在互联网上发布 Docker 镜像，以便在多节点集群上使用或与更广泛的社区共享。我们将在研讨会期间使用 Docker Hub 发布我们的函数。

你可以在这里注册: [Docker Hub](https://hub.docker.com)

打开终端或 Git Bash 窗口，并使用上面注册的用户名登录 Docker Hub。

```sh
$ docker login
```

> 注意：来自社区的提示-如果您在 Windows 机器上运行此命令时出错，请单击任务栏中的 Docker for Windows 图标，然后登录 Docker，而不是登录/创建 Docker ID。

- 设置 OpenFaaS 前缀

OpenFaaS 镜像存储在 Docker 注册表或 Docker Hub 中，我们可以设置一个环境变量，以便您的用户名自动添加到您创建的新函数中。这将为您在研讨会期间节省一些时间。

编辑 `~/.bashrc` 或 `~/.bash_profile` - 如果文件不存在就创建一个

现在添加以下内容-根据上面看到的更改 URL。

```sh
export OPENFAAS_PREFIX="" # Populate with your Docker Hub username
```

### 设置单节点集群

实验室使用 Kubernetes，OpenFaaS 社区不再支持 Swarm。一些实验将使用 faasd ，但是您可能需要更改命令，当使用 faasd 时，我们不为研讨会提供支持。

- Kubernetes: [实验 1b](./lab1b.md)
