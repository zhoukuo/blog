# UBUNTU 20.04 安装后要做哪些事情

zhoukuo@2020-04-26

最近 Ubuntu 发布了 20.04 LTS 版本，我也在第一时间安装体验。由于各种 Linux 发行版本并不像 MacOS、Windows 一样开箱即用，因此需要做很多配置。每次配置都需要查阅各种资料，虽然网络上有很多配置文章，但基本上都会存在一些问题。

```
安装搜狗拼音：sudo apt install fcitx-bin;sudo apt-get install fcitx-table;sudo dpkg -i sogoupinyin*.deb
参考：https://www.linuxprobe.com/ubuntu20-04-install-sogou.html
```
```
dock显示放到底部： 设置->外观->屏幕上位置
```
```
启用dock最小化功能：gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'
```
```
wifi驱动安装：软件和更新-> 附加驱动 -> 使用 realtek 8821C PICe WIFI driver
```
```
安装chrome浏览器： https://www.jianshu.com/p/3d02abeb6d15
```
```
安装邮件客户端：https://blog.csdn.net/gatieme/article/details/78174372
```
```
安装截图标记工具（火焰截图）：sudo apt install flameshot
```
```
安装wps：https://blog.csdn.net/qq_35451572/article/details/85856239#_15
```
```
安装team viewer: https://www.teamviewer.com/en/download/linux/
```
```
安装git: sudo apt install git
```
```
安装sublime text3：http://www.sublimetext.com/docs/3/linux_repositories.html#apt
```
```
安装gosublime插件：https://www.jianshu.com/p/333fd9de6de5
```
```
安装golang：sudo add-apt-repository ppa:longsleep/golang-backports;sudo apt-get update;sudo apt intall golang-go
```
```
安裝vscode：https://code.visualstudio.com
```
```
重新安装vim:  sudo apt-get remove vim-common;sudo apt-get install vim
```
```
安装vmm: sudo apt install virt-manager -y;sudo apt install ssh-askpass-gnome -y
```
