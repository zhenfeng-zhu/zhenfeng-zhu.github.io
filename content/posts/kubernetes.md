---
date: "2018-10-08 15:29:21"
title: kubernetes
---

### docker

利用Linux的cgroups和namespace，构建一个沙箱运行环境。

### docker镜像

其实就是一个压缩包，这个压缩包是由一个完整的操作系统的所有文件目录构成，包含了这个应用运行所需要的所有依赖，所以本地开发环境和测试环境是一样的。

解决了应用打包的根本性问题。

### 容器编排

对 Docker 容器的一系列定义、配置和创建动作的管理

> 容器本身没有价值，有价值的是“容器编排”。

### 原理

容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造一个“边界”。

在创建一个容器进程的时候，指定了这个进程所需要启动的一组Namespace参数，这样容器就只能看到当前Namespace所限定的资源、文件、设备、状态或配置。

Cgroups主要作用是为一个进程组设置资源上限，如CPU、内存、磁盘和带宽等。也可以设置进程优先级，审计，挂起，重启等。

因此，一个正在运行的Docker容器，其实就是一个启用了多个Namespace的应用进程，而这个进程能够使用的资源是由Cgroups来限制。

挂载在容器根目录上，用来为容器进程提供隔离后执行环境的文件系统，就是容器镜像，rootfs。

- 启动Namespace配置
- 设置Cgroups参数
- 切换进程根目录rootf

docker镜像设计时，引入了层（layer），用户制作镜像的每一步操作都会生成一个层，也就是一个增量的rootfs。AuFS，所以就有了共享层，镜像不用那么大。

一个进程，可以选择加入到某个进程已有的 Namespace当中，从而达到进入这个进程所在的容器的目的，这正是docker exec的实现原理。

volume机制，允许你将宿主机上指定的目录或文件，挂载到容器里面进行读取和修改操作。

### 主要依赖Linux依赖三大技术：

- Namespace
- Cgroups
- rootfs

### 和虚拟机比较

虚拟机是通过硬件虚拟化功能，模拟一套操作系统所需要的各种硬件，如CPU、内存、IO设备等，然后安装一个新的操作系统。

docker是利用Linux的Namespace原理，帮助用户启动的还是系统的应用进程，只是加了一些参数，限制其能看到的资源。因此相对于虚拟机资源消耗更小，而且轻量级，敏捷高性能。

不过缺点就是隔离不彻底，多个容器进程公用宿主机操作系统内核。有些资源和对象不可以被Namespace化的，如时间。



kubernetes要解决的问题

编排？调度？容器云？集群管理？

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fw117whrc6j31hc0u0gq5.jpg)

- master
  - kube-apiserver：API服务
  - kube-scheduler：调度
  - kube-controller-manager：编排
- node
  - kubelet：同容器运行时打交道。依赖于CRI（container runtime interface容器运行接口）远程调用接口，这个接口定义了容器运行时的各项核心操作。
  - 
- etcd

运行在大规模集群中的各种任务之间，实际存在各种各样的关系。这些关系的处理，才是作业编排和管理系统最困难的地方。

sudo

- 首先，通过一个编排对象，如pod，job或cronjob等，来描述你试图管理的应用；
- 然后，再为它定义一些服务对象，如service，secret，autoscaler等。这些对象，会负责具体的平台级功能。

这种使用方法，就是所谓的“声明式API”。这种API对应的编排对象和服务对象，都是k8s项目中的API对象。

## 简单使用

```
$ kubectl create -f 我的配置文件
```

pod就是k8s世界中的应用，而一个应用可以由多个容器组成。

使用一个API对象管理另一个API对象的方法，叫控制器模式。

每个API对象都有一个metadata字段，这个字段是API对象的标识，即元数据。主要用到的是labels，spec.selector.matchLabels就是k8s过滤的规则。与labels同层级的是annotations，这是由k8s所感兴趣的，而不是用户。

一个k8s的API对象都有metadata和spec两个部分。前者放的是对象的元数据，对所有API对象来讲，这部分的字段和格式基本一样；而后者存放的是属于这个对象独有的定义，用来描述它所要表达的功能。

```
$ kubectl create -f nginx-deployment.yaml

$ kubectl get pods -l app=nginx
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-67594d6bf6-9gdvr   1/1       Running   0          10m
nginx-deployment-67594d6bf6-v6j7w   1/1       Running   0          10m

$ kubectl describe pod nginx-deployment-67594d6bf6-9gdvr
Name:               nginx-deployment-67594d6bf6-9gdvr
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               node-1/10.168.0.3
Start Time:         Thu, 16 Aug 2018 08:48:42 +0000
Labels:             app=nginx
                    pod-template-hash=2315082692
Annotations:        <none>
Status:             Running
IP:                 10.32.0.23
Controlled By:      ReplicaSet/nginx-deployment-67594d6bf6
...
Events:

  Type     Reason                  Age                From               Message

  ----     ------                  ----               ----               -------
  
  Normal   Scheduled               1m                 default-scheduler  Successfully assigned default/nginx-deployment-67594d6bf6-9gdvr to node-1
  Normal   Pulling                 25s                kubelet, node-1    pulling image "nginx:1.7.9"
  Normal   Pulled                  17s                kubelet, node-1    Successfully pulled image "nginx:1.7.9"
  Normal   Created                 17s                kubelet, node-1    Created container
  Normal   Started                 17s                kubelet, node-1    Started container


$ kubectl apply -f nginx-deployment.yaml

# 修改 nginx-deployment.yaml 的内容

$ kubectl apply -f nginx-deployment.yaml

```

