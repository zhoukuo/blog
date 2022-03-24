# DroneCI持续交付平台使用指南

## 概述

持续交付平台主要包括以下功能：

- 自动编译
- 自动部署
- 自动回滚
- 自动发布
- 静态代码检查
- 自动化测试
- 邮件通知

支持语言包括：

- Java
- JavaScript
- c/c++

DroneCI的配置与其它持续平台不同，通过在代码库中配置文件进行配置，配置文件名称通常为.drone.yml，也可以修改。



![](../img/droneci/持续交付.png)



DroneCI访问地址：https://droneci.qingtian-sec.net/

账号：使用gitea账号

## Java

### 自动编译

Java自动编译同样依赖Maven工具，但不需要自己安装，通过配置中指定Maven容器，自动下载容器并启动执行编译命令，完成后容器自动释放。

具体的配置示例如下：

```yaml
kind: pipeline
type: docker
name: build
steps:
- name: build
  image: maven:3-jdk-11
  volumes:
    - name: cache
      path: /root/.m2
    - name: docker.sock
      path: /var/run/docker.sock
  commands:
    - mvn clean install deploy -f ./pom.xml -T 1C
volumes:
- name: cache
  host:
    path: /data/.m2
- name: docker.sock
  host:
    path: /var/run/docker.sock
trigger:
  branch:
  - master
  event:
  - push
  - pull_request
```

配置说明：

0.代码仓库的pom配置要和编译平台的配置保持一致。

1.name由用户决定，没有限制，通常表示这个步骤要做的事情。

2.image是编译环境的容器镜像名称，这里使用maven3

3.volumes是本地缓存，由于maven在编译过程中需要依赖大量的第三方依赖包，而通过docker容器编译，每次编译完成后，容器都会释放，每次都重新下载非常耗费时间，使用本地缓存可以节省大量时间。

4.commands就是实际要执行的命令，这里只执行maven的编译命令，如果在编译前或编译后有其它工作要做，可以在这里补充。这里的命令只要是系统里支持的shell命令都可以执行。

5.maven的deploy参数表示编译成功后会把产出物上传到对应制品仓库中(如nexus)，如果已经配置，那么执行成功后就会在制品仓库的目录中看到。需要在pom.xml文件中增加发布配置：

```yaml
<distributionManagement>
    <repository>
        <id>releases</id>
        <name>Releases</name>
        <url>https://nexus.qingtian-sec.net/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <name>Snapshot</name>
        <url>https://nexus.qingtian-sec.net/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>

```

![](../img/droneci/nexus-result.png)

6.trigger是触发执行的条件，这里对分支和事件做了限制，只有master分支的push和pullrequest（分支合并）事件才会触发自动编译。也可以使用排除的方式，对某些分支忽略，比如：只对非master分支的改动才会触发编译动作，可以使用exclude关键字来实现。


```yaml
trigger:
  branch:
  	exclude:
  	- master
  event:
  - push
  - pull_request
```
7.编译成功后在交付平台上会看到一条执行成功的记录。

![](../img/droneci/droneci-compile-done.png)


### 自动部署

自动部署有两个场景：

**场景一：构建成功后自动部署，不需要人为干预。这个部署环境因为未经过完整的测试，可能会存在很多bug。**

这种场景只需要在之前自动编译的pipeline中增加自动部署的步骤就可以了，具体配置如下：

```yaml
kind: pipeline
type: docker
name: build
steps:
- name: build
  image: maven:3-jdk-11
  volumes:
    - name: cache
      path: /root/.m2
    - name: docker.sock
      path: /var/run/docker.sock
  commands:
    - mvn clean install deploy -f ./pom.xml -T 1C

- name: auto deploy
  image: appleboy/drone-ssh:1.6.3
  settings:
    host: 
      192.168.40.121
    username:
      qingtian
    key:
      from_secret: SSH_KEY
    script:
      - echo "192.168.40.121"
      - mkdir -p /opt/auto/qtsec-auth;cd /opt/auto/qtsec-auth
      - wget -q http://gitea.qingtian-sec.net/devops/deploy/raw/branch/master/pkg-mvn.sh -O pkg.sh;chmod +x pkg.sh
      - ./pkg.sh update auth-api-0.0.1-SNAPSHOT.jar
      - touch ${DRONE_COMMIT_SHA:0:10}-${DRONE_BUILD_NUMBER}-${DRONE_BRANCH}.commit;
      - wget -q http://gitea.qingtian-sec.net/devops/deploy/raw/branch/master/restartall.sh -O restartall.sh;chmod +x restartall.sh
      - ./restartall.sh

volumes:
- name: cache
  host:
    path: /data/.m2
- name: docker.sock
  host:
    path: /var/run/docker.sock

trigger:
  branch:
  - master
  event:
  - push
  - pull_request
```

