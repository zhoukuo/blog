# Pipeline流水线设计与实现-shell版

完整的pipeline应该包括以下几个步骤：

```
CLONE -- UNIT TEST -- BUILD -- PUBLISH -- DEPLOY -- API TEST -- NOTIFY
```

但由于研发阶段需要频繁构建，完整的pipeline还是比较耗时，因此简化后的pipeline只包括以下几个步骤：

```
CLONE -- BUILD -- PUBLISH -- DEPLOY
```

完整的pipeline通常只有在测试阶段会使用。

## CLONE

以GIT为例：

```bash
root@desktop:~/$ git clone -b master https://"username:password"@github.com/username/blog.git
```

在Jenkins中，通常可以直接通过页面配置

![](../img/jenkins/git.png)

## UNIT TEST

单元测试一般在构建前执行，以JAVA为例，在Maven构建时会首先执行单元测试，除非禁止单元测试执行

## BUILD

每种语言都有自己的构建工具，这里以JAVA为例，JAVA最常用的是Maven：

```bash
root@desktop:~/$ clean install -pl hospital-web/hospital-web-doctor
```

在Jenkins中，通常可以直接通过页面配置

![](../img/jenkins/maven.png)

## PUBLISH

发布的含义是将构建的产出物上传到发布服务器上供其他人使用，具体步骤如下：

1. 创建临时目录，用来临时存放待上传文件

2. 收集构建的产出物、构建版本和commitid，并放到临时目录

3. 将临时目录所有文件上传到发布服务器上以build号命名的目录下，同时，也会放到build-latest目录，确保build-latest目录始终存放最新的build

使用方法：

```bash
./upload.sh $source_dir $appname $dest_ip $dest_dir
```

参数说明：

|参数|说明|备注|
|-|-|-|
|source_dir|布署文件所在的目录|通常是target目录|
|app_name|布署文件名称	|只填写jar、war的文件名，或者需要打包的静态页面工程：如mobile.tar.gz|
|dest_ip|指定本地或远程文件服务器|本地的文件服务器（http://192.168.126.39:8082）；云端的文件服务器（http://47.95.231.203:8082/）|
|dest_dir|在文件服务器存放的相对目录|相对路径，从环境类型一级开始，比如开发环境：dev/ywq/ywqSchedule|

完整示例：

```
#!/bin/sh
#等号两侧不要有空格
 
source_dir=hospital-web/hospital-web-patient/target
app_name=patient.jar
dest_ip=192.168.126.39
dest_dir=dev/hospital/hospital-api-patient
 
wget http://$dest_ip:8082/shared//devops/upload.sh -O upload.sh
sh upload.sh $source_dir $app_name $dest_ip $dest_dir
 
exit
```

## DEPLOY

部署动作包括以下几个步骤：

1. 清理布署环境
2. 拉取布署文件
3. 拉取布署依赖的脚本
4. 重启服务
5. 服务验证

其中的每一个步骤，可能由单独的脚本来完成，如果步骤比较简单，就直接执行对应的命令。

目前的deploy的第4步会调用service/tomcat脚本来完成，第五步调用verify来完成，前3步直接执行shell命令。

使用方法：

```bash
./deploy.sh $source_type $source_dir $app_name $app_type $dest_ip $dest_dir $jvm_heap
```

参数说明：

|参数|说明|备注|
|-|-|-|
|source_ip|文件服务器IP|可选项：本地的文件服务器（http://192.168.126.39:8082）；remote表示云端的文件服务器（http://47.95.231.203:8082/）|
|source_dir|Jenkins构建产出的所有布署文件|目前规范要求所有的文件都保存在文件服务器的对应目录上，文件服务器地址：http://192.168.126.39:8082，具体保存在哪个目录由该服务对应的Jenkins构建任务配置决定|
|app_name|jar包的文件名|用于service.sh脚本启动和停止服务|
|app_type|应用类型|maven/npm/static|
|dest_ip|目标服务器的IP地址|前开发环境每个服务器都有2个IP段，分别是122和126的，为了降低126网段的带宽压力，同时加快传输速度，建议使用122段的IP|
|dest_dir|服务即将布署的目录|规范要求所有JAVA服务都要布署在/opt/szyx/目录下，并且要单独创建一个目录，例如医网签项目的目录为：/opt/szyx/ywq，用于service.sh脚本启动和停止服务|
|jvm_heap|jvm堆内存分配|非必填项，示例：256m，表示 java -Xms256m -Xmx256m，默认值：线上：1536m，集成：512m，内网：256m|


完整示例：

```
#!/bin/sh
#等号两侧不要有空格
 
source_ip=192.168.126.39
source_dir=dev/hospital/hospital-api-patient/build-${BUILD_NUMBER}
app_name=patient.jar
app_type=spring_boot
dest_ip=192.168.126.11
dest_dir=/opt/hospital/patient
 
wget http://$source_ip:8082/shared//devops/deploy-sv.sh -O deploy.sh
sh deploy.sh  $source_ip $source_dir $app_name $app_type $dest_ip $dest_dir

exit
```

## API TEST

接口测试可以通过很多种方法实现：

- CURL命令
- 脚本语言，如：python
- 编译语言，如：c
- 其它工具，如：newman

无论使用哪种方式，只要可以在shell命令行下执行就可以与jenkins集成，具体不再赘述。


## NOTIFY

通知最常用的是邮件，最近几年钉钉和企业微信通知更受欢迎。

- 邮件
- 微信、企业微信
- 钉钉

这些通知通常都是通过webhook实现，这里不再赘述，只要按照相应的接口配置即可。

