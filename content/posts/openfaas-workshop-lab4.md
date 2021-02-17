---
date: "2018-07-02 09:32:59"
title: openfaas-workshop-lab4
---

# Lab 4 - 深入函数

<img src="https://github.com/openfaas/media/raw/master/OpenFaaS_Magnet_3_1_png.png" width="500px"></img>

在开始本实验之前，创建一个新的文件夹，把 lab3 的文件拷贝到 lab4 里：

```
$ cp -r lab3 lab4 \
   && cd lab4
```

## 通过环境变量注入配置

It is useful to be able to control how a function behaves at runtime, we can do that in at least two ways:

控制函数在运行时的行为很有用，我们至少可以通过两种方式来实现：

### 在部署时

- 在部署时设置环境变量

我们在 Lab3 时用了 write_debug 来做——你也可以在这里设置你想要的任何自定义的环境变量。例如：如果你想为 hello world 函数配置一种语言，可以引入一个 spoken_language 变量。

### 使用 HTTP 上下文——querystring / headers

- 使用 querystring 和 HTTP headers

另一个更为动态的选项是可以在每个请求级别上进行修改，即使用 querystrings 和 HTTP headers，这两者都可以通过 faas-cli 或者 curl 传递。

这些 headers 通过环境变量暴露出来，因此你可以很容易的在函数中使用。所有的 header 都以 Http*为前缀，并且所有的`-`都被替换为了` * `下划线

让我们用一个 querystring 和一个列出所有环境变量的函数来尝试一下：

- 部署一个函数，此函数使用内置的 BusyBox 命令来打印环境变量

```
$ faas-cli deploy --name env --fprocess="env" --image="functions/alpine:latest" --network=func_functions
```

- 用一个 querystring 去调用函数：

```
$ echo "" | faas-cli invoke env --query workshop=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=05e8db360c5a
fprocess=env
HOME=/root
Http_Connection=close
Http_Content_Type=text/plain
Http_X_Call_Id=cdbed396-a20a-43fe-9123-1d5a122c976d
Http_X_Forwarded_For=10.255.0.2
Http_X_Start_Time=1519729562486546741
Http_User_Agent=Go-http-client/1.1
Http_Accept_Encoding=gzip
Http_Method=POST
Http_ContentLength=-1
Http_Path=/function/env
...
Http_Query=workshop=1
...
```

在 python 代码中，你应该输入 os.getenv("Http_Query")

- 现在用一个 header 调用函数：

```
$ echo "" | curl http://127.0.0.1:8080/function/env --header "X-Output-Mode: json"
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=05e8db360c5a
fprocess=env
HOME=/root
Http_X_Call_Id=8e597bcf-614f-4ca5-8f2e-f345d660db5e
Http_X_Forwarded_For=10.255.0.2
Http_X_Start_Time=1519729577415481886
Http_Accept=*/*
Http_Accept_Encoding=gzip
Http_Connection=close
Http_User_Agent=curl/7.55.1
Http_Method=GET
Http_ContentLength=0
Http_Path=/function/env
...
Http_X_Output_Mode=json
...
```

在 python 代码中，你应该输入 os.getenv("Http_X_Outout_Mode")。

你可以看到，当 Http_Method 是 POST 方法时， 所有的其他 HTTP 上下文也被提供了出来，比如 Content-Length，`User_Agent`， Cookies 和其他 HTTP 请求中应有的参数。

## 利用日志

OpenFaaS 的 watchdog 是通过标准 IO 流的 stdin 和 stdout 处理 HTTP 请求和读取 HTTP 响应。这意味着作为一个函数的进程不需要知道任何 web 和 HTTP 的信息。

一个有趣的情况是当函数以非零的退出且 stderr 不为空。默认情况下，函数的 stdout/stderr 被合并了，且 stderr 不会被打印到日志里。

让我们使用 Lab3 的函数 hello-openfaas 验证一下。

修改 handler.py 为：

```
import sys
import json

def handle(req):

    sys.stderr.write("This should be an error message.\n")
    return json.dumps({"Hello": "OpenFaaS"})
```

构建并部署：

```sh
$ faas-cli build -f hello-openfaas.yml \
  && faas-cli push -f hello-openfaas.yml \
  && faas-cli deploy -f hello-openfaas.yml
```

然后调用函数：

```sh
$ echo | faas-cli invoke hello-openfaas
```

你应该可以看到合并之后的输出：

```
This should be an error message.
{"Hello": "OpenFaaS"}
```

> 说明：如果使用`docker service logs hello-openfaas`检查容器的日志，你应该看不到 stderr 输出。

在这个例子中，我们需要函数返回一个可被解析的合法 JSON。不幸的是日志信息使得输出不可用，所以我们需要重定向 stderr 的信息到容器的日志中。OpenFaaS 提供了一个解决方案即：只返回 stdout，因此你可以打印日志的错误信息并且保证函数的返回是清晰的。

