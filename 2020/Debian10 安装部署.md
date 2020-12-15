debian10 安装部署

1. 安装时不要设置IP，防止安装时自动更新，时间太长

2. 安装后手动设置静态IP

```
auto ens3
iface ens3 inet static
address 192.168.126.203
netmask 255.255.255.0
gateway 192.168.126.254
```

3. 修改源：把默认的cdrom注释，sudo vi /etc/apt/sources.list

4. 控制台中文乱码


docker run -d -v /data/rancher/mysql:/var/lib/mysql --restart=always -p 8080:8080 rancher/server