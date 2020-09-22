# Docker 入门
zhoukuo@2016-10-22

Docker是一个开源的应用容器引擎，开发者可以利用Docker打包自己的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。

## 简介
容器与管理程序虚拟化有所不同，管理程序虚拟化通过中间层将一台或多台独立的机器虚拟运行于物理硬件之上，而容器则是直接运行在操作系统内核之上的用户空间。容器可以让多个独立的用户空间运行在一台宿主机上。

由于“客居”于操作系统，容器只能运行与底层宿主机相同或相似的操作系统，例如可以在Ubuntu服务器中运行CentOS，但无法在Ubuntu服务器上运行“Windows”。

那么Docker有什么特别之处呢？Docker在虚拟化的容器执行环境中增加了一个应用程序引擎，该引擎的目标就是提供一个轻量、快速的环境，能够运行开发者的程序，并方便高效地将程序从开发者的笔记本部署到测试环境，然后再部署到生产环境。

使用Docker，开发人员只需要关心容器中运行的应用程序，而运维人员只需要关心如何管理容器。Docker设计的目的就是要加强开发人员写代码的开发环境与应用程序要部署的生产环境的一致性，从而降低那种“开发时一切都正常，肯定是运维的问题”的风险。

Docker还鼓励面向服务的架构和微服务架构。Docker推荐单个容器只运行一个应用程序或进程，这样就形成了一个分布式的应用程序模型。

## Docker组件

Docker的核心组件：

* Docker客户端和服务器
* Docker镜像
* Registry
* Docker容器

## Docker架构

Docker是一个CS架构的程序，Docker客户端只需向Docker服务器（守护进程）发出请求，服务器将完成所有工作并返回结果。用户可以在同一台宿主机上运行Docker服务器和客户端，也可以从本地的Docker的客户端连接到运行在另一台宿主机上的远程Docker服务器。

![Docker架构](http://images2015.cnblogs.com/blog/438458/201603/438458-20160314192315631-334072513.png)

## Docker镜像
镜像是构建Docker世界的基石，用户基于镜像来运行自己的容器。

## Registry
Docker用Registry来保存用户构建的镜像。Registry分为公共和私有两种，Docker公司运营的公共Registry叫作Docker Hub。用户可以在Docker Hub注册账号，分享保存自己的镜像。

## 容器
Docker可以帮用户构建和部署容器，用户只需要把自己的应用程序或服务打包放进容器即可。每个容器都包含一个软件镜像，容器可以被创建、启动、关闭、重启以及销毁。

## 安装Docker

### 安装Docker引擎
```bash
[root@localhost zhoukuo]# yum -y install docker
```
### 启动守护进程
```bash
[root@localhost zhoukuo]# systemctl start docker
```
### 确保已经就绪
```bash
[root@localhost zhoukuo]# docker info
```

## 创建我们的第一个容器
```bash
[root@localhost zhoukuo]# docker create -t -i centos
Unable to find image 'centos:latest' locally
Trying to pull repository docker.io/library/centos ... 
latest: Pulling from docker.io/library/centos
8d30e94188e7: Pull complete 
Digest: sha256:2ae0d2c881c7123870114fb9cc7afabd1e31f9888dac8286884f6cf59373ed9b
Status: Downloaded newer image for docker.io/centos:latest
```

## 启动容器
```bash
[root@localhost zhoukuo]# docker start container_1
```

## 停止容器
```bash
[root@localhost zhoukuo]# docker stop container_1
```

## 重启容器
```bash
[root@localhost zhoukuo]# docker restart container_1
```

## 附着到正在运行的容器
```bash
[root@localhost zhoukuo]# docker attach container_1
```

## 创建并启动容器
```bash
[root@localhost zhoukuo]# docker run -i -t centos 
```

## 容器命名
```bash
[root@localhost zhoukuo]# docker create --name container_1 -i -t centos
```

```bash
[root@localhost zhoukuo]# docker run --name container_1 -i -t centos
```

docker run = docker create + docker start + docker attach

## 创建守护进程
除了这些交互式运行的容器，也可以创建长期运行的容器，
```bash
[root@localhost zhoukuo]# docker run --name daemon_dave -d centos /bin/sh -c "while true; do echo hello world; sleep 1; done"
```
我们在上面的docker run 命令使用了-d参数，因此Docker会将容器放到后台运行。

## 容器内部都在干些什么
```bash
docker logs daemon_dave
```
## 查看容器列表
```bash
[root@localhost zhoukuo]# docker ps -a
```
## 查看正在运行的容器
```bash
[root@localhost zhoukuo]# docker ps
```

## 查看最后一个运行的容器
```bash
[root@localhost zhoukuo]# docker ps -l
```

## 查看容器内的进程
```bash
[root@localhost zhoukuo]# docker top container_1
```

## Docker统计信息
```bash
[root@localhost zhoukuo]# docker stats
```

## 删除容器
```bash
[root@localhost zhoukuo]# docker rm container_1
```

## 删除所有容器
```bash
[root@localhost zhoukuo]# docker rm `docker ps -s -q`
```
