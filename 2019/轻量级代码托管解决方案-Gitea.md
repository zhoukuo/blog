# 轻量级代码托管解决方案-Gitea

zhoukuo@2019-10-24

Gitea 是一个自托管的Git代码托管服务，他和GitHub，Gitlab，Bitbucket 等比较类似。

Gitea 的首要目标是创建一个极易安装，运行非常快速，安装和使用体验良好的自建 Git 服务。

项目采用 Go 作为后端语言，只要生成一个可执行程序即可。

最低的系统硬件要求为一个廉价的树莓派，如果用于团队项目，建议使用2核CPU及1GB内存。

横向对比：https://docs.gitea.io/zh-cn/comparison/

![](/img/compare.png)

## 二、Git下载安装

### 1. 卸载旧版本

```bash
yum remove git
```

### 2. 查看最新的版本

https://git-scm.com/downloads

### 3. 安装编译环境
确保安装gcc、g++以及编译git所需要的包

```bash
#安装编译所需的包
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-ExtUtils-MakeMaker
```

### 4. 下载安装Git包
```bash
#下载源码(*.tar.gz)
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.23.0.tar.gz

#解压安装包
tar zxvf git-2.23.0.tar.gz
 
#切换到指定目录 
cd git-2.23.0
 
#编译安装到指定目录
./configure --prefix=/usr/local/git && make install
 
#将编译好的目录添加到环境变量中
echo 'export PATH=$PATH:/usr/local/git/bin' >> /etc/profile
 
#使/etc/profile生效
source /etc/profile
```

### 5. 查看Git版本
```bash
git --version
```

## 三、Gitea下载安装

所有下载均包括 SQLite, MySQL 和 PostgreSQL 的支持，同时所有资源均已嵌入到可执行程序中，这一点和老版本有所不同。 基于二进制的安装非常简单，只要从 下载页面 选择对应平台，拷贝下载URL，执行以下命令即可（以Linux为例）：

```bash
wget https://dl.gitea.io/gitea/1.9.3/gitea-1.9.3-linux-amd64
chmod +x gitea-1.9.3-linux-amd64
```

## 四、初始化配置


## 五、仓库管理方式

### 方式一：按产品
- 组织 == 产品（医网信、互联网医院、点餐、病案、监管平台、电子合同、医信查、微服务），数量较多
- 团队 == 角色（服务端、移动端、前端、测试），数量较少，且固定
- 仓库 == 一个产品，服务端+移动端+前端，数量较少

### 方式二：按职能
- 组织 == 角色（服务端、移动端、前端），数量较少，且固定
- 团队 == 产品（医网信、互联网医院、点餐、病案、监管平台、电子合同、医信查、微服务），数量较多
- 仓库 == 所有产品，数量较多


## 六、人员角色
- 超级管理员：胡丹、周阔（所有权限：创建组织、删除仓库、配置管理员的所有操作）
- 配置管理员：蒲天恩（创建团队、创建仓库、为仓库添加团队、添加协作者）
- 开发人员：所有开发人员（读写仓库、添加协作者）


## 七、权限管理

### 组织权限
1. 只有超级管理员能创建组织
2. 组织可见性为受限（如果对外网开放，可见性调整为私有）
3. 不允许仓库管理员修改团队的访问权限

### 团队权限
1. 只有超级管理员、配置管理员能创建团队
2. 所有团队都分配管理员权限（添加协作者权限下放给开发人员）

### 仓库权限
1. 只有超级管理员、配置管理员能创建仓库
2. 仓库默认为私有，任何人都不可见
3. 仓库默认只有Master分支，且为受保护分支
3. 只有超级管理员、配置管理员能为仓库添加团队（团队只能是组织内的团队）
4. 团队成员都可以读写、添加协作者（个人可以是平台上的所有人）
5. 只有超级管理员有删除仓库等危险操作权限

### 用户权限
1. 用户可以自注册、也可以由超级管理员添加，超级管理员添加后默认密码为：1qaz@WSX，并强制用户更改密码
2. 用户权限只能由仓库管理员、配置管理员、超级管理员分配

## 八、数据管理

Gitea的数据包括三个部分：
- 配置数据库（sqllite3）
- 代码仓库文件
- git-lfs

### 数据存储
- db：/opt/gitea/data/gitea.db
- repo：/opt/gitea-repositories
- lfs：/opt/gitea/data/lfs
- log：/opt/gitea/log

### 数据备份
1. 代码仓库通过创建镜像仓库，定时同步的方式，进行数据的备份
2. 配置数据库通过shell脚本每天把数据库备份到gitea上
3. git-lfs暂时不使用，可以忽略

## 九、进阶配置
https://docs.gitea.io/zh-cn/config-cheat-sheet/

