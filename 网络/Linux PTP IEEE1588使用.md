# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [TI：在KeyStone器件实现IEEE1588时钟方案](https://www.eefocus.com/communication/339581)
> [One Step, Two Steps](https://www.jianshu.com/p/6cb197183a3c)
> [IEEE1588精确网络时钟同步协议简介 II](https://www.cnblogs.com/byeyear/archive/2012/10/14/2723336.html)
> [IEEE1588精密网络同步协议（PTP）-v2.0协议浅析](http://blog.sina.com.cn/s/blog_4b0cdab70100k4fv.html)
> [Precise Time Protocol (PTP)](https://wiki.wireshark.org/Protocols/ptp)
> [Ubuntu 设置PTP时间同步](https://blog.csdn.net/u013431916/article/details/83054369)
> [PTP简介](https://www.cnblogs.com/zhuchunling/p/5911814.html)
> [PTP(Precision Time Protocol)高精度时间同步协议+CS模式测试代码](https://blog.csdn.net/woswod/article/details/82345380)
> [IEEE 1588精确时钟同步协议的研究](https://blog.csdn.net/a746742897/article/details/53286040)
> [12 - 利用LinuxPTP进行时间同步(软/硬件时间戳) - 研一](https://blog.csdn.net/BUPTOctopus/article/details/86246335)
> [解剖PTP协议](https://www.cnblogs.com/dakewei/p/10881699.html)
> [The Linux PTP Project](http://linuxptp.sourceforge.net/)
> [一种IEEE 1588硬件的设计和实现](https://blog.csdn.net/a746742897/article/details/53285920)
> [stm32实现1588协议](https://blog.csdn.net/iodoo/article/details/9785871)
> [ptpd 1588协议关于多个定时器的实现方式解析](https://blog.csdn.net/linxing927/article/details/73649808)
> [编译安装 PTPdv2](https://blog.csdn.net/qq_30943197/article/details/87966486)
> [FSL 1588 PTPD简要分析！](https://blog.csdn.net/zjy900507/article/details/69747314)
> [工业级IEEE1588精密主时钟（从时钟）模块技术详解](https://blog.csdn.net/weixin_44990608/article/details/90716040)
> [ptpd 守护程序](https://blog.csdn.net/zjy900507/article/details/69744473)
> [IEEE1588 ( PTP ) 协议简介](https://www.cnblogs.com/adaminxie/p/6754644.html)

# ptp
[IEEE 1588PTP协议借鉴了NTP技术，具有容易配置、快速收敛以及对网络带宽和资源消耗少等特点。IEEE1588标准的全称是“网络测量和控制系统的精密时钟同步协议标准（IEEE 1588 Precision Clock Synchronization Protocol）”，简称PTP（Precision Timing Protocol），它的主要原理是通过一个同步信号周期性的对网络中所有节点的时钟进行校正同步，可以使基于以太网的分布式系统达到精确同步，IEEE 1588PTP时钟同步技术也可以应用于任何组播网络中。
　　IEEE 1588将整个网络内的时钟分为两种，即普通时钟（Ordinary Clock，OC）和边界时钟（Boundary Clock，BC），只有一个PTP通信端口的时钟是普通时钟，有一个以上PTP通信端口的时钟是边界时钟，每个PTP端口提供独立的PTP通信。其中，边界时钟通常用在确定性较差的网络设备（如交换机和路由器）上。从通信关系上又可把时钟分为主时钟和从时钟，理论上任何时钟都能实现主时钟和从时钟的功能，但一个PTP通信子网内只能有一个主时钟。整个系统中的最优时钟为最高级时钟GMC（Grandmaster Clock），有着最好的稳定性、精确性、确定性等。根据各节点上时钟的精度和级别以及UTC（通用协调时间）的可追溯性等特性，由最佳主时钟算法（Best Master Clock）来自动选择各子网内的主时钟；在只有一个子网的系统中，主时钟就是最高级时钟GMC。每个系统只有一个GMC，且每个子网内只有一个主时钟，从时钟与主时钟保持同步。](https://blog.csdn.net/weixin_44990608/article/details/90716040)
PTP中的名词，
![207](https://img-blog.csdnimg.cn/20191010214741879.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# ptp4l
使用记得加`sudo`
```shell
$ ptp4l -h

usage: ptp4l [options]

 Delay Mechanism

 -A        Auto, starting with E2E
 -E        E2E, delay request-response (default)
 -P        P2P, peer delay mechanism

 Network Transport

 -2        IEEE 802.3
 -4        UDP IPV4 (default)
 -6        UDP IPV6

 Time Stamping

 -H        HARDWARE (default)
 -S        SOFTWARE
 -L        LEGACY HW

 Other Options

 -f [file] read configuration from 'file'
 -i [dev]  interface device to use, for example 'eth0'
           (may be specified multiple times)
 -p [dev]  PTP hardware clock device to use, default auto
           (ignored for SOFTWARE/LEGACY HW time stamping)
 -s        slave only mode (overrides configuration file)
 -t        transparent clock
 -l [num]  set the logging level to 'num'
 -m        print messages to stdout
 -q        do not print messages to the syslog
 -v        prints the software version and exits
 -h        prints this message and exits
$ sudo ptp4l -m -i enp1s0f0
```
配合x520万兆网卡，
![205](https://img-blog.csdnimg.cn/20191009202008525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# ptpdv2
编译，
```shell
$ autoreconf -vi
$ ./configure
$ make
$ make install
```
交叉编译，生成的`config.h`中，使用系统malloc，
```c
#define malloc rpl_malloc
/* #undef malloc */
```

