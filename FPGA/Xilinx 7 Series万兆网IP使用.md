# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 方法
万兆网IP，axi-dma，axi-pcie-bridge，搭建一个基于FPGA的万兆网卡，移植petalinux的驱动到Ubuntu16.04.6上，测试结果如下，
```shell
qe@qe-pc:~/project$ ./iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 192.168.10.6, port 6668
[  5] local 192.168.10.8 port 5201 connected to 192.168.10.6 port 6669
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec   231 MBytes  1.93 Gbits/sec                  
[  5]   1.00-2.00   sec   235 MBytes  1.97 Gbits/sec                  
[  5]   2.00-3.00   sec   233 MBytes  1.96 Gbits/sec                  
[  5]   3.00-4.00   sec   236 MBytes  1.98 Gbits/sec                  
[  5]   4.00-5.00   sec   232 MBytes  1.94 Gbits/sec                  
[  5]   5.00-6.00   sec   232 MBytes  1.95 Gbits/sec                  
[  5]   6.00-7.00   sec   232 MBytes  1.95 Gbits/sec                  
[  5]   7.00-8.00   sec   232 MBytes  1.95 Gbits/sec                  
[  5]   8.00-9.00   sec   240 MBytes  2.01 Gbits/sec                  
[  5]   9.00-10.00  sec   239 MBytes  2.00 Gbits/sec                  
[  5]  10.00-10.04  sec  8.53 MBytes  2.01 Gbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-10.04  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-10.04  sec  2.30 GBytes  1.96 Gbits/sec                  receiver
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
iperf3: interrupt - the server has terminated
qe@qe-pc:~/project$ ./iperf3 -c 192.168.10.6 -w 4M
Connecting to host 192.168.10.6, port 5201
[  4] local 192.168.10.8 port 37564 connected to 192.168.10.6 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  37.3 MBytes   313 Mbits/sec    0   28.5 KBytes       
[  4]   1.00-2.00   sec  36.9 MBytes   310 Mbits/sec    0   28.5 KBytes       
[  4]   2.00-3.00   sec  36.9 MBytes   310 Mbits/sec    0   28.5 KBytes       
[  4]   3.00-4.00   sec  36.9 MBytes   309 Mbits/sec    0   28.5 KBytes       
[  4]   4.00-5.00   sec  36.9 MBytes   310 Mbits/sec    0   28.5 KBytes       
[  4]   5.00-6.00   sec  36.9 MBytes   310 Mbits/sec    0   28.5 KBytes       
[  4]   6.00-7.00   sec  36.9 MBytes   310 Mbits/sec    0   28.5 KBytes       
[  4]   7.00-8.00   sec  36.9 MBytes   310 Mbits/sec    0   28.5 KBytes       
[  4]   8.00-9.00   sec  36.9 MBytes   309 Mbits/sec    0   28.5 KBytes       
[  4]   9.00-10.00  sec  36.9 MBytes   310 Mbits/sec    0   28.5 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec   370 MBytes   310 Mbits/sec    0             sender
[  4]   0.00-10.00  sec   369 MBytes   310 Mbits/sec                  receiver

iperf Done.
```
参考我的[博文](https://blog.csdn.net/Zhu_Zhu_2009/article/details/100103495)，发送速度还不如一个zynq（80MB/s左右），原因是驱动没有优化，每发送一个包都需要中断响应一次，中断还要透过pcie msi传输，所以效率低，接收采用了napi，所以性能相比zynq有所提高，但是还是不太理想，需要进一步采用FPGA硬件加速。
