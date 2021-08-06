# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [wsl的GUI程序（wslg项目的使用）](https://www.bilibili.com/read/cv11041040)
> [微软WSLG(WSL GUI)安装](https://blog.csdn.net/weixin_43860494/article/details/116118008)
> [microsoft/wslg](https://github.com/microsoft/wslg)
> [Getting started with the Windows Insider Program](https://insider.windows.com/en-us/getting-started#flight)

# 安装
搜索打开`启用或关闭Windows功能`，开启`适用于Linux的Windows子系统`，WSL2还需开启`虚拟机平台`，
![1](https://img-blog.csdnimg.cn/2021071910270655.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
或者采用命令行，
```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```
搜索打开`Microsoft Store`，
![2](https://img-blog.csdnimg.cn/20210719090331468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
从开始菜单打开Ubuntu程序，进入终端后，输入用户名和密码，安装完毕，本地磁盘目录在`/mnt`上，
```bash
qe@DESKTOP-0VGL0B0:~$ ls /mnt/
c/ d/ e/
```
默认安装的是WSL，不是WSL2，
```bash
PS C:\Users\jingjiamicro> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu-18.04    Running         1
```
更新软件源，
```bash
$ sudo vi /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ bionic universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates universe
deb http://mirrors.aliyun.com/ubuntu/ bionic multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-security universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-security multiverse
```
配置代理，
```bash
$ export http_proxy=http://192.168.1.60:3333
$ export https_proxy=http://192.168.1.60:3333
$ sudo -E apt update # 必须执行
```
升级WSL2，安装内核更新包`wsl_update_x64.exe`，执行命令升级之前创建的虚拟机，
```bash
wsl.exe --set-version Ubuntu-18.04 2
```

# 共享文件
- WSL访问Windows，`/mnt`
- Windows访问WSL，`cd`到WSL主目录，`explore.exe .`

# U盘
U盘不会自动挂载，
```bash
$ sudo mkdir /mnt/zcu104
$ sudo mount -t drvfs F: /mnt/zcu104/
```
Windows弹出之前，需要先卸载，
```bash
$ sudo umount /mnt/zcu104
```

# gWSL
版本`Windows 10 Insider Preview build 21362+`才支持gWSL，点击检查更新，更新21H1，
![12](https://img-blog.csdnimg.cn/92816655ac7146b1bafcff0981896674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
更新完之后，再点击检查更新，安装[NVIDIA GPU driver for WSL](https://developer.nvidia.com/cuda/wsl)，

# 远程桌面
- 安装`VcXsrv`或`Xming`，到安装路径`C:\Program Files\VcXsrv`
- 对两个可执行文件vcxsrv.exe和xlaunch.exe执行操作`右键点击可执行文件 –> “属性” –> “兼容性” – > “更改高DPI设置” –> “替代高DPI缩放行为”`。
- `xlaunch`勾选`Disable access control`，其他默认，开启Xserver
- 在WSL中执行，`sudo apt install xfce4 xfce4-goodies`
- 在WSL1中执行`export DISPLAY=:0.0`或`echo 'export DISPLAY=:0.0' >> .profile`，在WSL2中执行`export DISPLAY=$(grep -m 1 nameserver /etc/resolv.conf | awk '{print $2}'):0`
- 安装`sudo apt install xeyes`用于测试