配置说明：

0.由于DroneCI和Git平台已经打通，不需要配置代码库地址，DroneCI会自动拉取代码。

1.host就是目标服务器的ip地址，因为是自动触发的部署，因此必须明确部署到哪个服务器。

2.usename是服务器上的用户名，为了安全通常不使用root用户。

3.key是通过ssh登录服务器是使用的私钥，这里为了保密用到了DroneCI提供的Secret功能，可以将保密的内容作为value放到这里，然后使用对应的name调用，这里的SSH_KEY就是我们事先准备好的私钥，并且所有服务器通用。

4.script和之前自动编译中的commands是相同的，都是要执行的shell命令，这里的命令做了两件事，第一件事是从nexus仓库里把之前编译好的最新的包下载到本地服务器并替换原来的包（原来的包不会删除，而是放到.old目录中用于回滚），第二件事重启服务，这两件事都是通过shell脚本来完成的。

5.pkg.sh 脚本需要两个参数，一个是动作，一个是包名，动作目前支持update和rollback两个动作(rollback后面会介绍)，包名要和maven编译后的包名保持一致。

6.restartall脚本是重启服务的脚本，这个脚本不管当前目录有多少java服务，都会启动。

7..commit文件记录了包文件对应的git commit id和DroneCI生成的build号以及分支名称。

**场景二：研发人员提交代码至代码仓库后，触发自动编译，编译成功后，由测试人员部署至测试环境进行测试。**

这种场景的配置和之前略有不同，需要配置单独的pipeline。具体配置如下：

```yaml
---
kind: pipeline
type: docker
name: deploy
steps:
- name: deploy
  image: appleboy/drone-ssh:1.6.3
  settings:
    host: 
      ${DRONE_DEPLOY_TO}
    username:
      qingtian
    key:
      from_secret: SSH_KEY
    script:
      - mkdir -p /opt/qtsec-auth;cd /opt/qtsec-auth
      - wget -q http://gitea.qingtian-sec.net/devops/deploy/raw/branch/master/pkg-mvn.sh -O pkg.sh;chmod +x pkg.sh
      - ./pkg.sh update auth-api-0.0.1-SNAPSHOT.jar
      - touch ${DRONE_COMMIT_SHA:0:10}-${DRONE_BUILD_NUMBER}-${DRONE_BRANCH}.commit;
      - wget -q http://gitea.qingtian-sec.net/devops/deploy/raw/branch/master/restartall.sh -O restartall.sh;chmod +x restartall.sh
      - ./restartall.sh
clone:
  disable: true
trigger:
  event:
  - promote
```

配置说明：

1.host字段在这里是一个环境变量，可以接收来自页面输入的IP地址，因此可以由用户决定部署到哪个服务器上。

2.只能部署nexus仓库中最后一个快照版本，不能选择。

3.这里由于不需要代码库，因此clone动作被禁用。

4.触发事件是promote，代表只能在DroneCI平台通过点击Promote按钮来触发。

![](../img/droneci/promote.png)

### 自动回滚

一旦发现部署的包有问题，需要立即进行回滚，尤其在生产环境。回滚的配置也是一个单独的pipeline，具体配置如下：

```yaml
---
kind: pipeline
type: docker
name: rollback
steps:
- name: rollback
  image: appleboy/drone-ssh:1.6.3
  settings:
    host:
      ${DRONE_DEPLOY_TO}
    username:
      qingtian
    key:
      from_secret: SSH_KEY
    script:
      - cd /opt/qtsec-auth
      - ./pkg.sh rollback auth-api-0.0.1-SNAPSHOT.jar
      - ./restartall.sh
clone:
  disable: true
trigger:
  event:
  - rollback

```

