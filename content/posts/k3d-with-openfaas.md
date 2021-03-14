---
title: "K3d With Openfaas"
date: 2021-03-10T20:09:11+08:00
draft: false
---

# openfaas

https://github.com/openfaas/workshop/blob/master/lab1b.md

## 安装docker
```
brew install homebrew/cask/docker
```

## 安装单节点K8S
```
brew install k3d
```

配置单节点K8S集群

```
k3d cluster create CLUSTER_NAME

k3d kubeconfig merge CLUSTER_NAME --kubeconfig-switch-context

kubectl get pods --all-namespaces

```


## 安装arkade
```
curl -SLsf https://dl.get-arkade.dev/ | sudo sh
```

## 安装openfaas客户端 faas-cli
```
brew install faas-cli
```

## 安装openfaas server端
```
arkade install openfaas
```

配置openfaas的ui界面
```
kubectl rollout status -n openfaas deploy/gateway

kubectl port-forward svc/gateway -n openfaas 8080:8080
```
这样就可以在浏览器里输入 127.0.0.1:8080 进入到openfaas的ui界面了。

但是当你打开页面的时候，要输入密码，那就需要下面的操作：
```
# This command retrieves your password
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)

# This command logs in and saves a file to ~/.openfaas/config.yml
echo -n $PASSWORD | faas-cli login --username admin --password-stdin

```

或者直接在命令行输入，拿到密码：

```
echo $(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
```
用户名是admin，密码输入到浏览器里即可。


