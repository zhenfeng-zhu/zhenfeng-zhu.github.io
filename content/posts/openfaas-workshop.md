---
title: "Openfaas Workshop"
date: 2021-03-30T09:53:12+08:00
draft: false
---

# openfaas-研讨会

这是一个自定进度的研讨会，用于学习如何使用 OpenFaaS 构建、部署和运行无服务器功能。

![](https://github.com/openfaas/media/raw/master/OpenFaaS_Magnet_3_1_png.png)

在这个研讨会中，您首先将 OpenFaaS 部署到您的笔记本电脑或带有 Docker for Mac 或 Windows 的远程集群中。然后，您将使用 OpenFaaS 用户界面、CLI 和函数商店来完成最基本的使用。构建完成后，在 Python 中部署调用自己的无服务器函数，您将继续学习以下主题：使用 pip 管理依赖关系、通过安全机密处理应用编程接口令牌、使用 Prometheus 监控函数、异步调用函数以及将函数链接在一起创建应用程序。实验中让人兴奋的是可以让您创建自己的 GitHub 机器人，它可以自动响应问题。同样的方法也可以通过 IFTTT.com 连接到在线事件流，这将使您能够构建机器人、自动响应器以及与社交媒体和物联网设备的集成。

最后，这些实验也涵盖了更高级的主题，并为进一步学习提供建议。

**其他语言**

- [日本語](./translations/ja)
- [中文](./translations/zh)

## 要求：

我们将介绍如何在[Lab 1](./lab1.md)中安装这些要求。请在参加讲师指导的研讨会之前完成[Lab 1](./lab1.md)

- 函数将用 Python 编写，因此有编程或脚本经验者优先
- 安装推荐的代码编辑器/IDE [VSCode](https://code.visualstudio.com/download)
- Windows 安装 [Git Bash](https://git-scm.com/downloads)
- 首选操作系统： MacOS，Windows 10Pro/Enterprise，Ubuntu Linux

Docker:

- Docker CE for [Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)/[Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) **Edge edition**
- Docker CE for Linux

> 注意：作为最后的手段，如果您有不兼容的 PC，您可以在https://labs.play-with-docker.com/.上运行研讨会

## 讲师主导的研讨会

如果你参加了一个由讲师领导的研讨会，那么将会通过一个链接来加入 OpenFaaS Slack 社区。使用研讨会指定的渠道讨论意见、问题和建议。

## 选择你的路径

在实验室 1 中，您将选择您的路径，然后在整个实验室中寻找您的容器编排器所需的任何特殊命令。

### Kubernetes

您还可以使用 OpenFaaS 了解 Kubernetes 上的 Serverless。

OpenFaaS 社区的建议是，您可以在生产中运行 Kubernetes，但是所有知识都是相通的，并且功能不必重建。

## [实验 1 - OpenFaaS 准备](./lab1.md)

- 安装先决条件
- 使用 Kubernetes 建立单节点集群
- Docker Hub 帐户
- OpenFaaS CLI
- 部署 OpenFaaS

## [实验 2 - 测试](./lab2.md)

- 使用 UI 门户
- 通过函数商店部署
- 学习 CLI
- 通过 Prometheus 监控

## [实验 3 - 函数介绍](./lab3.md)

- 生成新函数的脚手架
- 构建 astronaut-finder 函数
- 通过`pip`添加依赖
- 疑难解答：找到容器的日志
- 疑难解答：通过`write_debug`查看输出
- 使用自定义和第三方语言模板
- 在模板商店发现社区模板

## [实验 4 - 深入了解函数](./lab4.md)

- [通过环境变量注入](lab4.md#inject-configuration-through-environmental-variables)
  - 部署时使用 yaml
  - 动态使用 http 上下文 - querystring / headers 等
- 安全：只读文件系统
- [利用日志](lab4.md#making-use-of-logging)
- [创建工作流](lab4.md#create-workflows)
  - 客户端使用调用链
  - 从一个函数中调用另一个函数

## [实验 5 - 构建一个 GitHub bot](./lab5.md)

> 构建 `issue-bot` - GitHub Issues 自动响应程序

- 获取 GitHub 账户
- 通过 ngrok 内网穿透
- 为`issue-bot`创建 gou
- 从 GitHub 接收 webhooks
- 部署情感分析函数
- 通过 GitHub API 应用标签
- 完成函数

## [实验 6 - 函数中的 HTML](./lab6.md)

- 从一个函数中生成且返回基本的 HTML
- 从磁盘中读取并返回一个静态的 HTML 文件
- 与其他函数协作

## [实验 7 - 异步函数](./lab7.md)

- 同步调用 vs 异步调用
- 查看 queue-worker 的日志
- 使用 X-Callback-Url

## [实验 8 - 高级特性 - 超时](./lab8.md)

- 通过 read_timeout 调节超时时间
- 适应长时间运行函数

## [实验 9 - 高级特性 - 自动扩容](./lab9.md)

- 实操自动扩容
  - 最小和最大副本的一些见解
  - 访问本地的 Prometheus
  - 执行 Prometheus 查询
  - 使用 curl 调用函数
  - 观察自动扩容

## [实验 10 - 高级特性 - Secrets](./lab10.md)

- issue-bot 使用秘钥
  - 通过 faas-cli 创建 Kubernetes 秘钥
  - 在函数中访问秘钥

## [实验 11 - 高级特性 - 信任 HMAC](./lab11.md)

- 对使用 HMAC 的函数信任

您可以从第一个实验[实验 1](lab1.md)开始

## 停止/清理

你可以在[这里](https://docs.openfaas.com/deployment/troubleshooting/#uninstall-openfaas)找到如何停止和删除 OpenFaaS

## 下一步

如果你在一个讲师指导的研讨会，并且已经完成了实验，你可能想回到实验，edit/alter 代码，或者进行一些你自己的实验。

以下是后续任务/主题的一些想法：

### OpenFaaS Cloud

尝试 OpenFaaS 的多用户托管体验——要么在社区集群上，要么托管自己的 OpenFaaS 云。

- [Docs: OpenFaaS Cloud](https://docs.openfaas.com/openfaas-cloud/intro/)

### TLS

- [在 Kubernetes 入口在网关上启用 HTTPS](https://docs.openfaas.com/reference/ssl/kubernetes-with-cert-manager/)

### CI/CD

设置 Jenkins，Google Cloud Build 或 GitLab，并使用 OpenFaaS CLI 构建和部署您自己的函数：

- [CI/CD 简介](https://docs.openfaas.com/reference/cicd/intro/)

### 存储 / 数据库

- [尝试使用 Minio 开源对象存储](https://blog.alexellis.io/openfaas-storage-for-your-functions/)

- [使用 Mongo 尝试 OpenFaaS 存储数据](https://blog.alexellis.io/serverless-databases-with-openfaas-and-mongo/)

### 仪表盘 / 监控

- [探索在 Prometheus 可用的指标](https://docs.openfaas.com/architecture/metrics/#monitoring-functions)

### 额外的博客文章和教程

- [OpenFaaS 博客上的教程](https://www.openfaas.com/blog/)

- [社区博客帖子](https://github.com/openfaas/faas/blob/master/community.md)

### 附录

[附录](./appendix.md) 中包含一些附加内容。

## 致谢

感谢 @iyovcheva, @BurtonR, @johnmccabe, @laurentgrangeau, @stefanprodan, @kenfdev, @templum & @rgee0 对实验的贡献、测试和翻译。
