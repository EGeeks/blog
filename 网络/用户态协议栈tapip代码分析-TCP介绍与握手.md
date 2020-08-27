# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

TCP于1974年就已经提出来了，后来有很多拓展和修正，TCP在互联网上提供可靠的连接。

# TCP介绍
为了防止数据丢失，通信双方都保存一定数量的已经发送的数据，数据按发送顺序排列起来，就像一个个窗口胶片一样：
```c
                 Left window edge             Right window edge
                       |                             |
                       |                             |
          ---------------------------------------------------------
          ...|    3    |    4    |    5    |    6    |    7    |...
          ---------------------------------------------------------
                  ^     ^                            ^    ^
                  |      \                          /     |
                  |       \                        /      |
             Sent and           Window size: 3         Cannot be
             ACKed                                     sent yet
```
基于滑动窗口的特性很容易（alleviate）实现流控，另一方面，拥塞控制（Congestion control）有助于发送者和接收者之间的网络堆栈不会拥塞。有两种通用的方法：在明确的版本中，协议有一个字段专门通知发送者关于拥塞状态。在隐式版本中，发送者在网络拥塞时尝试猜测，并且应该抑制其输出。总的来说，拥塞控制是一个复杂的、经常出现的网络问题，伴随着这项研究至今仍在进行。
和IP类似，TCP需要检测包的完整性，和IP一样的checksum计算方法，但是TCP把包头和数据都计算进校验和，另外还加入了一个伪首部（pseudo-header），[伪首部包括12个字节，包含源地址、目的地址、协议、TCP长度信息](https://blog.csdn.net/u013401853/article/details/77482542)。
```c
uint16_t tcp_chksum(uint16_t initcksum, uint8_t *tcphead, int tcplen , uint32_t *srcaddr, uint32_t *destaddr)
{
    uint8_t pseudoheader[12];
    uint16_t calccksum;

    memcpy(&pseudoheader[0],srcaddr,IP_ADDR_LEN);
    memcpy(&pseudoheader[4],destaddr,IP_ADDR_LEN);
    pseudoheader[8] = 0; /* 填充零 */
    pseudoheader[9] = IPPROTO_TCP;
    pseudoheader[10] = (tcplen >> 8) & 0xFF;
    pseudoheader[11] = (tcplen & 0xFF);

    calccksum = ip_chksum(0,pseudoheader,sizeof(pseudoheader));
    calccksum = ip_chksum(calccksum,tcphead,tcplen);
    calccksum = ~calccksum;
    return calcchsum;
}
```
如果TCP接收到了不连续的报文段，则直接丢弃，不会通知应用，TCP通过ACK报文来同步数据窗口。
TCP包头格式，TCP包头长度为20字节：
```c
        0                            15                              31
       -----------------------------------------------------------------
       |          source port          |       destination port        |
       -----------------------------------------------------------------
       |                        sequence number                        |
       -----------------------------------------------------------------
       |                     acknowledgment number                     |
       -----------------------------------------------------------------
       |  HL   | rsvd  |C|E|U|A|P|R|S|F|        window size            |
       -----------------------------------------------------------------
       |         TCP checksum          |       urgent pointer          |
       -----------------------------------------------------------------
```
主机上依靠Source Port和Destination Port来建立不同的TCP连接，通常（prevalent）应用采用Berkeley sockets接口来访问网络，port,2字节，值从0到65535，Sequence Number表示TCP报文在窗口中的位置，在握手的时候，其值为初始序列号ISN（Initial Sequence Number），Acknowledgment Number包含了发送希望接收数据的窗口位置，握手完成之后，这个字段必须被正确配置，Header Length (HL)字段表示TCP头的长度，以4字节为单位。
接下来是一些标志位，rsvd长度4bit未使用，C拥塞窗口减少（Congestion Window Reduced）用来通知发送者减少发送线速，E（ECN Echo）通知发送者接收一个拥塞通知，U紧急数据指针（Urgent Pointer）表示这个报文包含优先数据（prioritized data），A（ACK）字段用来表示TCP握手的状态，P（PSH）提示接收者需要尽快将数据送往应用，R（RSH）复位TCP连接，S(SYN)用来同步Sequence Number在握手阶段，F(FIN)表示发送者数据已经发送完成。
Window Size用来告知窗口大小，这个字段表示接收者将能接收多少字节数据，所以最大窗口为65535字节（由于这个数值太小，后面再握手阶段会协商窗口的单位n，即n * Window Size），TCP Checksum用来验证TCP报文的正确性，Urgent Pointer在U标志有效的时候表示urgent data在报文中的位置。
在包头的后面，有几个选项（options ）字节可以提供，比如最大报文段大小MSS（Maximum Segment Size），随后就是数据，但是有时数据是不需要的，比如握手报文就没有数据部分。

# TCP握手
TCP连接通常经历以下几个阶段：连接设置（握手）、数据传输和连接的关闭。下面的图表描述了TCP常用的握手程序：
```c
          TCP A                                                TCP B
    	  
    1.  CLOSED                                               LISTEN
    	
    2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED
    	  
    3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED
    			
    4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED
    			  
    5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED
```
主机A的socket状态是关闭状态，同时，主机B的socket绑定到特定端口，开始监听连接，主机A开始连接主机B，主机A发出一个设置了SYN标志，Sequence Number为100的TCP报文，主机B返回一个设置了SYN和ACK，Sequence Number为300，Acknowledgment Number为101的TCP报文，主机A在返回一个Sequence Number为101，Acknowledgment Number为301的TCP报文。这样三步握手完成，两边的Sequence Number已经同步，下面可以开始传输数据。详细过程可以参考我的另一片博客：[TCP/IP握手过程](https://blog.csdn.net/zhu_zhu_2009/article/details/80424724)。
初始序列号很重要，不能被猜测到，否则可能会被TCP序列号攻击（TCP Sequence Number Attack），攻击者伪造一个虚假的主机，发送攻击数据。在最初的协议里，ISN从一个每4毫秒递增的计数器中得到，这很容易被攻击值猜测，现在的网络协议栈以一种非常复杂的方式生成ISN。
当通信双方同时（Simultaneous Open）向对方发送建立连接请求报文（SYN）,则通信双方多一个消息交换的过程，两边都发送ACK反馈和SYN-ACK请求，之后连接建立。
最后，TCP实现必须有一个计时器，用于知道何时放弃建立连接。尝试重新建立连接时，通常需要等待一段时间（exponential backoff），但一旦达到最大重试或时间阈值，连接被认为是不存在的。
在tcp/tcp_sock.c中，状态为TCP_SYN_SENT，
```c
static int tcp_connect(struct sock *sk, struct sock_addr *skaddr)
{
	struct tcp_sock *tsk = tcpsk(sk);
	int err;
	if (tsk->state != TCP_CLOSED)
		return -1;
	sk->sk_daddr = skaddr->dst_addr;
	sk->sk_dport = skaddr->dst_port;
	/* three-way handshake starts, send first SYN */
	tsk->state = TCP_SYN_SENT;
	tsk->iss = alloc_new_iss();
	tsk->snd_una = tsk->iss;
	tsk->snd_nxt = tsk->iss + 1;
	if (tcp_hash(sk) < 0) {
		tsk->state = TCP_CLOSED;
		return -1;
	}
	/*
	 * Race condition:
	 *  If we connect to localhost, then we will send syn
	 *  and recv packet. It wakes up tsk->wait_connect without
	 *  seting wait_connect.
	 *
	 * Fix:
	 *  set wait_connect before sending syn
	 */
	tcp_pre_wait_connect(tsk);
	tcp_send_syn(tsk, NULL);
	err = tcp_wait_connect(tsk);
	if (err || tsk->state != TCP_ESTABLISHED) {
		tcp_unhash(sk);
		tcp_unbhash(tsk);
		tsk->state = TCP_CLOSED;
		err = -1;
	}
	return err;
}
```
在tcp/tcp_state.c中，tcp_process调用tcp_synsent，
```c
/* Tcp state process method is implemented via RFC 793 #SEGMENT ARRIVE */
void tcp_process(struct pkbuf *pkb, struct tcp_segment *seg, struct sock *sk)
{
	struct tcp_sock *tsk = tcpsk(sk);
	struct tcp *tcphdr = seg->tcphdr;
	tcp_dbg_state(tsk);
	if (!tsk || tsk->state == TCP_CLOSED)
		return tcp_closed(tsk, pkb, seg);
	if (tsk->state == TCP_LISTEN)
		return tcp_listen(pkb, seg, tsk);
	if (tsk->state == TCP_SYN_SENT)
		return tcp_synsent(pkb, seg, tsk);
	if (tsk->state >= TCP_MAX_STATE)
		goto drop;
	/* first check sequence number */
	tcpsdbg("1. check seq");
	if (seq_check(seg, tsk) < 0) {
		/* incoming segment is not acceptable */
		if (!tcphdr->rst)
			tsk->flags |= TCP_F_ACKNOW; /*reply ACK seq=snd.nxt, ack=rcv.nxt*/
		goto drop;
	}
	/* second check the RST bit */
	tcpsdbg("2. check rst");
	if (tcphdr->rst) {
		/* abort a connection */
		switch (tsk->state) {
		case TCP_SYN_RECV:
			if (tsk->parent) {	/* passive open */
				tcp_unhash(&tsk->sk);
			} else {
				/*
				 * signal user "connection refused"
				 * when both users open simultaneously.
				 * XXX: test
				 */
				if (tsk->wait_connect)
					wake_up(tsk->wait_connect);
			}
			break;
		case TCP_ESTABLISHED:
		case TCP_FIN_WAIT1:
		case TCP_FIN_WAIT2:
		case TCP_CLOSE_WAIT:
			/* RECEIVE and SEND receive reset response */
			/* flush all segments queue */
			/* signal user "connection reset" */
			break;
		case TCP_CLOSING:
		case TCP_LAST_ACK:
		case TCP_TIME_WAIT:
			break;
		}
		tcp_set_state(tsk, TCP_CLOSED);
		tcp_unhash(&tsk->sk);
		tcp_unbhash(tsk);
		goto drop;
	}
	/* third check security and precedence (ignored) */
	tcpsdbg("3. NO check security and precedence");
	/* fourth check the SYN bit */
	tcpsdbg("4. check syn");
	if (tcphdr->syn) {
		/* only LISTEN and SYN-SENT can receive SYN */
		tcp_send_reset(tsk, seg);
		/* RECEIVE and SEND receive reset response */
		/* flush all segments queue */
		/* signal user "connection reset" */
		/*
		 * RFC 1122: error corrections of RFC 793:
		 * In SYN-RECEIVED state and if the connection was initiated
		 * with a passive OPEN, then return this connection to the
		 * LISTEN state and return.
		 * - We delete child tsk directly,
		 *   and its parent has been in LISTEN state.
		 */
		if (tsk->state == TCP_SYN_RECV && tsk->parent)
			tcp_unhash(&tsk->sk);
		tcp_set_state(tsk, TCP_CLOSED);
		free_sock(&tsk->sk);
	}
	/* fifth check the ACK field */
	tcpsdbg("5. check ack");
	/*
	 * RFC 793 say:
	 * 1. we should drop the segment and return
	 *    if the ACK bit is off.
	 * 2. Once in the ESTABLISHED state all segments must
	 *    carry current acknowledgment information.
	 * Should we do it ?
	 * -No for xinu
	 * -No for linux
	 */
	if (!tcphdr->ack)
		goto drop;
	switch (tsk->state) {
	case TCP_SYN_RECV:
		/*
		 * previous state LISTEN :
		 *  snd_nxt = iss + 1
		 *  snd_una = iss
		 * previous state SYN-SENT:
		 *  snd_nxt = iss+1
		 *  snd_una = iss
		 * Should we update snd_una to seg->ack here?
		 *  -Unknown for RFC 793
		 *  -Yes for xinu
		 *  -Yes for Linux
		 *  +Yes for tapip
		 * Are 'snd.una == seg.ack' right?
		 *  -Yes for RFC 793
		 *  -Yes for 4.4BSD-Lite
		 *  -Yes for xinu, although duplicate ACK
		 *  -Yes for Linux,
		 *  +Yes for tapip
		 */
		if (tsk->snd_una <= seg->ack && seg->ack <= tsk->snd_nxt) {
			if (tcp_synrecv_ack(tsk) < 0) {
				tcpsdbg("drop");
				goto drop;		/* Should we drop it? */
			}
			tsk->snd_una = seg->ack;
			/* RFC 1122: error corrections of RFC 793(SND.W**) */
			__tcp_update_window(tsk, seg);
			tcp_set_state(tsk, TCP_ESTABLISHED);
		} else {
			tcp_send_reset(tsk, seg);
			goto drop;
		}
		break;
	case TCP_ESTABLISHED:
	case TCP_CLOSE_WAIT:
	case TCP_LAST_ACK:
	case TCP_FIN_WAIT1:
	case TCP_CLOSING:
		tcpsdbg("SND.UNA %u < SEG.ACK %u <= SND.NXT %u",
				tsk->snd_una, seg->ack, tsk->snd_nxt);
		if (tsk->snd_una < seg->ack && seg->ack <= tsk->snd_nxt) {
			tsk->snd_una = seg->ack;
			/*
			 * remove any segments on the restransmission
			 * queue which are thereby entirely acknowledged
			 */
			if (tsk->state == TCP_FIN_WAIT1) {
				tcp_set_state(tsk, TCP_FIN_WAIT2);
			} else if (tsk->state == TCP_CLOSING) {
				tcp_set_timewait_timer(tsk);
				goto drop;
			} else if (tsk->state == TCP_LAST_ACK) {
				tcp_set_state(tsk, TCP_CLOSED);
				tcp_unhash(&tsk->sk);
				/* for tcp active open */
				tcp_unbhash(tsk);
				goto drop;
			}
		} else if (seg->ack > tsk->snd_nxt) {	/* something not yet sent */
			/* reply ACK ack = ? */
			goto drop;
		} else if (seg->ack <= tsk->snd_una) {	/* duplicate ACK */
			/*
			 * RFC 793 say we can ignore duplicate ACK.
			 * What does `ignore` mean?
			 * Should we conitnue and not drop segment ?
			 * -Yes for xinu
			 * -Yes for linux
			 * -Yes for 4.4BSD-Lite
			 * +Yes for tapip
			 *
			 * After three-way handshake connection is established,
			 * then SND.UNA == SND.NXT, which means next remote
			 * packet ACK is always duplicate. Although this
			 * happens frequently, we should not view it as an
			 * error.
			 *
			 * Close simultaneously in FIN_WAIT1 also causes this.
			 *
			 * Also window update packet will cause this situation.
			 */
		}
		tcp_update_window(tsk, seg);
		break;
	case TCP_FIN_WAIT2:
	/*
          In addition to the processing for the ESTABLISHED state, if
          the retransmission queue is empty, the user's CLOSE can be
          acknowledged ("ok") but do not delete the TCB. (wait FIN)
	 */
		break;
	case TCP_TIME_WAIT:
	/*
          The only thing that can arrive in this state is a
          retransmission of the remote FIN.  Acknowledge it, and restart
          the 2 MSL timeout.
	 */
		break;
	}

	/* sixth check the URG bit */
	tcpsdbg("6. check urg");
	if (tcphdr->urg) {
		switch (tsk->state) {
		case TCP_ESTABLISHED:
		case TCP_FIN_WAIT1:
		case TCP_FIN_WAIT2:
	/*
        If the URG bit is set, RCV.UP <- max(RCV.UP,SEG.UP), and signal
        the user that the remote side has urgent data if the urgent
        pointer (RCV.UP) is in advance of the data consumed.  If the
        user has already been signaled (or is still in the "urgent
        mode") for this continuous sequence of urgent data, do not
        signal the user again.
	 */
			break;
		case TCP_CLOSE_WAIT:
		case TCP_CLOSING:
		case TCP_LAST_ACK:
		case TCP_TIME_WAIT:
			/* ignore */
			/* Should we conitnue or drop? */
			break;
		case TCP_SYN_RECV:
			/* ?? */
			break;
		}
	}
	/* seventh process the segment text */
	tcpsdbg("7. segment text");
	switch (tsk->state) {
	case TCP_ESTABLISHED:
	case TCP_FIN_WAIT1:
	case TCP_FIN_WAIT2:
		if (tcphdr->psh || seg->dlen > 0)
			tcp_recv_text(tsk, seg, pkb);
		break;
	/*
	 * CLOSE-WAIT|CLOSING|LAST-ACK|TIME-WAIT:
	 *  FIN has been received, so we ignore the segment text.
	 *
	 * OTHER STATES: segment is ignored!
	 */
	}
	/* eighth check the FIN bit */
	tcpsdbg("8. check fin");
	if (tcphdr->fin) {
		switch (tsk->state) {
		case TCP_SYN_RECV:
			/*
			 * SYN-RECV means remote->local connection is established
			 * see TCP/IP Illustrated Vol.2, tcp_input() L1127-1134
			 */
		case TCP_ESTABLISHED:
			/* waiting user to close */
			tcp_set_state(tsk, TCP_CLOSE_WAIT);
			tsk->flags |= TCP_F_PUSH;
			tsk->sk.ops->recv_notify(&tsk->sk);
			break;
		case TCP_FIN_WAIT1:
			/* both users close simultaneously */
			tcp_set_state(tsk, TCP_CLOSING);
			break;
		case TCP_CLOSE_WAIT:	/* Remain in the CLOSE-WAIT state */
		case TCP_CLOSING:	/* Remain in the CLOSING state */
		case TCP_LAST_ACK:	/* Remain in the LAST-ACK state */
			/* dont handle it, must be duplicate FIN */
			break;
		case TCP_TIME_WAIT:	/* Remain in the TIME-WAIT state */
			/* restart the 2 MSL time-wait timeout */
			tsk->timewait.timeout = TCP_TIMEWAIT_TIMEOUT;
			break;
		case TCP_FIN_WAIT2:
			/* FIXME: turn off the other timers. */
			tcp_set_timewait_timer(tsk);
			break;
		}
		/* singal the user "connection closing" */
		/* return any pending RECEIVEs with same message */
		/* advance rcv.nxt over fin */
		tsk->rcv_nxt = seg->seq + 1;
		/* send ACK for FIN */
		tsk->flags |= TCP_F_ACKNOW;
		/*
		 * FIN implies PUSH for any segment text not yet delivered
		 * to the user.
		 */
	}
drop:
	/* TODO: use ack delay timer instead of sending ack now */
	if (tsk->flags & (TCP_F_ACKNOW|TCP_F_ACKDELAY))
		tcp_send_ack(tsk, seg);
	free_pkb(pkb);
}
```
在tcp/tcp_state.c中
```c
/*
 * OPEN CALL:
 * sent <SEQ=ISS><CTL=SYN>
 * SND.UNA = ISS, SND.NXT = ISS+1
 */
static void tcp_synsent(struct pkbuf *pkb, struct tcp_segment *seg,
			struct tcp_sock *tsk)
{
	struct tcp *tcphdr = seg->tcphdr;
	tcpsdbg("SYN-SENT");
	/* first check the ACK bit */
	tcpsdbg("1. check ack");
	if (tcphdr->ack) {
		/*
		 * Maybe we can reduce to `seg->ack != tsk->snd_nxt`
		 * Because we should not send data with the first SYN.
		 * (Just assert tsk->iss + 1 == tsk->snd_nxt)
		 */
		if (seg->ack <= tsk->iss || seg->ack > tsk->snd_nxt) {
			tcp_send_reset(tsk, seg);
			goto discarded;
		}
		/*
		 * RFC 793:
		 *   If SND.UNA =< SEG.ACK =< SND.NXT, the ACK is acceptable.
		 *   (Assert SND.UNA == 0)
		 */
	}
	/* second check the RST bit */
	tcpsdbg("2. check rst");
	if (tcphdr->rst) {
		if (tcphdr->ack) {
			/* connect closed port */
			tcpsdbg("Error:connection reset");
			tcp_set_state(tsk, TCP_CLOSED);
			if (tsk->wait_connect)
				wake_up(tsk->wait_connect);
			else
				tcpsdbg("No thread waiting for connection");
		}
		goto discarded;
	}
	/* third check the security and precedence (ignored) */
	tcpsdbg("3. No check the security and precedence");
	/* fouth check the SYN bit */
	tcpsdbg("4. check syn");
	if (tcphdr->syn) {
		tsk->irs = seg->seq;
		tsk->rcv_nxt = seg->seq + 1;
		if (tcphdr->ack)		/* No ack for simultaneous open */
			tsk->snd_una = seg->ack;	/* snd_una: iss -> iss+1 */
		/* delete retransmission queue which waits to be acknowledged */
		if (tsk->snd_una > tsk->iss) {	/* rcv.ack = snd.syn.seq+1 */
			tcp_set_state(tsk, TCP_ESTABLISHED);
			/* RFC 1122: error corrections of RFC 793 */
			tsk->snd_wnd = seg->wnd;
			tsk->snd_wl1 = seg->seq;
			tsk->snd_wl2 = seg->ack;
			/* reply ACK seq=snd.nxt, ack=rcv.nxt at right */
			tcp_send_ack(tsk, seg);
			tcpsdbg("Active three-way handshake successes!(SND.WIN:%d)", tsk->snd_wnd);
			wake_up(tsk->wait_connect);
			/*
			 * Data or controls which were queued for transmission
			 * may be included.  If there are other controls or text
			 * in the segment then continue processing at the sixth
			 * step * below where the URG bit is checked, otherwise
			 * return.
			 */
		} else {		/* simultaneous open */
			/* XXX: test */
			tcp_set_state(tsk, TCP_SYN_RECV);
			/* reply SYN+ACK seq=iss,ack=rcv.nxt */
			tcp_send_synack(tsk, seg);
			tcpsdbg("Simultaneous open(SYN-SENT => SYN-RECV)");
			/*
			 * queue text or other controls after established state
			 * has been reached
			 */
			return;
		}
	}
	/* fifth drop the segment and return */
	tcpsdbg("5. drop the segment");
discarded:
	free_pkb(pkb);
}
```
其中TCP_ESTABLISHED之后，调用tcp_send_ack，
```c
/*
 * Acknowledgment algorithm is not stated directly in RFC 793,
 * but we can conclude it from all acknowledgment situation.
 */
void tcp_send_ack(struct tcp_sock *tsk, struct tcp_segment *seg)
{
	/*
	 * SYN-SENT :
	 *         SEG: SYN, acceptable ACK, no RST   (SND.NXT = SEG.SEQ+1)
	 *         <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>
	 * SYN-RECEIVED / ESTABLISHED  / FIN-WAIT-1   / FIN-WAIT-2   /
	 * CLOSE-WAIT   / CLOSING      / LAST-ACK     / TIME-WAIT    :
	 *         SEG: no RST, ??ACK, ??SYN        (segment is not acceptable)
	 *         <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>
	 * ESTABLISHED  / FIN-WAIT-1  / FIN-WAIT-2  / process the segment text:
	 *         SEG: ACK, no RST
	 *         <SEQ=SND.NXT><ACK=RCV.NXT><CTL=ACK>
	 *         (This acknowledgment should be piggybacked on a segment being
	 *          transmitted if possible without incurring undue delay.)
	 */
	struct tcp *otcp, *tcphdr = seg->tcphdr;
	struct pkbuf *opkb;

	if (tcphdr->rst)
		return;
	opkb = alloc_pkb(ETH_HRD_SZ + IP_HRD_SZ + TCP_HRD_SZ);
	/* fill tcp head */
	otcp = (struct tcp *)pkb2ip(opkb)->ip_data;
	otcp->src = tcphdr->dst;
	otcp->dst = tcphdr->src;
	otcp->doff = TCP_HRD_DOFF;
	otcp->seq = _htonl(tsk->snd_nxt);
	otcp->ackn = _htonl(tsk->rcv_nxt);
	otcp->ack = 1;
	otcp->window = _htons(tsk->rcv_wnd);
	tcpdbg("send ACK(%u) [WIN %d] to "IPFMT":%d",
			_ntohl(otcp->ackn), _ntohs(otcp->window),
			ipfmt(seg->iphdr->ip_src), _ntohs(otcp->dst));
	tcp_send_out(tsk, opkb, seg);
}
```
至此，TCP三次握手过程建立。

# TCP选项
TCP包头之后保留了一些字段用于TCP选项，常用的选项如下：
最大报文长度MSS（Maximum Segment Size）表示协议栈能接收的报文大小，IPv4里这个值一般是1460字节。
选择验证SACK（Selective Acknowledgment）优化了下面的情况，当许多包丢失，接收的滑动窗口充满了空洞，为了恢复网络带宽，TCP可以用SACK设置哪些发送的包不需要接收ACK。
窗口因子（Window Scale）增加了窗口大小，由于Windows Size字段只有16bit的限制，在握手阶段，通过这个选项可以设置窗口大小为Window Scale * Windows Size。
时间戳（Timestamps）允许发送者在TCP报文中防止时间戳，这可以用来计算每一个ACK报文的往返时间RTT（Round Trip Time）时间，然后可以计算TCP重传的超时时间RTO（Retransmission TimeOut），参考这篇博文：[TCP中RTT的测量和RTO的计算](https://blog.csdn.net/zhangskd/article/details/7196707)
