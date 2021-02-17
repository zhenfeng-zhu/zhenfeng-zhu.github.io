---
date: "2018-06-27 18:10:47"
title: openfaas-workshop-lab2
---

# Lab2 测试一下

在开始这个实验之前，首先创建一个新的文件夹：

```
$ mkdir -p lab2 \
   && cd lab2
```

## 使用UI界面

你现在可以打开[http://127.0.0.1:8080](http://127.0.0.1:8080/)测试OpenFaaS的UI。如果是在一台LInux虚拟机中部署，将127.0.0.1替换为该虚拟机的ip地址。

> 备注：我们使用的是127.0.0.1而不是localhost，因为在一些Linux发行版中IPv4/IPv6出现不兼容。

我们可以部署一些样例代码来进行测试：

```
faas-cli deploy -f https://raw.githubusercontent.com/openfaas/faas/master/stack.yml
```

![](https://github.com/openfaas/workshop/raw/master/screenshot/markdown_portal.png)

你可以在UI里面选择Markdown函数进行测试，这个函数将markdown转换为html。

在Request一栏中输入：

```
## The **OpenFaaS** _workshop_
```

点击INVOKE按钮，然后在屏幕下方观察响应。

即：

```
<h2>The <strong>OpenFaaS</strong> <em>workshop</em></h2>
```

你同时也会发现在其他栏中的值：

- Status - 函数是否可以运行的状态。如果在UI中的状态不可用，你将不能invoke函数。
- Replicas - 运行在swarm集群中的函数副本。
- Image - 发布在docker hub或其他docker仓库中的镜像名字。
- Invocation count - 5秒更新一次，显示该函数已经被触发的次数。

点击INVOKE按钮几次，可以看到Invocation count在增加。

## 通过函数商店进行部署

你也可以从OpenFaas商店中部署一个函数。这个商店是是社区整理的一些免费函数集合。

- 点击*Deploy New Function*
- 点击*From Store*
- 点击*Figlet*或者从搜索栏中进入*figlet*，然后点击*Deploy*

Figlet函数将会在左侧的函数列表中。稍等几分钟，等待函数从Docker Hub中下载。然后就像Markdown函数一样，输入一些文字，点击INVOKE。

一个ASCII的logo将会生成：

```
 _  ___   ___ _  __
/ |/ _ \ / _ (_)/ /
| | | | | | | |/ / 
| | |_| | |_| / /_ 
|_|\___/ \___/_/(_)
```

## 学习CLI

你现在可以测试CLI，但是首先先做一些网关URL的说明：

如果你的网关不是部署在[http://127.0.0.1:8080](http://127.0.0.1:8080/)，那么你需要指定替代的地址。这里有几种方法：

1. 设置OPENFAAS_URL的环境变量，faas-cli将会在当前shell会话中指向此endpoint。比如export OPENFAAS_URL=http://openfaas.endpoint.com:8080
2. 使用-g或--gateway指向正确的endpoint：faas deploy --gateway http://openfaas.endpoint.com:8080
3. 在部署的YAML文件中，改变provider下面的gateway的值。

### 列出已经部署的函数

这个命令将会列出已经部署的函数，副本数和调用数。

```
$ faas-cli list
```

你应该可以看到markdown函数和figlet函数以及他们被调用的次数。

现在试一下verbose参数

```
$ faas-cli list --verbose
```

或

```
$ faas-cli list -v
```

你现在可以看到函数的docker镜像。

### 调用一个函数

选择一个faas-cli list命令列出的函数，比如markdown：

```
$ faas-cli invoke markdown
```

现在输入一些文字，按ctrl+d退出。

也可以使用其他命令，比如echo或者uname -a作为输入，和invoke命令形成管道：

```
$ echo Hi | faas-cli invoke markdown

$ uname -a | faas-cli invoke markdown
```

接下来你甚至可以将一个markdown文件转换为HTML文件：

```
$ git clone https://github.com/openfaas/workshop \
   && cd workshop

$ cat lab2.md | faas-cli invoke markdown
```

### 监控仪表盘

OpenFaas使用Prometheus自动监控函数的指标。使用一些免费且开源的软件如 [Grafana](https://grafana.com/)，将这些指标转换为可视化的仪表盘。

部署OpenFaaS的Grafana：

```
$ docker service create -d \
--name=grafana \
--publish=3000:3000 \
--network=func_functions \
stefanprodan/faas-grafana:4.6.3
```

服务创建之后，在浏览器中打开Grafana，使用admin/admin登录，然后进入预制的OpenFaaS的仪表盘：

<http://127.0.0.1:3000/dashboard/db/openfaas>

![](https://camo.githubusercontent.com/24915ac87ecf8a31285f273846e7a5ffe82eeceb/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4339636145364358554141585f36342e6a70673a6c61726765)

现在进入[Lab 3](https://github.com/openfaas/workshop/blob/master/lab3.md)

未完待续...