配置说明：

1.回滚和部署是相对的，因此大部分配置和部署的相同，最主要的不同就是执行pkg脚本的参数不同，部署时使用update，回滚使用rollback

2.触发回滚的事件是rollback，代表只能在DroneCI平台通过点击Rollback按钮来触发。

![](../img/droneci/rollback.png)


### 自动发布

发布其实也是编译，区别在于日常开发过程中的编译包是快照版本（snapshot），而发布用于对测试通过的产物进行编译并归档，是发布版本(release)，以便在交付和在生产环境部署时使用。交付物管理同样适用Nexus，区别在于日常开发的产物放在Snapshot目录下，而发布的产物放在Release目录下。

具体配置如下：

```yaml
---
kind: pipeline
type: docker
name: publish
steps:
- name: publish to nexus
  image: maven:3-jdk-11
  volumes:
    - name: cache
      path: /root/.m2
    - name: docker.sock
      path: /var/run/docker.sock
  commands:
    - mvn clean deploy -f pom.xml -T 1C
volumes:
- name: cache
  host:
    path: /data/.m2
- name: docker.sock
  host:
    path: /var/run/docker.sock
trigger:
  branch:
    exclude:
    - master
  event:
  - custom
```

配置说明：

1.发布前研发人员需要基于测试通过的代码版本合入到发布分支，修改发布版本为正确的版本，SNAPSHOT替换成RELEASE，并推送到git服务器。

2.发布人员通过DroneCI平台的手动编译按钮，触发发布动作。（事件触发中的配置的custom）

3.其它和之前的编译过程没有任何区别，由于pom中的SNAPSHOT替换成了RELEASE，maven在执行deploy动作的时候会自动判断应该放到哪个目录。

4.同一个版本不能重复发布，如果要更新，需要先手动删除旧的发布包。

![](../img/droneci/publish.png)

### 静态代码检查

静态代码检查使用sonar静态代码检查平台，需要事先在sonar平台生成一个token，并将token配置到DroneCI配置文件中，

sonar中maven项目的配置信息如下：

![](../img/droneci/sonar-token-maven.png)

Droneci具体配置如下：

```yaml
- name: static code analysis
  image: maven:3-jdk-11
  volumes:
    - name: cache
      path: /root/.m2
    - name: docker.sock
      path: /var/run/docker.sock
  commands:
    - mvn sonar:sonar -Dsonar.projectKey=helloworld-maven -Dsonar.host.url=https://sonar.qingtian-sec.net -Dsonar.login=02fe0af6e055acb08a7cb6c474e2a2bef3ede9ca -f ./parent/pom.xml
```

### 自动化测试

对于java项目来说，自动化测试主要是接口的自动化测试，在服务器端通过newman调用postman脚本来执行。

```yaml
- name: test
  image: appleboy/drone-ssh:1.6.3
  settings:
    host: 
      ${DRONE_DEPLOY_TO}
    username:
      qingtian
    key:
      from_secret: SSH_KEY
    script:
      - cd /opt/qtvsoc-csa/test
      - bash runtest.sh
```

配置说明：

1.测试脚本放在git仓库中，配置前，需要在目标服务器把测试代码拉取到test目录

2.需要提前在目标服务器上安装newman工具

3.测试代码中要包括从postman导出的测试脚本、环境变量、全局变量

### 邮件通知

邮件通知使用qq邮件服务器发送，具体配置如下：

```yaml
---
kind: pipeline
type: docker
name: notify
steps:
- name: email notification
  image: drillster/drone-email
  settings:
    host: smtp.qq.com
    port: 465
    username: 754888159@qq.com
    password: wzuzobhinajebfbi
    from: 754888159@qq.com
    skip_verify: true
  when:
    status: [ failure ]
clone:
  disable: true
depends_on: [ build, deploy, publish ]

```

配置说明：

1.status是触发邮件通知的条件，这里的failure代表仅在编译失败时触发。

