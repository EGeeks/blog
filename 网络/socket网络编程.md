# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [setsockopt ：SO_LINGER 选项设置](https://blog.csdn.net/factor2000/article/details/3929816)
> [Linux网络编程socket选项之SO_LINGER,SO_REUSEADDR](https://blog.csdn.net/feiyinzilgd/article/details/5894300)
> [TCP socket如何清空发送缓冲区](https://bbs.csdn.net/topics/30018461)
> [socket怎样随时刷新缓冲区的内容！！！](https://bbs.csdn.net/topics/30018461)
> [socket UDP广播的发送和接收示例](https://blog.csdn.net/mao0514/article/details/89203203)
> [关于UDP接收icmp端口不可达(port unreachable)](http://blog.chinaunix.net/uid-28458801-id-4990181.html)

# UDP收发
> [UDP通信](https://blog.csdn.net/fanle93/article/details/90666484)
> [linux下实现UDP通信](https://www.cnblogs.com/ssyfj/p/12107455.html)

# UDP服务器回复端口不可达
ECONNREFUSED是ICMP返回的，ICMP是一种专门的控制协议，控制和指示IP层发生的事件，所以UDP也可以通过ICMP来获知端口不可达，
```c
setsockopt(fd, IPPROTO_IP, IP_RECVERR , &val, sizeof(int)); 
```

# SO_LINGER SO_REUSEADDR SO_REUSEPORT
1. 设置 l_onoff为0，则该选项关闭，l_linger的值被忽略，等于内核缺省情况，close调用会立即返回给调用者，如果可能将会传输任何未发送的数据；
2. 设置 l_onoff为非0，l_linger为0，则套接口关闭时TCP夭折连接，TCP将丢弃保留在套接口发送缓冲区中的任何数据并发送一个RST给对方，而不是通常的四分组终止序列，这避免了TIME_WAIT状态；
3. 设置 l_onoff 为非0，l_linger为非0，当套接口关闭时内核将拖延一段时间（由l_linger决定）。如果套接口缓冲区中仍残留数据，进程将处于睡眠状态，直 到（a）所有数据发送完且被对方确认，之后进行正常的终止序列（描述字访问计数为0）或（b）延迟时间到。此种情况下，应用程序检查close的返回值是非常重要的，如果在数据发送完并被确认前时间到，close将返回EWOULDBLOCK错误且套接口发送缓冲区中的任何数据都丢失。close的成功返回仅告诉我们发送的数据（和FIN）已由对方TCP确认，它并不能告诉我们对方应用进程是否已读了数据。如果套接口设为非阻塞的，它将不等待close完成。

# TCP send之后，报文立刻从网卡发出
1. setsockopt SO_DONTLINGER
2. BOOL bNoDelay = TRUE;
setsockopt(skt, IPPROTO_TCP, TCP_NODELAY, (char FAR *)&bNoDelay, sizeof(BOOL));

# 组播（多播）
> [组播 IP_MULTICAST_LOOP回环在Linux和Windows的差异](https://blog.csdn.net/lucky_greenegg/article/details/84938565)
> [Linux之UDP组播示例——双向通信](https://blog.csdn.net/qq_26600237/article/details/81036817)
> [局域网发现之UDP组播](https://blog.csdn.net/lixin88/article/details/55209630)
> [windows下UDP组播](https://blog.csdn.net/a1173356881/article/details/81608868)
> [IP Multicast](https://docs.microsoft.com/zh-cn/windows/win32/winsock/ip-multicast-2)
> [组播 IP_MULTICAST_LOOP回环在Linux和Windows的差异](https://blog.csdn.net/lucky_greenegg/article/details/84938565)
> [UDP 单播、广播和多播](https://www.cnblogs.com/lidabo/p/5865045.html)
> [收不到组播问题 rp_filter](https://blog.csdn.net/sz_liao/article/details/9278849)
> [rp_filter与多播](https://blog.csdn.net/winlerman/article/details/49587381)
> [Linux udp 多网卡组播接收问题](https://bbs.csdn.net/topics/391070884)
> [linux 在多网卡下的设备的UDP 组播问题总结](https://blog.csdn.net/little_water/article/details/52550514?utm_source=blogxgwz8)
> [CentOS多网卡下 应用层无法收到组播的问题解决](https://blog.csdn.net/norsd/article/details/61196474)
> [UDP多网卡广播问题解决方案](https://blog.csdn.net/wmx313880747/article/details/44942499)
> [linux udp组播接收问题及原理分析](https://blog.csdn.net/rcfalcon/article/details/35221759)
> [基于 UDP 的 组播、广播详解](https://www.cnblogs.com/schips/p/12552534.html)
> [UDP组播的实现](https://blog.csdn.net/gua_MASS/article/details/51339096)
> [UDP 用户数据报格式（单播+组播）](https://blog.csdn.net/u010018619/article/details/79415128)
> [UDP 单播、广播、多播](https://www.cnblogs.com/yyy1234/p/10417383.html)
> [UDP 单播、广播和多播](https://www.cnblogs.com/lidabo/p/5865045.html)

IP 组播通信必须依赖于 IP 多播地址，在 IPv4 中它是一个 D 类 IP 地址，范围从 224.0.0.0 到 239.255.255.255，并被划分为局部链接多播地址、预留多播地址和管理权限多播地址3类：
- 局部链接多播地址范围在 224.0.0.0~224.0.0.255，这是为路由协议和其它用途保留的地址，路由器并不转发属于此范围的IP包；
- 预留多播地址为 224.0.1.0~238.255.255.255，可用于全球范围（如Internet）或网络协议；
- 管理权限多播地址为 239.0.0.0~239.255.255.255，可供组织内部使用，类似于私有 IP 地址，不能用于 Internet，可限制多播范围。

另外一种说法，
- 224.0.0.0～224.0.0.255为预留的组播地址（永久组地址），地址224.0.0.0保留不做分配，其它地址供路由协议使用；
- 224.0.1.0～224.0.1.255是公用组播地址，可以用于Internet；
- 224.0.2.0～238.255.255.255为用户可用的组播地址（临时组地址），全网范围内有效；
- 239.0.0.0～239.255.255.255为本地管理组播地址，仅在特定的本地范围内有效。

MAC地址的第8位为0时是单播地址，为1是是组播地址，组播MAC地址通常以01:00:5e或01:00:5f开头，后23位取IP地址的后23位，24位为0。

# 广播
广播IP为该子网网段的最后8字节为255，例如192.168.1.255，端口为 80，目的MAC地址为FF:FF:FF:FF:FF:FF。

## 多路组播一路收不到或者一路发不出
eth0 IP为192.168.5.196，eth1 IP为192.168.6.196，使用eth1收两路组播226.0.0.2:5409 226.0.0.6:5051，配置了路由route add default gw 192.168.5.254导致组播只能收到一路5051的，将路由改成route add default gw 192.168.6.254解决问题。