为了达到目的，你应该使用`combine_output`参数：

让我们尝试一下。打开`hello-openfaas.yaml`文件，然后添加下面这几行：

```yaml
    environment:
      combine_output: false
```

推送部署并调用函数。

输出应该是：

```
{"Hello": "OpenFaaS"}
```

检查容易的 stderr 日志。你应该可以看到如下类似的消息：

```
hello-openfaas.1.2xtrr2ckkkth@linuxkit-025000000001    | 2018/04/03 08:35:24 stderr: This should be an error message.
```

## 创建工作流程

在某些情况下，将一个函数的输出作为另一个函数的输入是很有用的。通过客户端或者 API 网关都可以实现。

### 在客户端的函数链

你可以使用 curl，faas-cli 或者你自己的其他代码将一个函数的结果传给另一个函数。这是一个例子：

优点：

- requires no code - can be done with CLI programs
- 无需代码——可以使用 CLI 程序完成
- fast for development and testing
- 快速开发测试
- easy to model in code
- 易于在代码中建模

缺点：

- additional latency - each function goes back to the server
- 额外的延迟——每个函数都要返回到服务器
- chatty (more messages)
- 繁琐（更多的消息）

例子：

- 从函数商店中部署一个 Nodeinfo 函数

- 把 NodeInfo 的输出传给 Markdown。

```sh
$ echo -n "" | faas-cli invoke nodeinfo | faas-cli invoke func_markdown
<p>Hostname: 64767782518c</p>

<p>Platform: linux
Arch: x64
CPU count: 4
Uptime: 1121466</p>
```

你现在将会看到 NodeInfo 函数的输出被 HTML 标签装饰了。

客户端函数链的另一个例子是生成一个图片，然后把它传给另一个加水印的函数。

### 从一个函数中调用另一个函数

最简单的函数间调用是通过 OpenFaaS 的 API 网关做一个 HTTP 调用。这个调用是不用知道外部域名或 IP 地址，他可以简单通过 DNS 条目把 API 网关作为一个 gateway。

在一个函数中访问 API 网关服务时，最好使用环境变量来配置主机名，这是很重要的，原因有两个——名字可能会改变，并且在 Kubernetes 中有时需要后缀

优点：

- 函数可以直接使用对方

- 因为是在同一网络中互相访问，延迟较低

缺点：

- 需要一个 HTTP 请求的库

例子：

在 Lab3 中，我们介绍了 requests 包并且使用它去调用 ISS 获取一个宇航员的名字。我们可以使用相同的技术调用部署在 OpenFaaS 中的其他函数。

- 打开函数商店然后部署*Sentiment Analysis*函数。

Sentiment Analysis 函数将会告诉你任意一个句子的主动性和倾向（积极性评级）。这个函数的结果是一个格式化的 JSON，如下面例子所示：

```sh
$ echo -n "California is great, it's always sunny there." | faas-cli invoke sentimentanalysis
{"polarity": 0.8, "sentence_count": 1, "subjectivity": 0.75}
```

这个结果显示我们的测试句子非常有主动性（75%）且很积极（80%）。这两个字段的值在-1.00 和 1.00 之间。

下面的代码被用于在任何一个函数中调用 *Sentiment Analysis*函数：

```
    test_sentence = "California is great, it's always sunny there."
    r = requests.get("http://gateway:8080/function/sentimentanalysis", text= test_sentence)
```

或者通过一个环境变量：

```
    gateway_hostname = os.getenv("gateway_hostname", "gateway") # uses a default of "gateway" for when "gateway_hostname" is not set
    test_sentence = "California is great, it's always sunny there."
    r = requests.get("http://" + gateway_hostname + ":8080/function/sentimentanalysis", text= test_sentence)
```

因为结果总是 JSON 格式的，所示我们可以使用.json()转化响应。

```
    result = r.json()
    if result["polarity"] > 0.45:
       return "That was probably positive"
    else:
        return "That was neutral or negative"
```

现在创建一个 Python 函数，然后合并在一起：

```
import os
import requests
import sys

def handle(req):
    """handle a request to the function
    Args:
        req (str): request body
    """

    gateway_hostname = os.getenv("gateway_hostname", "gateway") # uses a default of "gateway" for when "gateway_hostname" is not set

    test_sentence = req

    r = requests.get("http://" + gateway_hostname + ":8080/function/sentimentanalysis", data= test_sentence)

    if r.status_code != 200:
        sys.exit("Error with sentimentanalysis, expected: %d, got: %d\n" % (200, r.status_code))

    result = r.json()
    if result["polarity"] > 0.45:
        return "That was probably positive"
    else:
        return "That was neutral or negative"
```

- 记得把 requests 添加到 requirements.txt 文件里

说明：你不需要修改 SentimentAnalysis 函数的源码，我们已经把它部署了，可以通过 API 网关获取。

现在进入 Lab 5。
