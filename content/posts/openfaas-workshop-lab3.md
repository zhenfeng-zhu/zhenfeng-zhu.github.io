---
date: "2018-06-28 17:29:56"
title: openfaas-workshop-lab3
---

# Lab3 - 函数的介绍

<img src="https://github.com/openfaas/media/raw/master/OpenFaaS_Magnet_3_1_png.png" width="500px"></img>

在开始本实验之前，创建一个新文件夹：

```
$ mkdir -p lab3 \
   && cd lab3
```

## 创建一个新函数

创建函数有两种方式：

- 使用内置的或社区提供的代码末班创建一个函数脚手架（默认）

* 将现有的二进制文件作为函数（高级）

### 生成一个新函数

在使用模板创建一个新函数之前，首先确认你已经从 Github 上拉下来[模板文件](https://github.com/openfaas/templates)：

```
$ faas-cli template pull

Fetch templates from repository: https://github.com/openfaas/templates.git
 Attempting to expand templates from https://github.com/openfaas/templates.git
 Fetched 11 template(s) : [csharp dockerfile go go-armhf node node-arm64 node-armhf python python-armhf python3 ruby]
```

之后找到可用的语言：

```
$ faas-cli new --list
Languages available as templates:
- csharp
- dockerfile
- go
- go-armhf
- node
- node-arm64
- node-armhf
- python
- python-armhf
- python3
- ruby

Or alternatively create a folder containing a Dockerfile, then pick
the "Dockerfile" lang type in your YAML file.
```

此时你可以创建 Python, Python 3, Ruby, Go, Node, CSharp 等函数。

- 关于我们例子的说明

在这个 workshop 中的所有例子都已经被 OpenFaas 社区使用 Python3 测试通过，应该也兼容 Python2.7。

如果你倾向于 Python2.7 而不是 Python3，使用`faas-cli new --lang python`命令替换`faas-cli new --lang python3`。

### Python 里的 Hello World

我们将会用 Python 创建一个 hello-world 函数，然后也会学习附加的依赖项。

- 函数脚手架

```
$ faas-cli new --lang python3 hello-openfaas --prefix="<your-docker-username-here>"
```

—prefix 参数将会把`hello-openfaas.yml`文件里`image:` 变量中的 prefix 更新为你的 docker hub 的账户。OpenFaas 的镜像`image: functions/hello-openfaas`里的参数将会是`--prefix="functions"`。

如果你不想指定 prefix，那么在创建函数之后可以通过修改 YAML 文件。

这将会在文件夹中创建 3 个文件：

```
./hello-openfaas.yml
./hello-openfaas
./hello-openfaas/handler.py
./hello-openfaas/requirements.txt
```

YAML(.yml)文件用于配置 CLI 的构建，推送和部署。

> 说明：无论何时你需要在 Kubernetes 或远程 OpenFaaS 实例部署一个函数时，都必须在你构建之后。
>
> 在这个例子中，你也可以通过设置一个环境变量：`export OPENFAAS_URL=127.0.0.1:31112`覆盖掉默认的网关 URL：127.0.0.1：8080，

如下是 YAML 文件的内容：

```yaml
provider:
  name: faas
  gateway: http://127.0.0.1:8080

functions:
  hello-openfaas:
    lang: python3
    handler: ./hello-openfaas
    image: hello-openfaas
```

- 函数的名字是在`functions`的下面，如：`hello-openfaas`

* 语言是放在 lang 之下

* handler 是用于构建的文件夹，一定得是文件夹，不能是文件

* docker 的镜像名是放在 image 之下

请记住，网关的 URL 是可以在 YAML 文件中 provider 下的 gateway 字段覆写，或者是在 CLI 中使用—gateway 参数或是环境变量的的 OPENFAAS_URL。

如下是`handler.py`文件的内容：

```
def handle(req):
    """handle a request to the function
    Args:
        req (str): request body
    """

    return req
```

这个函数返回输入的值，和 echo 函数很像。

将返回值修改为`Hello OpenFaaS`，如：

```
    return "Hello OpenFaaS"
```

任何返回给标准输出的值随后都会被返回给调用的程序。或者说是一个 print()语句和显示的结果和调用的程序流程相似。

这是本地的开发流程：

```
$ faas-cli build -f ./hello-openfaas.yml
$ faas-cli push -f ./hello-openfaas.yml
$ faas-cli deploy -f ./hello-openfaas.yml
```

接下来通过 UI，CLI，curk 或者其他的程序去调用函数。

这些函数都会被分配一个路由，比如：

```
http://127.0.0.1:8080/function/<function_name>
http://127.0.0.1:8080/function/figlet
http://127.0.0.1:8080/function/hello-openfaas
```

> 专家提醒：如果你重命名了 YAML 文件为 stack.yml，不再需要是用-f 参数。

函数只能被 get 或 post 方法调用。

- 调用你的函数

使用 faas-cli invoke 测试函数，更多的命令请参考 faas-cli invoke --help

### 例子：发现宇航员

我们将会创建一个叫 astronaut-finder 函数，这个函数会从国际空间站上拉取一个随机的宇航员的名字。

```
$ faas-cli new --lang python3 astronaut-finder --prefix="<your-docker-username-here>"
```

这会为我们创建三个文件：

```
./astronaut-finder/handler.py
```

该函数的 handler —— 你会得到一个原始的请求 req 对象，而且会在控制台上打印结果。

```
./astronaut-finder/requirements.txt
```

在这个文件中列出你想要安装的 pip 模块，比如 requests 或 urllib

```
./astronaut-finder.yml
```

这个文件用于管理函数——在里面有函数名，docker 镜像和其他自定义的字段。

- Edit `./astronaut-finder/requirements.txt`
- 修改`./astronaut-finder/requirements.txt`

```
requests
```

这告诉函数，他需要使用一个名叫 requests 的第三方包用于使用 http 请求网站。

- 写函数代码：

我们将会从 http://api.open-notify.org/astros.json拉取数据

这是返回结果的例子：

```on
{
  "number": 6,
  "people": [
    { "craft": "ISS", "name": "Alexander Misurkin" },
    { "craft": "ISS", "name": "Mark Vande Hei" },
    { "craft": "ISS", "name": "Joe Acaba" },
    { "craft": "ISS", "name": "Anton Shkaplerov" },
    { "craft": "ISS", "name": "Scott Tingle" },
    { "craft": "ISS", "name": "Norishige Kanai" }
  ],
  "message": "success"
}
```

更新 handler.py：

```
import requests
import random

def handle(req):
    r = requests.get("http://api.open-notify.org/astros.json")
    result = r.json()
    index = random.randint(0, len(result["people"])-1)
    name = result["people"][index]["name"]

    return "%s is in space" % (name)
```

> 备注：在这个例子中我们不需要使用 req 参数，但是必须确保他在函数头里。

现在构建函数：

```
$ faas-cli build -f ./astronaut-finder.yml
```

> 提示：试着将 astronaut-finder.yml 重命名为 stack.yml，然后就可以使用 faas-cli build。
>
> stack.yml 是 CLI 的默认命名。

部署函数：

```
$ faas-cli deploy -f ./astronaut-finder.yml
```

调用函数

```
$ echo | faas-cli invoke astronaut-finder
Anton Shkaplerov is in space

$ echo | faas-cli invoke astronaut-finder
Joe Acaba is in space
```

## 故障排除：找到容器的日志

通过容器的日志你可以找到函数每次调用的高级别的信息：

```
$ docker service logs -f astronaut-finder
astronaut-finder.1.1e1ujtsijf6b@nuc    | 2018/02/21 14:53:25 Forking fprocess.
astronaut-finder.1.1e1ujtsijf6b@nuc    | 2018/02/21 14:53:26 Wrote 18 Bytes - Duration: 0.063269 seconds
```

## 故障排除：使用 write_debug 详细输出

让我们打开函数的详细输出。为了不让函数的日志和数据泛滥，这个功能默认是关闭的，在日志里有很多没有意义的二进制数据时，这点儿尤为重要。

这是标准的 YAML 配置：

```yaml
provider:
  name: faas
  gateway: http://127.0.0.1:8080

functions:
  astronaut-finder:
    lang: python3
    handler: ./astronaut-finder
    image: astronaut-finder
```

编辑 YAML 文件，添加 environment。

```yaml
  astronaut-finder:
    lang: python3
    handler: ./astronaut-finder
    image: astronaut-finder
    environment:
      write_debug: true
```

现在再次使用`faas-cli deploy -f ./astronaut-finder.yml`部署。

调用函数，然后观察函数的返回：

```
$ docker service logs -f astronaut-finder
astronaut-finder.1.1e1ujtsijf6b@nuc    | 2018/02/21 14:53:25 Forking fprocess.
astronaut-finder.1.szobw9pt3m60@nuc    | 2018/02/26 14:49:57 Query  
astronaut-finder.1.szobw9pt3m60@nuc    | 2018/02/26 14:49:57 Path  /function/hello-openfaas
astronaut-finder.1.1e1ujtsijf6b@nuc    | 2018/02/21 14:53:26 Hello World
astronaut-finder.1.1e1ujtsijf6b@nuc    | 2018/02/21 14:53:26 Duration: 0.063269 seconds
```

### 管理多个函数

CLI 的 YAML 文件也可以把函数打组为一个 stack 里，当在几个相关联的函数工作时是很有用的。

看一下如何生成两个函数：

```
$ faas-cli new --lang python3 first
```

第二个函数使用--apend 标志：

```
$ faas-cli new --lang python3 second --append=./first.yml
```

为了方便我们把 first.yml 重命名为 example.yml。

```
$ mv first.yml example.yml
```

现在看一下文件：

```
provider:
  name: faas
  gateway: http://127.0.0.1:8080

functions:
  first:
    lang: python3
    handler: ./first
    image: first
  second:
    lang: python3
    handler: ./second
    image: second
```

当一个函数栈工作时，这里有几个有用的标志。

- 并行构建

```sh
$ faas-cli build -f ./example.yml --parallel=2
```

- 只构建或者推送一个函数

```sh
$ faas-cli build -f ./example.yml --filter=second
```

更多信息请参考`faas-cli build --help` 和 `faas-cli push --help` 。

> 专家提醒：如果你不想传-f 参数，faas-cli 将会自动寻找 stack.yml 文件。

你也可以在使用`faas-cli -f https://....`将函数栈部署在 https 上。

### 使用自定义模板

如果你有自己的语言模板或者是从社区中找到的模板，比如 PHP。你可以使用下面的命令添加进来：

```
$ faas-cli template pull https://github.com/itscaro/openfaas-template-php

...

$ faas-cli new --list | grep php
- php
- php5
```

社区模板列表维护在[OpenFaaS CLI README](https://github.com/openfaas/faas-cli)页面。

继续可选的练习或者进入 [Lab 4](lab4.md)。

### 自定义二进制文件为函数（可选）

自定义二进制文件或者容器也可以被作为函数，但是大部分时间里使用语言模板文件应该可以涵盖大多数情况。

通过使用 dockerfile 语言，可以用自定义二进制或者 Dockerfile 创建一个新函数函数：

```
$ faas-cli new --lang dockerfile sorter --prefix="<your-docker-username-here>"
```

你将会看到 sorter 文件夹和 sorter.yml 文件被创建。

编辑 sorter/Docerfile，更新设置 fprocess 的那一行。让我们改变他为内置 bash 的 sort 命令。我们可以用这个命令对字符串按照字母数字顺序排序。

```
ENV fprocess="sort"
```

构建，推送和部署函数：

```
$ faas-cli build -f sorter.yml \
  && faas-cli push -f sorter.yml \
  && faas-cli deploy -f sorter.yml
```

从 UI 或者 CLI 中调用函数：

```
$ echo -n '
elephant
zebra
horse
ardvark
monkey'| faas-cli invoke sorter

ardvark
elephant
horse
monkey
zebra
```

在这个例子中我们使用内置于 BusyBox 的 sort 命令。也有其他有用的命令，比如 sha512sum，甚至可以是 bash 或者 shell 脚本，而且并不限制于这些命令。任何二进制或者存在的容器都可以通过添加到 OpenFaaS 的函数 watchdog 中来将它 serverless 化。

> 提示：你知道 OpenFaaS 也支持 WIndows 的二进制文件吗？比如 C#，VB 或者 PowerShell？

现在进入 [Lab 4](lab4.md)
