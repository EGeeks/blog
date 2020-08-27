# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# TCP传输控制块TCB（Transmission Control Block）
TCP利用TCB保存当前连接的状态，每新建一个连接就需要一个TCB，发送TCP报文相关的变量：
```c
    Send Sequence Variables
	
      SND.UNA - send unacknowledged
      SND.NXT - send next
      SND.WND - send window
      SND.UP  - send urgent pointer
      SND.WL1 - segment sequence number used for last window update
      SND.WL2 - segment acknowledgment number used for last window update
      ISS     - initial send sequence number
```
接收TCP报文相关的变量：
```c
    Receive Sequence Variables
											  
      RCV.NXT - receive next
      RCV.WND - receive window
      RCV.UP  - receive urgent pointer
      IRS     - initial receive sequence number
```
tapip的实现在tcp/tcp.h中：
```c
struct tcp_sock {
	struct sock sk;
	struct hlist_node bhash_list;	/* for bind hash table, e/lhash node is in sk */
	unsigned int bhash;		/* bind hash value */
	int accept_backlog;		/* current entries of accept queue */
	int backlog;			/* size of accept queue */
	struct list_head listen_queue;	/* waiting for second SYN+ACK of three-way handshake */
	struct list_head accept_queue;	/* waiting for third ACK of three-way handshake */
	struct list_head list;
	struct tcp_timer timewait;	/* TIME-WAIT TIMEOUT */
	struct tapip_wait *wait_accept;
	struct tapip_wait *wait_connect;
	struct tcp_sock *parent;
	unsigned int flags;
	struct cbuf *rcv_buf;
	struct list_head rcv_reass;	/* list head of unordered reassembled tcp segments */
	/* transmission control block (RFC 793) */
	unsigned int snd_una;	/* send unacknowledged */
	unsigned int snd_nxt;	/* send next */
	unsigned int snd_wnd;	/* send window */
	unsigned int snd_up;	/* send urgent pointer */
	unsigned int snd_wl1;	/* seq for last window update */
	unsigned int snd_wl2;	/* ack for last window update */
	unsigned int iss;	/* initial send sequence number */
	unsigned int rcv_nxt;	/* receive next */
	unsigned int rcv_wnd;	/* receive window */
	unsigned int rcv_up;	/* receive urgent pointer */
	unsigned int irs;	/* initial receive sequence number */
	unsigned int state;	/* connection state */
};
```
另外还有一些处理当前报文的变量：
```c
    Current Segment Variables
	
      SEG.SEQ - segment sequence number
      SEG.ACK - segment acknowledgment number
      SEG.LEN - segment length
      SEG.WND - segment window
      SEG.UP  - segment urgent pointer
      SEG.PRC - segment precedence value
```
tapip的实现在tcp/tcp.h中：
```c
/* host-order tcp current segment (RFC 793) */
struct tcp_segment {
	unsigned int seq;	/* segment sequence number */
	unsigned int ack;	/* segment acknowledgment number */
	unsigned int lastseq;	/* segment last sequence number */
	unsigned int len;	/* segment length */
	unsigned int dlen;	/* segment text length */
	unsigned int wnd;	/* segment window */
	unsigned int up;	/* segment urgent point */
	unsigned int prc;	/* segment precedence value(no used) */
	unsigned char *text;		/* segment text */
	struct ip *iphdr;
	struct tcp *tcphdr;
};
```

# TCP通信流程
当一个连接建立，数据流的处理就开始了，TCB中的三个重要变量用于追踪状态：

 - SND.NXT 这是发送将要使用的Sequence Number
 - RCV.NXT 这是接收期望接收的下一个Sequence Number
 - SND.UNA 这是发送者记录最老的没有ACK的Sequence Number
 
