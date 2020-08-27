# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [linux的反向路由检测](http://blog.51cto.com/wujingfeng/2104489)
> [Linux内核的 反向路由检查机制](https://blog.csdn.net/changqing1234/article/details/81068780)
> [linux双网卡配置路由的一次实战经历](https://blog.csdn.net/jackliu16/article/details/79369163)
> [在LINUX下配置网桥](https://blog.csdn.net/yyh_linux_note/article/details/90268912)
> [linux下brctl配置网桥](https://blog.csdn.net/xiangxizhishi/article/details/79438449)
> [linux配置网桥](https://www.cnblogs.com/liyuanhong/articles/10096854.html)


# 多网卡绑定


# 配置静态ip
编辑文件`/etc/network/interfaces`，
```bash
storage@storage-pc:~$ cat /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static 
address 192.168.26.7
netmask 255.255.255.0
gateway 192.168.26.254

auto enp3s0d1
iface enp3s0d1 inet static 
address 192.168.5.199
netmask 255.255.255.0
gateway 192.168.5.254
```
重启或者重新加载配置，
```bash
# sudo service network restart
sudo service network-manager restart
```

# 配置路由
```bash
route add -net 192.168.12.0 gw 192.168.6.254 netmask 255.255.255.0 dev eth1
route del 192.168.6.0 gw 192.168.6.254
```
# ping不通与反向路由
```shell
echo 0 > /proc/sys/net/ipv4/conf/bond0.20/rp_filter
echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter
/etc/sysctl.conf
net.ipv4.conf.bond0.23.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0

echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter 
echo 0 > /proc/sys/net/ipv4/conf/eth0/rp_filter 
echo 0 > /proc/sys/net/ipv4/conf/eth1/rp_filter 
```

# 双网卡报文转发
类似于一个3层软交换，`PC1<------>PC2<------>PC3`
| 主机名 | 所在网段 | 分配IP | 网关 |
|:--:|:----:|:-----:|:----:|
| PC1 | 192.168.2.0/24 | 192.168.2.2/24 | 192.168.2.1 |
| PC2与PC1相连的网卡eth1 |192.168.2.0/24|192.168.2.1/24| |
| PC2与PC3相连的网卡eth0 |192.168.1.0/24|192.168.1.1/24| |
| PC3 |192.168.1.0/24|192.168.1.2/24|192.168.1.1|

在PC2上输入，
```bash
# echo '1' > /proc/sys/net/ipv4/ip_forward
# route add -net 192.168.1.0 netmask 255.255.255.0 dev eth0
# route add -net 192.168.2.0 netmask 255.255.255.0 dev eth1
```


