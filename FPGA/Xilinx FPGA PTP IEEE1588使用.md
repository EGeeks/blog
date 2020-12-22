# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Zynq-7000 AP SoC - Precision Timing with IEEE1588 v2 Protocol Tech Tip](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841712/Zynq-7000+AP+SoC+-+Precision+Timing+with+IEEE1588+v2+Protocol+Tech+Tip)
> [Xilinx TSN Solution](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/25034864/Xilinx+TSN+Solution)
> [Linux AXI Ethernet driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842485/Linux+AXI+Ethernet+driver)
> [PTPv2与gPTP的不同点是什么？IEEE802.1AS and IEEE 1588的区别是什么](https://blog.csdn.net/yesxure/article/details/88636207)
> Xilinx pg157 (10 Gigabit Ethernet Subsystem v3.1 LogiCORE IP Product Guide)
> [1588v2（PTP）报文通用格式](http://www.023wg.com/message/message/cd_feature_1588v2_format-general.html)

# ieee1588
如果ptp报文承载在最常用的IPv4上，不同类型的报文，对应的IP多播地址不一样，
![211](https://img-blog.csdnimg.cn/20191011202158422.png)
不同类型的报文对应的UDP端口不一样，
![212](https://img-blog.csdnimg.cn/20191011202305910.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
ptp消息类型，
![213](https://img-blog.csdnimg.cn/20191011202341849.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
以Follow_Up为例，
![209](https://img-blog.csdnimg.cn/20191011201707581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# Xilinx 7 Series FPGA 10G IP
在7系列FPGA上，一个悲惨的事就是，10G以太网IP，被称之为legacy 10g，我估计有bug也不会有维护了，1588的时钟，systemtimer在上电开始的时候，从systemtimer_s_field和systemtimer_ns_field输入脚取一次值，然后开始递增，之后这个输入值再变化都不会传入内部了，每次IP都以第一次上电递增时间来打时间戳。
![208](https://img-blog.csdnimg.cn/20191011152448515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
时间戳端口上的ns部分竟然不能被5整除，即使systemtimer_clk为200MHz，
![210](https://img-blog.csdnimg.cn/20191011152805650.png)
如果我们自己内部维护一个ptp内部时钟源，可以通过ila抓收到的eth包来获取两个时钟的初始tick，内部时钟源为a，systemtimer为b（可以做到`a=b`），那么如果我们软件校准了内部时钟源，校准前内部时钟源为c，校准为d，一个包来到的时刻，内部时钟源为e，systemtimer为f，实际包的时间戳为：
```c
a  b                           a - b
c  b1                         c - b1 = a - b
d  b2 = b1 + 10ns             (d + 10) - b2 = d - c + (a - b)
e f                             
ts = e - (d - c + (a - b))
```
实际测试结果，`a!=b`，且相差还在变化，这里ptp的时钟源采用了200MHz，而10g以太网的时钟是165.25，不同源，不知道是不是这儿的原因导致的，暂时取个平均`a - b = 0xf4`，这个地方的精度误差达10ns，
```c
10g packet ts  : b13 c8c 4fd a23 9fde 15b b38 d1a
local packet ts: c10 d87 5f0 b17 a0d2 248 c2b e0c
delta          :  fd  fb  f3  f4   f4  ed  f3  f2

ts = e - (d - c + (a - b)) = e - (d - c + 244) = e + c - d - 244
```
时间戳都是由专门的axis接口读取，只有一个valid信号，官方是用axis fifo转成了axi接口，接收包的时间戳会有问题，如果包涌入量超过DMA搬运能力，这是包fifo溢出，但是时间戳fifo不一定溢出，这样包和时间戳就错位了，我觉得是有问题的，可以像千兆网IP那样，用逻辑把时间戳8字节插入包头，发送包的时间戳不是每个包都有，只有设置了ptp的cmd才会输出时间戳。
FPGA硬件上对软件发送的帧加上时间戳，所以承载在UDP上的PTPv2帧需要重新计算校验和，承载在RAW上的需要重新计算FCS，所以需要一个1588命令来控制，命令可以In-band（加在帧头）和Out-band（axis user信号）传输给IP核，硬件时间戳通过独立的axis接口读出。
# 授时原理
![TwoStepClockSynchronization](https://img-blog.csdnimg.cn/20191017171259132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
网络延时和时钟偏移，从钟根据Offset来纠偏。
```c
Delay = [ (t2-t1) + (t4-t3) ] / 2;
Offset = [ (t2-t1) - (t4-t3) ] / 2;
```

