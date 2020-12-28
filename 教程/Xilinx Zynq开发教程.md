QQ技术交流群：852283276
B站教学视频合集：[点我](https://www.bilibili.com/video/av96898695)
# xilinx zynq介绍
## 芯片架构

## arm cpu

## pl可编程逻辑部分

# 开发环境搭建
首先Xilinx开发arm cpu的工具有两种：裸机开发和嵌入式linux开发。如果不跑linux操作系统，那么就用vitis就够了，如果跑linux操作系统，那么则需要xilinx的petalinux工具，vitis支持linux和windows两种操作系统，但是petalinux只支持linux操作系统。比如petalinux-v2019.2的最小安装配置是，
1. 8GB内存
2. 2GHz的CPU，最少8核
3. 100GB磁盘空间
4. 支持的操作系统：
Red Hat Enterprise Workstation/Server 7.2, 7.3, 7.4 (64-bit)
CentOS 7.2, 7.3, 7.4 (64-bit)
Ubuntu Linux 16.04.3 (64-bit)

所以，如果你的电脑是linux操作系统，那么直接安装vitis和petalinux就可以了。如果你的电脑是windows操作系统，通过安装虚拟机软件，可以运行linux操作系统，在虚拟机的linux操作系统上安装petalinux，对于win10的用户还可以选择WSL/WSL2来代替虚拟机软件，常用的虚拟机软件有vmware和visualbox，我们选择vmware workstation player，[点击下载vmware workstation player](https://www.vmware.com/cn/products/workstation-player.html)，选择最新版下载即可，这个软件对个人用户是免费的，常用功能也没有阉割。linux操作系统我们使用ubuntu，版本16.04.7，[点击下载ubuntu](http://mirrors.aliyun.com/ubuntu-releases/16.04/)，选择`ubuntu-16.04.7-desktop-amd64.iso `下载，我最后的安装环境是，

1. win10
2. vmware workstation player
3. ubuntu16.04.7
4. 虚拟机配置：4核+8GB内存
5. 200GB磁盘

开发环境搭建教学视频链接：[点我](https://www.bilibili.com/video/av96898695)

## vitis安装
比较简单，双击安装文件，安装即可。

## 嵌入式linux开发环境安装
准备软件vmware workstation player，ubuntu16.04.7的镜像ISO文件，petalinux安装文件。
**1. 在虚拟机上安装ubuntu**
安装虚拟机和创建ubuntu虚拟机请参考博客[VMware安装和使用](https://blog.csdn.net/Zhu_Zhu_2009/article/details/80891427)。
**2. 安装petalinux**
首先在虚拟机上安装petalinux的依赖软件，
```bash
$ sudo apt-get install tftp-hpa tftpd-hpa dos2unix iproute2 gawk xvfb git make net-tools libncurses5-dev zlib1g-dev libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev screen pax gzip zlib1g:i386 minicom u-boot-tools mtd-utils ssh make
```
开始安装petalinux，比如安装在路径`/opt/pkg/petalinux-v2019.2-final`，可执行下面操作，
```bash
$ sudo mkdir -p /opt/pkg/petalinux-v2019.2-final
$ sudo chmod 777 -Rf /opt
$ ./petalinux-v2019.2-final-installer.run /opt/pkg/petalinux-v2019.2-final
```
**问题记录**
`awk: read error (Bad address)`，由于网络不好，依赖的软件包没有全部安装好，解决办法：重新执行安装命令。
`/bin/sh is not bash`，在ubuntu上会出现这个警告，执行`sudo dpkg-reconfigure dash`，在弹出的界面中选`No`。

# 搭建vivado工程
> [https://china.xilinx.com/support/answers/53051.html](https://china.xilinx.com/support/answers/53051.html)
> [从零开始，搭建zynq-7000的PS硬件平台--DDR3接口集成与配置](https://forums.xilinx.com/t5/%E5%B5%8C%E5%85%A5%E5%BC%8F-%E7%A1%AC%E4%BB%B6%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91/%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B-%E6%90%AD%E5%BB%BAzynq-7000%E7%9A%84PS%E7%A1%AC%E4%BB%B6%E5%B9%B3%E5%8F%B0-DDR3%E6%8E%A5%E5%8F%A3%E9%9B%86%E6%88%90%E4%B8%8E%E9%85%8D%E7%BD%AE/m-p/295051#M21)

颗粒的速度等级为多少，MT41K256M16TW，
![2020-10-24 00-30-00](https://img-blog.csdnimg.cn/20201024003312403.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)DDR时钟533.33MHz，下面两组参数DQS to Clock Delay(ns)和Board Delay(ns)需要用PCB软件计算出来。DQS to Clock Delay(ns)参数范围-0.1~100，Board Delay(ns)参数范围0.007～100，
![2020-10-24 00-41-04](https://img-blog.csdnimg.cn/20201024004130740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)

