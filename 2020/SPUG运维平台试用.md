
# SPUG运维平台试用

zhoukuo@2020-10-19

## 简介

这里直接引用官方的介绍：

面向中小型企业设计的轻量级无Agent的自动化运维平台，整合了主机管理、主机批量执行、主机在线终端、文件在线上传下载、应用发布部署、在线任务计划、配置中心、监控、报警等一系列功能。

## 安装

官方推荐使用docker方式安装：

```bash
$ docker pull registry.aliyuncs.com/openspug/spug

$ docker run -d --restart=always --name=spug -p 80:80 registry.aliyuncs.com/openspug/spug

# 持久化存储启动命令：
# mydata指的是本地磁盘路径，也可以是其他目录，但需要保证映射的本地磁盘路径已经存在，/data是容器内代码和数据初始化存储的路径
$ docker run -d --restart=always --name=spug -p 80:80 -v /mydata/:/data registry.aliyuncs.com/openspug/spug

# 以下操作会创建一个用户名为 admin 密码为 spug.dev 的管理员账户，可自行替换管理员账户。
$ docker exec spug init_spug admin spug.dev

# 执行完毕后需要重启容器
$ docker restart spug
```

最后，在浏览器中输入 http://localhost:80 访问。

```bash
用户名： admin  
密码： spug.dev
```

## 主机管理

对于开发和维护服务的人员来说，这里要解决如何远程登录到目标服务器的问题。

spug相对jumpserver来说实现的比较简单，只要配置号登录账号就可以登录了，而且登录操作体验比较流畅。


## 发布管理

### 创建发布

```
创建发布流程：创建环境-> 创建应用-> 创建发布
```

创建发布前需要先添加应用，基于应用才能创建发布。这里的应用可以理解为一个独立运行的web服务。

一旦应用创建完成后，就可以创建发布了，发布依赖发布环境，例如：生产环境，开发环境等。

**注意：每个应用在每个环境只能创建一个发布任务。**

### 发布配置

```
应用发布 --> 应用管理 --> 新建发布 --> 自定义发布 --> 编辑自定义发布 --> 提交
```

**本地动作1：拉取代码**

```bash
if [ "$SPUG_RELEASE" = "" ]; then
    echo '[ERROR] "SPUG_RELEASE"字段为版本号，不能为空！'
    exit -1
elif [ "$SPUG_DEPLOY_TYPE" = "2" ]; then
    echo "回滚操作，跳过拉取代码步骤 ..."
    exit 0
elif [ "$SPUG_RELEASE" = "latest" ]; then
    echo "跳过拉取代码步骤，直接部署latest版本"
    exit 0
fi

/bin/rm cpia-parent -fr
git clone -b develop http://deploy:deploy2020%40Gitea@gitea.51trust.com/server/cpia-parent.git
```

**本地动作2：构建**
```bash
if [ "$SPUG_DEPLOY_TYPE" = "2" ]; then
    echo "回滚操作，跳过构建步骤 ..."
    exit 0
elif [ "$SPUG_RELEASE" = "latest" ]; then
    echo "跳过构建步骤，直接部署latest版本"
    exit 0
fi

cd cpia-parent
mvn clean install -pl cpia-api-web -am -P dev
```

**本地动作3：发布**

```bash
if [ "$SPUG_DEPLOY_TYPE" = "2" ]; then
    echo "回滚操作，跳过发布步骤 ..."
    exit 0
elif [ "$SPUG_RELEASE" = "latest" ]; then
    echo "跳过发布步骤，直接部署latest版本"
    exit 0
fi

cd cpia-parent
source_dir=cpia-api-web/target
app_name=cpia.jar
dest_ip=192.168.126.39
dest_dir=dev/service/service-cpia/build-${SPUG_RELEASE}

wget http://$dest_ip:8082/shared//devops/upload_spug.sh -O upload.sh
sh upload.sh $source_dir $app_name $dest_ip $dest_dir
```

**目标主机动作1：部署**

```bash
source_ip=192.168.126.39
source_dir=dev/service/service-cpia/build-${SPUG_RELEASE}
app_name=cpia.jar
app_type=maven
dest_dir=/opt/cpia

wget http://$source_ip:8082/shared//devops/deploy_spug.sh -O deploy.sh
sh deploy.sh $source_ip $source_dir $app_name $app_type $dest_dir
```

![](../img/spug/spug-cd1.png)


这里有一点需要注意一下，spug的发布是基于发布任务的，也就是没有发布任务就不能发布。如果在开发环境这样部署服务的话，效率太低了。希望作者可以增加一个可以基于发布任务直接执行的方式，这样对于开发环境就比较友好了。

## 监控告警

### 告警配置

这里以企业微信为例，介绍一下告警配置。

配置流程：spug告警配置是基于联系组的，因此，首先添加告警联系人，然后将联系人添加到联系组，最后在监控任务中配置报警联系人组即可。

由于企业微信的告警是基于webhook机器人实现的，而机器人的消息在组内都可见，因此一个告警机器人产生的告警，组内的人都可见，因此在spug中只需要添加一个联系人即可。

1. 添加报警联系人

企业微信的webhook机器人和其他IM工具类似，都是在在群组中添加。

1.1 创建群组，只有管理员才能添加机器人

1.2 添加机器人，复制webhook地址

![](../img/spug/wechat-webhook.png)

1.3 把机器人的URL地址粘贴到“企业微信”的字段上

![](../img/spug/wechat.png)


2. 添加报警联系组

至此，告警配置完成。

### 添加监控项

spug内置了4种最常见的服务健康相关的监控项：

- PING检测
- 进程检测
- 端口检测
- 站点检测

此外，spug还支持自定义脚本，脚本执行退出状态码为 0 则判定为正常，其他为异常。

因此可以说spug提供的监控项还是非常实用的，都是服务维护者关心的指标。至于基础资源监控，还是让运维同学关注吧。

![](../img/spug/check2.png)

这里以端口检测为例，介绍一下监控的配置。

![](../img/spug/check1.png)


告警配置页面包括以下几方面内容：

- 监控频率
- 报警阀值
- 报警联系人组
- 报警方式
- 通道沉默时间

这里分别解释一下：

监控频率：系统执行检测脚本的间隔时间，时间越短，报警越及时

报警阀值：发现异常几次才会报警

通道沉默：相同的告警信息，无论触发报警多少次，沉默期内只发送一次

![](../img/spug/check3.png)

一旦监控配置完成，就可以测试一下告警了

这里是监控项的列表

![](../img/spug/alert1.png)

这里是告警历史列表

![](../img/spug/alert2.png)


## 角色权限

SPUG的权限控制比较全面，包括三个维度：

- 功能权限
- 发布权限
- 主机权限

功能权限就是按照左侧菜单进行管理的。

发布权限按照发布环境和应用的维度进行配置

主机权限就是单个主机进行配置。

基本可以满足任何权限配置的要求。


## 工作台

最后看一下工作台的展示页面：

![](../img/spug/workspace.png)

## 总结

整体来看，spug提供的功能都是中小企业服务管理所需要的，就是作者自己说的那样：“麻雀虽小，五脏俱全”，如果能够进一步提供用户登录的双因子认证，以及操作日志审计功能就更好了。

