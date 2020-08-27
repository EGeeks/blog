# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [网卡多队列技术与RSS功能介绍](https://blog.csdn.net/baidu_24553027/article/details/54927724)
> [ETHTOOL设置网卡接收哈希（RSS）](https://blog.csdn.net/sinat_20184565/article/details/88127851)
> [TSO、UFO、GSO、LRO、GRO和RSS介绍（ethtool命令）](https://blog.csdn.net/an_zhenwei/article/details/18732081)
> [Linux 内核协议栈的 TSO、GSO](https://blog.csdn.net/lovelylichqq/article/details/81625117)
> [linux内核网络协议栈学习笔记：关于GRO/GSO/LRO/TSO等patch的分析和测试](https://blog.csdn.net/majieyue/article/details/7929398)
> [GSO: Generic Segmentation Offload](https://lwn.net/Articles/188489/)
> [网络数据包分析 网卡Offload](https://blog.csdn.net/sdulibh/article/details/50108471)
> [linux内核网络协议栈学习笔记（6）](https://blog.csdn.net/majieyue/article/details/7980543)
> [GSO: Generic Segmentation Offload](https://lwn.net/Articles/188489/)
> [GSO/TSO/GRO等对VirtIO虚机的网络性能影响分析(by quqi99)](https://blog.csdn.net/quqi99/article/details/51066800)
> [linux tcp GSO和TSO实现](https://www.cnblogs.com/lvyilong316/p/6818231.html)
> [Segmentation Offloads in the Linux Networking Stack](https://www.kernel.org/doc/Documentation/networking/segmentation-offloads.txt)
> [linux内核网络路径数据包分片功能介绍](https://www.toutiao.com/a6436205299023544578/)
> [【网络协议】TCP分段与IP分片](https://blog.csdn.net/ns_code/article/details/30109789)

# GSO
`dev_hard_start_xmit`里判断`netif_needs_gso`判断网卡是否支持gso，如果不支持则调用`dev_gso_segment`里面又调用`skb_gso_segment`把报文分片，对于ipv4而言，实际调用了`tcp_tso_segment`，最后返回多个`sk_buff`组成的链表，头指针存在`skb->next`里，如果网卡本身支持的话，直接把大块的skb交给网卡：调用`netdev_ops->ndo_start_xmit`发送出去，判断netif_need_gso时，检查网卡的`netdev->features`，位于`include/linux/netdevice.h`，
```c
#define NETIF_F_SG      1   /* Scatter/gather IO. */
#define NETIF_F_IP_CSUM     2   /* Can checksum TCP/UDP over IPv4. */
#define NETIF_F_NO_CSUM     4   /* Does not require checksum. F.e. loopack. */
#define NETIF_F_HW_CSUM     8   /* Can checksum all the packets. */

#define NETIF_F_FRAGLIST    64  /* Scatter/gather IO. */

#define NETIF_F_GSO     2048    /* Enable software GSO. */

#define NETIF_F_GSO_SHIFT   16
#define NETIF_F_GSO_MASK    0x00ff0000
#define NETIF_F_TSO     (SKB_GSO_TCPV4 << NETIF_F_GSO_SHIFT)
#define NETIF_F_UFO     (SKB_GSO_UDP << NETIF_F_GSO_SHIFT)
```
对于要支持TSO的网卡而言，需要有`NETIF_F_SG | NETIF_F_TSO | NETIF_F_IP_CSUM`，相应如果要支持UFO，应该就需要`NETIF_F_SG | NETIF_F_UFO | NETIF_F_IP_CSUM`，命令`ethtool -k eth0`可查看网卡驱动的gso/tso特性：
```bash
$ ethtool -k enp1s0f0
Features for enp1s0f0:
rx-checksumming: on
tx-checksumming: on
	tx-checksum-ipv4: off [fixed]
	tx-checksum-ip-generic: on
	tx-checksum-ipv6: off [fixed]
	tx-checksum-fcoe-crc: on [fixed]
	tx-checksum-sctp: on
scatter-gather: on
	tx-scatter-gather: on
	tx-scatter-gather-fraglist: off [fixed]
tcp-segmentation-offload: on
	tx-tcp-segmentation: on
	tx-tcp-ecn-segmentation: off [fixed]
	tx-tcp6-segmentation: on
	tx-tcp-mangleid-segmentation: off
udp-fragmentation-offload: off [fixed]
generic-segmentation-offload: on
generic-receive-offload: on
large-receive-offload: off
rx-vlan-offload: on
tx-vlan-offload: on
ntuple-filters: off
receive-hashing: on
highdma: on [fixed]
rx-vlan-filter: on
vlan-challenged: off [fixed]
tx-lockless: off [fixed]
netns-local: off [fixed]
tx-gso-robust: off [fixed]
tx-fcoe-segmentation: on [fixed]
tx-gre-segmentation: on
tx-ipip-segmentation: on
tx-sit-segmentation: on
tx-udp_tnl-segmentation: on
fcoe-mtu: off [fixed]
tx-nocache-copy: off
loopback: off [fixed]
rx-fcs: off [fixed]
rx-all: off
tx-vlan-stag-hw-insert: off [fixed]
rx-vlan-stag-hw-parse: off [fixed]
rx-vlan-stag-filter: off [fixed]
busy-poll: off [fixed]
tx-gre-csum-segmentation: on
tx-udp_tnl-csum-segmentation: on
tx-gso-partial: on
tx-sctp-segmentation: off [fixed]
rx-gro-hw: off [fixed]
l2-fwd-offload: off
hw-tc-offload: off
rx-udp_tunnel-port-offload: on
```
命令`ethtool -K eth0 tso|gso off|on`可打开或关闭网卡驱动的gso/tso特性，采用`-k`得到的缩写`generic-receive-offload`是`gro`，
```bash
$ sudo ethtool -K enp1s0f0 gro off
```
- 物理网卡不支持GSO时，使用TSO时，TCP分段在驱动处调用硬件做，不使用TSO时，TCP分段在TCP协议处软件做。
- 物理网卡不支持UFO时，使用GSO时，在发送给驱动前一刻做，不使用GSO在IP层软件做。
- 物理网卡不支持TSO时，使用GSO时，在发送给驱动前一刻做，不使用GSO在TCP层软件做。

TSO与GSO的区别，
- TSO只有第一个分片有TCP头和IP头，接着的分段只有IP头。意味着第一个分段丢失，所有分段得重传。
- GSO在分段时会调用TCP或UDP的回调函数(udp4_ufo_fragment)为每个分段都加上IP头，由于分段是通过mss设置的（mss由发送端设置），所以长度仍然可能超过mtu值，所以在IP层还得再分片(代码位于dev_hard_start_xmit)。

# TSO

在发送处理方面，提供了tcp-segmentation-offload（TSO），udp-fragmentation-offload(UFO，generic-segmentation-offload(GSO)这3个特性。UFO是专门针对UDP协议的，使用了这个特性，用户层就可以发送大的数据包（最大长度是64K），而不需要由IP协议来分片。 UFO与TSO，GSO没有任何的联系，但是却需要tx-checksumming，scatter-gather这两个特性的支持。遗憾的是，现在还没有看到有网卡支持UFO。所以，默认情况下，该功能的状态是off。当前UFO被应用在虚拟设备(比如虚拟的bridge设备，bond设备)上。TSO是专门针对TCP协议，DCCP[1]协议的，与UFO一样，使用了这个特性，用户层就可以发送大的数据包。TSO的启用不需要GSO的支持，但是却需要tx-checksumming，scatter-gather这两个特性的支持。当前的网卡更多的是针对TCP协议进行功能强化，当前的网卡只支持TSO，而不支持UFO。这3个特性中只有TSO是纯硬件的功能。这个分片和IP分片相比有如下的优势：
- 硬件分片，速度更快；
- 每个分片后的包是一个单独的TCP包，即每个分片包都包含TCP头部。且IP头部不包含任何的与分片相关的值。避免了分片包的丢失，而导致所有的分片包被重传。
- 对于接收端而言，则省去了重组分片的工作。减轻了接收端的工作负荷。
- 减少了sk_buff(内核使用这个来管理数据包)的使用，并降低了CPU的使用率。从tcpdump捕获的数据包中可以看出TSO启用后的效果。一个数据包可以包含11824个字节，远远超过MTU。

以下是TSO和GSO的组合关系：
1. GSO开启， TSO开启: 协议栈推迟分段，并直接传递大数据包到网卡，让网卡自动分段
2. GSO开启， TSO关闭: 协议栈推迟分段，在最后发送到网卡前才执行分段
3. GSO关闭， TSO开启: 同GSO开启， TSO开启
4. GSO关闭， TSO关闭: 不推迟分段，在tcp_sendmsg中直接发送MSS大小的数据包

如果想用FPGA来做这个offloading，参考第2种情况来编写FPGA程序。驱动程序在注册网卡设备的时候默认开启GSO：`NETIF_F_GSO`，驱动程序会根据网卡硬件是否支持来设置TSO：`NETIF_F_TSO`。

# checksum offloading
mac地址字节序，
![248](https://img-blog.csdnimg.cn/2019121316005778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
UDP帧格式，
![257](https://img-blog.csdnimg.cn/20191216111442976.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
TCP帧格式，
![258](https://img-blog.csdnimg.cn/20191216111529640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
IP头校验，
```bash
原始         csum清零     16bit组合   取反        求和         32bit组合   取反        求和
0  1  2  3   0  1  2  3   0 1   2 3   0 1   2 3                0 1 2 3     0 1 2 3  
45 00 00 7a  45 00 00 7a  0045  7a00  ffba  85ff  7+d161=d168  7a000045    85ffffba    4+7BAA+55BA=d168
3c 52 40 00  3c 52 40 00  523c  0040  adc3  ffbf               0040523c    ffbfadc3  
40 06 68 d1  40 06 00 00  0640  0000  f9bf  ffff               00000640    fffff9bf  
c0 a8 0a 08  c0 a8 0a 08  a8c0  080a  573f  f7f5               080aa8c0    f7f5573f  
c0 a8 0a 02  c0 a8 0a 02  a8c0  020a  573f  fdf5               020aa8c0    fdf5573f
```
TCP，UDP校验，其中UDP报文长度可从报文直接提取，而TCP由IP报文长度减去IP头长度得到，
```bash
0000   30 09 f9 20 23 64 00 0a 35 00 00 c8 08 00 45 00
0010   00 3c 59 e2 40 00 40 06 4b 7f c0 a8 0a 08 c0 a8
0020   0a 02 8b 20 1f 90 5f 9b fd 3d 00 00 00 00 a0 02
0030   72 10 94 23 00 00 02 04 05 b4 04 02 08 0a 00 08
0040   a4 df 00 00 00 00 01 03 03 07

伪首部
c0 a8 0a 08  a8c0 080a
c0 a8 0a 02  a8c0 020a
00 06 00 28  0600 2800

TCP
8b 20 1f 90 5f 9b fd 3d 00 00 00 00 a0 02 72 10    208b    901f 9b5f 3dfd 0000 0000 02a0 1072
94 23 00 00 02 04 05 b4 04 02 08 0a 00 08 a4 df    2394(0) 0000 0402 b405 0204 0a08 0800 dfa4
00 00 00 00 01 03 03 07                            0000    0000 0301 0703
```
# IP分片
使用iperf3测UDP带宽时，8192字节的包超过MSS，导致IP分片，抓包发现带UDP包头的包在最后，不是《TCP/IP详解》中说的在最前面，或者说，我们需要需要通过off字段来重组，目前本人能想到的设计，接收端需要n个buffer，占用资源量又增加了，后面分片重组时，还得考虑流水线的出口的dma需要支持多通道Stream，按照off来把数据搬到正确的内存地址。在计算/检验（发送/接收）checksum的时候，每一片都要计算/检验IP头checksum，累计到最后一个片计算/检验UDP的checksum。显然把UDP包头放在最后一个片有利于计算/检验checksum。
![271](https://img-blog.csdnimg.cn/20200107161641919.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
