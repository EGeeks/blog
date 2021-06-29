# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [VMware虚拟机三种联网方式（图文详细解说）](https://blog.csdn.net/qq_28090573/article/details/78730552)
> [Ubuntu 18.04 永久修改DNS的方法](https://blog.csdn.net/weixin_43640082/article/details/83859885)
> [Ubuntu 18.04设置dns](https://www.cnblogs.com/breezey/p/9155988.html)
> [VMWARE虚拟机串口添加与调试](https://www.goupteam.com/tech-expand/1259.html)
> [VMware虚拟串口的设置和使用](https://blog.csdn.net/exbob/article/details/6313443)
> [VMware 将虚拟磁盘多个文件合并为单个文件](https://blog.csdn.net/weixin_44303986/article/details/105047775)

# 安装
安装前请在BIOS中使能处理器虚拟化支持，Intel的是VT-x，否则会出现下面的问题，
![40](https://img-blog.csdnimg.cn/20201228220933372.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

## Windows虚拟机
按照默认一路安装即可，配置共享文件夹，通过网上邻居添加文件夹，
![27](https://img-blog.csdnimg.cn/2020111323491052.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)

## Ubuntu虚拟机
文件`->`新建，选择网上下载的ISO镜像，一路默认选择，
![136](https://img-blog.csdnimg.cn/20190621155146526.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
大小填200GB，免得以后还要扩展，
![137](https://img-blog.csdnimg.cn/20190621155322234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击完成，开始安装，
![140](https://img-blog.csdnimg.cn/20190621155733905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
扩大磁盘大小，
```shell
zc@ubuntu:~/program$ sudo apt-get install gparted
```
sudo运行fdisk，删除swap分区，扩展分区（顺序不要颠倒），仔细核对，不行就按q从来。运行Gparted partition editor，扩展boot分区，留2GB，按之前的操作新建扩展和swap分区。
![这里写图片描述](https://img-blog.csdn.net/20180703234443613?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
配置好之后，点击工具栏的对号，出现错误点击Ignore继续，
![这里写图片描述](https://img-blog.csdn.net/20180703234828894?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
重启，
```shell
zc@ubuntu:~$ fdisk -l
fdisk: cannot open /dev/sda: Permission denied
zc@ubuntu:~$ sudo fdisk -l
[sudo] password for zc: 
Disk /dev/sda: 120 GiB, 128849018880 bytes, 251658240 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa976b913

Device     Boot     Start       End   Sectors  Size Id Type
/dev/sda1  *         2048 247463935 247461888  118G 83 Linux
/dev/sda2       247463936 251658239   4194304    2G  5 Extended
/dev/sda5       247465984 251658239   4192256    2G 82 Linux swap / Solaris
zc@ubuntu:~$ df -T
Filesystem     Type             1K-blocks     Used Available Use% Mounted on
udev           devtmpfs           1981720        0   1981720   0% /dev
tmpfs          tmpfs               401644    11612    390032   3% /run
/dev/sda1      ext4              80374600 69836784   6431988  92% /
tmpfs          tmpfs              2008216      212   2008004   1% /dev/shm
tmpfs          tmpfs                 5120        4      5116   1% /run/lock
tmpfs          tmpfs              2008216        0   2008216   0% /sys/fs/cgroup
vmhgfs-fuse    fuse.vmhgfs-fuse 116589564 64914144  51675420  56% /mnt/hgfs
tmpfs          tmpfs               401644       60    401584   1% /run/user/1000
```
磁盘分区是扩大了，但是文件系统依然只能识别80GB，网上的教程大都没有解释清楚，我也纳闷他们是怎么成功的，参考博客[ubuntu对根目录进行扩展](https://blog.csdn.net/maclinuxye/article/details/52901019)。
```shell
zc@ubuntu:~$ sudo resize2fs /dev/sda1
resize2fs 1.42.13 (17-May-2015)
Filesystem at /dev/sda1 is mounted on /; on-line resizing required
old_desc_blocks = 5, new_desc_blocks = 8
The filesystem on /dev/sda1 is now 30932736 (4k) blocks long.
zc@ubuntu:~$ df -T
Filesystem     Type             1K-blocks     Used Available Use% Mounted on
udev           devtmpfs           1981720        0   1981720   0% /dev
tmpfs          tmpfs               401644    11612    390032   3% /run
/dev/sda1      ext4             121658468 69844756  46030244  61% /
tmpfs          tmpfs              2008216      212   2008004   1% /dev/shm
tmpfs          tmpfs                 5120        4      5116   1% /run/lock
tmpfs          tmpfs              2008216        0   2008216   0% /sys/fs/cgroup
vmhgfs-fuse    fuse.vmhgfs-fuse 116589564 64914584  51674980  56% /mnt/hgfs
tmpfs          tmpfs               401644       60    401584   1% /run/user/1000
```
重新开机出现错误，导致开机时间增加1min30s，不可忍，更新fstab，
```shell
zc@ubuntu:~$ swapon --show
zc@ubuntu:~$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=1981720k,nr_inodes=495430,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=401644k,mode=755)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=29,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=643)
mqueue on /dev/mqueue type mqueue (rw,relatime)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,pagesize=2M)
vmware-vmblock on /run/vmblock-fuse type fuse.vmware-vmblock (rw,relatime,user_id=0,group_id=0,default_permissions,allow_other)
configfs on /sys/kernel/config type configfs (rw,relatime)
fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
tmpfs on /run/user/108 type tmpfs (rw,nosuid,nodev,relatime,size=401644k,mode=700,uid=108,gid=114)
vmhgfs-fuse on /mnt/hgfs type fuse.vmhgfs-fuse (rw,nosuid,nodev,relatime,user_id=0,group_id=0,allow_other)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=401644k,mode=700,uid=1000,gid=1000)
gvfsd-fuse on /run/user/1000/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
zc@ubuntu:~$ sudo blkid
[sudo] password for zc: 
/dev/sda1: UUID="2adca72f-3a1d-4bb8-9088-b178c9a339a7" TYPE="ext4" PARTUUID="a976b913-01"
/dev/sda5: UUID="019c9714-7f55-43a9-abfe-395dd2f7bca3" TYPE="swap" PARTUUID="a976b913-05"
zc@ubuntu:~$ sudo gedit /etc/fstab
```
原来的是，
```shell
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=2adca72f-3a1d-4bb8-9088-b178c9a339a7 /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=c35dd24f-2cce-404d-8cc8-08473225be79 none            swap    sw              0       0
/dev/fd0        /media/floppy0  auto    rw,user,noauto,exec,utf8 0       0
```
更新新的UUID
```shell
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=2adca72f-3a1d-4bb8-9088-b178c9a339a7 /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=019c9714-7f55-43a9-abfe-395dd2f7bca3 none            swap    sw              0       0
/dev/fd0        /media/floppy0  auto    rw,user,noauto,exec,utf8 0       0
```

# 网络配置
虚拟机安装时默认为NAT网卡，`编辑->虚拟网络编辑器`，选用NAT模式，
![141](https://img-blog.csdnimg.cn/20190621194645762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击NAT设置，
![142](https://img-blog.csdnimg.cn/20190621194724930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

## Ubuntu
虚拟机配置IP，
![在这里插入图片描述](https://img-blog.csdnimg.cn/201906211948179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
编辑`resolv`，否则无法联网，一旦不能联网，不能下载软件包，需要检查这个文件，
```shell
qe@ubuntu:/var/www/html/packages$ sudo gedit /etc/resolv.conf 
qe@ubuntu:/var/www/html/packages$ cat /etc/resolv.conf 
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
nameserver 192.168.91.2
# nameserver 127.0.1.1
```
但是这个文件重启会被覆盖成默认值，更新文件`/etc/systemd/resolved.conf`，添加`DNS=192.168.91.2 8.8.8.8`，多个ip用空格隔开，或者，
```bash
DNS=192.168.91.2
DNS=8.8.8.8
```
或，
![134](https://img-blog.csdnimg.cn/20200307233449574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 虚拟机拷贝
在笔记本上，我有一个130GB的虚拟机，安装petalinux的开发环境，当时我在笔记本上编译工程，然后直接通过FTP把虚拟机拷贝到台式机上使用，当我拷贝完之后，导入虚拟机，提醒我虚拟机正在使用状态，然后就无法开机了，恢复快照也有新的错误，无语了，当时拷贝过程中，我也预感会有问题，迫于百兆网交换机的速度只有8MB/s，舍不得从头拷贝。。。看来还是失策了。

# 合并虚拟机磁盘文件
使用`C\Program Files（x86）\VMware\VMware Player\vmware-vdiskmanager.exe`。

# USB配置
默认是询问，这个很烦，每次弹出个窗口，不注意还以为电脑不识别。
![384](https://img-blog.csdnimg.cn/20201231103832610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 串口配置
给Ubuntu添加一个调试串口，关闭虚拟机，点击菜单，`虚拟机->设置`，如果没有串口，则点击添加串口，选择管道模式，
![385](https://img-blog.csdnimg.cn/20201231104959383.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
我的显示的是`串行端口 2`，经测试对应`/dev/ttyS1`，minicom和putty使用默认设置，不需要关闭流控，
```bash                                                                                                                                           
qe@ubuntu:~$ sudo cat /proc/tty/driver/serial
serinfo:1.0 driver revision:
0: uart:16550A port:000003F8 irq:4 tx:24 rx:0 CTS|DSR|CD
1: uart:16550A port:000002F8 irq:3 tx:9 rx:0 CTS|DSR|CD
2: uart:unknown port:000003E8 irq:4
3: uart:unknown port:000002E8 irq:3
4: uart:unknown port:00000000 irq:0
5: uart:unknown port:00000000 irq:0
6: uart:unknown port:00000000 irq:0
7: uart:unknown port:00000000 irq:0
8: uart:unknown port:00000000 irq:0
9: uart:unknown port:00000000 irq:0
10: uart:unknown port:00000000 irq:0
11: uart:unknown port:00000000 irq:0
12: uart:unknown port:00000000 irq:0
13: uart:unknown port:00000000 irq:0
14: uart:unknown port:00000000 irq:0
15: uart:unknown port:00000000 irq:0
16: uart:unknown port:00000000 irq:0
17: uart:unknown port:00000000 irq:0
18: uart:unknown port:00000000 irq:0
19: uart:unknown port:00000000 irq:0
20: uart:unknown port:00000000 irq:0
21: uart:unknown port:00000000 irq:0
22: uart:unknown port:00000000 irq:0
23: uart:unknown port:00000000 irq:0
24: uart:unknown port:00000000 irq:0
25: uart:unknown port:00000000 irq:0
26: uart:unknown port:00000000 irq:0
27: uart:unknown port:00000000 irq:0
28: uart:unknown port:00000000 irq:0
29: uart:unknown port:00000000 irq:0
30: uart:unknown port:00000000 irq:0
31: uart:unknown port:00000000 irq:0
qe@ubuntu:~$ dmesg | grep ttys*
[    0.004000] console [tty0] enabled
[    1.956527] 00:05: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
[    1.979817] 00:06: ttyS1 at I/O 0x2f8 (irq = 3, base_baud = 115200) is a 16550A
```

# 找不到Windows共享文件夹
以前是自动出现在`/mnt/hgfs`下的，共享是已经成功了的，
```bash
$ vmware-hgfsclient 
D
```
下面关闭虚拟机重启，确保安装虚拟时的ISO文件不再占用CD/DVD光驱，点击菜单，
![43](https://img-blog.csdnimg.cn/20210101233100527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
这是会挂载一个光驱，把光驱中的文件拷贝到本地，
![44](https://img-blog.csdnimg.cn/20210101233441463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
执行命令，`vmware-install.pl`一路回车默认安装，安装完重启一下，
```bash
$ tar -zxvf VMwareTools-10.3.21-14772444.tar.gz 
$ cd vmware-tools-distrib/
$ sudo apt remove --purge open-vm-tools
$ sudo ./vmware-install.pl
```

# 问题
## 本地设备名已在使用中
1. 以前先开虚拟机，后点击文件夹就会避免这个问题。
2. 实在不行，就重启电脑。
3. 最近发现，把WMnet8禁用再启用就可以了。
4. 后发现，先用Ubuntu ping一下宿主机就可以了，注意不是vmnet1/vmnet8这两个网卡的IP。

# 主机ping不通虚拟机
关闭虚拟机防火墙，
![70](https://img-blog.csdnimg.cn/20210629221418319.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

