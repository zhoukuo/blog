
# SPUG运维平台试用

## 简介

## 安装

## 主机管理

## 发布管理

### SPUG 发布配置

```
应用发布 --> 应用管理 --> 新建发布 --> 自定义发布 --> 编辑自定义发布 --> 提交
```

本地动作1：拉取代码

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

本地动作2：构建
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

本地动作3：发布

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

目标主机动作1：部署

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


## 监控告警

## 角色权限
