# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Open Source: NTFS-3G](https://www.tuxera.com/community/open-source-ntfs-3g/)
> [在linux中如何解压.tgz](https://blog.csdn.net/weixin_40533355/article/details/80473223)
> [RedHat 挂载 NTFS](https://www.cnblogs.com/masterSoul/p/7567215.html)
> [RHEL7配置中文输入法-智能拼音](https://www.cnblogs.com/hi-frank/p/7355187.html)
> [redhat下软件安装](https://blog.csdn.net/zxy15771771622/article/details/78419555)
> [centos官方源](http://vault.centos.org/centos)
> [centos阿里源](https://mirrors.aliyun.com/centos/)
> [CentOS7下内核源码下载及编译步骤](https://blog.csdn.net/xh_xinhua/article/details/71629747)
> [初始值设置项里有未知的字段ndo_change_mtu](https://blog.csdn.net/tianyuzhixina/article/details/102496733)
> [Centos 7.0设置/etc/rc.local无效问题解决](https://www.cnblogs.com/wangshuyi/p/6599317.html)
> [CentOS7查看和关闭防火墙](https://blog.csdn.net/ytangdigl/article/details/79796961)
> [配置本地yum源找不到repomd.xml的解决方法](https://blog.csdn.net/csdn_kerrsally/article/details/78952503)
> [CentOS安装相应版本的内核源码](https://www.cnblogs.com/wanpengcoder/p/11768483.html)
> [Centos7下载linux内核源码](https://blog.csdn.net/wh_computers/article/details/114272949)
> [CentOS8 安装epel 使用阿里云镜像](https://www.cnblogs.com/kate7/p/13372624.html)
> [centos wiki I_need_the_Kernel_Source](https://wiki.centos.org/zh/HowTos/I_need_the_Kernel_Source)
> [阿里云Centos镜像源和EPEL源](https://developer.aliyun.com/article/500286)
> [构建CentOS dwarves包](https://blog.csdn.net/weixin_46949825/article/details/113790325)

# 安装系统
Centos7和Redhat7同版本之间是兼容的，但是不确定Centos7.4和Centos7.7之间是否兼容，而目前国内的源上已经无法下载老版本的Centos7只有最新的`CentOS-7-x86_64-DVD-1908.iso`或者Centos8。Centos7.4的版本号，我安装的时候选择的是`开发版本`，
```shell
$ uname -a
Linux localhost.localdomain 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
$ cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core)
$ cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core) 
```

# 内核
下载对应内核源码`3.10.0-693.el7.x86_64`，所有包的源码包rpm路径为`http://vault.centos.org/centos/7.4.1708/os/Source/SPackages/`。
下载Centos8内核源码，`https://vault.centos.org/8.3.2011/BaseOS/Source/SPackages/kernel-4.18.0-193.el8.src.rpm`
```bash
[   ]	kernel-4.18.0-193.1.2.el8_2.src.rpm	2020-06-04 13:35	110M	 
[   ]	kernel-4.18.0-193.6.3.el8_2.src.rpm	2020-06-10 12:36	110M	 
[   ]	kernel-4.18.0-193.14.2.el8_2.src.rpm	2020-07-29 22:04	110M	 
[   ]	kernel-4.18.0-193.19.1.el8_2.src.rpm	2020-09-16 20:44	110M	 
[   ]	kernel-4.18.0-193.28.1.el8_2.src.rpm	2020-10-22 03:00	110M	 
[   ]	kernel-4.18.0-193.el8.src.rpm	2020-05-29 15:54	110M	 
[   ]	kernel-4.18.0-240.1.1.el8_3.src.rpm	2020-11-19 23:52	113M	 
[   ]	kernel-4.18.0-240.10.1.el8_3.src.rpm	2021-01-19 19:59	113M	 
[   ]	kernel-4.18.0-240.15.1.el8_3.src.rpm	2021-03-02 16:27	113M	 
[   ]	kernel-4.18.0-240.22.1.el8_3.src.rpm	2021-04-08 20:13	113M	 
[   ]	kernel-4.18.0-240.el8.src.rpm	2020-09-28 16:45	113M
```
安装，在`/root/rpmbuild/SOURCES`有一个压缩包`linux-4.18.0-240.el8.tar.xz`即为内核源码包，
```bash
# groupadd mockbuild
# useradd mockbuild -g mockbuild
# rpm -ivh kernel-4.18.0-240.el8.src.rpm 
Updating / installing...
   1:kernel-4.18.0-240.el8            ################################# [100%]
[root@localhost ~]# ls -l /root/rpmbuild/SOURCES/
-rw-rw-r--. 1 mockbuild mockbuild 112560684 Sep 25  2020 linux-4.18.0-240.el8.tar.xz
```
编译，
```bash
# yum install rpm-build asciidoc audit-libs-devel binutils-devel bison dwarves elfutils-devel flex gcc git java-devel kabi-dw libbabeltrace-devel libbpf-devel libcap-devel libcap-ng-devel llvm-toolset m4 make ncurses-devel newt-devel nss-tools numactl-devel openssl-devel pciutils-devel perl perl-devel perl-generators pesign python3-devel python3-docutils xmlto xz-devel zlib-devel
No match for argument: dwarves
No match for argument: libbabeltrace-devel
No match for argument: libbpf-devel
Error: Unable to find a match: dwarves libbabeltrace-devel libbpf-devel
# cd /root/rpmbuild/SPECS/
# rpmbuild -bp --target=$(uname -m) kernel.spec
# cd /root/rpmbuild/BUILD/kernel-4.18.0-240.el8/linux-4.18.0-240.el8.x86_64
# cp /boot/config-4.18.0-240.el8.x86_64 .config
# make -j8
```

# 安装软件
- 中文输入法
- 可挂载ntfs格式文件系统

中文输入法用于百度搜索，ntfs挂载后可以把rhel的iso烤出来，下一步配置本地软件源。
```shell
$ ./configure 
$ make && make install
$ mkdir zc
$ mount -t ntfs-3g /dev/sdb1 ./zc
$ cp rhel-server-7.4-x86_64-dvd.iso  ../
$ mkdir rhel-7.4-yum
$ mount rhel-server-7.4-x86_64-dvd.iso ./rhel-7.4-yum
$ ls /etc/yum.repos.d/
$ sudo mkdir /etc/yum.repos.d/backup
$ sudo mv /etc/yum.repos.d/*.repo backup
$ vim /etc/yum.repos.d/yum.repo
$ cat /etc/yum.repos.d/yum.repo 
[centos7.4]
name=centos7.4
baseurl="file:///run/media/qe/CentOS 7 x86_64" #cdrom
gpgcheck=0
$ yum clean all
$ yum makecache
$ yum repolist
$ sudo yum install ftp
$ yum install minicom
$ yum install wireshark
$ yum install wireshark-gnome
$ wireshark
```
编译驱动，其实下面的已经安装了，
```shell
yum install gcc kernel-headers kernel-devel
```
可直接执行编译，
```makefile
#ccflags-y += -Wno-error=date-time

ifneq ($(KERNELRELEASE),)
obj-m:=pcie_ep.o
else
KERNELDIR:=/lib/modules/`uname -r`/build
PWD:=$(shell pwd)

modules:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules
modules_install:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

.PHONY: clean
clean:
	rm -rf *.o *.mod.c *.mod.o *.ko *.order *symvers .*.cmd
endif
```
网卡驱动从centos7.4迁移到centos7.6报错`初始值设置项里有未知的字段ndo_change_mtu`，修改`ndo_change_mtu`为`ndo_change_mtu_rh74`。

## 配置阿里源
epel企业源，
```bash
# yum install epel-release
# cp bak/CentOS-Linux-PowerTools.repo .
[root@localhost yum.repos.d]# vim CentOS-Linux-PowerTools.repo 
# CentOS-Linux-PowerTools.repo
#
# The mirrorlist system uses the connecting IP address of the client and the
# update status of each mirror to pick current mirrors that are geographically
# close to the client.  You should use this for CentOS updates unless you are
# manually picking other mirrors.
#
# If the mirrorlist does not work for you, you can try the commented out
# baseurl line instead.

[powertools]
name=CentOS Linux $releasever - PowerTools
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=PowerTools&infra=$infra
#baseurl=http://mirror.centos.org/$contentdir/$releasever/PowerTools/$basearch/os/
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```

# 开机自启动
由于/etc/rc.local是/etc/rc.d/rc.local的软连接，所以必须确保/etc/rc.local和/etc/rc.d/rc.local都有可执行权限，
```shell
$ ls -l /etc/rc.local
lrwxrwxrwx 1 root root 13  1月 21  2021 /etc/rc.local -> rc.d/rc.local
$ sudo chmod a+x /etc/rc.local
$ sudo chmod a+x /etc/rc.d/rc.local
systemctl enable rc-local
systemctl start rc-local
```
脚本中所有的路径都必须使用绝对路径，添加`[Install]`字段，
```bash
[storage@localhost ~]$ cat /etc/rc.d/rc.local
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this
                                                   #
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local
ifconfig enp14s0f0 192.168.1.10
mkdir -p /run/media/root/Kylin-Server-10
mount /home/storage/Kylin-Server-10-SP1-Build01-20210309-JUN-arm64.iso /run/media/root/Kylin-Server-10
[storage@localhost ~]$ cat /lib/systemd/system/rc-local.service
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

# This unit gets pulled automatically into multi-user.target by
# systemd-rc-local-generator if /etc/rc.d/rc.local is executable.
[Unit]
Description=/etc/rc.d/rc.local Compatibility
Documentation=man:systemd-rc-local-generator(8)
ConditionFileIsExecutable=/etc/rc.d/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.d/rc.local start
TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no

[Install]
WantedBy=multi-user.target
```

# 关闭防火墙
```bash
$ firewall-cmd --state
$ systemctl stop firewalld.service
```

# 本地源
可以从光盘内copy目录结构，由于我的repomd.xml抱错，使用命令重新建立repo，
```bash
$ cp /run/media/qe/CentOS 7 x86_64/* iso
$ creatrepo iso
```

# 配置网络
```bash
$ sudo vi /etc/sysconfig/network-scripts/ifcfg-enaphyt4i0
BOOTPROTO=static
ONBOOT=no
IPADDR=192.168.1.10
NETMASK=255.255.255.0
GATEWAY=192.168.1.254
NAME=enaphyt4i0
DEVICE=enaphyt4i0
$ sudo systemctl restart network
```

# 远程登录
```bash
$ yum -y install yum grouplist
$ yum groupinstall -y "GNOME Desktop"
$ systemctl set-default graphical.target
$ systemctl set-default multi-user.target
$ yum install tigervnc-server -y
$ rpm -qa|grep tigervnc-server
$ cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
$ vi /etc/systemd/system/vncserver\@\:1.service 
$ vncpasswd 
$ systemctl start vncserver@\:1.service
$ systemctl daemon-reload
$ ps -ef | grep vnc
$ netstat -ant | grep 5901
$ service iptables status
$ systemctl status iptables
$ systemctl status firewalld
$ systemctl stop firewalld
$ systemctl disable firewalld
```

