# Kong安装布署指南

zhoukuo@2019-10-31

在微服务架构中，由于系统和服务的细分，导致系统结构变得非常复杂， 为了跨平台，为了统一集中管理api，同时为了不暴露后置服务。甚至有时候需要对请求进行一些安全、负载均衡、限流、熔断、灰度等中间操作，基于此类种种的客观需求一个类似综合前置的系统就产生了，这就是API网关（API Gateway）。

API网关作为分散在各个业务系统微服务的API聚合点和统一接入点，外部请求通过访问这个接入点，即可访问内部所有的REST API服务。

## 安装Kong
参考：https://docs.konghq.com/install/centos/

### 1. YUM安装配置
```bash
$ sudo yum update -y
$ sudo yum install -y wget
$ wget https://bintray.com/kong/kong-rpm/rpm -O bintray-kong-kong-rpm.repo
$ export major_version=`grep -oE '[0-9]+\.[0-9]+' /etc/redhat-release | cut -d "." -f1`
$ sed -i -e 's/baseurl.*/&\/centos\/'$major_version''/ bintray-kong-kong-rpm.repo
$ sudo mv bintray-kong-kong-rpm.repo /etc/yum.repos.d/

```
### 2. 安装
```bash
$ sudo yum update -y
$ sudo yum install -y kong
```

### 3. 配置
Kong的配置可以使用数据库方式，也可以使用文件方式

Kong支持2种数据库：PostgreSQL 9.5+ ，Cassandra 3.x.x

#### 文件方式
这里我们使用文件方式，下面的命令会在当前目录生成一个配置文件kong.yml，通常会放在/etc/kong/目录下
```bash
$ kong config init
```

然后，按照如下方式配置：
cd /etc/kong;
mv kong.conf.default kong.conf
vi /etc/kong/kong.conf
```bash
database = off
declarative_config = /etc/kong/kong.yml
```

#### 数据库方式
生产环境建议使用数据库方式
如果使用数据库方式，需要提供一个用户和数据库，例如：
```bash
CREATE USER kong; CREATE DATABASE kong OWNER kong;
```


然后，修改配置文件：
```bash
#database = off
#declarative_config = /etc/kong/kong.yml
database = postgres              # Determines which of PostgreSQL or Cassandra

pg_host = 192.168.126.122       # Host of the Postgres server.
pg_port = 5432                  # Port of the Postgres server.
pg_timeout = 5000               # Defines the timeout (in ms), for connecting,

pg_user = kong                  # Postgres user.
pg_password =                   # Postgres user's password.
pg_database = kong              # The database name to connect to.

pg_ssl = on                      # Toggles client-server TLS connections
                                 # between Kong and PostgreSQL.
```

最后，初始化数据库：
```bash
kong migrations bootstrap [-c /path/to/kong.conf]
```

### 4. 启动
```bash
kong start -c /etc/kong/kong.conf
```
kong默认的配置文件/etc/kong/kong.conf，所以 这里的-c参数可以省略

### 5. 测试
```bash
curl -i http://localhost:8001
```

## 安装Konga
当前KONG的社区版是没有dashboard的，但是付费的企业版是有带的，排除自己去撸一个dashboard的这种选择，第三方开源的dashboard无疑是首选。
当前GitHub上还在更新维护的dashboard有两个，一个是kong-dashboard，另一个为konga，我们选择的便是KONGA，选择KONGA的主要原因为该开源dashboard支持到了官方最新版的0.14版，并且UI简洁。

### 安装nodejs
官方推荐版本为8.11.3

下载地址：https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-x64.tar.gz

```bash
$ cd /opt
$ mkdir nodejs;cd nodejs
$ wget https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-x64.tar.gz
$ tar -xvf node-v8.11.3-linux-x64.tar.gz
$ ln -s /opt/nodejs/node-v8.11.3-linux-x64/bin/node node
$ ln -s /opt/nodejs/node-v8.11.3-linux-x64/bin/npm npm
$ ln -s /opt/nodejs/node-v8.11.3-linux-x64/bin/npx npx
```

npm镜像替换成淘宝
```bash
$ npm config set registry http://registry.npm.taobao.org/
$ yarn config set registry http://registry.npm.taobao.org/
```
### 安装依赖
```bash
sudo npm install -g gulp
sudo npm install -g bower
sudo npm install -g sails
```
### 下载konga安装包
```bash
安装的时候增加--unsafe-perm 参加即可

```bash
$ cd /opt;mkdir konga;cd konga;
$ wget https://github.com/pantsel/konga/archive/0.14.7.tar.gz
$ tar -xvf 0.14.7.tar.gz;cd konga-0.14.7
```

### 安装konga

```bash
$ npm install konga
```

安装时如果提示：“gyp WARN EACCES user "root" does not have permission to access ...“，在安装命令后增加 "--unsafe-perm" 参数即可。

```bash
$ npm install konga --unsafe-perm
```

### 安装依赖
```bash
$ npm run bower-deps
$ npm install dotenv-extended
$ npm install angular
```

### 启动
```bash
#konga根目录
npm start
```
### 访问
http://localhost:1337

![](/img/kong/konga.jpg)



### 配置数据库信息
默认情况下，konga的配置存储在本地配置文件中，如果用于生产环境，建议使用数据库存储的方式。由于Kong也需要依赖数据库，并且他们共同支持的数据库只有postgres，这里我们选择postgres。

目前支持数据库: mysql, mongo, sqlserver, postgres

配置pgsql数据库：
```bash
$ pwd
$ /opt/konga/konga-0.14.7
$ mv.env_example .env
$ vi .env
PORT=1337
NODE_ENV=production
KONGA_HOOK_TIMEOUT=120000
DB_ADAPTER=postgres
DB_URI=postgresql://192.168.126.122:5432/konga
KONGA_LOG_LEVEL=warn
TOKEN_SECRET=some_secret_token
```

接下来初始化数据库表，在执行初始化脚本前，需要注意以下几点：
1. pgsql必须开启SSL
2. 数据库密码算法修改：md5 -> trust
3. pgsql需要创建和当前linux用户同名的用户，这里是root
4. pgsql必须准备一个空的数据库——konga


```bash
$ node ./bin/konga.js  prepare
```
最后启动dashboard
```bash
$ npm run production
```

## 参考

官网：https://konghq.com/kong/
