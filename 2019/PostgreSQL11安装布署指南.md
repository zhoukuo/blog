# PostgreSQL11安装布署指南

zhoukuo@2019-10-14

## 安装
在Centos7上最简单的安装方式是使用Yum在线安装

### 通过官网得到仓库的配置：

https://www.postgresql.org/download/linux/redhat/

### 下载安装：
```bash
$ yum install yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
$ yum install postgresql11-server
```

### 初始化并启动pgsql：
```bash
/usr/pgsql-11/bin/postgresql-11-setup initdb
systemctl enable postgresql-11
systemctl start postgresql-11
```

## 配置

### 修改管理员密码
登录服务器

使用管理员用户
```bash
$ su postgresql
bash-4.2$ psql 
postgres=# alter user postgres with password '******';
ALTER ROLE
```

### 允许远程连接
默认安装的postgres数据库配置只监控本地地址（localhost），其他主机是无法访问的，
这里通过一个简单的例介绍远程主机连接方式。

配置pg_hba.conf文件
```bash
$ vi /var/lib/pgsql/11/data/pg_hba.conf

```
在# IPv4 local connections:

下面添加一行，内容为  "host  all  all  192.168.40.1/24  md5"

```bash
$ vi /var/lib/pgsql/11/data/postgresql.conf
#新增
listen_addresses = '*'

```

重启服务
```bash
$ systemctl restart postgresql-11.service
```

### 开启SSL
```bash
$ cd /var/lib/pgsql/11/data
# 创建server.key文件
$ openssl genrsa -des3 -out server.key 1024
# 过程中回要求输入密码，随便设置一个即可，然后可使用以下命令删除设置的密码：
$ openssl rsa -in server.key -out server.key
# 创建基于server.key文件的服务器证书
$ openssl req -new -key server.key -days 3650 -out server.crt -x509
# 为了得到自己签名的证书，把生成的服务器证书作为受信任的根证书，复制并取一个合适的名字
$ copy server.crt root.crt
# 配置postgres.conf和pg_hba.conf
# 修改postgres.conf,修改ssl = on 配置ssl_ca_file = 'root.crt'
# 配置pg_hba.conf,新增ssl连接认证规则
$ hostssl all all 0.0.0.0/0	trust
# 重启数据库，规则生效
$ systemctl restart postgresql-11.service
```

### 添加用户和数据库

```bash
$ CREATE USER kong; CREATE DATABASE kong OWNER kong;
```
