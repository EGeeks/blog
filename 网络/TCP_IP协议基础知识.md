# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> TCP/IP详解卷1：协议
> [网络七层协议](https://blog.csdn.net/ght886/article/details/78901939)
> [Wireshark 分析ping报文](https://blog.csdn.net/u010999240/article/details/52969384)
> [EtherType ：以太网类型字段及值](https://blog.csdn.net/eydwyz/article/details/65446328)
> [IP分片报文的接收与重组](https://blog.csdn.net/sinat_20184565/article/details/82670126)
> [TCP 的那些事 | TCP报文格式解析](https://mp.weixin.qq.com/s?__biz=MzA3Nzk2NTY1MA==&mid=2649163946&idx=1&sn=f8c0c661b4a6c4a971c2bd1d4cd331e8&chksm=875a5897b02dd1815d0aca59aa0c4a8671fc3ff8c013756d9c2fe4214ca86a531a64a4204f47&scene=21#wechat_redirect)
> [TCP选项之SACK选项概述](https://blog.csdn.net/xiaoyu_750516366/article/details/87870712)
> [TCP 的那些事 | SACK](https://blog.csdn.net/u014023993/article/details/85041238)
> [IP分片和TCP分段](https://blog.csdn.net/lixiangminghate/article/details/82015722)
> [【网络协议】TCP分段与IP分片](https://blog.csdn.net/ns_code/article/details/30109789)
> [关于“TCP segment of a reassembled PDU”](https://blog.csdn.net/dog250/article/details/51809566)
> [流重组讲解四部曲（一）- IP、TCP流重组概念理解](https://blog.csdn.net/lsl_zhulin/article/details/79299153)
> [【TCP/IP协议】TCP中的MSS解读](https://www.cnblogs.com/zhangxian/articles/5558651.html)
> [Windows系统下的TCP参数优化](https://blog.csdn.net/baidu_18607183/article/details/51646522)
> [为什么没收到对端 MSS 选项，TCP 就采用 536 这个 MSS 值？](https://blog.csdn.net/chuanglan/article/details/80666086)
> [TCP选项之MSS](https://blog.csdn.net/xiaoyu_750516366/article/details/85316123)

# TCP/IP网络分层
- ICMP是IP协议的附属协议，IP层用它来与其他主机或路由器交换错误报文和其他重要信息，但应用程序也有可能访问它，比如ping和traceroute。
- IGMP是Internet组管理协议，它用来把一个UDP数据报多播到多个主机。
- ARP（地址解析协议）和RARP（逆地址解析协议）是某些网络接口（如以太网和令牌环网）使用的特殊协议，用来转换 IP层和网络接口层使用的地址。
![229](https://img-blog.csdnimg.cn/20191201145838468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
## OSI 7层协议模型
- 应用层 Application 网络服务与最终用户的一个接口。协议有：HTTP FTP TFTP SMTP SNMP DNS TELNET HTTPS POP3 DHCP
- 表示层 Presentation Layer 数据的表示、安全、压缩。（在五层模型里面已经合并到了应用层）格式有，JPEG、ASCll、DECOIC、加密格式等
- 会话层Session Layer 建立、管理、终止会话。（在五层模型里面已经合并到了应用层）对应主机进程，指本地主机与远程主机正在进行的会话
- 传输层 Transport  定义传输数据的协议端口号，以及流控和差错校验。协议有：TCP UDP，数据包一旦离开网卡即进入网络传输层
- 网络层 Network 进行逻辑地址寻址，实现不同网络之间的路径选择。协议有：ICMP IGMP IP（IPV4 IPV6） ARP RARP
- 数据链路层 Link 建立逻辑连接、进行硬件地址寻址、差错校验等功能。（由底层网络定义协议）将比特组合成字节进而组合成帧，用MAC地址访问介质，错误发现但不能纠正。
-  物理层Physical Layer 建立、维护、断开物理连接。（由底层网络定义协议）

# 网络数据包封装过程
TCP数据包的封装过程，UDP类似，但是UDP数据包的首部只有8字节。有一个特殊的情况，以太网帧中可以是一个完整的IP数据报，也可以是IP数据报的一个分片（fragment）。
- TCP和UDP报文首部含有源端口号和目的端口号，来表示不同的应用程序。知名端口号介于1～255之间。256～1023之间的端口号通常都是由Unix系统占用，以提供一些特定的Unix服务，一些只有Unix系统才有的、而其他操作系统可能不提供的服务，现在IANA管理1～1023之间所有的端口号。
- IP在首部中存入一个长度为8bit的协议域，1表示为ICMP协议，2表示为IGMP协议，6表示为TCP协议，17表示为UDP协议。
- 以太网的帧首部也有一个 16bit的帧类型域，以指明生成数据的网络层协议IP、ARP和RARP。
![230](https://img-blog.csdnimg.cn/20191201150354262.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 以太网帧格式
通常我们采用的是RFC894协议中的格式，IEEE802/RFC1042了解一下即可，
| 类型值 | 协议名 |
|--|--|
| 0800 | IP |
| 0806 | ARP |
| 8035 | RARP |
| 86dd | IPv6 |
![231](https://img-blog.csdnimg.cn/20191201152412147.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# IP数据报文格式
- 版本，4=IPv4。
- 首部长度指的是首部占32bit字的数目，包括任何选项。由于它是一个4比特字段，因此首部最长为60个字节。普通IP数据报（没有任何选择项）字段的值是5，即20个字节。
- 总长度字段是指整个IP数据报的长度，以字节为单位。利用首部长度字段和总长度字段，就可以知道IP数据报中数据内容的起始位置和长度。总长度字段是IP首部中必要的内容，因为一些数据链路（如以太网）需要填充一些数据以达到最小长度。尽管以太网的最小帧长为46字节（见图2-1），但是IP数据可能会更短。如果没有总长度字段，那么IP层就不知道46字节中有多少是IP数据报的内容。由于该字段长16比特，所以IP数据报最长可达65535字节，但是大多数的链路层都会对它进行分片。而且，主机也要求不能接收超过576字节的数据报。由于TCP把用户数据分成若干片，因此一般来说这个限制不会影响TCP。在后面的章节中将遇到大量使用UDP的应用（RIP，TFTP，BOOTP，DNS，以及SNMP），它们都限制用户数据报长度为512字节，小于576字节。但是，事实上现在大多数的实现（特别是那些支持网络文件系统NFS的实现）允许超过8192字节的IP数据报。
- 服务类型（TOS）字段包括一个3bit的优先权子字段（现在已被忽略），4bit的TOS子字段和1bit未用位但必须置0。4bit的TOS分别代表：最小时延、最大吞吐量、最高可靠性和最小费用。4 bit中只能置其中1 bit。如果所有4 bit均为0，那么就意味着是一般服务。
- 标识字段，标识主机发送的每一份数据报，通常每发送一份报文它的值就会加 1。
- 标志字段和片偏移字段，IP分片和重组时会使用它们。DF（Don't fragment）标志表示不可分片，在超过MSS时，这会导致包丢弃，并回传一个 ICMP不可达差错报文，这可以用来确定MTU值，MF（More fragments）示"更多的片"，除了最后一片外，其他每个组成数据报的片都要把该比特置 1。片偏移字段指的是该片偏移原始数据报开始处的位置。另外，当数据报被分片后，每个片的总长度值要改为该片的长度值。
- TTL（time-to-live）生存时间字段设置了数据报可以经过的最多路由器数。它指定了数据报的生存时间。TTL的初始值由源主机设置（通常为32或64），一旦经过一个处理它的路由器，它的值就减去1。当该字段的值为0时，数据报就被丢弃，并发送 ICMP报文通知源主机。traceroute程序使用这个字段来工作。
- 协议域，

| 协议值 | 协议名 |
|--|--|
| 1 | ICMP |
| 2 | IGMP |
| 6 | TCP |
| 17 | UDP |

- 首部检验和字段是根据IP首部计算的检验和码。它不对首部后面的数据进行计算。 ICMP、IGMP、UDP和TCP在它们各自的首部中均含有同时覆盖首部和数据检验和码。为了计算一份数据报的IP检验和，首先把检验和字段置为0。然后，对首部中每个16bit进行二进制反码求和（整个首部看成是由一串16bit的字组成），结果存在检验和字段中。当收到一份IP数据报后，同样对首部中每个16 bit进行二进制反码的求和。由于接收方在计算过程中包含了发送方存在首部中的检验和，因此，如果首部在传输过程中没有发生任何差错，那么接收方计算的结果应该为全1。如果结果不是全1（即检验和错误），那么IP就丢弃收到的数据报。但是不生成差错报文，由上层去发现丢失的数据报并进行重传。
- 选项，是数据报中的一个可变长的可选信息。目前，这些选项定义如下：
• 安全和处理限制（用于军事领域，详细内容参见 RFC1108[Kent 1991]）
• 记录路径（让每个路由器都记下它的 I P地址）
• 时间戳（让每个路由器都记下它的 I P地址和时间）
• 宽松的源站选路（为数据报指定一系列必须经过的 I P地址）
• 严格的源站选路（与宽松的源站选路类似，但是要求只能经过指定的这些地址，不能经过其他的地址）。
这些选项很少被使用，并非所有的主机和路由器都支持这些选项。选项字段一直都是以32bit作为界限，在必要的时候插入值为0的填充字节。这样就保证IP首部始终是32bit的整数倍（这是首部长度字段所要求的），**通过IP头长度可判断是否含有选项**。
![232](https://img-blog.csdnimg.cn/20191201153030945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
常用TOS字段值，
![233](https://img-blog.csdnimg.cn/20191201153834999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
## IP分片
![270](https://img-blog.csdnimg.cn/20200107153144612.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# ARP数据报文格式
- 以太网报头中的前两个字段是以太网的源地址和目的地址。目的地址为全1的特殊地址是广播地址。
- 两个字节长的以太网帧类型表示后面数据的类型。对于ARP请求或应答来说，该字段的值为0x0806。
- 硬件类型字段表示硬件地址的类型。它的值为1即表示以太网地址。
- 协议类型字段表示要映射的协议地址类型，比如0x0800即表示IP地址。
- 硬件地址长度指出硬件地址的长度，以字节为单位。对于以太网上的ARP请求或应答来说，它的值为6。
- 协议地址长度指出协议地址的长度，以字节为单位。对于IP地址的ARP请求或应答来说，它的值为4。
- 操作字段指出四种操作类型，它们是ARP请求（值为1）、ARP应答（值为2）、RARP请求（值为3）和RARP应答（值为4），这个字段必需的，因为ARP请求和ARP应答的帧类型字段值是相同的。
![234](https://img-blog.csdnimg.cn/20191201155401336.png)
对于一个ARP请求来说，除目的端硬件地址外的所有其他的字段都有填充值。当系统收到一份目的端为本机的ARP请求报文后，它就把硬件地址填进去，然后用两个目的端地址分别替换两个发送端地址，并把操作字段置为2，最后把它发送回去。

# RARP
具有本地磁盘的系统引导时，一般是从磁盘上的配置文件中读取IP地址。但是无盘机，如X终端或无盘工作站，则需要采用其他方法来获得IP地址。RARP分组的格式与ARP分组基本一致。它们之间主要的差别是RARP请求或应答的帧类型代码为0x8035，而且RARP请求的操作代码为3，应答操作代码为4。对应于ARP，RARP请求以广播方式传送，而RARP应答一般是单播( unicast)传送的。

# ICMP报文
类型字段可以有15个不同的值，以描述特定类型的ICMP报文。某些ICMP报文还使用代码字段的值来进一步描述不同的条件。
检验和字段覆盖整个ICMP报文。使用的算法与I P首部检验和算法相同。
![235](https://img-blog.csdnimg.cn/20191201160901599.png)
![236](https://img-blog.csdnimg.cn/20191201160926226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![237](https://img-blog.csdnimg.cn/20191201161112311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
## ping
对于其他类型的ICMP查询报文，服务器必须响应标识符和序列号字段。另外，客户发送的选项数据必须回显，假设客户对这些信息都会感兴趣。
Unix系统在实现ping程序时是把ICMP报文中的标识符字段置成发送进程的ID号。这样即使在同一台主机上同时运行了多个ping程序实例，ping程序也可以识别出返回的信息。序列号从0开始，每发送一次新的回显请求就加1。ping程序打印出返回的每个分组的序列号，允许我们查看是否有分组丢失、失序或重复。IP是一种最好的数据报传递服务，因此这三个条件都有可能发生。
旧版本的ping程序曾经以这种模式运行，即每秒发送一个回显请求，并打印出返回的每个回显应答。但是，新版本的实现需要加上- s选项才能以这种模式运行。默认情况下，新版本的ping程序只发送一个回显请求。如果收到回显应答，则输出`host is alive`；否则，在20秒内没有收到应答就输出`no answer`。
![260](https://img-blog.csdnimg.cn/20191223194620285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# IGMP
IGMP的校验和位置核ICMP是一样的，
![261](https://img-blog.csdnimg.cn/20191225143707290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# UDP报文格式
- 端口号表示发送进程和接收进程。由于IP层已经把IP数据报分配给TCP或UDP（根据IP首部中协议字段值），因此TCP端口号与UDP端口号是相互独立的。
- UDP长度字段指的是UDP首部和UDP数据的字节长度。该字段的最小值为8字节（发送一份0字节的UDP数据报是OK）。这个UDP长度是有冗余的。IP数据报长度指的是数据报全长，因此UDP数据报长度是全长减去IP首部的长度（该值在首部长度字段中指定）。
- UDP检验和覆盖UDP首部和UDP数据。回想IP首部的检验和，它只覆盖IP的首部 — 并不覆盖IP数据报中的任何数据。UDP和TCP在首部中都有覆盖它们首部和数据的检验和。 UDP的检验和是可选的，而TCP的检验和是必需的。尽管UDP检验和的基本计算方法与IP首部检验和计算方法相类似（16 bit字的二进制反码和），但是它们之间存在不同的地方。首先，UDP数据报的长度可以为奇数字节，但是检验和算法是把若干个16bit字相加。解决方法是必要时在最后增加填充字节0，这只是为了检验和的计算（也就是说，可能增加的填充字节不被传送）。其次，UDP数据报和TCP段都包含一个12字节长的伪首部，它是为了计算检验和而设置的。伪首部包含IP首部一些字段。其目的是让 UDP两次检查数据是否已经正确到达目的地（例如，IP没有接受地址不是本主机的数据报，以及IP没有把应传给另一高层的数据报传给UDP）。
![238](https://img-blog.csdnimg.cn/20191201161326500.PNG)
![239](https://img-blog.csdnimg.cn/20191201161308431.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
UDP数据报中的伪首部格式，在该图中，我们特地举了一个奇数长度的数据报例子，因而在计算检验和时需要加上填充字节。注意，UDP数据报的长度在检验和计算过程中出现两次。如果检验和的计算结果为 0，则存入的值为全 1（65535），这在二进制反码计算中是等效的。如果传送的检验和为0，说明发送端没有计算检验和。**UDP和TCP的伪首部格式是一样的**。
![240](https://img-blog.csdnimg.cn/20191201162015854.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
```clike
unsigned short checksum(unsigned short *buf,int nword)
{
    unsigned long sum;
  
    for(sum = 0; nword > 0; nword--)
        sum += *buf++;
    sum = (sum >> 16) + (sum & 0xffff);
    sum += (sum >> 16);
  
    return ~sum;
}
```

# TCP报文格式
- 每个TCP段都包含源端和目的端的端口号，用于寻找发端和收端应用进程。这两个值加上IP首部中的源端IP地址和目的端IP地址唯一确定一个TCP连接。
- 有时，一个IP地址和一个端口号也称为一个插口（socket）。这个术语出现在最早的TCP规范（RFC793）中，后来它也作为表示伯克利版的编程接口。插口对（socket pair）(包含客户IP地址、客户端口号、服务器 IP地址和服务器端口号的四元组 )可唯一确定互联网络中每个TCP连接的双方。
- 序号用来标识从TCP发端向TCP收端发送的数据字节流，它表示在这个报文段中的的第一个数据字节。如果将字节流看作在两个应用程序间的单向流动，则 TCP用序号对每个字节进行计数。序号是32bit的无符号数，序号到达`2^32-1`后又从0开始。当建立一个新的连接时，SYN标志变1。序号字段包含由这个主机选择的该连接的初始序号ISN（Initial Sequence Number）。该主机要发送数据的第一个字节序号为这个ISN加1，因为SYN标志消耗了一个序号。既然每个传输的字节都被计数，确认序号包含发送确认的一端所期望收到的下一个序号。因此，确认序号应当是上次已成功收到数据字节序号加 1。只有ACK标志为1时确认序号字段才有效。发送ACK无需任何代价，因为32bit的确认序号字段和ACK标志一样，总是TCP首部的一部分。因此，我们看到一旦一个连接建立起来，这个字段总是被设置， ACK标志也总是被设置为1。TCP为应用层提供全双工服务。这意味数据能在两个方向上独立地进行传输。因此，连接的每一端必须保持每个方向上的传输数据序号。TCP可以表述为一个没有选择确认或否认的滑动窗口协议。我们说TCP缺少选择确认是因为TCP首部中的确认序号表示发方已成功收到字节，但还不包含确认序号所指的字节。当前还无法对数据流中选定的部分进行确认。例如，如果1～1024字节已经成功收到，下一报文段中包含序号从2049～3072的字节，收端并不能确认这个新的报文段。它所能做的就是发回一个确认序号为1025的ACK。它也无法对一个报文段进行否认。例如，如果收到包含1025～2048字节的报文段，但它的检验和错， TCP接收端所能做的就是发回一个确认序号为1025的ACK。
- 首部长度给出首部中32bit字的数目。需要这个值是因为任选字段的长度是可变的。这个字段占4 bit，因此TCP最多有60字节的首部。然而，没有任选字段，正常的长度是20字节。在TCP首部中有 6个标志比特。它们中的多个可同时被设置为1。

| 标志 | 含义 |
|--|--|
| URG | 紧急指针（urgent pointer）有效 |
| ACK | 确认序号有效 |
| PSH | 接收方应该尽快将这个报文段交给应用层 |
| RST | 重建连接 |
| SYN | 同步序号用来发起一个连接 |
| FIN | 发端完成发送任务 |

- TCP的流量控制由连接的每一端通过声明的窗口大小来提供。窗口大小为字节数，起始于确认序号字段指明的值，这个值是接收端正期望接收的字节。窗口大小是一个16bit字段，因而窗口大小最大为65535字节。如果需要更大的窗口，可以通过窗口刻度选项，它允许这个值按比例变化以提供更大的窗口。
- 检验和覆盖了整个的 TCP报文段：TCP首部和TCP数据。这是一个强制性的字段，一定是由发端计算和存储，并由收端进行验证。 TCP检验和的计算和UDP检验和的计算相似，使用一个伪首部。
- 只有当URG标志置1时紧急指针才有效。紧急指针是一个正的偏移量，和序号字段中的值相加表示紧急数据最后一个字节的序号。 TCP的紧急方式是发送端向另一端发送紧急数据的一种方式。
- 选项，最常见的可选字段是最长报文大小，又称为MSS (Maximum Segment Size)。每个连接方通常都在通信的第一个报文段（为建立连接而设置SYN标志的那个段）中指明这个选项。时间戳选项使发送方在每个报文段中放置一个时间戳值，发送方在第 1个字段中放置一个 32bit的值，接收方在应答字段中回显这个数值，包含这个选项的TCP首部长度将从正常的20字节增加为32字节。
![241](https://img-blog.csdnimg.cn/20191201170730221.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![242](https://img-blog.csdnimg.cn/20191201170751594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
TCP报文段中的数据部分是可选的。在一个连接建立和一个连接终止时，双方交换的报文段仅有TCP首部。如果一方没有数据要发送，也使用没有任何数据的首部来确认收到的数据。在处理超时的许多情况中，也会发送不带任何数据的报文段。

## TCP选项
TCP最多有60字节的首部，正常的长度是20字节，所以选项最多为40字节，每个选项的开始是1字节kind字段，说明选项的类型。kind字段为0和1的选项仅占1个字节。其他的选项在kind字节后还有len字节。它说明的长度是指总长度，包括kind字节和len字节。设置无操作选项的原因在于允许发方填充字段为 4字节的倍数。
![280](https://img-blog.csdnimg.cn/20200219164751543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
下面是一个TCP3次握手的抓包，用到了3个选项，最长报文大小（MSS），窗口扩大选项（WS），SACK
![276](https://img-blog.csdnimg.cn/20200219162511564.png)
- 窗口扩大选项
窗口扩大选项使TCP的窗口定义从16bit增加为32bit。这并不是通过修改TCP首部来实现
的，TCP首部仍然使用16bit ，而是通过定义一个选项实现对16bit 的扩大操作(scaling operation)来完成的。于是TCP在内部将实际的窗口大小维持为32bit的值。
在图中，字节的移位记数器取值为2。表示窗口大小为262140字节（`65535×(2^2)`）。这个选项只能够出现在一个SYN报文段中，因此当连接建立起来后，在每个方向的扩大因子是固定的。为了使用窗口扩大，两端必须在它们的SYN报文段中发送这个选项。主动建立连接的一方在其 SYN中发送这个选项，但是被动建立连接的一方只能够在收到带有这个选项的SYN之后才可以发送这个选项。每个方向上的扩大因子可以不同。
如果主动连接的一方发送一个非零的扩大因子，但是没有从另一端收到一个窗口扩大选项，它就将发送和接收的移位记数器置为0。这就允许较新的系统能够与较旧的、不理解新选项的系统进行互操作。
Host Requirements RFC要求TCP接受在任何报文段中的一个选项（只有前面定义的一个选项，即最大报文段大小，仅在SYN报文段中出现）。它还进一步要求TCP忽略任何它不理解的选项。这就使事情变得容易，因为所有新的选项都有一个长度字段（图1 8 - 2 0）。
假定我们正在使用窗口扩大选项，发送移位记数为S，而接收移位记数则为R。于是我们从另一端收到的每一个16bit的通告窗口将被左移 R位以获得实际的通告窗口大小。每次当我们向对方发送一个窗口通告的时候，我们将实际的 32 bit窗口大小右移S比特，然后用它来替换TCP首部中的16 bit的值。
TCP根据接收缓存的大小自动选择移位计数。这个大小是由系统设置的，但是通常向应用进程提供了修改途径。
MSS选项没有被设置的时候，本端就会使用536这个值，这个值的来源在于IPv4有一个最小重组缓冲区大小，其值为576字节，是IPv4的任何实现都必须保证支持的最小IP数据报大小，IPv6对应的值为1500字节。从运输层到IP层，PDU增加了IP首部，20字节，因此TCP包为556字节，再去掉首部20字节，即为最小的MSS，即536字节。
- SACK
标准的TCP确认机制中，如果发送方发送了0-1000序号之间的数据，接收方收到了0-100、300-1000，那么接收方只能向发送方确认101，这时发送方会重传所有101-1000之间的数据，实际上这是不必要的，因为有可能仅仅是丢了一小段而已，但是在标准的TCP确认机制中，发送方无法感知这一事情，只能重传从101开始的所有数据。

# TCP分段
UDP会造成IP分片，但是TCP不会造成IP分片，这是因为TCP自己支持分段，将数据拆分成小包传给网络层，最大的分段大小MSS相当于MTU减去IP头和TCP头，所以一个MSS恰好装进MTU中，几乎不会造成IP分片。TCP在建立连接进行的三次握手的前两个握手包中双方互相声明自己的MSS。
IP数据报分片后，只有第一片带有UDP首部或ICMP首部，其余的分片只有IP头部，到了端点后根据IP头部中的信息再网络层进行重组。而TCP报文段的每个分段中都有TCP首部，到了端点后根据TCP首部的信息在传输层进行重组。IP数据报分片后，只有到达目的地后才进行重组，而不是向其他网络协议，在下一站就要进行重组。对IP分片的数据报来说，即使只丢失一片数据也要重新传整个数据报（既然有重传，说明传输层使用的是具有重传功能的协议，如TCP协议）。这是因为IP层本身没有超时重传机制，必须由更高层协议来负责超时和重传。当来自TCP报文段的某一段（在IP数据报的某一片中）丢失后，TCP在超时后会重发整个TCP报文段，该报文段对应于一份IP数据报（可能有多个IP分片），没有办法只重传数据报中的一个数据分片。
TCP是一个字节流协议，TCP绝不会以杂乱的次序给接收应用程序发送数据。因此，TCP接收端可能会被迫先保持大序列号的数据不交给应用程序，直到缺失的小序列号的报文段被填满，最终按序将数据提交给应用程序，这也是TCP重组的目的。
