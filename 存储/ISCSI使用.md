# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Linux搭建ISCSI存储服务器](https://zhuanlan.zhihu.com/p/39362930)
> [CentOS7系列--3.2CentOS7中配置iSCSI服务](https://www.cnblogs.com/gispathfinder/p/8833488.html)
> [CentOS7下配置iSCSI服务端](https://blog.csdn.net/cl403007095/article/details/88073823)
> [PuTTY 和 SSH 免密码登录](https://blog.csdn.net/chentaichi6002/article/details/100920753)
> [linux集群管理工具clustershell](https://www.cnblogs.com/wangchengshi/p/11165246.html)
> [Putty使用ssh免密登录Linux](https://blog.csdn.net/zhaoxixc/article/details/82314957)
> [Linux集群批量管理工具parallel-ssh(PSSH)的安装与使用](https://www.linuxidc.com/Linux/2013-08/88547.htm)
> [Linux系统之工具篇（一）DRBD 单双主模式区别详解，Centos6.5（64bit）与nfs文件系统使用结合测试](https://www.cnblogs.com/dantezhao/p/5365211.html)
> [Linux系统之工具篇（二）集群管理软件clustershell](https://www.cnblogs.com/dantezhao/p/5365206.html)
> [linux系统之工具篇（三）集群管理工具Nmap](https://www.cnblogs.com/dantezhao/p/5365205.html)
> [简单好用的服务器运维面板](https://www.bt.cn/)
> [开源、强大的Linux服务器集群管理工具，比宝塔好用！](https://blog.csdn.net/yjh1271845364/article/details/105833719/)
> [8款基于Web控制面板的服务器管理工具，开源免费，系统管理员利器 ](https://www.sohu.com/a/391085213_100159565)
> [20个最佳的开源及商业的 Linux 服务器管理面板](https://www.open-open.com/news/view/118af3b)
> [Linux系统之工具篇（二）集群管理软件clustershell](https://blog.csdn.net/weixin_30273763/article/details/97421664)
> [centos7中启用rc-local服务](https://blog.csdn.net/x356982611/article/details/90414752)
> [Ubuntu 下iscsi initiator的安装与使用](https://blog.csdn.net/vah101/article/details/6238191)
> [ubuntu 12.04中iscsi target和initiator的安装和使用](http://blog.chinaunix.net/uid-20940095-id-3487049.html)
> [Ubuntu 中 iSCSI Target 和 Initiator 的使用](https://www.iteye.com/blog/e2718282-1739281)
> [iscsi磁盘挂载并设置为开机自动挂载](https://blog.csdn.net/jiyiyun/article/details/103798730)

# target
## centos
```bash
$ sudo yum -y install iscsi-initiator-utils perl-Config-General scsi-target-utils
$ uuidgen -t
7447a8ee-ce34-11e9-9b31-04a0c9210400
$ sudo vim /etc/tgt/targets.conf
$ sudo cat /etc/tgt/targets.conf
# This is a sample config file for tgt-admin.
#
# The "#" symbol disables the processing of a line.

# Set the driver. If not specified, defaults to "iscsi".
default-driver iscsi

# Set iSNS parameters, if needed
#iSNSServerIP 192.168.111.222
#iSNSServerPort 3205
#iSNSAccessControl On
#iSNS On

# Continue if tgtadm exits with non-zero code (equivalent of
# --ignore-errors command line option)
#ignore-errors yes

<target iqn.2021-04.com.jingjiamicro:7447a8ee-ce34-11e9-9b31-04a0c9210400>
        backing-store /dev/nvme0n1
</target>

<target iqn.2021-04.com.jingjiamicro:a600ad3c-ce2e-11e9-ae3f-3c6a2cb259c4>
        backing-store /dev/md5
</target>

$ sudo systemctl start tgtd
$ sudo systemctl enable tgtd #$ sudo chkconfig tgtd on
# 防火墙
$ firewall-cmd --add-service=iscsi-target --permanent
$ firewall-cmd --reload
# or
$ firewall-cmd --permanent --add-port=3260/tcp
$ firewall-cmd --reload
# or close firewalld
$ systemctl stop firewalld
$ systemctl disable firewalld

# 查看target状态，
$ netstat -antpl | grep :3260
$ tgtadm --lld iscsi --op show --mode target
```
卸载，


## ubuntu
```bash
$ sudo apt install iscsitarget iscsitarget-dkms
```

# Linux Initiator
## centos
```bash
$ sudo yum -y install iscsi-initiator-utils
# 搜索
$ sudo iscsiadm -m discovery -t sendtargets -p 192.168.40.70:3260
192.168.40.70:3260,1 iqn.2021-04.com.jingjiamicro:ceb39004-ce39-11e9-b856-3c6a2cb26120
$ sudo iscsiadm -m discovery -t sendtargets -p 192.168.40.71:3260
192.168.40.71:3260,1 iqn.2021-04.com.jingjiamicro:41f3f950-ce3f-11e9-843a-3c6a2cb23b26
$ sudo iscsiadm -m discovery -t sendtargets -p 192.168.40.72:3260
192.168.40.72:3260,1 iqn.2021-04.com.jingjiamicro:930c964a-ce2f-11e9-9ca3-3c6a2cb259c4

# 连接 72连接71
$ sudo iscsiadm -m node -T iqn.2021-04.com.jingjiamicro:a03fb190-ce2e-11e9-8d84-3c6a2cb26142 --login
Logging in to [iface: default, target: iqn.2021-04.com.jingjiamicro:a03fb190-ce2e-11e9-8d84-3c6a2cb26142, portal: 192.168.40.70,3260]
Login to [iface: default, target: iqn.2021-04.com.jingjiamicro:a03fb190-ce2e-11e9-8d84-3c6a2cb26142, portal: 192.168.40.70,3260] successful.
$ sudo iscsiadm -m node -T iqn.2021-04.com.jingjiamicro:41f3f950-ce3f-11e9-843a-3c6a2cb23b26 --login
Logging in to [iface: default, target: iqn.2021-04.com.jingjiamicro:a3c335bc-ce2e-11e9-8b0f-3c6a2cb23b26, portal: 192.168.40.71,3260]
Login to [iface: default, target: iqn.2021-04.com.jingjiamicro:a3c335bc-ce2e-11e9-8b0f-3c6a2cb23b26, portal: 192.168.40.71,3260] successful.
$ sudo iscsiadm -m node -T iqn.2021-04.com.jingjiamicro:930c964a-ce2f-11e9-9ca3-3c6a2cb259c4 --login
Logging in to [iface: default, target: iqn.2021-04.com.jingjiamicro:a600ad3c-ce2e-11e9-ae3f-3c6a2cb259c4, portal: 192.168.40.72,3260]
Login to [iface: default, target: iqn.2021-04.com.jingjiamicro:a600ad3c-ce2e-11e9-ae3f-3c6a2cb259c4, portal: 192.168.40.72,3260] successful.
```
先登出共享磁盘，再删除共享磁盘，
```bash
$ sudo iscsiadm -m node -T iqn.2021-04.com.jingjiamicro:a600ad3c-ce2e-11e9-ae3f-3c6a2cb259c4 -p 192.168.40.72,3260 -u 
$ sudo iscsiadm -m node -T iqn.2021-04.com.jingjiamicro:a600ad3c-ce2e-11e9-ae3f-3c6a2cb259c4 -p 192.168.40.72,3260 -o delete 
```
自动挂载，
```bash
$ sudo iscsiadm -m node -T iqn.2021-04.com.jingjiamicro:930c964a-ce2f-11e9-9ca3-3c6a2cb259c4 --op update -n node.startup -v automatic
```
配置文件，
```bash
/var/lib/iscsi/nodes/iqn.2021-04.com.jingjiamicro:930c964a-ce2f-11e9-9ca3-3c6a2cb259c4/192.168.40.72,3260,1/default
```

## ubuntu
```bash
$ sudo apt-get install open-iscsi
```

# Windows Initiator
Windows管理工具里打开iSCSI发起程序，选择是，
![402](https://img-blog.csdnimg.cn/20210426163818631.PNG)
填入服务器的IP，
![403](https://img-blog.csdnimg.cn/20210426164054525.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击自动配置，
![404](https://img-blog.csdnimg.cn/20210426164117136.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
这个时候可以看到分区信息，可以进行速度测试，
![405](https://img-blog.csdnimg.cn/2021042616413378.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)




