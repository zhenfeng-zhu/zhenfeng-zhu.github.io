---
title: "Openfaas Workshop Lab1b"
date: 2021-03-30T09:55:20+08:00
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

# 实验 1 - 使用 Kubernetes 设置 OpenFaaS

<img src="https://kubernetes.io/images/kubernetes-horizontal-color.png" width="500px"></img>

## 安装 `kubectl`

使用下面的说明或[官方文档](https://kubernetes.io/docs/tasks/tools/install-kubectl/)为您的操作系统安装`kubectl`。

- Linux

```sh
export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin/
```

- MacOS

```sh
export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/darwin/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin/
```

- Windows

```sh
export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/windows/amd64/kubectl.exe
chmod +x kubectl.exe
mkdir -p $HOME/bin/
mv kubectl $HOME/bin/
```

## 设置 Kubernetes 集群

您可以在使用 Kubernetes 时遵循实验，但您可能需要在此过程中进行一些小的更改。网关的服务地址从`http://gateway:8080` 更改为`http://gateway.openfaas:8080`。尽可能记录这些差异，并在每个实验室提供替代方案。

### 在笔记本电脑上创建本地集群

#### _k3s 和 k3d_

如果您的计算机上有 Docker，那么您可以使用 Rancher Labs 的 k3d。它安装了一个名为 k3s 的 Kubernetes 轻量级版本，并在 Docker 容器中运行，这意味着它可以在任何有 Docker 的计算机上工作。

- [安装 k3d](https://github.com/rancher/k3d)

- 启动一个集群

1. `k3d cluster create CLUSTER_NAME` 创建一个新的单节点集群 (= 1 个运行 k3s 的容器+1 个负载均衡器容器)
2. `k3d kubeconfig merge CLUSTER_NAME --switch-context` 更新默认的 kubeconfig 并且 切换到当前的上下文。
3. 执行一下命令，比如 `kubectl get pods --all-namespaces`
   如果你想要删除一个集群 `k3d cluster delete CLUSTER_NAME`

#### _Docker for Mac_

- [安装 Docker for Mac](https://docs.docker.com/v17.12/docker-for-mac/install/)

> 请注意，Kubernetes 仅适用于 Mac 17.12 CE 及更高版本的 Docker

#### _使用 Minikube_

- 要安装 Minikube，请根据您的平台从 [latest release](https://github.com/kubernetes/minikube/releases) 进行下载安装。

- 现在通过如下命令启动 Minikube

```
$ minikube start
```

Minikube VM 通过一个仅限主机的 IP 地址暴露给主机系统。通过`minikube ip`检测这个 IP。
这是稍后将用于网关 URL 的 IP。

> 注意： Minikube 还需要一个 Hypervisor，如 VirtualBox 或 Hyperkit（在 MacOS 上）。请遵循 Minikube 说明和文档

### 在云上创建远程集群

您可以在云中创建远程集群，享受与本地开发相同的体验，同时节省 RAM/CPU 和电池。运行集群 1-2 天的成本是最低的。

#### _在 DigitalOcean 上运行 Kubernetes 服务_

您可以使用免费学分通过 DigitalOcean 的 UI 创建集群。

然后，DigitalOcean 的 dashboard 将指导您如何配置用于实验室的`kubectl` 和`KUBECONFIG` 文件。

- [申请您的免费信用-30 天内的 50 美元信用。](https://m.do.co/c/8d4e75e9886f)

即使您已经申请了免费信用，2-3 个节点集群 24-48 小时的运行成本也是可以忽略不计的。

- 单击 dashboard 左侧面板的 _Kubernetes_， 然后点击 "Enable Limited Access"

- 登录后, 单击 _Kubernetes_ 菜单创建一个集群。

建议使用最新的 Kubernetes 版本和选择最近的数据中心以最大限度地减少延迟。

- 在 "Add node pool(s)" 下面

使用 2x 4GB / 2vCPU

> 注意：如果需要，您可以在以后添加更多容量。

- 下载 [doctl](https://github.com/digitalocean/doctl#installing-doctl) CLI， 并且放到自己的路径下面。

- 创建 [API Key](https://cloud.digitalocean.com/account/api/tokens/new)

跟踪 API key (复制到剪贴板)

- 授权 CLI

```sh
$ doctl auth init
```

粘贴 API key

- 现在获取集群的名称：

```sh
$ doctl k8s cluster list
GUID    workshop-lon1      nyc1      1.13.5-do.1    provisioning    workshop-lon1-1
```

- 保存一个配置文件， 这样`kubectl` 就可以指向一个新集群：

```sh
$ doctl k8s cluster kubeconfig save workshop-lon1
```

现在需要将 Kubernetes 上下文切换到指向新集群。

如果在`kubectl config get-contexts`中找不到高亮显示的集群名称，则使用`kubectl config set-context <context-name>`。

#### _Run on GKE (Google Kubernetes Engine)_

登录谷歌云，创建一个项目并为其启用计费。如果你没有账户，你可以在这里注册[here](https://cloud.google.com/free/)。

安装 [Google Cloud SDK](https://cloud.google.com/sdk/docs) - 这将可以使用 `gcloud` 和 `kubectl` 命令。
对于 windows，请参考 [documentation](https://cloud.google.com/sdk/docs/#windows).

安装 gcloud 命令之后，通过`gcloud init`配置默认项目、计算区域等（将 PROJECT_ID 替换为您自己的项目）。

```sh
$ gcloud config set project PROJECT_ID
$ gcloud config set compute/region us-central1
$ gcloud config set compute/zone us-central1-a
```

启用 Kubernetes service:

```sh
$ gcloud services enable container.googleapis.com
```

安装 kubectl:

```sh
gcloud components install kubectl
```

创建 Kubernetes cluster:

```sh
$ gcloud container clusters create openfaas \
--zone=us-central1-a \
--num-nodes=1 \
--machine-type=n1-standard-2 \
--disk-size=30 \
--no-enable-cloud-logging
```

给`kubectl`设置 credentials:

```sh
$ gcloud container clusters get-credentials openfaas
```

创建一个集群管理员：

```sh
$ kubectl create clusterrolebinding "cluster-admin-$(whoami)" \
--clusterrole=cluster-admin \
--user="$(gcloud config get-value core/account)"
```

现在验证 `kubectl` 已经被配置到了 GKE 集群：

```
$ kubectl get nodes
NAME                                   STATUS    ROLES     AGE       VERSION
gke-name-default-pool-eceef152-qjmt   Ready     <none>    1h        v1.10.7-gke.2
```

## 部署 OpenFaaS

部署 OpenFaaS 的说明会不时更改，因为我们努力让这变得更容易。

### 安装 OpenFaaS

有三种安装 OpenFaaS 的方法，您可以选择对您和您的团队有意义的方法。在这个研讨会中，我们将使用官方安装程序`arkade`。

- `arkade 应用安装` - arkade 使用官方的 helm chart 安装 OpenFaaS。 它还可以提供其他具有用户友好型 CLI 的软件，比如 `cert-manager` 和 `nginx-ingress`。 这是启动和运行的最简单、最快的方法。

- Helm chart - 易于通过 YAML 或 CLI 标志进行配置。对于那些在限制性环境中工作的人来说，也存在安全选项， 比如 `helm template` 或 `helm 3`。

- 普通 YAML 文件 - 硬编码 settings/values。如 Kustomise 可以提供自定义设置。

#### `arkade`

- 获取 arkade

For MacOS / Linux:

```sh
curl -SLsf https://dl.get-arkade.dev/ | sudo sh
```

For Windows:

```sh
curl -SLsf https://dl.get-arkade.dev/ | sh
```

- 安装 OpenFaaS

如果使用提供负载均衡器的托管云 Kubernetes 服务，则运行以下操作：

```sh
arkade install openfaas --load-balancer
```

> 注意：'--load-balancer'标志默认为`false`，因此需要通过传递该标志给云厂商。

如果使用本地 Kubernetes 群集或 VM，请运行：

```sh
arkade install openfaas
```

#### helm (高级)

如果您愿意，您可以使用[ [helm chart](https://github.com/openfaas/faas-netes/blob/master/chart/openfaas/README.md)。

### 登录 OpenFaaS gateway

- 检查 gateway 已经可用

```sh
kubectl rollout status -n openfaas deploy/gateway
```

如果您使用的是笔记本电脑、VM 或任何其他类型的 Kubernetes 发行版，请改为运行以下内容：

```sh
kubectl port-forward svc/gateway -n openfaas 8080:8080
```

此命令将打开一条从 Kubernetes 集群到本地计算机的隧道，以便您可以访问 OpenFaaS 网关。还有其他方法可以访问 OpenFaaS，但这超出了本研讨会的范围。

Gateway URL 是： `http://127.0.0.1:8080`

如果使用托管云 Kubernetes 服务，则从下面的命令中的`EXTERNAL-IP` 字段获取 LoadBalancer 的 IP 地址或 DNS 地址。

```sh
kubectl get svc -o wide gateway-external -n openfaas
```

- 登录：

```sh
export OPENFAAS_URL="" # Populate as above

# This command retrieves your password
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)

# This command logs in and saves a file to ~/.openfaas/config.yml
echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```

- 检查 `faas-cli list`：

```sh
faas-cli list
```

### 永久保存您的 OpenFaaS URL

编辑 `~/.bashrc` or `~/.bash_profile` 。

现在添加以下内容——根据上面看到的内容更改 URL 地址。

```sh
export OPENFAAS_URL="" # populate as above
```

现在进入 [实验 2](lab2.md)
