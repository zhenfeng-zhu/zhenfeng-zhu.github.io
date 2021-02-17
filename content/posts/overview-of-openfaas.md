---
date: "2018-07-26 17:41:33"
title: overview-of-openfaas
---

## OpenFaaS概览


> 无服务器函数变得简单。

![](https://ws1.sinaimg.cn/large/b831e4c7gy1ftucj00kylj20xc0j511g.jpg)

### 函数监视器

* 你可以通过添加*函数监视器* (一个小型的Golang HTTP服务)把任何一个Docker镜像变成无服务器函数。
* *函数监视器*是允许HTTP请求通过STDIN转发到目标进程的入口点。响应会从你应用写入STDOUT返回给调用者。

### API网关/UI门户

* API网关为你的函数提供外部路由，并通过Prometheus收集云原生指标。
* 你的API网关将会根据需求更改Docker Swarm 或 Kubernetes API中的服务副本数来实现伸缩性。
* UI是允许你在浏览器中调用函数或者根据需要创建新的函数。

> API网关是一个RESTful形式的微服务，你可以在这里查看[Swagger文档](https://github.com/openfaas/faas/tree/master/api-docs)。

### 命令行

Docker中的任何容器或者进程都可以是FaaS中的一个无服务器函数。使用[FaaS CLI](http://github.com/openfaas/faas-cli) ，你可以快速的部署函数。

可以从Node.js, Python, [Go](https://blog.alexellis.io/serverless-golang-with-openfaas/) 或者更多的语言模板中创建新的函数。如果你无法找到一个合适的模板，甚至可以使用一个Dockerfile。

> CLI实际上是API网关的一个RESTful客户端。

在配置好OpenFaaS之后，你可以在这里开始学习CLI[开始学习CLI](https://blog.alexellis.io/quickstart-openfaas-cli/)

### 函数示例

你可以通过 使用FaaS-CLI和其内置的模板创建新函数，也可以在Docker中使用Windows或Linux的二进制文件。

* Python示例：

```
import requests

def handle(req):
    r =  requests.get(req, timeout = 1)
    print(req +" => " + str(r.status_code))
```
*handler.py*

* Node.js示例：

```
"use strict"

module.exports = (callback, context) => {
    callback(null, {"message": "You said: " + context})
}
```
*handler.js*

在Github仓库中提供了一系列编程语言的其他[示例函数](https://github.com/openfaas/faas/tree/master/sample-functions) 。