2.depends_on表示依赖哪个pipeline执行，这里在执行build、deploy或publish的pipeline后执行。



## Javascript

### 自动编译

Javascript自动编译依赖npm工具，但不需要自己安装，通过配置中指定node容器，自动下载容器并启动执行编译命令，完成后容器自动释放。

具体的配置示例如下：

```yaml
kind: pipeline
type: docker
name: build
steps:
- name: build
  image: node:14.16.1-alpine3.13
  volumes:
    - name: modules
      path: /drone/src/node_modules
    - name: npm
      path: /root/.npm
    - name: npmrc
      path: /root/.npmrc
  commands:
    - npm install && npm run build
    - sed -i s/npm-releases/npm-snapshots/ package.json && \cp package.json dist
    - cd dist; touch ${DRONE_COMMIT_SHA:0:10}-${DRONE_BUILD_NUMBER}-${DRONE_BRANCH}.commit; npm publish
volumes:
- name: modules
  host:
    path: /data/node_modules
- name: npm
  host:
    path: /data/.npm
- name: npmrc
  host:
    path: /data/.npm/.npmrc
trigger:
  branch:
  - master
  event:
  - push
  - pull_request

```

配置说明：

1.name由用户决定，没有限制，通常表示这个步骤要做的事情。

2.image是编译环境的容器镜像名称，这里使用node:14.16.1-alpine3.13。

3.volumes是本地缓存，由于node在编译过程中需要依赖大量的第三方依赖包，而通过docker容器编译，每次编译完成后，容器都会释放，每次都重新下载非常耗费时间，使用本地缓存可以节省大量时间。

4.commands就是实际要执行的命令，这里只执行node的编译相关的命令，如果在编译前或编译后有其它工作要做，可以在这里补充。这里的命令只要是系统里支持的shell命令都可以执行。

5.node的publish命令表示编译成功后会把产出物上传到对应制品仓库中(如nexus)，如果已经配置，那么执行成功后就会在制品仓库的目录中看到。相应的代码库中的package.json文件中，要增加如下发布配置。

```yaml
  "publishConfig": {
    "registry": "https://nexus.qingtian-sec.net/repository/npm-releases/"
  },
```

![](../img/droneci/npm-nexus-result.png)

6.trigger是触发执行的条件，这里对分支和事件做了限制，只有master分支的push和pullrequest（分支合并）事件才会触发自动编译。也可以使用排除的方式，对某些分支忽略，比如：只对非master分支的改动才会触发编译动作，可以使用exclude关键字来实现。

```yaml
trigger:
  branch:
  	exclude:
  	- master
  event:
  - push
  - pull_request
```

### 自动部署

自动部署有两个场景：

**场景一：构建成功后自动部署，不需要人为干预。这个部署环境因为未经过完整的测试，可能会存在很多bug。**

这种场景只需要在之前自动编译的pipeline中增加自动部署的步骤就可以了，具体配置如下：

```yaml
kind: pipeline
type: docker
name: build
steps:
- name: build
  image: node:14.16.1-alpine3.13
  volumes:
    - name: modules
      path: /drone/src/node_modules
    - name: npm
      path: /root/.npm
    - name: npmrc
      path: /root/.npmrc
  commands:
    - npm install && npm run build
    - sed -i s/npm-releases/npm-snapshots/ package.json && \cp package.json dist
    - cd dist; touch ${DRONE_COMMIT_SHA:0:10}-${DRONE_BUILD_NUMBER}-${DRONE_BRANCH}.commit; npm publish
- name: auto deploy
  image: appleboy/drone-ssh:1.6.3
  settings:
    host:
      192.168.40.120
    username:
      qingtian
    password:
      key:
        from_secret: SSH_KEY
    script:
      - echo "192.168.40.120"
      - mkdir -p /usr/share/nginx/html/auto; cd /usr/share/nginx/html/auto
      - wget -q http://gitea.qingtian-sec.net/devops/deploy/raw/branch/master/pkg-npm.sh -O pkg.sh;chmod +x pkg.sh
      - ./pkg.sh update csms-1.0.0.tgz
volumes:
- name: modules
  host:
    path: /data/node_modules
- name: npm
  host:
    path: /data/.npm
- name: npmrc
  host:
    path: /data/.npm/.npmrc
trigger:
  branch:
  - master
  event:
  - push
  - pull_request
```

