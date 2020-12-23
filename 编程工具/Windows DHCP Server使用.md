# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Windows上构建DHCP Server](https://blog.csdn.net/fengel_cs/article/details/80632985)

# 使用
我这里下载到了DHCP Server 2.5.2，打开`dhcpwiz.exe`，一路默认，这里参数配一下IP范围，
![367](https://img-blog.csdnimg.cn/20201222182430801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击`Write INI file`，
![368](https://img-blog.csdnimg.cn/20201222182843524.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
打开`dhcpsrv.exe`，点击`Continue as tray app`，
![369](https://img-blog.csdnimg.cn/20201222182942453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
在板卡上使用DHCP客户端，有一个报错，参考[ip route 增删路由 报错问题 RTNETLINK answers: no such process解决办法](https://blog.csdn.net/one312/article/details/105006577)，应该是DHCP设路由的时候出的问题，本来就是同一网段的，不需要路由。
```bash
$ udhcpc -i eth0 -b -R -p /var/run/udhcpc.pid
udhcpc (v1.23.1) started
Sending discover...
Sending select for 192.168.26.10...
Lease of 192.168.26.10 obtained, lease time 86400
RTNETLINK answers: No such device
root@zynq:~# ifconfig
eth0      Link encap:Ethernet  HWaddr 16:25:C6:0F:D0:E9  
          inet addr:192.168.26.10  Bcast:192.168.26.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:22 errors:352 dropped:0 overruns:0 frame:352
          TX packets:25 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:3024 (2.9 KiB)  TX bytes:6210 (6.0 KiB)
          Interrupt:145 Base address:0xb000 
```
抓包，板子IP由192.168.26.21变成192.168.26.10，
![370](https://img-blog.csdnimg.cn/20201222183439199.png)

