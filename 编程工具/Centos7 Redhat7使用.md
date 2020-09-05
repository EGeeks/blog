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
# 安装系统
Centos7和Redhat7同版本之间是兼容的，但是不确定Centos7.4和Centos7.7之间是否兼容，而目前国内的源上已经无法下载老版本的Centos7只有最新的`CentOS-7-x86_64-DVD-1908.iso`或者Centos8。Centos7.4的版本号，我安装的时候选择的是`开发版本`，
```shell
$ uname -a
Linux localhost.localdomain 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
$ cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core)
```

## 获取内核源代码
下载对应内核源码`3.10.0-693.el7.x86_64`，所有包的源码包rpm路径为`http://vault.centos.org/centos/7.4.1708/os/Source/SPackages/`。

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
网卡驱动从centos7.4迁移到centos7.6报错`初始值设置项里有未知的字段ndo_change_mtu`，修改`ndo_change_mtu`为`ndo_change_mtu_rh74`

# 开机自启动
由于/etc/rc.local是/etc/rc.d/rc.local的软连接，所以必须确保/etc/rc.local和/etc/rc.d/rc.local都有可执行权限，
```shell
sudo chmod +x /etc/rc.local
sudo chmod +x /etc/rc.d/rc.local
```
脚本中所有的路径都必须使用绝对路径。
# 关闭防火墙
```bash
$ firewall-cmd --state
$ systemctl stop firewalld.service
```

