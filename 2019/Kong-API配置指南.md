# Kong-API配置指南(运维视角)

zhoukuo@2019-11-2

本文介绍如何配置Kong-API网关。

## 核心概念
- Upstream
- Target
- Service
- Routes

### Upstream
Upstream 对象表示虚拟主机名，可用于通过多个服务（目标）对传入请求进行负载均衡。例如：service.v1.xyz 为Service对象命名的上游host是service.v1.xyz对此服务的请求将代理到上游定义的目标。

### Target
目标IP地址/主机名，其端口表示后端服务的实例。每个上游都可以有多个target，并且可以动态添加Target。

### Service
服务实体是每个上游服务的抽象，有自己的服务端口，可能会部署多个节点，服务的主要属性是它的URL（其中，Kong应该代理流量），其可以被设置为单个串或通过指定其protocol， host，port和path。

- Name：服务名称，标识一个服务
- Protocol：http 或者 https
- Host：服务布署节点的IP，如果布署了多个节点，这里就是对应的Upstream的名称（需要使用Upstreams功能配置负载）
- Port：服务的端口，如果使用Upstream，可忽略
- Path：用来区分服务的相对路径，例如：/hisca

服务与路由相关联（服务可以有许多与之关联的路由）。路由是Kong的入口点，并定义匹配客户端请求的规则。一旦匹配路由，Kong就会将请求代理到其关联的服务。

### Route
路由实体定义规则以匹配客户端的请求。每个Route与一个Service相关联，一个服务可能有多个与之关联的路由。与给定路由匹配的每个请求都将代理到其关联的Service上。路由的条件至少是Host、Path、Method中的一种，通常我们只使用Path方式的路由。

Service和Route的组合（以及它们之间的关注点分离）提供了一种强大的路由机制，通过它可以在Kong中定义细粒度的入口点，从而使基础架构路由到不同上游服务。

### 实体关系图	
|实体|关系|
|-|-|
|Upstream ：Target|1 : N|
|Service : Upstream|1 : 1 或 1 : 0|
|Service : Route|1 : N|


## 使用流程
1. 添加upstream，配置target（可选）
2. 添加service，关联upstream/添加host，配置route

## 命名规范

|实体|命名|
|-|-|
|Upstream|<服务名>.api，例如：hisca.api|
|Service|<服务名>.service，例如：hisca.service|
|Route|<服务名>.route，例如：hisca.route|




