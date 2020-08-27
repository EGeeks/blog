# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 声明
参考[level-ip](https://github.com/saminiir/level-ip)翻译加工。

# 以太网帧格式
[MAC地址的分类](https://blog.csdn.net/weixin_39055688/article/details/79883850)
- 单播MAC  （01001100）  第一个字节的最后一位是0
- 组播MAC  （00000001）  第一个字节的最后一位是1
- 广播MAC    （11111111） 全F

以太网的各种标准构成了现在计算机局域网的基础，以太网标准成型于Digital Equipment Corporation, Intel and Xerox in 1980提出的ethernet 2标准。相比现在的以太网，初代的以太网速度很慢，而且是半双工的，所以有介质访问（MAC）协议，即使现在，在半双工模式里，依然需要CSMA/CD协议。从100BASE-T开始，以太网变成全双工模式，另外，以太网交换机的出现使得CSMA/CD协议在大多数情况下不再需要。以太网标准由IEEE 802.33工作组负责维护。
下面是以太网枕头格式，以C语言来描述：
```c
#include <linux/if_ether.h>

struct eth_hdr
{
    unsigned char dmac[6];
    unsigned char smac[6];
    uint16_t ethertype;
    unsigned char payload[];
} __attribute__((packed));
```
dmac和smac从命名就可以看出来，分别是目的和源的MAC地址，后面多余的两字节，如果他的值大于等于1536，则表示payload的类型，IPv4或ARP，否则，表示payload的长度。在type字段后，有可能存在一些其他的以太网帧的标志字段，用于表示VLAN或者QoS帧等，我们自定义协议栈里不实现这些功能，所以代码里没有体现，payload字段指向以太网帧的负载，在我们的程序里，这可能是ARP或者IPv4包，如果payload的长度小于48字节，我们会补一些空白字节在包尾以满足最小帧长度64字节的需求。最后，以太网帧结尾为帧校验FCS字段，是整个帧的CRC校验，这一部分通常为硬件实现，代码里不计算FCS。
以太网帧的接收分流处理，用简单的强制转换实现：
```c
struct eth_hdr *hdr = (struct eth_hdr *) buf;
```
为了可移植需求，用宏来实现：
```c
if (tun_read(buf, BUFLEN) < 0) {
    print_error("ERR: Read from tun_fd: %s\n", strerror(errno));
}
struct eth_hdr *hdr = init_eth_hdr(buf);
handle_frame(&netdev, hdr);
```
handle_frame函数将分析ethertype字段，来决定下一步的处理。在net/netdev.c
```c
/* L2 protocol parsing */
void net_in(struct netdev *dev, struct pkbuf *pkb)
{
	struct ether *ehdr = eth_init(dev, pkb);
	if (!ehdr)
		return;
	l2dbg(MACFMT " -> " MACFMT "(%s)",
				macfmt(ehdr->eth_src),
				macfmt(ehdr->eth_dst),
				ethpro(pkb->pk_pro));
	pkb->pk_indev = dev;
	switch (pkb->pk_pro) {
	case ETH_P_RARP:
//		rarp_in(dev, pkb);
		break;
	case ETH_P_ARP:
		arp_in(dev, pkb);
		break;
	case ETH_P_IP:
		ip_in(dev, pkb);
		break;
	default:
		l2dbg("drop unkown-type packet");
		free_pkb(pkb);
		break;
	}
}
```
net_in函数在net/veth.c中：
```c
static void veth_rx(void)
{
	struct pkbuf *pkb = alloc_netdev_pkb(veth);
	if (veth_recv(pkb) > 0)
		net_in(veth, pkb);	/* pass to upper */
	else
		free_pkb(pkb);
}
void veth_poll(void)
{
	struct pollfd pfd = {};
	int ret;
	while (1) {
		pfd.fd = tap->fd;
		pfd.events = POLLIN;
		pfd.revents = 0;
		/* one event, infinite time */
		ret = poll(&pfd, 1, -1);
		if (ret <= 0)
			perrx("poll /dev/net/tun");
		/* get a packet and handle it */
		veth_rx();
	}
}
```

# ARP协议

ARP即地址解析协议（Address Resolution Protocol），用来动态获取协议地址（IPv4）对应的48bit以太网MAC地址，有了ARP协议，很多L3层的协议才能使用，不仅仅是IPv4，CHAOS协议对应16bit的协议地址。通常情况下，已知LAN中某服务的IP地址，为了建立通信，还需要知道MAC地址，ARP用来广播查询整个网络，要求该IPv4地址的拥有者返回它的MAC地址。
ARP包格式为：
```c
struct arp_hdr
{
    uint16_t hwtype;
    uint16_t protype;
    unsigned char hwsize;
    unsigned char prosize;
    uint16_t opcode;
    unsigned char data[];
} __attribute__((packed));
```
ARP头部字段hwtype，2字节，表示链路层类型，以太网时这个值为0x0001。protype字段，2字节，表示协议类型，IPv4对应0x0800。hwsize和prosize字段，1字节，表示MAC和协议字段的大小，MAC地址对应值为6，IPv4对应值为4。opcode字段，2字节，表示ARP消息的类型，有四种：ARP请求（1），ARP回复（2），RARP请求（3），RARP回复（4），RARP是ARP反过来，由MAC地址得到IP地址，data字段包含了ARP消息的负载，在我们的协议栈中，表示IPv4的特定信息：
```c
struct arp_ipv4
{
    unsigned char smac[6];
    uint32_t sip;
    unsigned char dmac[6];
    uint32_t dip;
} __attribute__((packed));
```
字段意思显而易见，源MAC地址，源IP地址，目的MAC地址，目的IP地址。
ARP协议的算法过程大致如下（参考[ARP协议分析](https://blog.csdn.net/tigerjibo/article/details/7351992)）：

 1. 当主机A向本局域网上的某个主机B发送IP数据报时，就先在自己的ARP缓冲表中查看有无主机B的IP地址。
 2. 如果有，就可以查出其对应的硬件地址，再将此硬件地址写入MAC帧，然后通过以太网将数据包发送到目的主机中。
 3. 如果查不到主机B的IP地址的表项。可能是主机B才入网，也可能是主机A刚刚加电。其高速缓冲表还是空的。在这中情况下，主机A就自动运行ARP。
 - ARP进程在本局域网上广播一个ARP请求分组。ARP请求分组的主要内容是表明：我的IP地址是192.168.0.2，我的硬件地址是00-00-C0-15-AD-18.我想知道IP地址为192.168.0.4的主机的硬件地址。
 - 在本局域网上的所有主机上运行的ARP进行都收到此ARP请求分组。主机B在ARP请求分组中见到自己的IP地址，就向主机A发送ARP响应分组，并写入自己的硬件地址。其余的所有主机都不理睬这个ARP请求分组。ARP响应分组的主要内容是表明：“我的IP地址是192.168.0.4,我的硬件地址是08-00-2B-00-EE-AA”,请注意：虽然ARP请求分组是广播发送的，但ARP响应分组是普通的单播，即从一个源地址发送到一个目的地址。
 - 主机A收到主机B的ARP响应分组后，就在其ARP高速缓冲表中写入主机B的IP地址到硬件地址的映射。

tapip的ARP协议在arp/arp.c中实现：
```c
void arp_recv(struct netdev *dev, struct pkbuf *pkb)
{
	struct ether *ehdr = (struct ether *)pkb->pk_data;
	struct arp *ahdr = (struct arp *)ehdr->eth_data;
	struct arpentry *ae;

	/* real arp process */
	arpdbg(IPFMT " -> " IPFMT, ipfmt(ahdr->arp_sip), ipfmt(ahdr->arp_tip));

	/* drop multi target ip(refer to linux) */
	if (MULTICAST(ahdr->arp_tip)) {
		arpdbg("multicast tip");
		goto free_pkb;
	}

	if (ahdr->arp_tip != dev->net_ipaddr) {
		arpdbg("not for us");
		goto free_pkb;
	}

	ae = arp_lookup(ahdr->arp_pro, ahdr->arp_sip);
	if (ae) {
		/* passive learning(REQUEST): update old arp entry in cache */
		hwacpy(ae->ae_hwaddr, ahdr->arp_sha);
		/* send waiting packet (maybe we receive arp reply) */
		if (ae->ae_state == ARP_WAITING)
			arp_queue_send(ae);
		ae->ae_state = ARP_RESOLVED;
		ae->ae_ttl = ARP_TIMEOUT;
	} else if (ahdr->arp_op == ARP_OP_REQUEST) {
		/* Unsolicited ARP reply is not accepted */
		arp_insert(dev, ahdr->arp_pro, ahdr->arp_sip, ahdr->arp_sha);
	}

	if (ahdr->arp_op == ARP_OP_REQUEST) {
		arp_reply(dev, pkb);
		return;
	}
free_pkb:
	free_pkb(pkb);
}
```
流程很清晰，说一下arp_queue_send，如果ARP是waiting（suspend等等）状态，这个时候的IP包是发送不出去的，在ARP表项更新了之后，把积压的包发送出去，参考[地址转换协议ARP](https://www.cnblogs.com/coder2012/p/3445092.html) 。
在ip/ip_out.c中：
```c
void ip_send_dev(struct netdev *dev, struct pkbuf *pkb)
{
	struct arpentry *ae;
	unsigned int dst;
	struct rtentry *rt = pkb->pk_rtdst;

	if (rt->rt_flags & RT_LOCALHOST) {
		ipdbg("To loopback");
		netdev_tx(dev, pkb, pkb->pk_len - ETH_HRD_SZ,
					ETH_P_IP, dev->net_hwaddr);
		return;
	}

	/* next-hop: default route or remote dst */
	if ((rt->rt_flags & RT_DEFAULT) || rt->rt_metric > 0)
		dst = rt->rt_gw;
	else
		dst = pkb2ip(pkb)->ip_dst;

	ae = arp_lookup(ETH_P_IP, dst);
	if (!ae) {
		arpdbg("not found arp cache");
		ae = arp_alloc();
		if (!ae) {
			ipdbg("arp cache is full");
			free_pkb(pkb);
			return;
		}
		ae->ae_ipaddr = dst;
		ae->ae_dev = dev;
		list_add_tail(&pkb->pk_list, &ae->ae_list);
		arp_request(ae);
	} else if (ae->ae_state == ARP_WAITING) {
		arpdbg("arp entry is waiting");
		list_add_tail(&pkb->pk_list, &ae->ae_list);
	} else {
		netdev_tx(dev, pkb, pkb->pk_len - ETH_HRD_SZ,
				ETH_P_IP, ae->ae_hwaddr);
	}
}
```
实验一下（tapip TOP1）：
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ sudo ./tapip 
[18671]loop_dev_init lo ip address: 127.0.0.1
[18671]loop_dev_init lo netmask:    255.0.0.0
[18671]getname_tap net device: tap0
[18671]getmtu_tap mtu: 1500
[18671]veth_dev_init veth ip address: 192.168.116.138
[18671]veth_dev_init veth hw address: 00:34:45:67:89:ab
[18671]arp_cache_init ARP CACHE INIT
[18671]arp_cache_init ARP CACHE SEMAPHORE INIT
[18671]rt_init route table init
[18671]raw_init raw ip init
[18671]net_stack_run thread 0: net_timer
[18671]net_stack_run thread 1: tcp_timer
[18671]net_stack_run thread 2: netdev_interrupt
[18671]net_stack_run thread 3: shell worker
[net shell]: debug arp
enter ^C to exit debug mode
[18674]arp_recv arp 192.168.116.1 -> 192.168.116.2
[18674]arp_recv arp not for us
[18674]arp_recv arp 192.168.116.130 -> 192.168.116.130
[18674]arp_recv arp not for us
[18674]arp_recv arp 192.168.116.130 -> 192.168.116.2
[18674]arp_recv arp not for us
[18674]arp_recv arp 192.168.116.1 -> 192.168.116.2
[18674]arp_recv arp not for us
[18674]arp_recv arp 192.168.116.1 -> 192.168.116.2
[18674]arp_recv arp not for us
[18674]arp_recv arp 192.168.116.1 -> 192.168.116.138
[18674]arp_lookup arp pro:2048 192.168.116.1
[18674]arp_reply arp replying arp request
[18674]arp_lookup arp pro:2048 192.168.116.1
[18674]arp_recv arp 192.168.116.1 -> 192.168.116.2
[18674]arp_recv arp not for us
[18674]arp_lookup arp pro:2048 192.168.116.1
[18674]arp_lookup arp pro:2048 192.168.116.1
[18674]arp_recv arp 192.168.116.1 -> 192.168.116.2
[18674]arp_recv arp not for us
[18674]arp_recv arp 192.168.116.1 -> 192.168.116.2
[18674]arp_recv arp not for us
[18674]arp_lookup arp pro:2048 192.168.116.1
[18674]arp_recv arp 192.168.116.1 -> 192.168.116.2
[18674]arp_recv arp not for us
^C
exit debug mode
[net shell]: exit
[18675]shell_worker shell worker exit
```
在host上ping tap0 IP地址：
![这里写图片描述](https://img-blog.csdn.net/20180528232407711?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
host查看结果：
```bash
Microsoft Windows [版本 6.1.7601]
版权所有 (c) 2009 Microsoft Corporation。保留所有权利。

C:\Users\aj>ping 192.168.116.138

正在 Ping 192.168.116.138 具有 32 字节的数据:
来自 192.168.116.138 的回复: 字节=32 时间=1ms TTL=128
来自 192.168.116.138 的回复: 字节=32 时间<1ms TTL=128
来自 192.168.116.138 的回复: 字节=32 时间<1ms TTL=128
来自 192.168.116.138 的回复: 字节=32 时间<1ms TTL=128

192.168.116.138 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 0ms，最长 = 1ms，平均 = 0ms

C:\Users\aj>arp -a

接口: 192.168.0.103 --- 0xd
  Internet 地址         物理地址              类型
  192.168.0.1           ec-26-ca-6a-76-2a     动态
  192.168.0.255         ff-ff-ff-ff-ff-ff     静态
  224.0.0.22            01-00-5e-00-00-16     静态
  224.0.0.251           01-00-5e-00-00-fb     静态
  224.0.0.252           01-00-5e-00-00-fc     静态
  239.255.255.250       01-00-5e-7f-ff-fa     静态
  255.255.255.255       ff-ff-ff-ff-ff-ff     静态

接口: 192.168.70.1 --- 0x11
  Internet 地址         物理地址              类型
  192.168.70.254        00-50-56-eb-a6-c5     动态
  192.168.70.255        ff-ff-ff-ff-ff-ff     静态
  224.0.0.22            01-00-5e-00-00-16     静态
  224.0.0.251           01-00-5e-00-00-fb     静态
  224.0.0.252           01-00-5e-00-00-fc     静态
  239.255.255.250       01-00-5e-7f-ff-fa     静态
  255.255.255.255       ff-ff-ff-ff-ff-ff     静态

接口: 192.168.116.1 --- 0x12
  Internet 地址         物理地址              类型
  192.168.116.128       00-0c-29-f6-52-51     动态
  192.168.116.130       00-0c-29-f6-52-47     动态
  192.168.116.138       00-34-45-67-89-ab     动态
  192.168.116.255       ff-ff-ff-ff-ff-ff     静态
  224.0.0.22            01-00-5e-00-00-16     静态
  224.0.0.251           01-00-5e-00-00-fb     静态
  224.0.0.252           01-00-5e-00-00-fc     静态
  239.255.255.250       01-00-5e-7f-ff-fa     静态
```
tapip TOP2，打开tty1
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ sudo ./tapip 
[42979]loop_dev_init lo ip address: 127.0.0.1
[42979]loop_dev_init lo netmask:    255.0.0.0
[42979]getname_tap net device: tap0
[42979]getmtu_tap mtu: 1500
[42979]gethwaddr_tap mac addr: ba:f2:bd:65:51:3a
[42979]setipaddr_tap set IPaddr: 10.0.0.2
[42979]getipaddr_tap get IPaddr: 10.0.0.2
[42979]setnetmask_tap set Netmask: 255.255.255.0
[42979]setup_tap ifup tap0
[42979]veth_dev_init veth ip address: 10.0.0.1
[42979]veth_dev_init veth hw address: 00:34:45:67:89:ab
[42979]arp_cache_init ARP CACHE INIT
[42979]arp_cache_init ARP CACHE SEMAPHORE INIT
[42979]rt_init route table init
[42979]raw_init raw ip init
[42979]net_stack_run thread 0: net_timer
[42979]net_stack_run thread 1: tcp_timer
[42979]net_stack_run thread 2: netdev_interrupt
[42979]net_stack_run thread 3: shell worker
[net shell]: debug arp
enter ^C to exit debug mode
[42983]arp_recv arp 10.0.0.2 -> 10.0.0.1
[42983]arp_lookup arp pro:2048 10.0.0.2
[42983]arp_reply arp replying arp request
[42983]arp_recv arp 10.0.0.2 -> 10.0.0.1
[42983]arp_lookup arp pro:2048 10.0.0.2
[42983]arp_reply arp replying arp request
[42983]arp_recv arp 10.0.0.2 -> 10.0.0.1
[42983]arp_lookup arp pro:2048 10.0.0.2
[42983]arp_reply arp replying arp request
[42983]arp_recv arp 10.0.0.2 -> 10.0.0.1
[42983]arp_lookup arp pro:2048 10.0.0.2
[42983]arp_reply arp replying arp request
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp not found arp cache
[42983]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp not found arp cache
[42983]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_queue_drop arp drop pending packet
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp not found arp cache
[42983]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp not found arp cache
[42983]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp not found arp cache
[42983]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp not found arp cache
[42983]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
[42983]arp_lookup arp pro:2048 10.0.0.2
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp not found arp cache
[42983]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42983]arp_lookup arp pro:2048 10.0.0.255
[42983]ip_send_dev arp arp entry is waiting
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_request arp 10.0.0.1(00:34:45:67:89:ab)->10.0.0.255(request)
[42981]arp_queue_drop arp drop pending packet
[42981]arp_queue_drop arp drop pending packet
#这儿是tty2执行了ping之后的打印
[42983]arp_lookup arp pro:2048 10.0.0.2
[42983]arp_lookup arp pro:2048 10.0.0.2
[42983]arp_lookup arp pro:2048 10.0.0.2
```
打开tty2
```bash
zc@ubuntu:~$ arping 10.0.0.1 -I tap0
ARPING 10.0.0.1 from 10.0.0.2 tap0
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  1.573ms
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  0.957ms
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  0.941ms
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  0.910ms
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  0.918ms
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  0.877ms
^CSent 6 probes (1 broadcast(s))
Received 6 response(s)
zc@ubuntu:~$ arping 10.0.0.1 -I tap0
ARPING 10.0.0.1 from 10.0.0.2 tap0
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  0.720ms
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  0.869ms
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  0.894ms
Unicast reply from 10.0.0.1 [00:34:45:67:89:AB]  2.191ms
^CSent 4 probes (1 broadcast(s))
Received 4 response(s)
zc@ubuntu:~$ arp
Address                  HWtype  HWaddress           Flags Mask            Iface
bogon                    ether   00:34:45:67:89:ab   C                     tap0
bogon                    ether   00:50:56:fb:3c:1f   C                     ens33
bogon                    ether   00:50:56:c0:00:08   C                     ens33
bogon                    ether   00:50:56:e8:f9:29   C                     ens33
bogon                    ether   00:34:45:67:89:ab   C                     ens33
zc@ubuntu:~$ arp -a
bogon (10.0.0.1) at 00:34:45:67:89:ab [ether] on tap0
bogon (192.168.116.2) at 00:50:56:fb:3c:1f [ether] on ens33
bogon (192.168.116.1) at 00:50:56:c0:00:08 [ether] on ens33
bogon (192.168.116.254) at 00:50:56:e8:f9:29 [ether] on ens33
bogon (192.168.116.138) at 00:34:45:67:89:ab [ether] on ens33
zc@ubuntu:~$ ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=0.845 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=0.364 ms
64 bytes from 10.0.0.1: icmp_seq=3 ttl=64 time=0.327 ms
^C
--- 10.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 0.327/0.512/0.845/0.235 ms
zc@ubuntu:~$ 
```
