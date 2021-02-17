---
date: "2018-03-24 17:42:58"
title: docker
---

# docker常用命令

## docker

1. 获取镜像

   docker pull 


2. 新建并启动

   docker run


3. 列出镜像

   docker image ls

   docker images


4. 删除虚悬镜像

   docker image prune


5. 删除本地镜像

   docker iamge rm

6. 查看应用信息

   docker logs

## dockerfile

### 一般步骤：

- 在一个目录里，新建一个文件，命名为Dockerfile
- 在Dockerfile的目录内，执行docker build 

### 常用指令

1. FROM 指定基础镜像，且是第一条命令

2. RUN 执行命令

   shell格式

   exec格式

3. COPY和ADD指令是复制文件

4. CMD指令和RUN类似，容器启动命令

   shell格式

   exec格式

   参数列表格式

5. ENV 设置环境变量

6. EXPOSE 声明对外暴露的端口

7. WORKDIR 指定工作目录

## compose

### 两个重要的概念

- service 服务：一个应用的容器，实际上可以包括若干运行相同镜像的实例。
- project 项目：由一组关联的容器组成一个完整业务单元，在docker-compose.yml文件中定义。

### 一般步骤：

- 在一个项目目录里，新建一个Dockerfile

- 新建一个文件docker-compose.yml

  模板格式

  ```
  version: 3.0
  services:
  	web:
  		build: .
  		ports:
  			- "5000:5000"
  			
  	redis:
  		images: "redis:alpine"
  ```

- docker-compose up运行项目

### 常用命令：

1. docker-compose build 重新构建项目中的服务容器
2. config 验证compose文件格式是否正确
3. down 停止up命令所启动的容器
4. images 列出compose文件中包含的镜像
5. exec 进入指定的容器
6. kill 强制停止服务容器
7. ps 列出目前所有容器
8. rm 删除停止状态的容器
9. top 显示所有容器的进程

### compose模板文件：

每个服务都必须通过image指令指定镜像或者build指令（需要dockerfile）来构建生成的镜像。

1. build

   指定dockerfile所在的文件夹路径，compose将会利用它来自动构建这个镜像，然后使用。

2. depends_on

   解决容器的依赖和先后启动问题。但是不会等待完成启动之后再启动，而是在他们启动之后就去启动。


3. environment

   设置环境变量，在这里指定程序或者容器启动时所依赖的环境参数。


4. expose

   指定暴露的端口，只被连接的服务访问。


5. image

   指定镜像名称，如果本地不存在则去拉取这个镜像。


6. labels

   为容器添加docker元数据信息，即一些辅助说明。


7. ports

   暴露端口信息，宿主端口:容器端口，或者只指定容器端口。

