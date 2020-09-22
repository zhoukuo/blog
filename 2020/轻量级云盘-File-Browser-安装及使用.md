# 轻量级云盘 File Browser 安装及使用

zhoukuo@2020-06-03

File Browser 是一个基于 Web 的文件管理器。它可以使你随时随地的对设备的文件进行基本的管理操作，如：创建、删除、移动、复制等。它除了可以让你进行文件管理之外，还有一些其他的功能。它支持多个用户的管理，而且每个用户可以拥有自己可以访问的文件和权限。它还支持文件分享，就像网盘那样，你可以通过它来向你的朋友分享文件。你还可以用它来执行一些 Linux 命令，比如你想要在当前目录下克隆一个代码库，就可以用它来执行git等命令。

![](https://cdn.mivm.cn/www.mivm.cn/archives/filebrowser/File-Browser.png)

## 安装及配置

File Browser 适用于全平台，任何操作系统都可以安装它，当然，我会以 Linux 为主。

File Browser 在 Linux 的安装非常简单，只需要一条命令就可以搞定：curl -fsSL https://filebrowser.xyz/get.sh | bash，你也可以手动下载可执行文件进行安装：https://github.com/filebrowser/filebrowser/releases/latest

当安装好之后，你并不能立即使用它，需要修改一些配置。

以下设置方法只适用 File Browser 2.0 +

```
创建配置数据库：filebrowser -d /etc/filebrowser.db config init

设置监听地址：filebrowser -d /etc/filebrowser.db config set --address 0.0.0.0

设置监听端口：filebrowser -d /etc/filebrowser.db config set --port 8088

设置语言环境：filebrowser -d /etc/filebrowser.db config set --locale zh-cn

设置日志位置：filebrowser -d /etc/filebrowser.db config set --log /var/log/filebrowser.log

添加一个用户：filebrowser -d /etc/filebrowser.db users add root password --perm.admin，
            其中的root和password分别是用户名和密码，根据自己的需求更改。
```

有关更多配置的选项，可以参考官方文档：https://docs.filebrowser.xyz/

配置修改好以后，就可以启动 File Browser 了，使用-d参数指定配置数据库路径。示例：filebrowser -d /etc/filebrowser.db

启动成功就可以使用浏览器访问 File Browser 了，在浏览器输入 IP:端口，示例：http://192.168.1.1:8088

然后会看到 File Browser 的登陆界面，用刚刚创建的用户登陆。

登陆以后，默认会看到 File Browser 运行目录下的文件，需要更改一下当前用户的文件夹位置。

点击 [设置] → [用户设置] → 编辑用户 admin → 将目录范围改为你想要显示的文件夹，例如：/mnt → 修改完成后点击最下方的保存即可。

这样，File Browser 的基本安装和配置就搞定了。

![](https://cdn.mivm.cn/www.mivm.cn/archives/filebrowser/01.jpg)

## 常见问题

### 后台运行

File Browser 默认是前台运行，如何让它后台运行呢？

第一种是 nohup 大法：

```
运行：nohup filebrowser -d /etc/filebrowser.db >/dev/null 2>&1 &

停止运行：kill -9 $(pidof filebrowser)

开机启动：sed -i '/exit 0/i\nohup filebrowser -d \/etc\/filebrowser.db >\/dev\/null 2>&1 &' /etc/rc.local

取消开机启动：sed -i '/nohup filebrowser -d \/etc\/filebrowser.db >\/dev\/null 2>&1 &/d' /etc/rc.local

```
第二种是 systemd 大法：

首先下载 File Browser 的 service 文件：curl https://cdn.mivm.cn/www.mivm.cn/archives/filebrowser/filebrowser.service -o /lib/systemd/system/filebrowser.service

如果你的运行命令不是/usr/local/bin/filebrowser -d /etc/filebrowser.db，需要对 service 文件进行修改，将文件的 ExecStart 改为你的运行命令，更改完成后需要输入systemctl daemon-reload。

```
运行：systemctl start filebrowser.service

停止运行：systemctl stop filebrowser.service

开机启动：systemctl enable filebrowser.service

取消开机启动：systemctl disable filebrowser.service

查看运行状态：systemctl status filebrowser.service
```

我推荐使用 systemd 的方法来后台运行，当然，前提是你所使用的操作系统支持 systemd。

### HTTPS

File Browser 2.0 起开始内建 HTTPS 支持，只需要配置 SSL 证书即可。

配置 SSL：filebrowser -d /etc/filebrowser.db config set --cert example.com.crt --key example.com.key，其中example.com.crt和example.com.key分别是 SSL 证书和密钥路径，根据自身情况进行更改。配置完 SSL 后，只可以使用 HTTPS 访问，不可以使用 HTTP。

取消 SSL：filebrowser -d /etc/filebrowser.db config set --cert "" --key ""

当然，你也可以使用 Nginx 等 Web 服务器对 File Browser 进行反向代理，以达到 HTTPS 访问的目的。

还有就是使用 Caddy，这是一个开源、支持 HTTP/2 的 Web 服务器，它的一个显著特点就是默认启用 HTTPS 访问，会自己申请 SSL 证书，同时支持大量的插件，File Browser 就可以作为其插件运行。

### 外网访问

每个人的情况不同，外网访问的配置方法也不一样。

如果你有公网 IP 地址，不管是 v4 还是 v6，在防火墙上打开相应的端口以及设置好端口转发即可。

如果你没有公网IP地址，那么你想要外网访问可能就需要内网穿透了，可以参考我之前写的文章：《OpenWrt 使用 frp 实现内网穿透》


好了，以上就是 File Browser 在 Linux 系统上安装以及使用的方法，有关于更多的问题，欢迎加入 QQ 群与我探讨。