如果经过很长一段时间，没有传输发生，这三个变量最终会相等，例如TCP A发送一个segment然后增加SND.NXT，TCP B接收报文确认Sequence Number然后发送ACK，TCP A接收ACK，增加SND.UNA。三个变量增加的数量取决于报文中的数据长度，linux上用tcpdump命令来监控TCP的数据流程，参考[tcpdump参数解析及使用详解](https://blog.csdn.net/hzhsan/article/details/43445787)。
tapip TOP1的TCP/UDP通信会有checksum错误。tapip TOP2，打开tty1，
```c
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
[net shell]: snc -b 10.0.0.1:12345
#### User configure  ###############
CONFIG_DEBUG = n
CONFIG_DEBUG_PKB = n
CONFIG_DEBUG_WAIT = n
CONFIG_DEBUG_SOCK = n
CONFIG_DEBUG_ARP_LOCK = n
CONFIG_DEBUG_ICMPEXCFRAGTIME = n
CONFIG_TOPLOGY = 2
#### End of User configure #########

# Use 'make V=1' to see the full commands
ifeq ("$(origin V)", "command line")
	Q =
else
	Q = @
endif
export Q

MAKEFLAGS += --no-print-directory

LD = ld
CC = gcc
CFLAGS = -Wall -I../include
LFLAGS = -pthread
export LD CC CFLAGS

ifeq ($(CONFIG_DEBUG), y)
	CFLAGS += -g
endif

ifeq ($(CONFIG_DEBUG), y)
	CFLAGS += -DDEBUG_PKB
endif

ifeq ($(CONFIG_DEBUG_SOCK), y)
	CFLAGS += -DSOCK_DEBUG
endif

ifeq ($(CONFIG_DEBUG_ICMPEXCFRAGTIME), y)
	CFLAGS += -DICMP_EXC_FRAGTIME_TEST
endif

ifeq ($(CONFIG_DEBUG_WAIT), y)
	CFLAGS += -DWAIT_DEBUG
endif

ifeq ($(CONFIG_DEBUG_ARP_LOCK), y)
	CFLAGS += -DDEBUG_ARPCACHE_LOCK
endif

ifeq ($(CONFIG_TOPLOGY), 1)
	CFLAGS += -DCONFIG_TOP1
else
	CFLAGS += -DCONFIG_TOP2
endif

NET_STACK_OBJS =	shell/shell_obj.o	\
			net/net_obj.o		\
			arp/arp_obj.o		\
			ip/ip_obj.o		\
			socket/socket_obj.o	\
			udp/udp_obj.o		\
			tcp/tcp_obj.o		\
			app/app_obj.o		\
			lib/lib_obj.o

all:tapip
tapip:$(NET_STACK_OBJS)
	@echo " [BUILD] $@"
	$(Q)$(CC) $(LFLAGS) $^ -o $@

shell/shell_obj.o:shell/*.c
	@make -C shell/
net/net_obj.o:net/*.c
	@make -C net/
arp/arp_obj.o:arp/*.c
	@make -C arp/
ip/ip_obj.o:ip/*.c
	@make -C ip/
udp/udp_obj.o:udp/*.c
	@make -C udp/
tcp/tcp_obj.o:tcp/*.c
	@make -C tcp/
lib/lib_obj.o:lib/*.c
	@make -C lib/
socket/socket_obj.o:socket/*.c
	@make -C socket/
app/app_obj.o:app/*.c
	@make -C app/

test:cbuf
# test program for circul buffer
cbuf:lib/cbuf.c lib/lib.c
	@echo " [CC] $@"
	$(Q)$(CC) -DCBUF_TEST -Iinclude/ $^ -o $@

tag:
	ctags -R *

clean:
	find . -name *.o | xargs rm -f
	rm -f tapip cbuf

lines:
	@echo "code lines:"
	@wc -l `find . -name \*.[ch]` | sort -n

[net shell]: 
```
打开tty2，
```bash
cat Makefile | nc 10.0.0.1 12345
```
打开tty3，tcpdump抓包，S为SYN，S.为SYN和ACK，前三个包为three way握手过程，[其中seq和ack为1](https://blog.csdn.net/qq_32503701/article/details/53559759)，表示的是相对位置，方便理解（Relative sequence numbers are shown by default with tcpdump to ease readability），之后开始数据传输。注意：seq = ack + data_length。
```bash
zc@ubuntu:~$ sudo tcpdump -i tap0 tcp -vvv -XX
[sudo] password for zc: 
tcpdump: listening on tap0, link-type EN10MB (Ethernet), capture size 262144 bytes
01:59:30.676806 IP (tos 0x0, ttl 64, id 58146, offset 0, flags [DF], proto TCP (6), length 60)
    bogon.36292 > bogon.12345: Flags [S], cksum 0xbee9 (correct), seq 1403716138, win 29200, options [mss 1460,sackOK,TS val 885176942 ecr 0,nop,wscale 7], length 0
	0x0000:  0034 4567 89ab baf2 bd65 513a 0800 4500  .4Eg.....eQ:..E.
	0x0010:  003c e322 4000 4006 4397 0a00 0002 0a00  .<."@.@.C.......
	0x0020:  0001 8dc4 3039 53ab 022a 0000 0000 a002  ....09S..*......
	0x0030:  7210 bee9 0000 0204 05b4 0402 080a 34c2  r.............4.
	0x0040:  ba6e 0000 0000 0103 0307                 .n........
01:59:30.677062 IP (tos 0x0, ttl 64, id 7, offset 0, flags [none], proto TCP (6), length 40)
    bogon.12345 > bogon.36292: Flags [S.], cksum 0x15ee (correct), seq 12345682, ack 1403716139, win 4096, length 0
	0x0000:  baf2 bd65 513a 0034 4567 89ab 0800 4500  ...eQ:.4Eg....E.
	0x0010:  0028 0007 0000 4006 66c7 0a00 0001 0a00  .(....@.f.......
	0x0020:  0002 3039 8dc4 00bc 6152 53ab 022b 5012  ..09....aRS..+P.
	0x0030:  1000 15ee 0000                           ......
01:59:30.678226 IP (tos 0x0, ttl 64, id 58147, offset 0, flags [DF], proto TCP (6), length 40)
    bogon.36292 > bogon.12345: Flags [.], cksum 0xb3de (correct), seq 1, ack 1, win 29200, length 0
	0x0000:  0034 4567 89ab baf2 bd65 513a 0800 4500  .4Eg.....eQ:..E.
	0x0010:  0028 e323 4000 4006 43aa 0a00 0002 0a00  .(.#@.@.C.......
	0x0020:  0001 8dc4 3039 53ab 022b 00bc 6153 5010  ....09S..+..aSP.
	0x0030:  7210 b3de 0000                           r.....
01:59:30.678426 IP (tos 0x0, ttl 64, id 58148, offset 0, flags [DF], proto TCP (6), length 576)
    bogon.36292 > bogon.12345: Flags [.], cksum 0x8905 (correct), seq 1:537, ack 1, win 29200, length 536
	0x0000:  0034 4567 89ab baf2 bd65 513a 0800 4500  .4Eg.....eQ:..E.
	0x0010:  0240 e324 4000 4006 4191 0a00 0002 0a00  .@.$@.@.A.......
	0x0020:  0001 8dc4 3039 53ab 022b 00bc 6153 5010  ....09S..+..aSP.
	0x0030:  7210 8905 0000 2323 2323 2055 7365 7220  r.....####.User.
	0x0040:  636f 6e66 6967 7572 6520 2023 2323 2323  configure..#####
	0x0050:  2323 2323 2323 2323 2323 0a43 4f4e 4649  ##########.CONFI
	0x0060:  475f 4445 4255 4720 3d20 6e0a 434f 4e46  G_DEBUG.=.n.CONF
	0x0070:  4947 5f44 4542 5547 5f50 4b42 203d 206e  IG_DEBUG_PKB.=.n
	0x0080:  0a43 4f4e 4649 475f 4445 4255 475f 5741  .CONFIG_DEBUG_WA
	0x0090:  4954 203d 206e 0a43 4f4e 4649 475f 4445  IT.=.n.CONFIG_DE
	0x00a0:  4255 475f 534f 434b 203d 206e 0a43 4f4e  BUG_SOCK.=.n.CON
	0x00b0:  4649 475f 4445 4255 475f 4152 505f 4c4f  FIG_DEBUG_ARP_LO
	0x00c0:  434b 203d 206e 0a43 4f4e 4649 475f 4445  CK.=.n.CONFIG_DE
	0x00d0:  4255 475f 4943 4d50 4558 4346 5241 4754  BUG_ICMPEXCFRAGT
	0x00e0:  494d 4520 3d20 6e0a 434f 4e46 4947 5f54  IME.=.n.CONFIG_T
	0x00f0:  4f50 4c4f 4759 203d 2032 0a23 2323 2320  OPLOGY.=.2.####.
	0x0100:  456e 6420 6f66 2055 7365 7220 636f 6e66  End.of.User.conf
	0x0110:  6967 7572 6520 2323 2323 2323 2323 230a  igure.#########.
	0x0120:  0a23 2055 7365 2027 6d61 6b65 2056 3d31  .#.Use.'make.V=1
	0x0130:  2720 746f 2073 6565 2074 6865 2066 756c  '.to.see.the.ful
	0x0140:  6c20 636f 6d6d 616e 6473 0a69 6665 7120  l.commands.ifeq.
	0x0150:  2822 2428 6f72 6967 696e 2056 2922 2c20  ("$(origin.V)",.
	0x0160:  2263 6f6d 6d61 6e64 206c 696e 6522 290a  "command.line").
	0x0170:  0951 203d 0a65 6c73 650a 0951 203d 2040  .Q.=.else..Q.=.@
	0x0180:  0a65 6e64 6966 0a65 7870 6f72 7420 510a  .endif.export.Q.
	0x0190:  0a4d 414b 4546 4c41 4753 202b 3d20 2d2d  .MAKEFLAGS.+=.--
	0x01a0:  6e6f 2d70 7269 6e74 2d64 6972 6563 746f  no-print-directo
	0x01b0:  7279 0a0a 4c44 203d 206c 640a 4343 203d  ry..LD.=.ld.CC.=
	0x01c0:  2067 6363 0a43 464c 4147 5320 3d20 2d57  .gcc.CFLAGS.=.-W
	0x01d0:  616c 6c20 2d49 2e2e 2f69 6e63 6c75 6465  all.-I../include
	0x01e0:  0a4c 464c 4147 5320 3d20 2d70 7468 7265  .LFLAGS.=.-pthre
	0x01f0:  6164 0a65 7870 6f72 7420 4c44 2043 4320  ad.export.LD.CC.
	0x0200:  4346 4c41 4753 0a0a 6966 6571 2028 2428  CFLAGS..ifeq.($(
	0x0210:  434f 4e46 4947 5f44 4542 5547 292c 2079  CONFIG_DEBUG),.y
	0x0220:  290a 0943 464c 4147 5320 2b3d 202d 670a  )..CFLAGS.+=.-g.
	0x0230:  656e 6469 660a 0a69 6665 7120 2824 2843  endif..ifeq.($(C
	0x0240:  4f4e 4649 475f 4445 4255 4729 2c20       ONFIG_DEBUG),.
01:59:30.678432 IP (tos 0x0, ttl 64, id 58149, offset 0, flags [DF], proto TCP (6), length 576)
    bogon.36292 > bogon.12345: Flags [.], cksum 0xa3c4 (correct), seq 537:1073, ack 1, win 29200, length 536
	0x0000:  0034 4567 89ab baf2 bd65 513a 0800 4500  .4Eg.....eQ:..E.
	0x0010:  0240 e325 4000 4006 4190 0a00 0002 0a00  .@.%@.@.A.......
	0x0020:  0001 8dc4 3039 53ab 0443 00bc 6153 5010  ....09S..C..aSP.
	0x0030:  7210 a3c4 0000 7929 0a09 4346 4c41 4753  r.....y)..CFLAGS
	0x0040:  202b 3d20 2d44 4445 4255 475f 504b 420a  .+=.-DDEBUG_PKB.
	0x0050:  656e 6469 660a 0a69 6665 7120 2824 2843  endif..ifeq.($(C
	0x0060:  4f4e 4649 475f 4445 4255 475f 534f 434b  ONFIG_DEBUG_SOCK
	0x0070:  292c 2079 290a 0943 464c 4147 5320 2b3d  ),.y)..CFLAGS.+=
	0x0080:  202d 4453 4f43 4b5f 4445 4255 470a 656e  .-DSOCK_DEBUG.en
	0x0090:  6469 660a 0a69 6665 7120 2824 2843 4f4e  dif..ifeq.($(CON
	0x00a0:  4649 475f 4445 4255 475f 4943 4d50 4558  FIG_DEBUG_ICMPEX
	0x00b0:  4346 5241 4754 494d 4529 2c20 7929 0a09  CFRAGTIME),.y)..
	0x00c0:  4346 4c41 4753 202b 3d20 2d44 4943 4d50  CFLAGS.+=.-DICMP
	0x00d0:  5f45 5843 5f46 5241 4754 494d 455f 5445  _EXC_FRAGTIME_TE
	0x00e0:  5354 0a65 6e64 6966 0a0a 6966 6571 2028  ST.endif..ifeq.(
	0x00f0:  2428 434f 4e46 4947 5f44 4542 5547 5f57  $(CONFIG_DEBUG_W
	0x0100:  4149 5429 2c20 7929 0a09 4346 4c41 4753  AIT),.y)..CFLAGS
	0x0110:  202b 3d20 2d44 5741 4954 5f44 4542 5547  .+=.-DWAIT_DEBUG
	0x0120:  0a65 6e64 6966 0a0a 6966 6571 2028 2428  .endif..ifeq.($(
	0x0130:  434f 4e46 4947 5f44 4542 5547 5f41 5250  CONFIG_DEBUG_ARP
	0x0140:  5f4c 4f43 4b29 2c20 7929 0a09 4346 4c41  _LOCK),.y)..CFLA
	0x0150:  4753 202b 3d20 2d44 4445 4255 475f 4152  GS.+=.-DDEBUG_AR
	0x0160:  5043 4143 4845 5f4c 4f43 4b0a 656e 6469  PCACHE_LOCK.endi
	0x0170:  660a 0a69 6665 7120 2824 2843 4f4e 4649  f..ifeq.($(CONFI
	0x0180:  475f 544f 504c 4f47 5929 2c20 3129 0a09  G_TOPLOGY),.1)..
	0x0190:  4346 4c41 4753 202b 3d20 2d44 434f 4e46  CFLAGS.+=.-DCONF
	0x01a0:  4947 5f54 4f50 310a 656c 7365 0a09 4346  IG_TOP1.else..CF
	0x01b0:  4c41 4753 202b 3d20 2d44 434f 4e46 4947  LAGS.+=.-DCONFIG
	0x01c0:  5f54 4f50 320a 656e 6469 660a 0a4e 4554  _TOP2.endif..NET
	0x01d0:  5f53 5441 434b 5f4f 424a 5320 3d09 7368  _STACK_OBJS.=.sh
	0x01e0:  656c 6c2f 7368 656c 6c5f 6f62 6a2e 6f09  ell/shell_obj.o.
	0x01f0:  5c0a 0909 096e 6574 2f6e 6574 5f6f 626a  \....net/net_obj
	0x0200:  2e6f 0909 5c0a 0909 0961 7270 2f61 7270  .o..\....arp/arp
	0x0210:  5f6f 626a 2e6f 0909 5c0a 0909 0969 702f  _obj.o..\....ip/
	0x0220:  6970 5f6f 626a 2e6f 0909 5c0a 0909 0973  ip_obj.o..\....s
	0x0230:  6f63 6b65 742f 736f 636b 6574 5f6f 626a  ocket/socket_obj
	0x0240:  2e6f 095c 0a09 0909 7564 702f 7564       .o.\....udp/ud
01:59:30.678434 IP (tos 0x0, ttl 64, id 58150, offset 0, flags [DF], proto TCP (6), length 576)
    bogon.36292 > bogon.12345: Flags [.], cksum 0x19a9 (correct), seq 1073:1609, ack 1, win 29200, length 536
	0x0000:  0034 4567 89ab baf2 bd65 513a 0800 4500  .4Eg.....eQ:..E.
	0x0010:  0240 e326 4000 4006 418f 0a00 0002 0a00  .@.&@.@.A.......
	0x0020:  0001 8dc4 3039 53ab 065b 00bc 6153 5010  ....09S..[..aSP.
	0x0030:  7210 19a9 0000 705f 6f62 6a2e 6f09 095c  r.....p_obj.o..\
	0x0040:  0a09 0909 7463 702f 7463 705f 6f62 6a2e  ....tcp/tcp_obj.
	0x0050:  6f09 095c 0a09 0909 6170 702f 6170 705f  o..\....app/app_
	0x0060:  6f62 6a2e 6f09 095c 0a09 0909 6c69 622f  obj.o..\....lib/
	0x0070:  6c69 625f 6f62 6a2e 6f0a 0a61 6c6c 3a74  lib_obj.o..all:t
	0x0080:  6170 6970 0a74 6170 6970 3a24 284e 4554  apip.tapip:$(NET
	0x0090:  5f53 5441 434b 5f4f 424a 5329 0a09 4065  _STACK_OBJS)..@e
	0x00a0:  6368 6f20 2220 5b42 5549 4c44 5d20 2440  cho.".[BUILD].$@
	0x00b0:  220a 0924 2851 2924 2843 4329 2024 284c  "..$(Q)$(CC).$(L
	0x00c0:  464c 4147 5329 2024 5e20 2d6f 2024 400a  FLAGS).$^.-o.$@.
	0x00d0:  0a73 6865 6c6c 2f73 6865 6c6c 5f6f 626a  .shell/shell_obj
	0x00e0:  2e6f 3a73 6865 6c6c 2f2a 2e63 0a09 406d  .o:shell/*.c..@m
	0x00f0:  616b 6520 2d43 2073 6865 6c6c 2f0a 6e65  ake.-C.shell/.ne
	0x0100:  742f 6e65 745f 6f62 6a2e 6f3a 6e65 742f  t/net_obj.o:net/
	0x0110:  2a2e 630a 0940 6d61 6b65 202d 4320 6e65  *.c..@make.-C.ne
	0x0120:  742f 0a61 7270 2f61 7270 5f6f 626a 2e6f  t/.arp/arp_obj.o
	0x0130:  3a61 7270 2f2a 2e63 0a09 406d 616b 6520  :arp/*.c..@make.
	0x0140:  2d43 2061 7270 2f0a 6970 2f69 705f 6f62  -C.arp/.ip/ip_ob
	0x0150:  6a2e 6f3a 6970 2f2a 2e63 0a09 406d 616b  j.o:ip/*.c..@mak
	0x0160:  6520 2d43 2069 702f 0a75 6470 2f75 6470  e.-C.ip/.udp/udp
	0x0170:  5f6f 626a 2e6f 3a75 6470 2f2a 2e63 0a09  _obj.o:udp/*.c..
	0x0180:  406d 616b 6520 2d43 2075 6470 2f0a 7463  @make.-C.udp/.tc
	0x0190:  702f 7463 705f 6f62 6a2e 6f3a 7463 702f  p/tcp_obj.o:tcp/
	0x01a0:  2a2e 630a 0940 6d61 6b65 202d 4320 7463  *.c..@make.-C.tc
	0x01b0:  702f 0a6c 6962 2f6c 6962 5f6f 626a 2e6f  p/.lib/lib_obj.o
	0x01c0:  3a6c 6962 2f2a 2e63 0a09 406d 616b 6520  :lib/*.c..@make.
	0x01d0:  2d43 206c 6962 2f0a 736f 636b 6574 2f73  -C.lib/.socket/s
	0x01e0:  6f63 6b65 745f 6f62 6a2e 6f3a 736f 636b  ocket_obj.o:sock
	0x01f0:  6574 2f2a 2e63 0a09 406d 616b 6520 2d43  et/*.c..@make.-C
	0x0200:  2073 6f63 6b65 742f 0a61 7070 2f61 7070  .socket/.app/app
	0x0210:  5f6f 626a 2e6f 3a61 7070 2f2a 2e63 0a09  _obj.o:app/*.c..
	0x0220:  406d 616b 6520 2d43 2061 7070 2f0a 0a74  @make.-C.app/..t
	0x0230:  6573 743a 6362 7566 0a23 2074 6573 7420  est:cbuf.#.test.
	0x0240:  7072 6f67 7261 6d20 666f 7220 6369       program.for.ci
01:59:30.678463 IP (tos 0x0, ttl 64, id 7, offset 0, flags [none], proto TCP (6), length 40)
    bogon.12345 > bogon.36292: Flags [.], cksum 0x15ef (correct), seq 1, ack 537, win 3560, length 0
	0x0000:  baf2 bd65 513a 0034 4567 89ab 0800 4500  ...eQ:.4Eg....E.
	0x0010:  0028 0007 0000 4006 66c7 0a00 0001 0a00  .(....@.f.......
	0x0020:  0002 3039 8dc4 00bc 6153 53ab 0443 5010  ..09....aSS..CP.
	0x0030:  0de8 15ef 0000                           ......
```

# TCP通信关闭
关闭TCP连接可以通过RST报文强制关闭或FIN报文协商，一般是FIN报文。
主动关闭连接的一方发送FIN报文，另一方回复ACK报文，然后完成自己的关闭操作，即发送FIN报文，等待ACK，这样双方互发FIN报文和收到ACK，连接关闭。显然，关闭一个连接需要4个报文，而建立连接只要3个。
另外，TCP是一个双向协议，如果一方断开连接，而另一方还处于等待接收状态，这时称之为半关闭状态。由于网络通信会经过很多交换节点，如果在FIN报文协商的过程中，报文丢失，这可能会导致TCP连接处被挂起，在linux的实现中，有一个定时器，如果FIN报文协商超时，则强制关闭连接，虽然不符合协议要求，但是可以避免DoS攻击。
用RST报文来断开连接适用于如下场景，连接到一个不存在的端口或接口，一方的TCP状态机崩溃，且不可恢复，导致现在连接无法开始，所以，只要TCP数据传输不出错，就不会用到RST报文。
前面的报文里面没有FIN报文，为什么呢，首先看看[TCP连接关闭时不发FIN包的奇怪行为分析](https://blog.csdn.net/zhangxinrun/article/details/51123458)，但是会发RST包，来看看调用，
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ strace cat Makefile | nc 10.0.0.1 12345
execve("/bin/cat", ["cat", "Makefile"], [/* 63 vars */]) = 0
brk(NULL)                               = 0x137c000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=108742, ...}) = 0
mmap(NULL, 108742, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fe48afff000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\t\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1868984, ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fe48affe000
mmap(NULL, 3971488, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fe48aa2b000
mprotect(0x7fe48abeb000, 2097152, PROT_NONE) = 0
mmap(0x7fe48adeb000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1c0000) = 0x7fe48adeb000
mmap(0x7fe48adf1000, 14752, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fe48adf1000
close(3)                                = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fe48affd000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fe48affc000
arch_prctl(ARCH_SET_FS, 0x7fe48affd700) = 0
mprotect(0x7fe48adeb000, 16384, PROT_READ) = 0
mprotect(0x60b000, 4096, PROT_READ)     = 0
mprotect(0x7fe48b01a000, 4096, PROT_READ) = 0
munmap(0x7fe48afff000, 108742)          = 0
brk(NULL)                               = 0x137c000
brk(0x139d000)                          = 0x139d000
open("/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=2981280, ...}) = 0
mmap(NULL, 2981280, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fe48a753000
close(3)                                = 0
fstat(1, {st_mode=S_IFIFO|0600, st_size=0, ...}) = 0
open("Makefile", O_RDONLY)              = 3
fstat(3, {st_mode=S_IFREG|0666, st_size=1854, ...}) = 0
fadvise64(3, 0, 0, POSIX_FADV_SEQUENTIAL) = 0
mmap(NULL, 139264, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fe48afda000
read(3, "#### User configure  ###########"..., 131072) = 1854
write(1, "#### User configure  ###########"..., 1854) = 1854
read(3, "", 131072)                     = 0
munmap(0x7fe48afda000, 139264)          = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```

# socket API
BSD Socket API是最著名的Socket API，源自于1983年4.2BSD UNIX，Linux Socket API兼容BSD Socket API。
tapip的socket API设计，
```c
Linux socket model
====================================================
socket
   `------> +---------+-+-+
            |  sock   | | |
            +---------+ | |
            | inet_sock | |   --> PF_INET      [family]
            +-----------+ |
            | tcp_sock    |   --> SOCK_STREAM  [type]
            +-------------+       IPPROTO_TCP  [protocol]

 tcp_sock is Transmission Control Block(TCB) of RFC 793

----------------------------------------------------
inet_protosw (inetsw_array[])
+---------+
| type    | (SOCK_[type])                      [type]
+---------+
|protocol | (IPPROTO_[protocol])               [protocol]
+---------+
| ops     | (inet_[type]_ops)                  [type]
+---------+
| prot   ----->  proto         (tcp_prot)      [protocol]
+---------+     +-------------+
                | operations  |
                |  .accept    |
                |  .close     |
                |  .......    |
                +-------------+

Fucntion calling path:
 sys_connect
  .. -> inet_[type]_ops->connect()
   .. -> sock->sk_prot->connect()
 (sock->sk_prot == inet_protosw->prot)


Tapip socket model
====================================================

socket
   `------> +---------+-+
            |  sock   | |   --> PF_INET      [family]
            +---------+ |
            | tcp_sock  |   --> SOCK_STREAM  [type]
            |           |
            | /* tcb */ |       IPPROTO_TCP  [protocol]
            |           |
            +-----------+

 tcp_sock::somefield is Transmission Control Block(TCB) of RFC 793
----------------------------------------------------
socket->ops (inet_ops)
sock->ops   (tcp_ops)

Fucntion calling path:
  _send()
    -> socket->ops->send() [inet_send]
      -> sock->ops->send() [tcp_send]
```
tapip TOP2，测试udp发送，打开tty，更正一下，原作者的脚本需加上-u指定UDP，否则无法监听。
```bash
zc@ubuntu:~/xilinx/app/tapip-master$ sudo iptables -I INPUT -p udp -s 10.0.0.1 -j ACCEPT
[sudo] password for zc: 
zc@ubuntu:~/xilinx/app/tapip-master$ nc -l 12345
^C
zc@ubuntu:~/xilinx/app/tapip-master$ nc -u -l 12345
asdf
asd
asd
asdf
```
打开tty2，
```bash
[net shell]: snc -u -c 10.0.0.2:12345
heloo
hello
^C
[net shell]: snc -u -c 10.0.0.2:12345
hello
^C
[net shell]: snc -u -c 10.0.0.2:12345
hello
asdf
asdf
asdfsadf


#从这儿开始，tty1收到数据
asdf
asdf
asd
asd
asdf
```