在命令行中，所有 key-value 格式的参数，都使用“=“而不是”：“表示。

在k8s执行过程中，对API对象的所有重要操作，都会被记录在这个对象的events中。



在线业务

Deployment

StatefunSet

DaemonSet



离线业务

Job



restartPolicy在job对象里只被允许设置为never和onFailure；而在Deployment对象中，只被允许设置为always。



### 声明式API和Kubernetes编程范式

创建一个两个Nginx容器的步骤：

首先写一个Deployment的yaml文件：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

```

然后使用kubectl create命令在Kubernetes中创建一个Deployment对象：

```
$ kubectl create -f nginx.yaml
```

这样两个Nginx pod就运行起来了。

如果要更新的话，只需要修改yaml文件，然后使用kubectl apply命令更新，触发了滚动更新。

这个apply命令就是声明式API。



![](https://ws1.sinaimg.cn/large/006tNbRwgy1fwzjwny2o8j31hc0u0abg.jpg)

istio项目中，最根本的组件是运行在每个pod里的envoy容器。这个代理服务以sidecar容器的方式，把整个pod的进出流量接管下来。istio的控制层的pilot组件，通过调用每个envoy的API，实现微服务的治理。

利用Kubernetes的Admission Control，也叫：Initializer，先创建一个Pod，然后istio就是在pod的yaml给Kubernetes之后，自动加上envoy的配置。

- 所谓的声明式，指的就是我只需要提交一个定义好的API对象来声明我所期望的状态是什么样子。
- 其次，声明式API允许有多个API写端，以PATCH的方式对API对象进行修改，而无需关心原始的YAML文件的内容。
- 最后，Kubernetes基于对API对象的增删改查，在无需外界干预的情况下，完成对实际状态和期望状态的调谐。

一个API对象在etcd中完整路径是由：group（API组），version（API版本）和Resource（API资源类型）三个部分组成的。

```yaml
apiVersion: batch/v2
kind: CronJob
```

batch是组，v2是版本，CronJob是类型。

对于核心API对象：Pod，Node等，不需要group的。非核心对象是需要组。

匹配规则就是：

/apis/batch/v2/CronJob

- 首先yaml文件被提交给了APIServer

  过滤，授权，超时处理或审计等

- 进入路由流程

  根据yaml，按照匹配规则去找

- 根据定义，按照yaml中的字段，创建一个对象

- 进行Amission和Validation。

- 把验证过的对象，序列化存到etcd中



### RBAC

基于角色的控制

role：角色，一组规则，定义Kubernetes API对象的操作权限

subject：被作用者，可以是人，也可以是机器，也可以是Kubernetes定义的用户

rolebinding：定义被作用者和角色的绑定关系



ServiceAccount，会被自动创建分配一个secret对象。



所谓角色就是一组权限规则列表，而我们分配这些权限的方式，就是通过创建rolebinding对象，将被作用者和权限列表进行绑定。

另外，与之对应的ClusterRole和ClusterRoleBinding，则是Kubernetes集群级别的Role和RoleBinding，它们的作用范围不受Namespace限制。

尽管被作用者有很多种（如User、Group），但在我们平常使用的时候，最普遍的还是ServiceAccount。



### 网络模型

Veth Pair 常常被用作连接不同 Network Namespace的网线。veth pair虚拟设备。总是以两张虚拟网卡形式成对出现。并且，从一个网卡中发出的数据包，可以直接出现在另一张网卡上，哪怕这两个网卡在不同的network Namespace里。

一旦一张虚拟网卡被插在网桥上，他就会变成该网桥的从设备。从设备会降级成为网桥的一个端口，不能处理数据包，只能接收流入的数据包交给对应的网桥。

两个容器的虚拟网卡都插在宿主机的一个网桥上，这个网桥就扮演一个交换机的角色。当两个容器进行网络交互时，从一个容器的发出请求到宿主机，由于Veth Pair 的机制，另一个容器就看到有数据流入。

因此默认情况下，被限制在network Namespace的容器进程，实际就是通过veth pair设备+宿主机网桥的方式，实现了跟其他容器的数据交换。

![img](https://ws1.sinaimg.cn/large/006tNbRwgy1fwzjwayfppj31bn0rngn5.jpg)



跨主通信，需要有一个集群公用的网桥，所有容器都连接到该网桥上，就可以相互通信，这就是overlay network（覆盖网络）