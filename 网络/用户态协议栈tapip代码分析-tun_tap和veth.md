# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 代码分析
本人比较low，vim一直入门状态，习惯共享用si看代码
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ ls -l
total 372
drwxrwxrwx 2 zc zc   4096 May 24 08:15 app
drwxrwxrwx 2 zc zc   4096 May 24 08:15 arp
drwxrwxrwx 2 zc zc   4096 May 25 01:50 doc
drwxrwxrwx 2 zc zc   4096 Dec 12  2016 include
drwxrwxrwx 2 zc zc   4096 May 24 08:15 ip
drwxrwxrwx 2 zc zc   4096 May 24 08:15 lib
-rw-rw-rw- 1 zc zc   1854 May 24 08:15 Makefile
-rwxrwxrwx 1 zc zc    125 Dec 12  2016 mmleak.sh
drwxrwxrwx 2 zc zc   4096 May 24 08:15 net
-rw-rw-rw- 1 zc zc   1685 Dec 12  2016 README.md
drwxrwxrwx 2 zc zc   4096 May 24 08:15 shell
drwxrwxrwx 2 zc zc   4096 May 24 08:15 socket
-rwxrwxr-x 1 zc zc 319408 May 24 08:15 tapip
drwxrwxrwx 2 zc zc   4096 May 24 08:15 tcp
-rw-rw-rw- 1 zc zc    895 Dec 12  2016 TODO
drwxrwxrwx 2 zc zc   4096 May 24 08:15 udp
```
进入net文件夹，主要是tun/tap 和veth的实现，关于tun的应用模型，这个帖子讲的不错：[Linux下Tun/Tap设备通信原理](https://blog.csdn.net/flyforfreedom2008/article/details/46038853) 和这个结合的DPDK的TUN应用：[TAP/TUN浅析（一）](https://www.cnblogs.com/yml435/p/5917628.html) 和：[linux中 tun/tap 的实现](https://blog.csdn.net/u013982161/article/details/51816162) 。
执行brcfg.sh之后，形成的网络拓扑如下所示
```bash
=============================================================================
TOP1 (Use tapip instead of linux kernel TCP/IP stack)
=============================================================================

+----------+
| internet |
+----|-----+
+----|-----+
|   eth0   |
+----|-----+
+----|-----+
|  bridge  |
+----|-----+
+----|---------+
|   tap0       |
|--------------|
| /dev/net/tun |
+--|----|---|--+
  poll  |   |
   |  read  |
   |    |  write
+--|----|---|--+
| my netstack  |
+--------------+

=============================================================================
real network of TOP1

      gateway (10.20.133.20)
        |
      eth0 (0.0.0.0)
        |
       br0 (0.0.0.0)
        |
       tap0
      (0.0.0.0)
        |
      /dev/net/tun
        |
      veth0 (10.20.133.21)
      usermode
      network

=============================================================================
```
比较重要的一点，0.0.0.0表示所有ip，这是tapip和外部网关通信的关键，包的收发过程
```c
kernel-usermode stream of TOP1 ( usermode network stack receiving packet )

 network
    |
    | packet recv
   \|/
xxx_interrupt() --> netif_rx()
 { eth0 }                  |
                           |
                           |
    .----------------------'
    |  raise irq
   \|/
softirq: net_rx_action() --> netif_receive_skb() --> handle_bridge()
                                                        { br0 }
                                                           |
                                                           |
    .------------------------------------------------------'
    |
   \|/
dev_queue_xmit(skb) --> tun_net_xmit()
 [skb->dev == tap0]     1. put skb into tun's skbqueue
                        2. wake up process(./tapip) waiting
                                    for /dev/net/tun (read/poll)
                             |
                             |
                            \|/
                         process (read/poll)
                       ( ./tapip            )
                       { usermode network stack }


=============================================================================
kernel-usermode stream of TOP1 ( usermode network stack sending packet )

 usermode
 network
 stack
   |
   | write
  \|/
/dev/net/tun --> tun_chr_aio_write() --> tun_get_user()
                                         1. copy data from usermode
                                         2. make skb(sending packet)
                                         3. netif_rx_ni
                                             |
                                             |
   .-----------------------------------------'
   |
  \|/
netif_rx(skb)
1. put packet into queue
2. raise softirq
   |
  \|/
softirq: net_rx_action() --> netif_receive_skb() --> handle_bridge()
                                                        { br0 }
                                                           |
                                                           |
    .------------------------------------------------------'
    |
   \|/
dev_queue_xmit(skb) --> {eth0 netdevice}_hard_xmit()
 [skb->dev == eth0]                       |
                                          |  packet send
                                         \|/
                                       network
```
上节我们发现tun建立，虚拟机就断网，新建一块网卡做一下实验
```bash
zc@ubuntu:~$ ifconfig
ens33     Link encap:Ethernet  HWaddr 00:0c:29:f6:52:47  
          inet addr:192.168.116.130  Bcast:192.168.116.255  Mask:255.255.255.0
          inet6 addr: fe80::1f8d:22fe:c0c0:5025/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6474 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1421 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:8509256 (8.5 MB)  TX bytes:116323 (116.3 KB)

