# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [tcpdump & libpcap官网](https://www.tcpdump.org/)
> [使用PCAP获取数据包纳秒（ns）级精度的时间戳（timestamp）](http://www.hesonglin.com/2017/11/09/%E4%BD%BF%E7%94%A8pcap%E8%8E%B7%E5%8F%96%E6%95%B0%E6%8D%AE%E5%8C%85%E7%BA%B3%E7%A7%92%EF%BC%88ns%EF%BC%89%E7%BA%A7%E7%B2%BE%E5%BA%A6%E7%9A%84%E6%97%B6%E9%97%B4%E6%88%B3%EF%BC%88timestamp%EF%BC%89/)
> [tcpdump/libpcap中捕获数据包的时间戳](https://blog.csdn.net/huaxi1902/article/details/8792084)
> [基于libpcap多网卡抓包编程心得](https://blog.csdn.net/wuyingzhouzhou/article/details/80942996)
> [在LINUX系统下使用libpcap，一些流程](https://blog.csdn.net/zhuguorong11/article/details/52204116)
> [Python-对Pcap文件进行处理，获取指定TCP流](https://www.cnblogs.com/sunpudding/archive/2018/08/26/9538889.html)
> [PCAP文件格式分析(做抓包软件之必备)](https://www.cnblogs.com/VseYoung/p/pcap.html)
> [tcpdump使用方法总结](https://www.cnblogs.com/gsophy/p/10119201.html)

# pcap文件格式
## Pcap文件头24B各字段说明：
- Magic：4B：0x1A 2B 3C 4D:用来标示文件的开始
- Major：2B，0x02 00:当前文件主要的版本号
- Minor：2B，0x04 00当前文件次要的版本号
- ThisZone：4B当地的标准时间；全零
- SigFigs：4B时间戳的精度；全零
- SnapLen：4B最大的存储长度
- LinkType：4B链路类型

常用类型：
- 0 BSD loopback devices, except for later OpenBSD
- 1            Ethernet, and Linux loopback devices
- 6            802.5 Token Ring
- 7            ARCnet
- 8            SLIP
- 9            PPP
- 10           FDDI
- 100         LLC/SNAP-encapsulated ATM
- 101         “raw IP”, with no link
- 102         BSD/OS SLIP
- 103         BSD/OS PPP
- 104         Cisco HDLC
- 105         802.11
- 108         later OpenBSD loopback devices (with the AF_value in network byte order)
- 113         special Linux “cooked” capture
- 114         LocalTalk

## Packet 包头和Packet数据组成
字段说明：
Timestamp：时间戳高位，精确到seconds
Timestamp：时间戳低位，精确到microseconds
Caplen：当前数据区的长度，即抓取到的数据帧长度，由此可以得到下一个数据帧的位置。
Len：离线数据长度：网络中实际数据帧的长度，一般不大于caplen，多数情况下和Caplen数值相等。
Packet 数据：即 Packet（通常就是链路层的数据帧）具体内容，长度就是Caplen，这个长度的后面，就是当前PCAP文件中存放的下一个Packet数据包，PCAP文件里面并没有规定捕获的Packet数据包之间有什么间隔字符串，下一组数据在文件中的起始位置，我们需要靠第一个Packet包确定。

# 使用
先编译libpcap，这里交叉编译，用于arm平台，
```shell
./configure --host=arm-xilinx-linux-gnueabi --prefix=$cur_path/zynq-$PETALINUX_VER
```
同样方法，交叉编译tcpdump，可以自动发现libpcap，采用静态编译的库，可直接拷贝到板卡运行，
```shell
...
checking for local pcap library... ../libpcap-1.9.0/libpcap.a
checking for pcap-config... ../libpcap-1.9.0/pcap-config
...
```
# 包格式
pcap文件的包格式，`caplen`为捕获到长度，`len`为数据包原始长度，也就是说`caplen`可能比`len`小，`struct timeval`在这里长度为64字节。
```c
/*
 * Generic per-packet information, as supplied by libpcap.
 *
 * The time stamp can and should be a "struct timeval", regardless of
 * whether your system supports 32-bit tv_sec in "struct timeval",
 * 64-bit tv_sec in "struct timeval", or both if it supports both 32-bit
 * and 64-bit applications.  The on-disk format of savefiles uses 32-bit
 * tv_sec (and tv_usec); this structure is irrelevant to that.  32-bit
 * and 64-bit versions of libpcap, even if they're on the same platform,
 * should supply the appropriate version of "struct timeval", even if
 * that's not what the underlying packet capture mechanism supplies.
 */
struct pcap_pkthdr {
	struct timeval ts;	/* time stamp */
	bpf_u_int32 caplen;	/* length of portion present */
	bpf_u_int32 len;	/* length this packet (off wire) */
};
```
`libpcap-1.9.0`和`tcpdump-4.9.2`支持设置时间精度为毫秒或者纳秒，左边毫秒，右边纳秒，
![181](https://img-blog.csdnimg.cn/20190817153821404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
代码，
```c
// libpcap-1.9.0\pcap\pcap.h line404
/*
 * Time stamp resolution types.
 * Not all systems and interfaces will necessarily support all of these
 * resolutions when doing live captures; all of them can be requested
 * when reading a savefile.
 */
#define PCAP_TSTAMP_PRECISION_MICRO	0	/* use timestamps with microsecond precision, default */
#define PCAP_TSTAMP_PRECISION_NANO	1	/* use timestamps with nanosecond precision */

// tcpdump-4.9.2\tcpdump.c line945
static pcap_t *
open_interface(const char *device, netdissect_options *ndo, char *ebuf)
{
...
#ifdef HAVE_PCAP_SET_TSTAMP_PRECISION
	status = pcap_set_tstamp_precision(pc, ndo->ndo_tstamp_precision);
	if (status != 0)
		error("%s: Can't set %ssecond time stamp precision: %s",
			device,
			tstamp_precision_to_string(ndo->ndo_tstamp_precision),
			pcap_statustostr(status));
#endif
...
}
// tcpdump-4.9.2\tcpdump.c line1495 main
#ifdef HAVE_PCAP_SET_TSTAMP_PRECISION
		case OPTION_TSTAMP_PRECISION:
			ndo->ndo_tstamp_precision = tstamp_precision_from_string(optarg);
			if (ndo->ndo_tstamp_precision < 0)
				error("unsupported time stamp precision");
			break;
#endif
```
# 遍历网卡
代码，
```c
	char errbuf[PCAP_ERRBUF_SIZE];//存放错误信息的缓冲
	pcap_if_t *it;
	int r;
	
	r=pcap_findalldevs(&it,errbuf);
 	if(r < 0) {
	  printf("err:%s\n",errbuf);
	  return r;
	}
	
	while(it) {
	  printf(":%s\n",it->name);
	  it=it->next;
	}

	pcap_freealldevs(it);
```
其中`pcap_if_t`的`name`成员用于`pcap_open_live`函数，
```c
/*
 * Item in a list of interfaces.
 */
struct pcap_if {
	struct pcap_if *next;
	char *name;		/* name to hand to "pcap_open_live()" */
	char *description;	/* textual description of interface, or NULL */
	struct pcap_addr *addresses;
	bpf_u_int32 flags;	/* PCAP_IF_ interface flags */
};
```
# 过滤
只抓数据部分：`src host 192.168.6.6 and src port 8080 and tcp[tcpflags] & (tcp-push) != 0`，
```c
int pcap_compile(pcap_t *p, struct bpf_program *fp,char *str, int optimize, bpf_u_int32 netmask)
```
字符串str是过滤参数，program参数是一个指向bpf_program结构体的指针，optimize参数用于控制是否采用最优化的结果，netmask用于指定IPv4的网络子网掩码，这个参数仅仅在检查过滤程序中的IPv4广播地址时才会使用。
```c
struct bpf_program filter;
pcap_compile(fip->nic, &filter, fip->filter, 1, 0);
pcap_setfilter(fip->nic, &filter);
```
只抓取网卡接收到的包，不抓取发出去的包，0xac是本机的mac地址，过滤源mac地址不等于0xac就达到效果了。
```shell
$ sudo tcpdump -i enp1s0 -v "ether[6] != 0xac" -w a.pcap
```
抓取报文后隔指定的时间保存一次，
- G 选项后面接时间，单位为秒
- Z (大写) 表示下面的新文件也是用root权限来执行的
- w 抓包的名字以时间戳命名 `%Y_%m%d_%H%M_%S.cap`
- s 指定数据包截断的长度，0表示不截断，tcpdump默认截断68字节
```shell
$ sudo tcpdump -s 0 -G 60 -Z root -w %Y_%m%d_%H%M_%S.cap
```
抓取报文后达到指定的大小保存一次，
- C（大写）表示每当文件达到指定大小时进行重新保存一个新文件，单位是MB（1 000 000 B）
- Z (大写) 表示下面的新文件也是用root权限来执行的，如果用 - C 时必须配合-Z（大写Z）
- w 直接将分组写入文件中
- W 限制文件的个数，达到个数后开始从最早的文件覆盖
```shell
$ sudo tcpdump -s 0 -C 1 -Z root -W 1 -w test.cap
```

