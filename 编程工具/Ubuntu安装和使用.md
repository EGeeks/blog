# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [ubuntu 显示或者隐藏 grub选择菜单](http://blog.chinaunix.net/uid-26527046-id-3748986.html)
> [sudo: /etc/sudoers is world writable 错误解决方案](https://blog.csdn.net/whatday/article/details/84784494)
> [Ubuntu18.04（Gnome桌面）主题美化，Mac私人定制](https://blog.csdn.net/zyqblog/article/details/80152016)
> [Ubuntu18.04（mac主题）主题美化](https://blog.csdn.net/qq_37774171/article/details/87543614)
> [Ubuntu18.04 主题美化以及常用软件](https://www.jianshu.com/p/7d153a484f72)
> [《系统相关》Windows + Ubuntu双系统时间不一致](https://blog.csdn.net/u013162035/article/details/79151640)
> [经验体会：解决Ubuntu 18.04+Windows双系统时间不同步的问题](https://www.jianshu.com/p/cf445a2c55e8)
> [Ubuntu18.04开机动画（bootsplash）安装](https://blog.csdn.net/wobushixiaobailian/article/details/85246204)
> [ubuntu 自定义开机画面](https://blog.csdn.net/WMX843230304WMX/article/details/80254618)
> [Ubuntu16.04LTS更改开机背景和开机动画](https://blog.csdn.net/ArthurCaoMH/article/details/88093054)
> [Ubuntu18.04修改登录页面背景](https://blog.csdn.net/TalonZhang/article/details/82498114)
> [Windows 10 太难用，如何定制你的 Ubuntu？](https://blog.csdn.net/csdnnews/article/details/108093966)
> [利用logrotate 防止linux系统日志文件过大](https://blog.csdn.net/qq_26614295/article/details/79923056)
> [ubuntu系统日志的配置和使用](https://blog.csdn.net/brosnan2880/article/details/84295775)
> [logrotate - 如何防止日志变得太大？](https://www.kutu66.com/ubuntu/article_161023)


# QQ
官网下载QQ Linux版本deb包，安装，如果版本更新后登录出现闪退情况，请删除 ~/.config/tencent-qq/#你的QQ号# 目录后重新登录，
```bash
$ sudo apt install libgtk2.0-0
$ sudo apt install -y ./linuxqq_2.0.0-b2-1084_amd64.deb
```
卸载，
```bash
$ sudo dpkg -r linuxqq
```

# 华硕主板启动Ubuntu18.04
在启动里面开启CSM，存储设备选择忽略，默认是仅Legacy。

# 关闭unattended-upgrade
```bash
$ cat /etc/apt/apt.conf.d/20auto-upgrades
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "0";
```

# 限制日志大小
在`/etc/logrotate.d`新建一个logrotate的配置文件，
```bash
/var/log/* {
 #daily, hourly, weekly, monthly, yearly, maxsize, maxage
 size 10M
 rotate 10
 delaycompress
 compress
 notifempty
 missingok
}
```

# 输入法配置
> [Ubuntu系统中添加中文输入法及其快捷键的设置](https://blog.csdn.net/qq_27806947/article/details/80157419)
> [Ubuntu 安装中文输入法](https://blog.csdn.net/Chamico/article/details/89788324)
> [在Ubuntu中配置中文输入法](https://blog.csdn.net/nanhuaibeian/article/details/85851335)

安装后重启即可，
```bash
$ sudo apt install fcitx
$ sudo apt install fcitx-pinyin
$ sudo apt install fcitx-table-wbpy # 五笔
```

# Ubuntu和Windows时间差8小时
Ubuntu18.04，
```bash
$ sudo timedatectl set-local-rtc 1
$ sudo apt-get install ntpdate
$ sudo ntpdate time.windows.com
$ sudo hwclock --localtime --systohc
```
或者设置Windows，重启系统后即可生效。更改Windows时区为UTC，更改虚拟机Ubuntu时区为上海。
```bash
$ reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```

# NTFS硬盘在Linux挂载失败
挂载时报错`The disk contains an unclean file system (0, 0). Metadata kept in Windows ca`，在电源按钮功能设置界面，关掉快速启动。
![311](https://img-blog.csdnimg.cn/2020053015551449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# NTFS硬盘在Linux挂载成只读文件系统
在Linux下执行，
```bash
$ mount -t ntfs-3g -o remove_hiberfile /dev/sda4 ./win10
```

# 换源
打开`软件和更新`，可以选择阿里的。

# 美化
安装软件，
```bash
$ sudo apt install gnome-tweak-tool gnome-shell-extensions chrome-gnome-shell gtk2-engines-pixbuf libxml2-utils
```
安装浏览器扩展，进入[https://extensions.gnome.org](https://extensions.gnome.org)，
![2020-08-21-17-14-51](https://img-blog.csdnimg.cn/20200821171635258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)刷新界面，搜索安装插件`User Themes`，`Dash to Dock`，`Gnome Global Application Menu `，`TopIcons Plus`，点击开关切换到On状态即可，过一会自动弹出安装确认对话框。打开Tweaks，可用搜索的方式打开，到扩展选项卡中使能刚刚安装的插件。
到网站[https://www.gnome-look.org](https://www.gnome-look.org)下载主题包或者shell包之类的资源，然后安装。
```bash
$ mkdir .local/share/themes
$ cd .local/share/themes/
$ unzip Material-Black-Pistachio_1.8.6.zip 
$ rm Material-Black-Pistachio_1.8.6.zip 
```

# 安装wireshark
执行完下面命令后重启。
```bash
$ sudo add-apt-repository ppa:wireshark-dev/stable
$ sudo apt update
$ sudo apt-get install wireshark
# 在ubuntu18.04上会自动执行dpkg-reconfigure，不需要在执行一边了
$ sudo dpkg-reconfigure wireshark-common
# 由于您已允许非超级用户捕获数据包，因此必须将用户添加到wireshark组。 使用usermod命令将您自己添加到wirehark组
$ sudo usermod -aG wireshark $(whoami)
```

# sudo: /etc/sudoers is world writable
恢复，
```bash
$ pkexec chmod 0440 /etc/sudoers
```
添加写权限，使用vim，`wq!`写入，
```bash
$ pkexec chmod 555 /etc/sudoers
$ pkexec chmod 555 /etc/sudoers.d/README
```

# 添加桌面快捷方式
英文版操作系统桌面快捷方式路径为`~/home`，中文版桌面快捷方式路径为`~/桌面`，创建`.Desktop`文件，
```bash
qe@qe-pc:~/桌面$ cat Xilinx\ SDK\ 2018.2.desktop 
[Desktop Entry]
Encoding=UTF-8
Type=Application
Name=Xilinx SDK 2018.2
Comment=Xilinx SDK 2018.2
Icon=/opt/Xilinx/SDK/2018.2/data/sdk/images/sdk_logo.png
Exec=/opt/Xilinx/SDK/2018.2/bin/xsdk
```

# 添加Dash启动项
```bash
$ cat /usr/share/applicastions/drcom.desktop
[Desktop Entry]
Name=DrClient
Comment=DrClient
Exec=/home/haiyang/DrClient/DrClientLinux
Icon=/home/haiyang/DrClient/icon.xpm
Terminal=false
Type=Application
StartupNotify=true
Categories=Accessibility;Utility;
OnlyShowIn=Unity;
```

# 直接进入ubuntu不在grub菜单等待
修改`/etc/default/grub.cfg`文件，修改`GRUB_TIMEOUT=0`，执行`sudo update-grub`。

# 默认的切换输入法快捷键
切换到下一个输入法，
ubuntu16.04 `Ctrl + Space`
ubuntu18.04 `Super+ Space`
ubuntu20.04 `Shift`
切换到上一个输入法在上面的基础上加上`Shift`

# 更新nvdia驱动、
> [Ubuntu 18.04 安装 NVIDIA 显卡驱动](https://zhuanlan.zhihu.com/p/59618999)

不更新之前，屏幕闪屏，搜索打开`软件和更新`，附加驱动，选择安装nvdia的驱动。
卸载驱动，重新安装，
```bash
$ ubuntu-drivers devices
$ sudo apt-get remove --purge nvidia-*
$ sudo apt-get remove --purge libnvidia-*
$ sudo apt-get remove --purge libnvidia-compute-440:i386 libnvidia-decode-440:i386 libnvidia-encode-440:i386 libnvidia-fbc1-440:i386
$ sudo ubuntu-drivers autoinstall
```

# 安装工具
安装常用工具，
```bash
$ sudo apt install tftp-hpa tftpd-hpa dos2unix iproute2 gawk xvfb git make net-tools libncurses5-dev zlib1g-dev libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev screen pax gzip zlib1g:i386 minicom u-boot-tools mtd-utils memtool devmem2
```