ens38     Link encap:Ethernet  HWaddr 00:0c:29:f6:52:51  
          inet addr:192.168.116.128  Bcast:192.168.116.255  Mask:255.255.255.0
          inet6 addr: fe80::43bc:db53:ecd5:b9e6/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:703 errors:0 dropped:0 overruns:0 frame:0
          TX packets:56 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:62602 (62.6 KB)  TX bytes:7072 (7.0 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:580 errors:0 dropped:0 overruns:0 frame:0
          TX packets:580 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:45615 (45.6 KB)  TX bytes:45615 (45.6 KB)
```
修改tapip源代码netcfg.h如下
```bash
/* see doc/brcfg.sh & doc/net_topology # TOP1 */
#ifdef CONFIG_TOP1
#define FAKE_TAP_ADDR 0x00000000	/* 0.0.0.0 */
#define FAKE_IPADDR 0x8A74A8C0//192.168.116.138//0x1585140a		/* 10.20.133.21 */
/* veth mac cannot be eth0 mac, otherwise eth0 will received arp packet */
#define FAKE_HWADDR "\x00\x34\x45\x67\x89\xab"
#define DEFAULT_GW 0x0274A8C0//192.168.116.138//0x1485140a		/* 10.20.133.20 */
#endif
```
这样修改之后就好了，配置和虚拟机同一个网段的ip，网关设置成虚拟机NAT网络配置中的网关（参考wmware配置）。
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ sudo ./doc/brcfg.sh open
open ok
zc@ubuntu:~/xilinx/app/tapip-master$ sudo ./tapip 
[117781]loop_dev_init lo ip address: 127.0.0.1
[117781]loop_dev_init lo netmask:    255.0.0.0
[117781]getname_tap net device: tap0
[117781]getmtu_tap mtu: 1500
[117781]veth_dev_init veth ip address: 192.168.116.138
[117781]veth_dev_init veth hw address: 00:34:45:67:89:ab
[117781]arp_cache_init ARP CACHE INIT
[117781]arp_cache_init ARP CACHE SEMAPHORE INIT
[117781]rt_init route table init
[117781]raw_init raw ip init
[117781]net_stack_run thread 0: net_timer
[117781]net_stack_run thread 1: tcp_timer
[117781]net_stack_run thread 2: netdev_interrupt
[117781]net_stack_run thread 3: shell worker
[net shell]: ping 192.168.116.128
PING 192.168.116.128 56(84) bytes of data
64 bytes from 192.168.116.128: icmp_seq=0 ttl=64
64 bytes from 192.168.116.128: icmp_seq=1 ttl=64
64 bytes from 192.168.116.128: icmp_seq=2 ttl=64
64 bytes from 192.168.116.128: icmp_seq=3 ttl=64
^C
--- 192.168.116.128 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss
[net shell]: ping 192.168.6.4
64 bytes from 192.168.6.4: icmp_seq=0 ttl=128
64 bytes from 192.168.6.4: icmp_seq=1 ttl=128
64 bytes from 192.168.6.4: icmp_seq=2 ttl=128
^C
--- 192.168.6.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
[net shell]: exit
[117785]shell_worker shell worker exit
```
brctg.sh中删除配置虚拟机网卡的脚本，即保持状态不变。
源代码没什么需要特别分析的，很简单，read/write tun字符设备就好了。

# tun/tap设备使用
为了从linux内核底层获取网络数据包，使用tap设备，简单地说，tun/tap设备被用户态应用用于操纵L3（tun）和L2（tap）层数据包，一个典型的应用就是建立隧道，一个网络数据包被嵌入另外一个数据包。
tun/tap设备的优点是易于使用，已经被用于很多程序，比如OpenVPN。
我们自己实现用户态协议栈，所以使用tap设备。
```bash
/*
 * Taken from Kernel Documentation/networking/tuntap.txt
 */
int tun_alloc(char *dev)
{
    struct ifreq ifr;
    int fd, err;

    if( (fd = open("/dev/net/tap", O_RDWR)) < 0 ) {
        print_error("Cannot open TUN/TAP dev");
        exit(1);
    }

    CLEAR(ifr);

    /* Flags: IFF_TUN   - TUN device (no Ethernet headers)
     *        IFF_TAP   - TAP device
     *
     *        IFF_NO_PI - Do not provide packet information
     */
    ifr.ifr_flags = IFF_TAP | IFF_NO_PI;
    if( *dev ) {
        strncpy(ifr.ifr_name, dev, IFNAMSIZ);
    }

    if( (err = ioctl(fd, TUNSETIFF, (void *) &ifr)) < 0 ){
        print_error("ERR: Could not ioctl tun: %s\n", strerror(errno));
        close(fd);
        return err;
    }

    strcpy(dev, ifr.ifr_name);
    return fd;
}
```
得到的fd可以用于向虚拟网卡上读写数据。IFF_NO_PI参考[虚拟网卡 TUN/TAP 驱动程序设计原理](https://blog.csdn.net/macrossdzh/article/details/5734736)
IFF_TUN: 创建一个点对点设备
IFF_TAP: 创建一个以太网设备
IFF_NO_PI: 不包含包信息，默认的每个数据包当传到用户空间时，都将包含一个附加的包头来保存包信息
IFF_ONE_QUEUE: 采用单一队列模式，即当数据包队列满的时候，由虚拟网络设备自已丢弃以后的数据包直到数据包队列再有空闲。
配置的时候，IFF_TUN 和 IFF_TAP必须择一，其他选项则可任意组合。其中IFF_NO_PI没有开启时所附加的包信息头如下:
```c
struct tun_pi {
    unsigned short flags;
    unsigned short proto;
};
```
目前，flags只在收取数据包的时候有效，当它的TUN_PKT_STRIP标志被置时，表示当前的用户空间缓冲区太小，以致数据包被截断。proto成员表示发送/接收的数据包的协议。
