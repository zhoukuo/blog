# 使用drone和gitea搭建CI/CD系统

zhoukuo@2020-09-15

Drone 是一款基于 Docker 的 CI/CD 工具，所有编译、测试、发布的流程都在 Docker 容器中进行。

开发者只需在项目中包含 .drone.yml 文件，将代码推送到 git 仓库，Drone 就能够自动化的进行编译、测试、发布。

## 零. 为什么使用 Drone 作为CI/CD 工具

```
功能灵活强大：构建、测试、发布、部署，你想干什么都可以，一套系统全搞定
```
```
兼容性好：支持所有SCM、所有平台、所有语言
```
```
环境部署简单：原生支持Docker容器，启动两个容器就完成了部署，其它构建、测试、部署工具在使用时会自动从docker仓库拉取
```
```
扩展性强：强大的插件系统，丰富的插件可以免费使用，也可以自定义
```
```
配置简单：正如官方宣传的那样，“configuration as a code”，所有功能、步骤、工具、命令，一个yaml配置文件全搞定
```
```
维护简单：直接复用SCM的账号体系和权限管理，无需注册用户、分配权限
```


## 一. 安装前准备

### 创建新的 OAuth2 应用程序

创建一个Gitea的 OAuth2 应用程序，“客户端ID”和“客户端密钥”用于授权访问Gitea的资源。

重定向 URI配置必须按照下面示例的格式和路径，并且必须是真实存在的。

```
客户端ID
434eec5a-3e7d-4ab4-9d2f-0374de99d87c
```

```
客户端密钥
1-MEs8whyiJ78TSo35WISPVNS69XKVm2Nb5Kh7asYB0=
```

```
应用名称
Drone
```

```
重定向URI
http://192.168.126.201:9000/login
```

### 创建新的共享密钥

创建一个新的共享密钥，用于授权Runners和Drone Server之间进行通信。

你可以使用openssl命令生成一个共享密钥：

```bash
$ openssl rand -hex 16
7945e09c23e3c84f0f49c922b80d3b2e
```


## 二. 下载

Drone Server 以轻量级的Docker镜像的形式发布，镜像是自包含的，没有任何外部依赖。

```bash
$ docker pull drone/drone:latest
```

## 三. 配置

Drone Server 使用环境变量进行配置，这里引用了配置项的一个子集，完整的配置请参考：https://docs.drone.io/server/reference

```
- DRONE_GITEA_CLIENT_ID
  Required string value provides your Gitea oauth Client ID.
```

```
- DRONE_GITEA_CLIENT_SECRET
  Required string value provides your Gitea oauth Client Secret.
```

```
- DRONE_GITEA_SERVER
  Require string value provides your Gitea server address. For example https://gitea.company.com
```

```
- DRONE_GIT_ALWAYS_AUTH
  Optional boolean value configures Drone to authenticate when cloning public repositories.
```

```
- DRONE_RPC_SECRET
  Required string value provides the shared secret generated in the previous step. This is used 
  to authenticate the rpc connection between the server and runners. The server and runner must 
  be provided the same secret value.
```

```
- DRONE_SERVER_HOST
  Required string value provides your external hostname or IP address. If using an IP address you
  may include the port. For example drone.company.com.
```

```
- DRONE_SERVER_PROTO
  Required string value provides your external protocol scheme. This value should be set to http 
  or https. This field defaults to https if you configure ssl or acme.

```

```
- DRONE_USER_CREATE
  Optional user account that should be created on startup. This should be used to seed the system 
  with an administrative account. It can be a real account.
```

## 四. Drone 服务启动

