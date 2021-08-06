# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [vnc移植成功 VNC移植到arm开发板（4.1.3）](https://blog.csdn.net/guoqianqian5812/article/details/47167537)
> [关于TigerVNC编译安装](https://www.jianshu.com/p/2fcd0854050e)
> [TigerVNC官网](https://tigervnc.org/)
> [Ubuntu18.04 vnc4server 配置](https://zhuanlan.zhihu.com/p/75654743)
> [使用 VNC 显示 Ubuntu Server 的图形化界面](https://www.jianshu.com/p/04892f67b9df)
> [Ubuntu18.04 LTS 安装 VNC Server](https://blog.csdn.net/yidichaxiang/article/details/96429007)
> [Ubuntu如何安装vncserver](https://www.cnblogs.com/ningmengcaokanyu/p/10185457.html)
> [Linux 上安装配置 VNC Server](https://blog.csdn.net/llag_haveboy/article/details/84960479)
> [Linux下开启VNCserver服务（远程连接）](https://blog.51cto.com/13043516/2055574)
> [Ubuntu16.04 远程桌面连接（VNC-viewer）](https://blog.csdn.net/qq_27009517/article/details/86308611)
> [实现Windows直接远程访问Ubuntu桌面和解决VNC连接Ubuntu桌面灰色的问题解决](https://blog.csdn.net/sinat_28371057/article/details/109288076)
> [Download VNC® Viewer](https://www.realvnc.com/en/connect/download/viewer/)
> [ubuntu下使用vnc viewer](https://www.cnblogs.com/penny772866/p/5927796.html)
> [Ubuntu18.04LTS安装TigerVNC](https://blog.csdn.net/fjmsonic/article/details/104366421)
> [如何在 Ubuntu 18.04 上安装和配置 VNC](https://www.linuxidc.com/Linux/2019-08/159849.htm)
> [wsl使用可视化界面_启用Windows10的Linux子系统并安装图形界面](https://blog.csdn.net/weixin_39797393/article/details/113016351)

# VNC Viewer
安装深信服的软件，垃圾啊，请注意，
> [Mac 上移除 EasyConnect 常驻后台进程](https://www.dazhuanlan.com/2020/01/06/5e12b412bbd0c/)
> [警惕Easy Connect，此工具会偷偷在后台上传数据包！](https://www.jianshu.com/p/3e81fa164fb1)

```bash
$ sudo apt install ./EasyConnect_x64.deb 
```
提示不兼容，弹出浏览器下载安装兼容版本，
```bash 
$ sudo apt remove easyconnect
$ sudo apt install ./EasyConnect_x64_7_6_7_3.deb 
```
下载安装VNC Viewer，
```bash 
$ sudo apt install ./VNC-Viewer-6.20.529-Linux-x64.deb 
```
关闭深信服，
```bash
➜  ~ systemctl EasyMonitor.service  stop
Unknown operation EasyMonitor.service.
➜  ~ systemctl EasyMonitor.service      
➜  ~ systemctl list-unit-files | grep EasyMonitor.service
EasyMonitor.service                        enabled        
➜  ~ systemctl  stop EasyMonitor.service
➜  ~ systemctl  stop EasyMonitor.service
```

# VNC Server
进入设置，打开共享，开启桌面共享，没有的话在Ubuntu应用商店搜索安装`桌面共享`，
安装软件，
```bash
$ sudo apt install xrdp vnc4server xbase-clients dconf-editor
```
打开`dconf-editor`搜索`remote-access`关闭，`requlre-encryption`，使用mstsc远程桌面，输入IP，不输入用户名，连接，选择vnc-any协议，输入IP，密码登录即可。

## xfce
```bash
# 安装xfce
$ sudo -E apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils
# 关闭防火墙
$ sudo ufw disable
$ sudo ufw status
# 安装vncserver
$ sudo -E apt install tigervnc-standalone-server tigervnc-common tigervnc-xorg-extension
# 运行vncserver，第一次运行会要求设置密码，这里自动选择端口
$ vncserver
# 或vncpasswd设置密码，最少6位
$ vncpasswd
Passwork:12346
Verify:123456
Would you like to enter a view-only password  (y/n)? n
# 编辑xstartup
$ sudo vim ~/.vnc/xstartup
$ cat ~/.vnc/xstartup
#!/bin/sh

unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
startxfce4 &    #启动xface4
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey    #设置背景色
$ sudo chmod u+x ~/.vnc/xstartup
# 启动vnc，:2是用了5902端口，-localhost no是为了让其他计算机可以访问，而不仅仅是本地。
$ sudo vncserver :1 -localhost no
# 监听地址是0.0.0.0，而不是默认的127.0.0.1
$ sudo netstat -ntupl|grep vnc
# vnc列表
$ vncserver -list
# 关闭vncserver
$ sudo vncserver -kill :1
$ sudo vncserver -kill :*
```

# 问题
登录服务器查看发现端口是2，3，之前用1登录的，现在登不进去了，用2登录黑屏，用3登录就可以了，
```bash
# ps -ef | grep vnc
root      6445     1  0 11月16 ?      00:00:00 /usr/bin/Xvnc :2 -auth /root/.Xauthority -desktop localhost.localdomain:2 (root) -fp catalogue:/etc/X11/fontpath.d -geometry 1024x768 -pn -rfbauth /root/.vnc/passwd -rfbport 5902 -rfbwait 30000
root      6933     1  0 11月16 ?      00:00:12 /usr/bin/Xvnc :3 -auth /root/.Xauthority -desktop localhost.localdomain:3 (root) -fp catalogue:/etc/X11/fontpath.d -geometry 1024x768 -pn -rfbauth /root/.vnc/passwd -rfbport 5903 -rfbwait 30000
root      7020     1  0 11月16 ?      00:00:00 /bin/sh /root/.vnc/xstartup
root      9652 16975  0 17:00 pts/2    00:00:00 grep --color=auto vnc
```

