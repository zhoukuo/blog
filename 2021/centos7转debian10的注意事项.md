
# centos7转debian10的注意事项
zhoukuo@20210308

### 0. 地区选择中国，安装后控制台乱码，怎么解决？
忽略，通过远程ssh访问不受影响

### 1. 无法ssh远程登录

```bash
# 安装ssh服务
>apt install ssh
# 允许通过密码登录
PasswordAuthentication yes
```bash

### 2. 重启系统后无法ssh

```bash
systemctl enable ssh
```

### 3. 重启命令找不到

```bash
>/sbin/reboot
```

### 4. 查看系统版本

```bash
>cat /etc/debian_version
10.8
```
### 5. apt 和apt-get的区别
https://blog.csdn.net/liudsl/article/details/79200134

### 6. 查看IP

```bash
ip a|grep 192 | awk '{print $2}' | awk -F/ '{print $1}'
```

### 7. 修改ip

```bash
vi /etc/network/interfaces
```

### 8. 使用restart networking 无法重启网络服务

```bash
auto ens3
```

### 9. 系统更新

```bash
apt update && apt full-upgrade -y
```

### 10. 更新源

```bash
/etc/apt/source.list
```

### 11. 怎么配置防火墙

```bash
apt install nftables
```

### 12. apt purge 和 apt remove 区别

```bash
apt remove 保留配置文件
```

### 13. debian 10 默认未安装sudo，通过su切换用户