通过以下命令可以启动Drone服务，容器通过环境变量配置，如果想要查看完整的配置参数，请查看配置参考(https://docs.drone.io/server/reference)

```bash
$ docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITEA_SERVER={{DRONE_GITEA_SERVER}} \
  --env=DRONE_GITEA_CLIENT_ID={{DRONE_GITEA_CLIENT_ID}} \
  --env=DRONE_GITEA_CLIENT_SECRET={{DRONE_GITEA_CLIENT_SECRET}} \
  --env=DRONE_RPC_SECRET={{DRONE_RPC_SECRET}} \
  --env=DRONE_SERVER_HOST={{DRONE_SERVER_HOST}} \
  --env=DRONE_SERVER_PROTO={{DRONE_SERVER_PROTO}} \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:latest
```

示例如下:

```bash
$ docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITEA_SERVER=http://192.168.126.201:8080 \
  --env=DRONE_GITEA_CLIENT_ID=434eec5a-3e7d-4ab4-9d2f-0374de99d87c \
  --env=DRONE_GITEA_CLIENT_SECRET=1-MEs8whyiJ78TSo35WISPVNS69XKVm2Nb5Kh7asYB0= \
  --env=DRONE_RPC_SECRET=7945e09c23e3c84f0f49c922b80d3b2e \
  --env=DRONE_SERVER_HOST=192.168.126.201:9000 \
  --env=DRONE_SERVER_PROTO=http \
  --publish=9000:80 \
  --publish=9443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:latest
```

### 启用 Drone Trusted

如果需要挂载 volumes，需要先启用 Drone Trusted。

必须在 Drone 启动命令中增加配置：DRONE_USER_CREATE=username:(Git 仓库当前授权的账号名),admin:true，才有该菜单。

```bash
docker run \
...
--env=DRONE_USER_CREATE=username:root,admin:true \
...
```

## 五. 安装 Runners

一旦你的服务已启动并运行，你就可以安装runners来执行你的构建流水线(pipeline).

Drone runners 轮询服务器以查找要执行的工作任务，这里提供了几种不同的runners针对不同用户场景和运行时环境进行了优化，你可以根据你的情况安装一个或多个，一种或多种。

```
Docker Runner
```
```
Kubernetes Runner
```
```
Exec Runner
```
```
SSH Runner
```
```
Digital Ocean Runner
```
```
Macstadium Runner
```


这里我以最常用的 Docker Runner 为例，介绍一下如何配置 runner。

### Docker Runner

Docker runner 是一个守护进程，它在一个短生命周期容器中执行流水线（pipeline）任务。你可以安装一个单独的 Docker runner，或者在多台机器上安装来创建一个构建集群。

Docker runner 是一个通用的 runner，针对可以在无状态容器中运行测试和编译代码的项目进行了优化。

Docker runner 不太适合不能在容器内运行测试或编译代码的项目，包括以 Docker 不支持的操作系统或体系结构为目标的项目，如macOS。

### 在Linux上安装 Docker Runner

#### 1. 下载

安装Docker服务并且拉取公共镜像：

```bash
$ docker pull drone/drone-runner-docker:latest
```

#### 2. 配置

Docker Runner 使用环境变量进行配置。这里提供了部分参数，完成的参数请参考：https://docs.drone.io/runner/docker/configuration/reference

```
- DRONE_RPC_HOST
  provides the hostname (and optional port) of your Drone server. The runner connects to the server 
  at the host address to receive pipelines for execution.
```

```
- DRONE_RPC_PROTO
  provides the protocol used to connect to your Drone server. The value must be either http or https.
```

```
- DRONE_RPC_SECRET
  provides the shared secret used to authenticate with your Drone server. This must match the secret 
  defined in your Drone server configuration.
```
#### 3. 安装

下面的命令用于创建并且启动一个 Docker runner。*请注意要把这里的环境变量替换为实际的参数*

```bash
$ docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DRONE_RPC_PROTO=http \
  -e DRONE_RPC_HOST=192.168.126.201:9000 \
  -e DRONE_RPC_SECRET=7945e09c23e3c84f0f49c922b80d3b2e \
  -e DRONE_RUNNER_CAPACITY=4 \
  -e DRONE_RUNNER_NAME=${HOSTNAME} \
  -p 3000:3000 \
  --restart always \
  --name runner \
  drone/drone-runner-docker:latest
```

#### 4. 验证

使用 docker logs 命令查看日志来验证 runner 是否与 Drone sever 成功建立一个连接。

```bash
$ docker logs runner
INFO[0000] starting the server
INFO[0000] successfully pinged the remote server 
```

## 六. 流水线配置

流水线（pipeline）帮助你自动化软件交付过程，例如：构建、测试、部署。

流水线执行由代码库触发，代码的更改会触发一个webhook到Drone，并运行相应的流水线，其它常见的触发器包括计划任务或用户启动的工作流。

流水线是通过一个放置在代码库根目录中的.drone.yml文件进行配置。yaml 语法的设计易于阅读和表达，因此任何查看存储库的人都可以理解。

之前的 runner 使用的是 docker runner，这里还是以 docker pipeline 为例介绍如何配置构建流水线。

以下是一个pipeline的配置示例：

```yaml
---
kind: pipeline
type: docker
name: default

steps:
- name: backend
  image: golang
  commands:
  - go build
  - go test

- name: frontend
  image: node
  commands:
  - npm install
  - npm run test

...
```
Drone 支持不同类型的流水线（pipeline）,针对不同的用户场景和运行时环境进行优化。

```
Docker PipeLines
```
```
Kubernetes PipeLines
```
```
Exec PipeLines
```
```
SSH PipeLines
```
```
Digital Ocean PipeLines
```
```
Macstadium PipeLines
```

### Docker Pipeline


Docker pipeline 在短生命周期容器中执行管道命令。Docker 容器提供了隔离，允许在同一台机器上安全地执行并发管道。

pipeline 配置示例如下：

```yaml
---
kind: pipeline
type: docker
name: default

steps:
- name: greeting
  image: golang:1.12
  commands:
  - go build
  - go test
...
```

kind 和 type 属性表示这是一个 Docker pipeline。

```yaml
---
kind: pipeline
type: docker
```

steps 定义了一组 shell 命令，这些命令在 Docker 容器内部作为入口点执行。

如果任何一个命令返回非0，pipeline 就会失败并退出。

```yaml
steps:
- name: greeting
  image: golang:1.12
  commands:
  - go build
  - go test
```

## 七. 总结

如今，越来越多的企业开始重视软件研发的效率和质量，也有越来越多的企业开始采用DevOps理念来提升效率和质量。在整个软件研发全生命周期中，CICD是非常重要并且提升效率最为明显的阶段。通过实现自动化，能够大幅缩短从开发到测试到部署的时间。

随着Docker、Kubernetes等容器技术的成熟，特别是一线互联网公司已经将自己的业务部署在云端。在不远的未来，云计算必将成为软件系统运行的基础设施环境，就像如今的水和电一样想用就用。云计算的普遍使用，也催生了云原生技术的发展，同时也催生了云原生下的CICD平台的崛起，Drone就是其中一员。

Jenkins在一段时期内是CICD的代名词，界面化的操作和配置根本无法谈增效，Jenkins 2.0后，通过配置即代码的最佳实践，将流水线的构建过程配置到jenkinsfile里，提交到代码仓库下也纳入到了版本控制下，但真正用起来的并不多见。Jenkins X是下一代基于云原生的CICD框架，以Docker和Kubernetes容器生态为基础组件，通过命令行的方式实现CICD的所有功能。

JenkinsX和Drone都属于云原生下的CICD框架，都能充分利用容器的天然优势，提高CICD的灵活性和效率，JenkinsX目前仍在开发中，Drone目前来看已经在多个案例使用。如果打算构建容器环境的CICD平台，Drone可以是个不错的选择。
