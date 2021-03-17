# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# Xilinx Zynq介绍
## 芯片架构

## Arm CPU

## PL可编程逻辑

# 开发环境搭建
首先Xilinx开发arm cpu的工具有两种：裸机开发和嵌入式linux开发。如果不跑linux操作系统，那么就用vitis就够了，如果跑linux操作系统，那么则需要xilinx的petalinux工具，vitis支持linux和windows两种操作系统，但是petalinux只支持linux操作系统。比如petalinux-v2019.2的最小安装配置是，
1. 8GB内存
2. 2GHz的CPU，最少8核
3. 100GB磁盘空间
4. 支持的操作系统：
Red Hat Enterprise Workstation/Server 7.2, 7.3, 7.4 (64-bit)
CentOS 7.2, 7.3, 7.4 (64-bit)
Ubuntu Linux 16.04.3 (64-bit)

所以，如果你的电脑是linux操作系统，那么直接安装vitis和petalinux就可以了。如果你的电脑是windows操作系统，通过安装虚拟机软件，可以运行linux操作系统，在虚拟机的linux操作系统上安装petalinux，对于win10的用户还可以选择WSL/WSL2来代替虚拟机软件，常用的虚拟机软件有vmware和visualbox，我们选择vmware workstation player，[点击下载vmware workstation player](https://www.vmware.com/cn/products/workstation-player.html)，选择最新版下载即可，这个软件对个人用户是免费的，常用功能也没有阉割。linux操作系统我们使用ubuntu，版本16.04.x，[点击下载ubuntu](http://mirrors.aliyun.com/ubuntu-releases/16.04/)，选择`ubuntu-16.04.x-desktop-amd64.iso `下载，我最后的安装环境是，

1. win10
2. vmware workstation player
3. ubuntu16.04.6
4. 虚拟机配置：4核+8GB内存
5. 200GB磁盘

开发环境搭建教学视频链接：[点我](https://www.bilibili.com/video/av96898695)

## Xilinx Vitis安装
Vitis安装比较简单，双击安装文件，安装即可。Vitis最早从2019.2开始引入，以前是没有这个软件的，以前只有Vivado，现在Vitis把Vivado包含进去了，提供了更丰富的功能。

## 嵌入式Linux开发环境安装
准备软件VMware Workstation Player，Uubuntu16.04.x的镜像ISO文件，Petalinux安装文件。
**1. 在虚拟机上安装ubuntu**
安装虚拟机和创建Ubuntu虚拟机请参考博客[VMware安装和使用](https://blog.csdn.net/Zhu_Zhu_2009/article/details/80891427)。
**2. 安装petalinux**
首先在虚拟机上安装Petalinux的依赖软件，
```bash
$ sudo apt install ssh make tftp-hpa tftpd-hpa dos2unix iproute2 gawk xvfb git make net-tools libncurses5-dev zlib1g-dev libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev screen pax gzip zlib1g:i386 minicom u-boot-tools mtd-utils
```
开始安装Petalinux，比如安装在路径`/opt/pkg/petalinux-v2019.2-final`，可执行下面操作，
```bash
$ sudo mkdir -p /opt/pkg/petalinux-v2019.2-final
$ sudo chmod 777 -Rf /opt
$ ./petalinux-v2019.2-final-installer.run /opt/pkg/petalinux-v2019.2-final
```
**问题记录**
`awk: read error (Bad address)`，由于网络不好，依赖的软件包没有全部安装好，解决办法：重新执行安装命令。
`/bin/sh is not bash`，在ubuntu上会出现这个警告，执行`sudo dpkg-reconfigure dash`，在弹出的界面中选`No`。

# 配置CPU和开发PL逻辑功能
配置CPU和开发PL逻辑功能需要使用Xilinx Vivado创建工程，而Vivado在安装Vitis时会自动安装。

# 裸机软件开发
裸机软件开发需要使用Vitis（2019年之前是使用Xilinx SDK）创建工程。

# Linux开发
Linux开发需要使用Petalinux工具，使用Petalinux工具之前需要打开中断，执行命令，
```bash
$ source /opt/Xilinx/Petalinux/2018.2/settings.sh
```

## 创建工程
1. 如果你使用自己开发的Vivado工程，那么你使用的是自定义硬件，参考博客[Xilinx Petalinux安装和使用](https://blog.csdn.net/Zhu_Zhu_2009/article/details/87692304)。

2. 如果你使用官方下载BSP或者第三方提供的BSP，将bsp文件拷贝到虚拟机，执行下面命令，例如，
```bash
$ petalinux-create -t project -s zynq-v2018.2.bsp
```

## 编译工程
进入工程路径，执行命令，例如，
```bash
$ cd zynq-v2018.2/
$ petalinux-build
```

