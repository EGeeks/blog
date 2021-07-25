# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [PCIE 3.0中使用的动态均衡概念](https://www.cnblogs.com/hammerqiu/p/10653681.html)
> [PCIE training](https://www.cnblogs.com/hammerqiu/p/10643692.html)
> [芯片中的数学——均衡器EQ和它在高速外部总线中的应用](https://zhuanlan.zhihu.com/p/48343011)
> [PCI/PCIe基础——配置空间](https://blog.csdn.net/jiangwei0512/article/details/51603525)
> [PCIE各种包结构及常用资料汇总](https://blog.csdn.net/li_hu/article/details/10330825)
> [开发者分享 | 使用 lspci 和 setpci 调试 PCIe 问题](https://mp.weixin.qq.com/s/D938nUGiZU7WOxVXLXfyzQ)

# 物理层
# 传输层
## TLP
TLP报文，通常情况下Type[4:0]与Fmt[2:0]合起来定义具体的TLP事务类型，TC[2:0] Trafic Class 3bits, Byte1 Bit6:4, 总共定义了 8 个等级 Trafic Class，Traffic Class顾名思义可以理解为交通等级，控制报文传输优先级。
![317](https://img-blog.csdnimg.cn/20200623112518126.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
为了区分Requester与Completer之间的事务类型, PCIe Spec定义了一个事务描述块(Transaction Descriptor Fields)。下图红色区域合起来就称为一个事务描述块。Transaction ID包含了Request ID和Tag，内容有Request Device具体的位置(Bus/Device/Function)。主要是用于Requester发出事务的完成TLP回报。
![318](https://img-blog.csdnimg.cn/20200623153715522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

## Visual Channel
在接收端有一个缓存空间叫作 VC buffer， 其中 VC 代表的是 Virtual Channel，翻译过来就是虚拟通道。VC buffer 可以存放从发送端传过来的TLPs。接受端的可用 VC buffer 是以信用机制的方式告知发送端，在这里，信用积分代表接收端 VC buffer 的可用空间。接收端会通过发送 DLLPs(Data Link Layer Packets)来告知发送端 VC buffer的信用积分(也就是告知 VC buffer 可用空间)，当缓存空间快满的时候，发送端会停止发送 TLP，以防 VC buffer 容量不足而造成发送端传输的数据被丢弃，从而避免要求发送端重新发送刚才发送的数据，最终达到更加有效利用网络带宽。
![319](https://img-blog.csdnimg.cn/20200623154120679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 应用层
## 配置空间
PCIe能力寄存器，类型有下面几种，
```c
//include/uapi/linux/pci_regs.h
/* Capability lists */
#define PCI_CAP_LIST_ID   0 /* Capability ID */
#define  PCI_CAP_ID_PM    0x01 /* Power Management */
#define  PCI_CAP_ID_AGP   0x02 /* Accelerated Graphics Port */
#define  PCI_CAP_ID_VPD   0x03 /* Vital Product Data */
#define  PCI_CAP_ID_SLOTID  0x04 /* Slot Identification */
#define  PCI_CAP_ID_MSI   0x05 /* Message Signalled Interrupts */
#define  PCI_CAP_ID_CHSWP 0x06  /* CompactPCI HotSwap */
#define  PCI_CAP_ID_PCIX  0x07 /* PCI-X */
#define  PCI_CAP_ID_HT    0x08 /* HyperTransport */
#define  PCI_CAP_ID_VNDR  0x09 /* Vendor specific */
#define  PCI_CAP_ID_DBG   0x0A /* Debug port */
#define  PCI_CAP_ID_CCRC  0x0B /* CompactPCI Central Resource Control */
#define  PCI_CAP_ID_SHPC  0x0C /* PCI Standard Hot-Plug Controller */
#define  PCI_CAP_ID_SSVID 0x0D  /* Bridge subsystem vendor/device ID */
#define  PCI_CAP_ID_AGP3  0x0E /* AGP Target PCI-PCI bridge */
#define  PCI_CAP_ID_EXP   0x10 /* PCI Express */
#define  PCI_CAP_ID_MSIX  0x11 /* MSI-X */
#define  PCI_CAP_ID_AF    0x13 /* PCI Advanced Features */
#define PCI_CAP_LIST_NEXT 1 /*Next capability in the list */
#define PCI_CAP_FLAGS   2 /* Capability defined flags (16 bits) */
#define PCI_CAP_SIZEOF    4
```

# 热插拔
> [PCIE热插拔技术](https://blog.csdn.net/baidu_25816669/article/details/88999985)
> [NVMe盘暴力热插拔 学习记录](https://blog.csdn.net/qq_33632004/article/details/105972147)
> [认识和使用热插拔的正确姿势](https://blog.csdn.net/memblaze_2011/article/details/79203441)
> [Linux2.6.10内核下PCIExpressNative热插拔框架的实现机制](https://blog.csdn.net/fcryuuhou/article/details/8609531)
> [Linux内核笔记之PCIe hotplug介绍及代码分析](https://blog.csdn.net/yhb1047818384/article/details/99705972)
> [对NVMe SSD热插拔时，我需要注意什么？](https://blog.csdn.net/Memblaze_2011/article/details/52870727)
> [PCIe SSD之SFF-8639和备受关注的热插拔功能](https://blog.csdn.net/Memblaze_2011/article/details/47168755)
> [PCIe扫盲——热插拔简要介绍](https://blog.csdn.net/kunkliu/article/details/95042934)
> [PCIE Hot Plug 一般流程](https://blog.csdn.net/weixin_33843947/article/details/92768874)
> [[PCIe]NVMe 热插拔过程以及常见问题（一）](https://mp.weixin.qq.com/s/M_QEqyYrweOY15ru8LBrDw)
