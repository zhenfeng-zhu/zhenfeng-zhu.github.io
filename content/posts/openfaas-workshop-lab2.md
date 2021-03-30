---
title: "Openfaas Workshop Lab2"
date: 2021-03-30T09:57:01+08:00
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

# 实验 2 - 测试

<img src="https://github.com/openfaas/media/raw/master/OpenFaaS_Magnet_3_1_png.png" width="500px"></img>

在开始实验之前，先创建一个文件夹：

```sh
$ mkdir -p lab2 \
   && cd lab2
```

## 使用 UI 门户

您现在可以测试 OpenFaaS UI：

如果你已经设置了 `$OPENFAAS_URL` ，请获取 URL，并点开:

```sh
echo $OPENFAAS_URL
http://127.0.0.1:31112
```

如果还没有设置 `$OPENFAAS_URL` ，那么默认的值通常是： [http://127.0.0.1:8080](http://127.0.0.1:8080).

我们可以部署一些示例函数，然后使用它们进行测试：

```sh
$ faas-cli deploy -f https://raw.githubusercontent.com/openfaas/faas/master/stack.yml
```

![](./screenshot/markdown_portal.png)

您可以在用户界面中试用它们，例如 Markdown 函数，它将 Markdown 代码转换成超文本标记语言。

在 _Request_ 字段中输入如下的内容：

```sh
## The **OpenFaaS** _workshop_
```

现在点击 _Invoke_ ，然后就可以看到响应出现在屏幕的下半部分。

I.e.

```sh
<h2>The <strong>OpenFaaS</strong> <em>workshop</em></h2>
```

您将会看到如下的字段：

- Status - 函数是否准备好运行。在状态显示就绪之前，您将无法从用户界面调用该函数。
- Replicas - 在集群中运行的函数的副本数量。
- Image - 发布到 Docker Hub 或 Docker 存储库的 Docker 镜像名称和版本。
- Invocation count - 这显示了函数被调用的次数，并且每 5 秒更新一次。

多次点击 _Invoke_ ，就可以看到 _Invocation count_ 在递增。

## 通过函数商店部署

你也可以从 OpenFaaS 商店部署一个函数。这个商店是社区策划的函数集合。

- 点击 _Deploy New Function_
- 点击 _From Store_
- 点击 _Figlet_ 或者在搜索栏里输入 _figlet_ ，然后点击 _Deploy_

Figlet 函数将会出现在左侧函数列表中，耐心等几分钟，它需要从 docker hub 下载。 下载完成之后，输入一些文本，然后点击 invoke。

你会看的一个 ASCII logo：

```sh
 _  ___   ___ _  __
/ |/ _ \ / _ (_)/ /
| | | | | | | |/ /
| | |_| | |_| / /_
|_|\___/ \___/_/(_)
```

## 学习 CLI

你现在可以测试 CLI，但是首先看下网关的 URL：

如果 gateway 没有部署在http://127.0.0.1:8080，你需要按如下的几种方法指定一下：

1. 设置环境变量`OPENFAAS_URL`，`faas-cli`将会在 shell 会话中使用这个。比如：`export OPENFAAS_URL=http://openfaas.endpoint.com:8080`。这个已经在[实验 1](./lab1.md)中设置好了。
2. 使用`-g` 或 `--gateway`指定正确的端点：`faas deploy --gateway http://openfaas.endpoint.com:8080`。
3. 在 YAML 文件中，在`provider:`下修改一下`gateway:`。

### 列出已经部署的函数

这个会显示有多少函数，多少个副本以及调用次数。

```sh
$ faas-cli list
```

现在尝试使用 verbose 命令

```sh
$ faas-cli list --verbose
```

或

```sh
$ faas-cli list -v
```

现在可以在函数旁边看的 Docker 镜像。

### 调用函数

选择`faas-cli list`中看的的函数，比如 `markdown`:

```sh
$ faas-cli invoke markdown
```

然后你会被要求输入一些文本，然后按 Control + D 结束。

或者也可以使用`echo` 或 `uname -a`作为调用的输入。

```sh
$ echo Hi | faas-cli invoke markdown

$ uname -a | faas-cli invoke markdown
```

你甚至可以使用一个 HTML 文件：

```sh
$ git clone https://github.com/openfaas/workshop \
   && cd workshop

$ cat lab2.md | faas-cli invoke markdown
```

## 监控面板

OpenFaas 使用 Prometheus 监控函数的指标。这些指标可以通过[Grafana](https://grafana.com)变成一个可视化的仪表盘。

在 OpenFaaS 的 Kubernetes 命名空间下运行 Grafana：

```sh
kubectl -n openfaas run \
--image=stefanprodan/faas-grafana:4.6.3 \
--port=3000 \
grafana
```

通过 NodePort 暴露：

```sh
kubectl -n openfaas expose pod grafana \
--type=NodePort \
--name=grafana
```

找到 Grafana 的 node port 地址

```sh
$ GRAFANA_PORT=$(kubectl -n openfaas get svc grafana -o jsonpath="{.spec.ports[0].nodePort}")
$ GRAFANA_URL=http://IP_ADDRESS:$GRAFANA_PORT/dashboard/db/openfaas
```

其中 `IP_ADDRESS` 就是 Kubernetes 的对应 IP。

或者你也可以用端口转发命令，这样就可以在`http://127.0.0.1:3000`Grafana。

```sh
$ kubectl port-forward pod/grafana 3000:3000 -n openfaas
```

如果使用 Kubernetes 1.17 或更高版本，请使用`deploy/grafana` ，而不是用 `pod/` 。

创建服务之后，在浏览器中打开 Grafana，然后输入用户名`admin` 和密码 `admin`登录，并以 `$GRAFANA_URL`找到之前已经做好的 OpenFaaS 仪表盘。

<a href="https://camo.githubusercontent.com/24915ac87ecf8a31285f273846e7a5ffe82eeceb/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4339636145364358554141585f36342e6a70673a6c61726765"><img src="https://camo.githubusercontent.com/24915ac87ecf8a31285f273846e7a5ffe82eeceb/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4339636145364358554141585f36342e6a70673a6c61726765" width="600px" /></a>

_Pictured: example of an OpenFaaS dashboard with Grafana_

现在进入 [实验 3](./lab3.md)
