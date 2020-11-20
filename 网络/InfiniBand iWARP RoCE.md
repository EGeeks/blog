# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [InfiniBand, RDMA, iWARP, RoCE](http://blog.163.com/guaiguai_family/blog/static/20078414520141023103953705/)
> [RDMA Vs DPDK](https://www.jianshu.com/p/09b4b756b833?utm_campaign)
> [有哪些比较好的基于dpdk实现的tcp/ip stack?](https://www.zhihu.com/question/47396779/answer/139271653)
> [关于Infiniband的一些介绍](http://aze12.blog.163.com/blog/static/8620441220086411596665/)
> [InfiniBand：还会有多少人想起我](http://www.360doc.com/content/07/0904/16/494_718778.shtml)

# InfiniBand iWARP RoCE
parallel file system比如PVFS2/OrangeFS，Lustre，它们都声称支持 InfiniBand 网络连接技术，InfiniBand原生支持RDMA，可以不通过OS内核以及TCP/IP协议栈在网络上传输数据，因此延迟低，CPU消耗少。InfiniBand，FibreChannel，10Gbps Ethernet之间互相竞争。RDMA 技术有好几种规范来实现，
- InfiniBand：这是正统，InfiniBand设计之初就考虑了RDMA，InfiniBand从硬件级别保证可靠传输;
- iWARP：基于TCP or SCTP做RDMA，利用TCP or SCTP达到可靠传输，对网络设备的要求比较少;
- RoCE：基于Ethernet做RDMA，消耗的资源比iWARP少，支持的特性比iWARP多，需要FCoE做可靠传输。从wikipedia的评价看 RoCE 还是比正统的 InfiniBand 差点。

上面三种实现都是需要硬件支持的，IB需要支持IB规范的网卡和交换机，iWARP和 RoCE都可以使用普通的以太网交换机，但是需要支持iWARP或者RoCE的网卡。软件上Solaris、Linux、Windows都有支持，在API层面这篇文章有个入门的介绍：`Introduction to Remote Direct Memory Access (RDMA) `，可以使用`http://www.openfabrics.org/` 提供的 libibverbs库(Debian Linux有提供)，这个库似乎也支持Windows上的原生RDMA API `"Network Direct"`。另外也有一些其它 API 规范，比如DAT组织制定的kDAPL(让kernel driver可以访问RDMA功能)和uDAPL(让user space进程可以访问RDMA功能), OpenGroup制定的IT-API和RNICPI: 
`https://software.intel.com/en-us/articles/access-to-infiniband-from-linux`
`http://www.zurich.ibm.com/sys/rdma/interfaces.html`
`http://rdma.sourceforge.net/`
另外IETF制定了iSCSI Extensions for RDMA(iSER)和SDP(Sockets Direct Protocol，基于RDMA替换TCP的流式传输层协议，RDMA本身提供了可靠传输机制)两个协议。Java 7引入了对SDP的支持:`https://docs.oracle.com/javase/tutorial/sdp/sockets/index.html`，Apache Qpid 消息队列也支持 RDMA`https://packages.debian.org/sid/librdmawrap2` ，学习RDMA的网站：`http://www.rdmamojo.com/2013/06/08/tips-and-tricks-to-optimize-your-rdma-code/`

# 背板互联总线
看来以太网可以取代SRIO，毕竟有RDMA，而RDMA不是很新的技术，我突然记得在学校的时候听说过六院的超算天河用的Infiniband作为背板互联，教研室以前一个车载的存储阵列也是用的InfiniBand。RDMA不是新的技术，只是以前在InfiniBand上，导致比较小众，现在IB移植到RoCE（v2）上，扩展了应用场景。DPDK需要造的轮子太多，并且支持的厂家主要是intel，主要是intel自己的标准和协议，需要程序员做的事情太多，开发量太大，相当于程序员要把整个IP协议底层实现一遍。作为同样的类似技术，我推荐已经标准化的，兼容性好很多，不仅仅是intel，也不仅仅是以太网，还有InfiniBand等其他网络，都支持这个协议，一次写完，多种硬件，多种平台，多种厂家，国际标准，价格便宜，现在很多杂牌三四百元的万兆以太网卡都已经支持的RDMA/ROCE技术，不需要自己实现底层， 底层代码全部都已经国际标准化了，直接拿来用就可以了，不需要自己造轮子，性能与DPDK基本不相上下，也是在用户层实现了整个TCP/IP协议，国家超算中心已经有很多应用都是RDMA/ROCE技术开发的网络层，还有最近2届高校大学生超算全国性竞赛，都是指定必须使用RDMA技术作为网络层接口, 也说明了国家层面上的倾向（作者：郭忠明 链接：`https://www.zhihu.com/question/47396779/answer/139271653`）。

# FPGA RDMA IP（RNIC）
Xilinx RDMA pg294
![这里写图片描述](https://img-blog.csdn.net/20180603223751849?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
NVMEoF
![这里写图片描述](https://img-blog.csdn.net/20180603223650881?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
