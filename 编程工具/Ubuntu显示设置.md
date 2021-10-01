# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [linux系统配置x11,配置Xorg X11窗口系统](https://blog.csdn.net/weixin_33181159/article/details/116907964)
> [恢复ubuntu20.04默认桌面管理器](https://blog.csdn.net/irober/article/details/112666189)
> [Ubuntu18.04多显卡配置显示输出，指定某个独显输出图形界面，解决黑屏无法进入图形界面](https://blog.csdn.net/qq_34361099/article/details/106741432)

# 多显卡切换
一块GT-7-1-0，一块RTX-2-0-8-0，安装系统时显示器接的RTX-2-0-8-0，后面接到GT-7-1-0上也是黑屏，无法显示，
```bash
# 查看到gdm service，所以ubuntu20.04使用的gdm管理桌面
$ systemctl list-units
$ systemctl stop gdm
$ cd /etc/X11/
# 获取生成的Xorg配置文件/root/xorg.conf.new，编辑新的xorg配置文件
$ sudo Xorg -configure
$ sudo mv /root/xorg.conf.new /etc/X11/xorg.conf
$ systemctl start gdm # 成功切换
```

