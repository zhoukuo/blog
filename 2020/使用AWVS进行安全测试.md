# 使用AWVS进行安全测试

Acunetix Web Vulnerability Scanner（AWVS）是一款知名的Web网络漏洞扫描工具，它通过网络爬虫测试你的网站安全，检测流行安全漏洞，最近更新到了13的版本。

## 安装部署

AWVS支持windows、linux、macos等多种操作系统，但个人更推荐linux，部署方式上Docker最方便，部署起来也很简单，建议大家直接使用Docker版本。

```bash
#  pull 拉取下载镜像
docker pull secfa/docker-awvs

#  将Docker的3443端口映射到物理机的 13443端口
docker run -it -d -p 13443:3443 secfa/docker-awvs

# 容器的相关信息
awvs13 username: admin@admin.com
awvs13 password: Admin123
AWVS版本：13.0.200217097
```
浏览器访问：https://127.0.0.1:13443/ 即可。


## 使用

#### 第一步：添加目标

#### 第二布：新建扫描任务（扫描类型、报告类型、计划）

#### 第三步：查看报告


