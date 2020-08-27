# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 自动重发请求机制

很多保证可靠性的协议都有自动重发请求ARQ（Automatic Repeat reQuest）特性，在ARQ中，接收机发送已接收到的数据的ACK，发送者会重传未接收到ACK的数据。TCP保存发送数据的序列号，已经发送的数据被放入重传队列，并启动与数据相关联的定时器。如果在定时器用完之前没有接收到数据序列的确认，则会发起重传。但这样会产生很多问题，比如发送者应该等待多久就发起重传，这会影响到性能。TCP通过SACK对无序数据ACK，避免不必要的往返确认交互，浪费效率。

# TCP重传
TCP考重传机制来保证数据的可靠性。当TCP传输一个带有数据的报文段的时候，TCP就会将它复制一份放到重传队列中，然后开启一个定时器，当收到对应的ACK之后，就从队列中删除这个报文，如果产生超时，则该报文会被重传。
最新的重传方法在RFC62986中描述，下面三个变量用来计算RTO

 - srtt 是RTT的平均值
 - rttvar 是RTT值
 - rto 是重传超时时间
 - g 是时钟精度

在第一次计算RTT之前，
```
rto = 1000ms
```
第一次计算RTT，
```c
srtt = R
rttvar = R/2
rto = srtt + max(g, 4*rttvar)
```
在接下来的计算中，
```c
alpha = 0.125
beta = 0.25
rttvar = (1 - beta) * rttvar + beta * abs(srtt - r)
srtt = (1 - alpha) * srtt + alpha * r
rto = srtt + max(g, 4*rttvar)
```
计算出RTO值之后，如果小于1s，那么向上取整到1s，最大是按60s取整，这是为了避免产生太多重传，引起网络阻塞。时钟精度可以去相当大的值，比如500ms到1s之间，但现在的操作系统比如linux，RTO最小为200ms，同时g取值为1ms。linux下的RTO计算参考博客[TCP中RTT的测量和RTO的计算](https://blog.csdn.net/zhangskd/article/details/7196707)。

# Karn算法

Karn算法是为了防止RTT测算出错误的结果，它要求RTT的采样值不能是发生重传的包，因为我们无法确定某个ACK包是前面报文的ACK还是重传包的ACK，但是如果开启了TCP时间戳选项，那就可以分辨出ACK对应的是哪个报文，就可以应用这个ACK来更新RTT。

#RTO定时器管理

RFC6298推荐用下面的方法来管理RTO定时器，

 - 当发送报文时，RTO定时器若没有开启，则打开RTO定时器，设置超时时间为变量rto的值
 - 当所有数据报文都收到ACK，关闭RTO定时器
 - 当收到ACK之后，用新的的rto重启定时器
 
当RTO定时器超时（expire），重传最早的没有收到ACK的报文，增加（back off）超时重传时间（比如rto = rto * 2，指数增加），重启定时器。当back off发生之后，再次测算到RTT之后，RTO可能会迅速减小，协议栈可能会清除srtt和rttvar当back off等待ACK的时候。

#请求重传

TCP不仅仅是依靠发送端的定时器来控制丢包，接收方也可以主动通知发送方重传丢失报文，利用SACK（Selective Acknowledgment），接收端可以通过发送带SACK选项的报文段准确告诉发送方需要重传哪个报文。SACK具体参考博文[TCP-IP详解：SACK选项（Selective Acknowledgment）](https://blog.csdn.net/wdscq1234/article/details/52503315?locationNum=3)，和[TCP 选择性应答的性能权衡](https://www.ibm.com/developerworks/cn/linux/l-tcp-sack/)。

