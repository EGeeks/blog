# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [LWIP移植文件介绍](https://www.cnblogs.com/st-home/p/10899214.html)
> [手把手教你移植LWIP（ENC28J60）](https://blog.csdn.net/m0_37777700/article/details/83573726)
> [LwIP学习笔记——LwIP无操作系统移植](https://blog.csdn.net/tichimi3375/article/details/80721734)
> [LwIP BUG之ARP缓存](http://blog.sina.com.cn/s/blog_62a85b950102vs9x.html)
> [Lwip ARP分析(1)](https://blog.csdn.net/weijitao/article/details/53586167)
> [Lwip之如何动态更改IP地址](https://blog.csdn.net/zhaozhiyuan111/article/details/83026659)

# ping不通
项目里pc无法ping通单片机，抓包发现是ARP包没有返回，通过底层打印发现可以收包，
从`ethernetif_input`查看lwip对arp包处理的是否正确，
```c
//sys_arch.c
u32_t sys_now()
{
  return LocaTime;
}
```

# 动态修改IP

```cpp
tcp_close(u_sTcp_pcb[i]);
netif_set_down(&u_sNetif); //先禁用网卡
netif_set_gw(&u_sNetif, &GW_updata);        //重新设置网关地址
netif_set_netmask(&u_sNetif, &Mask_update); //重新设置子网掩码
netif_set_ipaddr(&u_sNetif, &ip_update);    //重新设置IP地址
//  netif_set_addr(&u_sNetif, &ip_update, &Mask_update, &GW_updata);
netif_set_up(&u_sNetif);  //启用网卡
```