配置说明：

0.由于DroneCI和Git平台已经打通，不需要配置代码库地址，DroneCI会自动拉取代码

1.host就是目标服务器的ip地址，因为是自动触发的部署，因此必须明确部署到哪个服务器

2.usename是服务器上的用户名，为了安全通常不使用root用户

3.key是通过ssh登录服务器是使用的私钥，这里为了保密用到了DroneCI提供的Secret功能，可以将保密的内容作为value放到这里，然后使用对应的name调用，这里的SSH_KEY就是我们事先准备好的私钥，并且所有服务器通用。

4.script和之前自动编译中的commands是相同的，都是要执行的shell命令，这里的命令做了两件事，第一件事是从nexus仓库里把之前编译好的最新的包下载到本地服务器并替换原来的包（原来的包不会删除，而是放到.old目录中用于回滚），第二件事重启服务，这两件事都是通过shell脚本来完成的。

5.pkg.sh 脚本需要两个参数，一个是动作，一个是包名，动作目前支持update和rollback两个动作(rollback后面会介绍)，包名要和npm编译后的包名保持一致。

6..commit文件记录了包文件对应的git commit id和DroneCI生成的build号以及分支名称

**场景二：研发人员提交代码至代码仓库后，触发自动编译，编译成功后，由测试人员部署至测试环境进行测试。**

这种场景的配置和之前略有不同，需要配置单独的pipeline。具体配置如下：

```yaml
---
kind: pipeline
type: docker
name: deploy
steps:
- name: deploy
  image: appleboy/drone-ssh:1.6.3
  settings:
    host:
      ${DRONE_DEPLOY_TO}
    username:
      qingtian
    key:
      from_secret: SSH_KEY
    script:
      - echo ${DRONE_DEPLOY_TO}
      - cd /usr/share/nginx/html
      - wget -q http://gitea.qingtian-sec.net/devops/deploy/raw/branch/master/pkg-npm.sh -O pkg.sh;chmod +x pkg.sh
      - ./pkg.sh update csms-1.0.0.tgz

trigger:
  event:
  - promote
clone:
  disable: true
```

配置说明：

1.host字段在这里是一个环境变量，可以接收来自页面输入的IP地址，因此可以由用户决定部署到哪个服务器上。

2.这里由于不需要代码库，因此clone动作被禁用。

3.触发事件是promote，代表只能在DroneCI平台通过点击Promote按钮来触发。

4.发布的目录是nginx服务器的静态资源目录，需要事先在nginx配置文件中配置好。

![](../img/droneci/npm-promote.png)

### 自动回滚

一旦发现部署的包有问题，需要立即进行回滚，尤其在生产环境。回滚的配置也是一个单独的pipeline，具体配置如下：

```yaml
---
kind: pipeline
type: docker
name: rollback
steps:
- name: rollback
  image: appleboy/drone-ssh:1.6.3
  settings:
    host:
      ${DRONE_DEPLOY_TO}
    username:
      qingtian
    key:
      from_secret: SSH_KEY
    script:
      - cd /usr/share/nginx/html/
      - ./pkg.sh update csms-1.0.0.tgz
trigger:
  event:
  - rollback
clone:
  disable: true
```

配置说明：

1.回滚和部署是相对的，因此大部分配置和部署的相同，最主要的不同就是执行pkg脚本的参数不同，部署时使用update，回滚使用rollback

2.触发回滚的事件是rollback，代表只能在DroneCI平台通过点击Rollback按钮来触发。

![](../img/droneci/npm-rollback.png)

### 自动发布

发布其实也是编译，区别在于日常开发过程中的编译包是快照版本（snapshot），而发布用于对测试通过的产物进行编译并归档，是发布版本(release)，以便在交付和在生产环境部署时使用。交付物管理同样适用Nexus，区别在于日常开发的产物放在Snapshot目录下，而发布的产物放在Release目录下。

