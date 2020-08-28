# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

Finace-FPGA方案设计文档
# 硬件设计
## 板卡原理框图
![251](https://img-blog.csdnimg.cn/20191214151122261.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
## 技术指标
- 总线类型：PCI Express 2.0 规范，兼容 PCI Express 1.1；
- 接口规格：PCI Express x8（可选 x1、x4 模式），全高半长；
- 板载 1 个 4 位拨码开关；
- 板载 1 片 16 位 128MByte 容量的 BPI Flash；
- 光纤传输性能：4 路 SFP+，支持单模与多模光纤，最大 10Gbps/lane；
- 光纤传输协议：10G 以太网、Serial RapidIO（最高 6.25Gbps/lane）、Aurora64b/66b；
- 动态缓存：1 组独立的 64 位 DDR3-1600 SDRAM，最大支持 4GByte 容量；
- PCIe DMA 性能：独立 DMA 通道，上行与下行传输带宽可以达到 3.2GByte/s；
- 数字离散 IO 性能：16 路 GPIO，18 对 LVDS IO；
- 支持 4 路 SATA3.0 接口；

## 物理与电气特征
- 板卡尺寸：106.65 x 167.55 mm；
- 工作温度：-20°C~+70°C；
- 存储温度：-40°C~+85°C；
- 工作湿度：5%~60%，非凝结；
- 散热方式：风冷散热；

## 电源管理
板卡为标准 PCIe 卡，正常工作时从 PCIe 金手指取电，+12V 为主电源轨，最大功耗 24W（静态功耗约 4W）。

## 时钟管理
1. 板卡提供 1 个 156.25MHz 的差分晶振，通过时钟 Buffer，分成 4 路：
- 2 路供给 4 路光纤参考时钟，作为 FPGA 光纤接口 GTX 的参考时钟；
- 1 路供给 FPGA 的全局时钟网络；
- 1 路作为 DDR3 接口的 System Clcok；
2. 板卡提供 1 个 100MHz 的 LVDS 差分时钟，作为 PCIe 接口备用参考时钟；
3. 板卡提供 1 个 200MHz 的 LVDS 差分时钟，作为 DDR3 接口参考时钟；
4. 板卡提供 1 个 150MHz 的 LVDS 差分时钟，作为 SATA 接口的参考时钟；
5. 板卡提供 1 个 50MHz 的 LVCOMS 单端时钟，作为 FPGA 同步逻辑时钟；
6. 板卡提供 1 个 66MHz 的 LVCOMS 单端时钟，作为 FPGA 的 EMCCLK；

# FPGA设计
## 10G以太网设计
### 10G以太网结构
10G以太网接口分为10G PHY和10G MAC两部分。如下图所示。
![253](https://img-blog.csdnimg.cn/20191214155450391.png)
本设计中使用了Xilinx公司提供的10GEthernet PCS/PMA IP核充当连接10GMAC的PHY芯片，然后将该IP核约束到光模块上构建完整的物理层。需要说明的是本设计主要是完成以太网二层逻辑设计，不涉及PHY层的逻辑设计，如：bit同步、字节同步、字同步、64b/66b编解码等。
### 10G以太网接口PHY
10G EthernetPCS/PMA的整体结构如下图所示，其核心是基于RocketIO GTH/GTX来实现的。从图中可知，该模块分为PCS层和PMA层，对于发送数据，PCS层主要功能是对数据进行64B/66B编码、扰码、发送变速等功能。同时在测试模式下还提供了一个测试激励源，用于对链路进行检测。PMA层的主要功能是提供并串转换、对串行信号进行驱动并发送等功能。对于接收数据，PMA层的主要功能是将接收到的高速差分信号进行串并转换、bit同步、时钟恢复等功能，PCS层对于从PMA层接收到的数据进行块同步、解扰码、64B/66B解码、弹性缓存等。同时在测试模式下还提供测试激励检测功能，用于检测链路工作状态。
![254](https://img-blog.csdnimg.cn/20191214155516594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
在接口调试过程中，可能用到PMA层的近端环回和远端环回功能。PMA近端回环，用于测试IP核内部自回环；PMA远端回环，用于将接收到的远端10G PHY发送的的数据在PMA层直接回环发送给远端10G PHY，而不经过本地的PCS层。
### 10G以太网接口时钟布局设计
由于10G Ethernet PCS/PMA是Xilinx官方提供的一款IP核，所以我们需要做的工作是结合开发板的实际情况，为该IP核以及其他模块设计合理的时钟电路，使其能够正常工作。本文选用Xilinx VC709开发板作为上板调试的硬件平台，因此我们的时钟布局需要充分考虑此开发板的结构来设计，具体的时钟布局如下图所示。
![255](https://img-blog.csdnimg.cn/20191214155532132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
对于FPGA内部的时钟布局主要分为以下4部分：
（a）输入的差分参考时钟经过一个参考钟专用缓存（IBUFDS_GTE2）变为单端时钟refclk，然后将refclk分为两路，一路接到QPLL（QuadraturephasePhase Locking Loop），另一路时钟经过一个BUFG后转变为全局时钟coreclk，继续将coreclk分为两路，一路作为10G MAC核XGMII接口的收发时钟（xgmii_rx_clk和xgmii_tx_clk），另一路用于驱动10G Ethernet PCS/PMA IP核内部用户侧的逻辑。
（b） 对于QPLL输出的两路时钟qplloutclk和qplloutrefclk，主要是用于IP核内GTH收发器使用的高性能时钟，其中qplloutclk直接用于驱动GTH内发送端的串行信号，其频率为5.15625GHz。qplloutrefclk用于驱动GTH内部部分逻辑模块，频率为156.25MHz。
（c） txoutclk是由10G Ethernet PCS/PMA IP产生的一个322.26MHz的时钟，该时钟经过BUFG后分为两路，其中txusrclk用于驱动IP核内GTH的32bits总线数据，txusrclk2用于驱动IP核内PCS层部分模块。
（d）200MHz的晶振产生差分时钟输入到FPGA内的PLL（Phase LockingLoop）模块，PLL模块以200MHz差分钟为驱动时钟生成192MHz用户钟（sys_clk）发送给10G MAC核用户侧。
## PCIe DMA设计
DMA功能是PCIe总线传输数据的一种重要方式，FPGA完成DMA控制器的功能，能够将数据通过PCIe通道直接搬运到PC机的内存中，提供了极大的数据传输带宽。基于Xilinx Gen2 Integrated Block for PCIe IP核的PCIe DMA控制器满足 PCI Express Gen2 标准，采用Bus Master DMA的数据交互方式，在整个数据传输的过程中，完全不需要主机CPU的干预，适合于海量数据传输、高带宽数据交互的应用场景。PCIe DMA 控制器示意图如下所示，
![252](https://img-blog.csdnimg.cn/20191214154431773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
## 代码介绍
FPGA工程开发采用Xilinx的Vivado来实现，设计中Xilinx官方用到的IP核均放在Block Design中实现，通过原理图完成设计，包括：

- `10G Ethernet Subsystem` IP为10G以太网MAC，完成物理层、链路层
- `AXI Memory Mapped To PCI Express` IP完成板卡与PC直接的PCIe通信
- `AXI Direct Memory Access` IP完成网络数据包发送到`AXI Memory Mapped To PCI Express` IP
- `AXI4-Stream Data FIFO` IP完成网络数据包的缓存，考虑以太网`jumbo frame` FIFO深度大于9KB
- `AXI Interconnect` IP完成多路以太网到PCIe的数据仲裁，分时复用PCIe总线
- `AXI Interrupt Controller` IP完成中断功能，数据包成功发送到PC后触发PCIe中断

用户功能以RTL源代码形式实现，对接收到的以太网包，加上秒和纳秒的时间戳之后在发送到PC，数据流动方向如下，
```bash
SFP网口 > MAC > 缓存   > 带硬件时间戳的数据包 > 缓存 > DMA > PCIe > PC
              > 时间戳 >
```
源码中的警告来自Xilinx官方IP内部，是正常的，不影响功能的正确性，
![262](https://img-blog.csdnimg.cn/20191227192021269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 软件设计
网卡驱动符合linux内核网卡驱动框架，支持libpcap，下图为linux网卡驱动框架，
![256](https://img-blog.csdnimg.cn/20191214162815455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

