# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 上手
下载[tapip源代码](https://github.com/chobits/tapip)到虚拟机，本人为ubuntu 16.04
```bash
zc@ubuntu:~/xilinx/app$ unzip tapip-master.zip
zc@ubuntu:~/xilinx/app$ cd tapip-master/
zc@ubuntu:~/xilinx/app/tapip-master$ make
zc@ubuntu:~/xilinx/app/tapip-master$ ls -l
total 192
drwxrwxr-x 2 zc zc   4096 May 24 06:13 app
drwxrwxr-x 2 zc zc   4096 May 24 06:13 arp
drwxrwxr-x 2 zc zc   4096 Dec 12  2016 doc
drwxrwxr-x 2 zc zc   4096 Dec 12  2016 include
drwxrwxr-x 2 zc zc   4096 May 24 06:13 ip
drwxrwxr-x 2 zc zc   4096 May 24 06:13 lib
-rw-rw-r-- 1 zc zc   1854 Dec 12  2016 Makefile
-rwxr-xr-x 1 zc zc    125 Dec 12  2016 mmleak.sh
drwxrwxr-x 2 zc zc   4096 May 24 06:13 net
-rw-rw-r-- 1 zc zc   1685 Dec 12  2016 README.md
drwxrwxr-x 2 zc zc   4096 May 24 06:13 shell
drwxrwxr-x 2 zc zc   4096 May 24 06:13 socket
-rwxrwxr-x 1 zc zc 132976 May 24 06:13 tapip
drwxrwxr-x 2 zc zc   4096 May 24 06:13 tcp
-rw-rw-r-- 1 zc zc    895 Dec 12  2016 TODO
drwxrwxr-x 2 zc zc   4096 May 24 06:13 udp
```
生成的tapip即可执行程序。
tapip底层收发包接口利用TAP/TUN虚拟网卡来调试，[Tun/Tap都是虚拟网卡，没有直接映射到物理网卡，是一种纯软件的实现。Tun是三层虚拟设备，能够处理三层即IP包，Tap是二层设备，能处理链路层网络包如以太网包。使用虚拟网络设备，可以实现隧道，如OpenVPN的实现](https://www.cnblogs.com/woshiweige/p/4532207.html)。
确定内核是否支持tun
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ modinfo tun
modinfo: ERROR: Module tun not found.
```
好了，不支持，现在得下载内核源代码自己编译了
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ uname -a
Linux ubuntu 4.13.0-41-generic #46~16.04.1-Ubuntu SMP Thu May 3 10:06:43 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
zc@ubuntu:~/xilinx/app/tapip-master$ apt-cache search linux-source
linux-source - Linux kernel source with Ubuntu patches
linux-source-4.4.0 - Linux kernel source for version 4.4.0 with Ubuntu patches
linux-source-4.10.0 - Linux kernel source for version 4.10.0 with Ubuntu patches
linux-source-4.11.0 - Linux kernel source for version 4.11.0 with Ubuntu patches
linux-source-4.13.0 - Linux kernel source for version 4.13.0 with Ubuntu patches
linux-source-4.15.0 - Linux kernel source for version 4.15.0 with Ubuntu patches
linux-source-4.8.0 - Linux kernel source for version 4.8.0 with Ubuntu patches
zc@ubuntu:~/xilinx/app/tapip-master$ sudo apt-get install linux-source-4.13.0
zc@ubuntu:~$ mkdir x86
zc@ubuntu:~$ cd x86/
zc@ubuntu:~/x86$ tar -jxvf linux-source-4.13.0.tar.bz2 
zc@ubuntu:~/x86$ cd linux-source-4.13.0/
zc@ubuntu:~/x86/linux-source-4.13.0$ make menuconfig
```
进入device driver > network device support > tun
![这里写图片描述](https://img-blog.csdn.net/20180524220649614?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
发现内核内检了tun/tap支持，所以应该不用编译内核了。
编译运行top1程序
```bash
#### User configure  ###############
CONFIG_DEBUG = n
CONFIG_DEBUG_PKB = n
CONFIG_DEBUG_WAIT = n
CONFIG_DEBUG_SOCK = n
CONFIG_DEBUG_ARP_LOCK = n
CONFIG_DEBUG_ICMPEXCFRAGTIME = n
CONFIG_TOPLOGY = 1
#### End of User configure #########
```
安装tunctl
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ sudo ./doc/brcfg.sh open
[sudo] password for zc: 
./doc/brcfg.sh: line 31: tunctl: command not found
./doc/brcfg.sh: line 34: brctl: command not found
./doc/brcfg.sh: line 37: brctl: command not found
./doc/brcfg.sh: line 38: brctl: command not found
SIOCSIFADDR: No such device
eth0: ERROR while getting interface flags: No such device
eth0: ERROR while getting interface flags: No such device
SIOCSIFADDR: No such device
tap0: ERROR while getting interface flags: No such device
tap0: ERROR while getting interface flags: No such device
SIOCSIFADDR: No such device
br0: ERROR while getting interface flags: No such device
br0: ERROR while getting interface flags: No such device
open ok
zc@ubuntu:~/xilinx/app/tapip-master$ sudo apt-get install uml-utilities
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  linux-headers-4.13.0-39 linux-headers-4.13.0-39-generic linux-image-4.13.0-39-generic linux-image-extra-4.13.0-39-generic
Use 'sudo apt autoremove' to remove them.
Suggested packages:
  user-mode-linux
The following NEW packages will be installed:
  uml-utilities
0 upgraded, 1 newly installed, 0 to remove and 281 not upgraded.
Need to get 49.4 kB of archives.
After this operation, 268 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu xenial/universe amd64 uml-utilities amd64 20070815-1.4 [49.4 kB]
Fetched 49.4 kB in 0s (127 kB/s)         
Selecting previously unselected package uml-utilities.
(Reading database ... 266152 files and directories currently installed.)
Preparing to unpack .../uml-utilities_20070815-1.4_amd64.deb ...
Unpacking uml-utilities (20070815-1.4) ...
Processing triggers for systemd (229-4ubuntu21.1) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up uml-utilities (20070815-1.4) ...
Processing triggers for systemd (229-4ubuntu21.1) ...
Processing triggers for ureadahead (0.100.0-19) ...
```
安装brctl
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ sudo ./doc/brcfg.sh open
./doc/brcfg.sh: line 34: brctl: command not found
./doc/brcfg.sh: line 37: brctl: command not found
./doc/brcfg.sh: line 38: brctl: command not found
SIOCSIFADDR: No such device
eth0: ERROR while getting interface flags: No such device
eth0: ERROR while getting interface flags: No such device
SIOCSIFADDR: No such device
br0: ERROR while getting interface flags: No such device
br0: ERROR while getting interface flags: No such device
open ok
zc@ubuntu:~/xilinx/app/tapip-master$ apt-cache search brctl
zc@ubuntu:~/xilinx/app/tapip-master$ sudo apt-get install bridge-utils
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  linux-headers-4.13.0-39 linux-headers-4.13.0-39-generic linux-image-4.13.0-39-generic linux-image-extra-4.13.0-39-generic
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  bridge-utils
0 upgraded, 1 newly installed, 0 to remove and 281 not upgraded.
Need to get 28.6 kB of archives.
After this operation, 102 kB of additional disk space will be used.
Get:1 http://mirrors.aliyun.com/ubuntu xenial/main amd64 bridge-utils amd64 1.5-9ubuntu1 [28.6 kB]
Fetched 28.6 kB in 0s (120 kB/s)        
Selecting previously unselected package bridge-utils.
(Reading database ... 266184 files and directories currently installed.)
Preparing to unpack .../bridge-utils_1.5-9ubuntu1_amd64.deb ...
Unpacking bridge-utils (1.5-9ubuntu1) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up bridge-utils (1.5-9ubuntu1) ...
zc@ubuntu:~/xilinx/app/tapip-master$ 
```
修改脚本，我的网卡不是eth0，是ens33
```bash
#!/bin/bash
# See doc/net_topology #TOP1 for more information
#
# TOPOLOGY:
#
#             br0
#            /   \
#        eth0     tap0
#         /         \
#     network(gw)   /dev/net/tun
#                     \
#                    veth (./tapip)
#
# NOTE: We view tap0, /dev/net/tun and veth(./tapip) as one interface,
#       They should have only one mac address(eth0 mac), which will
#       handle arp protocol right.
#       Then eth0, br0, tap0, /dev/net/tun and veth(./tapip) can be
#       viewed as only one interface, which is similar as the original
#       real eth0 interface, and ./tapip will become its internal
#       TCP/IP stack!
#
#       veth mac address must _NOT_ be eth0 mac address,
#       otherwise br0 dont work, in which case arp packet will not be passed
#       to veth
#
# WARNING: This test will suspend real kernel TCP/IP stack!
#          And we dont need kernel route table and arp cache.

ethname=ens33 # add by aj

openbr() {
	#create tap0
	tunctl -b -t tap0 > /dev/null

	#create bridge
	brctl addbr br0

	#bridge config
	brctl addif br0 $ethname #eth0 # add by aj
	brctl addif br0 tap0

	# net interface up (clear ip, ./tapip will set it)
	#ifconfig $ethname 0.0.0.0 up # comment by aj #eth0 
	ifconfig tap0 0.0.0.0 up
	ifconfig br0 0.0.0.0 up
	echo "open ok"
}

closebr() {
        ifconfig tap0 down
	ifconfig br0 down
	brctl delif br0 tap0
	brctl delif br0 $ethname #eth0 # add by aj
	brctl delbr br0
	tunctl -d tap0 > /dev/null
	echo "close ok"
}

usage() {
	echo "Usage: $0 [OPTION]"
	echo "OPTION:"
	echo "      open    config TOP1"
	echo "      close   unconfig TOP1"
	echo "      help    display help information"
}

if [ $UID != 0 ]
then
	echo "Need root permission"
	exit 0
fi

if [[ $# != 1 ]]
then
	usage
fi

case $1 in
open)
	openbr
	;;
close)
	closebr
	;;
*)
	usage
	;;
esac
```
执行doc下brctl.sh脚本，创建top1执行环境
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ sudo ./doc/brcfg.sh open
zc@ubuntu:~/xilinx/app/tapip-master$ ifconfig
br0       Link encap:Ethernet  HWaddr 00:0c:29:f6:52:47  
          inet6 addr: fe80::7c78:9eff:fe7b:1244/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:17 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:910 (910.0 B)  TX bytes:400 (400.0 B)

ens33     Link encap:Ethernet  HWaddr 00:0c:29:f6:52:47  
          inet addr:192.168.116.130  Bcast:192.168.116.255  Mask:255.255.255.0
          inet6 addr: fe80::1f8d:22fe:c0c0:5025/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3723985 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4176214 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1084576821 (1.0 GB)  TX bytes:5450197890 (5.4 GB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:11565 errors:0 dropped:0 overruns:0 frame:0
          TX packets:11565 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:749435 (749.4 KB)  TX bytes:749435 (749.4 KB)

tap0      Link encap:Ethernet  HWaddr 7e:78:9e:7b:12:44  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:12 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
zc@ubuntu:~/xilinx/app/tapip-master$ brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.000c29f65247	no		ens33
									tap0
```
执行tapip程序
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ sudo ./tapip 
[103694]loop_dev_init lo ip address: 127.0.0.1
[103694]loop_dev_init lo netmask:    255.0.0.0
[103694]getname_tap net device: tap0
[103694]getmtu_tap mtu: 1500
[103694]veth_dev_init veth ip address: 10.20.133.21
[103694]veth_dev_init veth hw address: 00:34:45:67:89:ab
[103694]arp_cache_init ARP CACHE INIT
[103694]arp_cache_init ARP CACHE SEMAPHORE INIT
[103694]rt_init route table init
[103694]raw_init raw ip init
[103694]net_stack_run thread 0: net_timer
[103694]net_stack_run thread 1: tcp_timer
[103694]net_stack_run thread 2: netdev_interrupt
[103694]net_stack_run thread 3: shell worker
[net shell]: debug arp
enter ^C to exit debug mode
[103697]arp_recv arp 192.168.116.1 -> 192.168.116.2
[103697]arp_recv arp not for us
^C
exit debug mode
[net shell]: ping 192.168.116.130
PING 192.168.116.130 56(84) bytes of data
^C
--- 192.168.116.130 ping statistics ---
72 packets transmitted, 0 received, 100% packet loss
```
这里会有一个问题，我是虚拟机测试的，而我的虚拟机是以NAT模式和主机共享IP（[Vmware虚拟机三种网络模式详解](https://blog.csdn.net/noob_f/article/details/51099040)），当我执行脚本的时候，虚拟机的网就和主机断了，导致si不能访问共享文件夹，这是个bug，不知道桥接模式会不会有不同，可能是我创建软桥br0把我的网卡ens33链接到这个桥之后，导致和主机的网就断了，更改脚本如下，发现只要创建了tun之后，自动断网，不知道是为什么啊？
```bash
openbr() {
	#create tap0
	tunctl -b -t tap0
	ifconfig tap0 192.168.116.138 up
	echo "open ok"
}

closebr() {
    ifconfig tap0 down
	tunctl -d tap0
	echo "close ok"
}
```
先作罢，看下tapip怎么实现veth，看看有没有什么思路。