具体配置如下：

```yaml
---
kind: pipeline
type: docker
name: publish to nexus
steps:
- name: build
  image: node:14.16.1-alpine3.13
  volumes:
    - name: modules
      path: /drone/src/node_modules
    - name: npm
      path: /root/.npm
    - name: npmrc
      path: /root/.npmrc
  commands:
    - npm install && npm run build
    - cp package.json dist
    - cd dist; touch ${DRONE_COMMIT_SHA:0:10}-${DRONE_BUILD_NUMBER}-${DRONE_BRANCH}.commit; npm publish
volumes:
- name: modules
  host:
    path: /data/node_modules
- name: npm
  host:
    path: /data/.npm
- name: npmrc
  host:
    path: /data/.npm/.npmrc
trigger:
  branch:
    exclue:
    - master
  event:
  - custom
```

配置说明：

1.发布前研发人员需要基于测试通过的代码版本合入到发布分支，修改发布版本为正确的版本，SNAPSHOT替换成RELEASE，并推送到git服务器。

2.发布人员通过DroneCI平台的手动编译按钮，触发发布动作。（事件触发中的配置的custom）

3.其它和之前的编译过程没有任何区别，由于package.json中的SNAPSHOT替换成了RELEASE，npm在执行deploy动作的时候会自动判断应该放到哪个目录。

4.同一个版本不能重复发布，如果要更新，需要先手动删除旧的发布包。

![](../img/droneci/npm-publish.png)

### 静态代码检查

静态代码检查使用sonar静态代码检查平台，需要事先在sonar平台生成一个token，并将token配置到DroneCI配置文件中，具体配置如下：

sonar中js项目配置如下：

![](../img/droneci/sonar-token.png)

Droneci具体配置如下：

```yaml
- name: code-analysis
  image: aosapps/drone-sonar-plugin
  settings:
    sonar_host:
      from_secret: https://sonar.qingtian-sec.net
    sonar_token:
      from_secret: 5abc00e9a43d6e5f4fe584fa5e56c78a1095ac32
```

### 自动化测试

<待补充>

### 邮件通知

邮件通知使用qq邮件服务器发送，具体配置如下：

```yaml
---
kind: pipeline
type: docker
name: notify
steps:
- name: email notification
  image: drillster/drone-email
  settings:
    host: smtp.qq.com
    port: 465
    username: 754888159@qq.com
    password: wzuzobhinajebfbi
    from: 754888159@qq.com
    skip_verify: true
  when:
    status: [ failure ]
clone:
  disable: true
depends_on: [ build, deploy, publish ]

```

配置说明：

1.status是触发邮件通知的条件，这里的failure代表仅在编译失败时触发。

2.depends_on表示依赖哪个pipeline执行，这里在执行build、deploy或publish的pipeline后执行。



## C/C++

### 自动编译

C/C++的自动编译不依赖外部编译工具，而是使用公司开发的工具，具体配置如下：

```yaml
kind: pipeline
type: exec
name: default

platform:
  os: linux
  arch: amd64
clone:
  disable: true

steps:
- name: clone
  commands:
  - git clone qingtian@git.qingtian-sec.net:qtivi/qtivi-idps.git .
- name: build
  commands:
  - bash acs_build.sh qtivi-idps $DRONE_BRANCH $BCLOUD $PROJECT
```

1.pipeline的类型不同于之前的maven和node，而是使用exec的方式，主要是考虑编译的性能问题。

2.没有使用DroneCI提供的默认代码拉取功能(clone:disable true)，而是使用自定义的clone脚本，主要是这里有一个bug，无法自动拉取https方式的git仓库的代码。

3.build脚本需要提供几个参数：DRONE_BRANCH表示分支，BCLOUD参数有两个值，一个是change、一个是module，change表示只编译变更部分，module代表整个仓库都编译，PROJECT是项目名称，和config/project目录中的项目名称要对应。

### 自动部署

暂不支持

### 自动回滚

暂不支持

### 自动发布

<待补充>

### 静态代码检查

<待补充>

### 自动化测试

<待补充>

### 邮件通知

同Maven或Node项目

