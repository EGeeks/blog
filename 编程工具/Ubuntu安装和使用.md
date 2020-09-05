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

# Ubuntu和Windows时间差8小时
Ubuntu18.04，
```bash
$ sudo timedatectl set-local-rtc 1
$ sudo apt-get install ntpdate
$ sudo ntpdate time.windows.com
$ sudo hwclock --localtime --systohc
```
或者设置Windows，重启系统后即可生效。
```bash
$ reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
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
添加写权限，
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
