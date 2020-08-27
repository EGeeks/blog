# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

目前tapip协议栈只支持IPv4，不支持IPv6，另一个比较好的协议栈是uIP，超轻量级（lwIP），广泛用于8位单片机系统，也可以在linux用户态运行，uIP现在已经支持IPv6，

# IPv4
IPv4是L3层协议，是传输层协议TCP和UDP的基础，它是无连接的，不像TCP，IPv4数据包在网络协议栈中单独处理，所以IP数据包可能会乱序。IP数据包同样不保证成功送达，类似于UDP。如果你需要通信的可靠性，则可以使用TCP协议，它构建于IP之上，高层协议来负责错误检测。
IP数据包头格式：
```c
struct iphdr {
    uint8_t version : 4;
    uint8_t ihl : 4;
    uint8_t tos;
    uint16_t len;
    uint16_t id;
    uint16_t flags : 3;
    uint16_t frag_offset : 13;
    uint8_t ttl;
    uint8_t proto;
    uint16_t csum;
    uint32_t saddr;
    uint32_t daddr;
} __attribute__((packed));
```
IPv4包头长度一般为20字节，IPv4头也可能会包含一些附加选项，tapip不考虑这种情况，version字段表示包头类型，这里值为4，表示IPv4，ihl（Internet Header Length）字段表示IP包头的长度（ihl * 4字节），ihl字段长度为4bit，所以表示的最大包头长度为60字节，tos（type of service）字段在IP协议的发展过程中被分成了几个更小的字段，表示IP的服务质量，len字段表示整个IP数据报文长度，2字节，所以最大长度为65535字节，大的IP数据报文会被分段，为了满足通信接口的MTU，id字段用来标志报文，基本上是用于将分段的IP数据包重新拼接起来，发送端将这个字段递增，接收端根据这个来重排序IP分段报文，flag字段表示IP报文的控制标志，发送者通过设置这个字段来表示报文是否可以被分段传输，是否是最后一个分段或者中间的分段，flag_offset字段表示分段在报文中的位置，所以第一个报文的值为0，ttl（time to live）字段，[它告诉网络，数据包在网络中的时间是否太长而应被丢弃。有很多原因使包在一定时间内不能被传递到目的地。解决方法就是在一段时间后丢弃这个包，然后给发送者一个报文，由发送者决定是否要重发。TTL的初值通常是系统缺省值，是包头中的8位的域。TTL的最初设想是确定一个时间范围，超过此时间就把包丢弃。由于每个路由器都至少要把TTL域减一，TTL通常表示包在被丢弃前最多能经过的路由器个数。当记数到0时，路由器决定丢弃该包，并发送一个ICMP报文给最初的发送者](https://blog.csdn.net/zhongguoren666/article/details/7377732)。proto字段表示IP数据负载的协议类型，通常这个字段值为16（UDP）或6（TCP）。csum（header checksum）字段用来保证IP头的完整性。saddr和daddr分别表示源IP地址和目的IP地址，尽管这个字段有32bit，可以提供大概45亿的地址，但即将被用尽，IPv6协议扩展了地址长度到128bit，保证了地址范围不被耗尽。
校验和的计算，先设置csum为0，然后按16bit字段计算IP包头的和，
```c
uint16_t checksum(void *addr, int count)
{
    /* Compute Internet Checksum for "count" bytes
     *         beginning at location "addr".
     * Taken from https://tools.ietf.org/html/rfc1071
     */

    register uint32_t sum = 0;
    uint16_t * ptr = addr;

    while( count > 1 )  {
        /*  This is the inner loop */
        sum += * ptr++;
        count -= 2;
    }

    /*  Add left-over byte, if any */
    if( count > 0 )
        sum += * (uint8_t *) ptr;

    /*  Fold 32-bit sum to 16 bits */
    while (sum>>16)
        sum = (sum & 0xffff) + (sum >> 16);

    return ~sum;
}
```
当校验和计算出来后，填入csum字段，这时再用函数计算，若结果为0，则表示IP包头正确，否则错误。

# ICMPv4
由于IP协议不保证可靠性，所以需要一些方法来获知网络是否畅通，ICMPv4（Internet Control Message Protocol version 4）被用来诊断测量网络状态，比如，如果一个网关无法连接，网络协议栈发现之后，会发送一个“Gateway Unreachable” 消息给对方。
ICMP包头格式，ICMP是IP包的负载：
```c
struct icmp_v4 {
    uint8_t type;
    uint8_t code;
    uint16_t csum;
    uint8_t data[];
} __attribute__((packed));
```
type字段，表示消息的任务类型，共有42种取值，常用的有8种，tapip实现了0（Echo Reply），3（Destination Unreachable），5（Redirect）关于redirect功能参考[关于ICMP Redirect路由的一个不是bug的bug](https://blog.csdn.net/dog250/article/details/49977507)，8（Echo Request）。code字段进一步描述消息的任务，比如，当type为3（Destination Unreachable）时，code字段表示Destination Unreachable的原因，例如当包无法路由到网络：发送方一般会收到type为3 code为0的ICMP的消息。csum字段是ICMP包的校验和，计算方法和IPv4包头一样，但是这里把payload也计算进去。payload字段包含查询、通知和错误消息。
[我们日常使用最多的ping，就是响应请求（Type=8）和应答（Type=0），一台主机向一个节点发送一个Type=8的ICMP报文，如果途中没有异常（例如被路由器丢弃、目标不回应ICMP或传输失败），则目标返回Type=0的ICMP报文，说明这台主机存在，更详细的tracert通过计算ICMP报文通过的节点来确定主机与目标之间的网络距离](https://blog.csdn.net/wuheshi/article/details/50973386)。type8和type0的payload格式一样：
```c
struct icmp_v4_echo {
    uint16_t id;
    uint16_t seq;
    uint8_t data[];
} __attribute__((packed));
```
id由host设置，决定哪一个进程处理echo reply，比如设置进程id到这个字段，seq字段是响应请求包的序列号，简单的从0开始递增每当新的响应请求包建立，可以用来判断包是否丢失或者发送时乱序。data字段是可选的，通常包含了响应请求的时间戳，可以用来估算两者之间来回程的时间。
最常见的ICMPv4错误消息Destination Unreachable type3的payload格式：
```c
struct icmp_v4_dst_unreachable {
    uint8_t unused;
    uint8_t len;
    uint16_t var;
    uint8_t data[];
} __attribute__((packed));
```
第一个字节不用，len代表数据报文长度，以4字节为单位，var字段取决图ICMP code字段，data字段尽可能存放引起Destination Unreachable 状态的IP报文。
tapip在收到ICMP type0包处理，在ip/icmp.c中：
```c
static void icmp_echo_request(struct icmp_desc *icmp_desc, struct pkbuf *pkb)
{
	struct ip *iphdr = pkb2ip(pkb);
	struct icmp *icmphdr = ip2icmp(iphdr);
	icmpdbg("echo request data %d bytes icmp_id %d icmp_seq %d",
			(int)(iphdr->ip_len - iphlen(iphdr) - ICMP_HRD_SZ),
			_ntohs(icmphdr->icmp_id),
			_ntohs(icmphdr->icmp_seq));
	if (icmphdr->icmp_code) {
		icmpdbg("echo request packet corrupted");
		free_pkb(pkb);
		return;
	}
	icmphdr->icmp_type = ICMP_T_ECHORLY;
	/*
	 * adjacent the checksum:
	 * If  ~ >>> (cksum + x + 8) >>> == 0
	 * let ~ >>> (cksum` + x ) >>> == 0
	 * then cksum` = cksum + 8
	 */
	if (icmphdr->icmp_cksum >= 0xffff - ICMP_T_ECHOREQ)
		icmphdr->icmp_cksum += ICMP_T_ECHOREQ + 1;
	else
		icmphdr->icmp_cksum += ICMP_T_ECHOREQ;
	iphdr->ip_dst = iphdr->ip_src;	/* ip_src is set by ip_send_out() */
	ip_hton(iphdr);
	/* init reused input packet */
	pkb->pk_rtdst = NULL;
	pkb->pk_indev = NULL;
	pkb->pk_type = PKT_NONE;
	ip_send_out(pkb);
}
```
