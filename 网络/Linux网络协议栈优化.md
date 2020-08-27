# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Linux转发性能评估与优化(转发瓶颈分析与解决方案)](https://blog.csdn.net/dog250/article/details/46666029?locationNum=1)
> [Linux性能优化 第七章 性能工具：网络](https://www.cnblogs.com/tcicy/p/10080421.html)
> [Linux性能调优方法总结（一）](https://blog.csdn.net/twypx/article/details/80290759)
> [Linux性能优化-网络性能评估](https://blog.csdn.net/hixiaoxiaoniao/article/details/87170521)
> [Linux性能优化-网络基础](https://blog.csdn.net/hixiaoxiaoniao/article/details/87079414)
> [Zynq-7000 AP SoC - Performance - Ethernet Packet Inspection - Bare Metal - Redirecting Packets to PL Tech Tip](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841889/Zynq-7000+AP+SoC+-+Performance+-+Ethernet+Packet+Inspection+-+Bare+Metal+-+Redirecting+Packets+to+PL+Tech+Tip)

# 目标
求教，Xilinx Zynqmp，4核cortex-a53，主频1.3GHz，SMP架构无NUMA，单独L1 Cache，共享L2 Cache，外接10G以太网，ARM单核性能弱，iperf3测试TCP只有280MB/s，网卡支持多达16通道通道DMA，但是所有发送通道共用一个中断，所有接收通道共用一个CPU中断，也就是说总共两个中断，驱动中断函数中通过读寄存器判断是哪个通道的中断（所以这里即使多通道不共用中断，中断函数也没有根据中断号直接定位哪个通道进行数据收发），驱动支持NAPI，NAPI的权重设置为64，测试发现，此数值改大改小均会引起性能下降，由于单核性能上不去，打算运行两个TCP socket，希望带宽可以乘以2，两个iperf3用-A进行绑核到CPU2和CPU3，网卡中断在CPU0，top命令也看到绑核成功，两路iperf3带宽相加还是280MB/s，将发送和接收中断分别放到CPU0和CPU1，性能下降。尝试针对每个通道预分配多个skb，用完再一次性分配多个skb，此时该通道NAPI驱动的接收函数应该会卡住一小段时间，这个时候CPU会自动从其他通道发送吗，实际测试性能下降，看来还是CPU还是一核干活，多核围观，求教了。

# TCP Win
> [linux下如何配置TCP参数设置详解](https://www.cnblogs.com/jdonson/p/4746094.html)
> [Linux网络编程-tcp缓存设置](https://www.cnblogs.com/whuqin/p/5580895.html)

内核设置的套接字缓存
- /proc/sys/net/core/rmem_default，net.core.rmem_default，套接字接收缓存默认值 (bit)
- /proc/sys/net/core/wmem_default，net.core.wmem_default，套接字发送缓存默认值 (bit)
- /proc/sys/net/core/rmem_max，net.core.rmem_max，套接字接收缓存最大值 (bit)
- /proc/sys/net/core/wmem_max，net.core.wmem_max，发送缓存最大值 (bit)

tcp缓存
- /proc/sys/net/ipv4/tcp_rmem：net.ipv4.tcp_rmem，接收缓存设置，依次代表最小值、默认值和最大值(bit)
- /proc/sys/net/ipv4/tcp_wmem：net.ipv4.tcp_wmem，发送缓存设置，依次代表最小值、默认值和最大值(bit)
- /proc/sys/net/ipv4/tcp_mem: net.ipv4.tcp_mem，tcp整体缓存设置，对所有tcp内存使用状况的控制，单位是页，依次代表TCP整体内存的无压力值、压力模式开启阀值、最大使用值，用于控制新缓存的分配是否成功

```bash
sysctl -w net.core.rmem_max=8388608
sysctl -w net.ipv4.tcp_mem='8388608 8388608 8388608'
```

