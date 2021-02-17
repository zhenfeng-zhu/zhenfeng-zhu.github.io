---
date: "2018-06-24 15:43:46"
title: openfaas
---

对于mac环境来讲，首先安装新版docker:

```
brew cask install docker
```

然后启动docker。

命令行登陆docker hub

```
docker login
```



启动docker swarm

```
docker swarm init
```



安装faas-cli

```
brew install faas-cli
```



clone下来代码：

```
git clone https://github.com/openfaas/faas
```

然后执行

```
./deploy_stack.sh
```



部署一些示例

```
faas-cli deploy -f https://raw.githubusercontent.com/openfaas/faas/master/stack.yml
```

使用浏览器打开 http://127.0.0.1:8080  就可以看到ui界面了。



安装grafana进行监控

```
docker service create -d \
--name=grafana \
--publish=3000:3000 \
--network=func_functions \
stefanprodan/faas-grafana:4.6.3
```

浏览器打开： http://127.0.0.1:3000  登陆admin  admin 查看。





常用命令：

```
$ faas-cli new --list
$ faas-cli build -f ./hello-openfaas.yml
$ faas-cli push -f ./hello-openfaas.yml
$ faas-cli deploy -f ./hello-openfaas.yml
```

