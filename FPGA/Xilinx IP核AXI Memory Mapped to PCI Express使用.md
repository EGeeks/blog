# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 性能
测试平台：长城的FT1500A/4台式机，DDR3双通道内存，FPGA直接挂载CPU直出的PCIe上。（附：长城的FT1500A/4台式机必须把显卡放在x16卡槽，必须是x16，我们用一个x1转接卡不行，bios锁死了，开不了机，有点恶心）
```bash
[  170.257983] pcie_chipset 0000:02:00.0: vendor[0x10ee], device[0x7038], subvendor[0x10ee], subdevice[0x10ee], class[0x58000]
[  170.257992] PCI: enabling device 0000:02:00.0 (0000 -> 0002)
[  170.258987] pcie_chipset 0000:02:00.0: BAR0, start=0x64000000[0xffffff8019400000], end=0x67ffffff, len=0x4000000, flags=0x140204
[  170.259210] pcie_chipset 0000:02:00.0: BAR2, start=0x68000000[0xffffff801d480000], end=0x68ffffff, len=0x1000000, flags=0x140204
[  170.259214] pcie_chipset 0000:02:00.0: find ucode device chipset_intc
[  170.259218] pcie_chipset 0000:02:00.0: find ucode device pmem
[  170.259222] pcie_chipset 0000:02:00.0: find ucode device xaxicdma
[  170.259225] pcie_chipset 0000:02:00.0: probe ucode device chipset_intc
[  170.259280] pcie_chipset 0000:02:00.0: chipset intc support 1 irqs, irq number:
[  170.259284] pcie_chipset 0000:02:00.0:   38
[  170.259289] pcie_chipset 0000:02:00.0: probe ucode device pmem
[  170.259357] pcie_chipset 0000:02:00.0: probe ucode device xaxicdma
kylin@kylin-os:~/kylin_os_repo/software/pcie_chipset$ sudo ./dma_benchmark -i xaxicdma -c c2h -s cmem -d 0x100000000 -m cdev:xaxicdma2@68a00000 -l 0x400000 -n 128
global params: 
  dma type: xaxicdma read 0x400000 bytes from 0xfdc00000 to 0x100000000
  cmd type: read
  poll time: 0
  test cycle: 128
  xaxicdma type: cdev name: /dev/xaxicdma2@68a00000
speed: 6023.53MB/s, cost times: 85ms
kylin@kylin-os:~/kylin_os_repo/software/pcie_chipset$ sudo ./dma_benchmark -i xaxicdma -c h2c -s cmem -d 0x100000000 -m cdev:xaxicdma2@68a00000 -l 0x400000 -n 128
global params: 
  dma type: xaxicdma write 0x400000 bytes from 0xfdc00000 to 0x100000000
  cmd type: write
  poll time: 0
  test cycle: 128
  xaxicdma type: cdev name: /dev/xaxicdma2@68a00000
speed: 5019.61MB/s, cost times: 102ms
kylin@kylin-os:~/kylin_os_repo/software/pcie_chipset$ 
kylin@kylin-os:~/kylin_os_repo/software/pcie_chipset$ 
kylin@kylin-os:~/kylin_os_repo/software/pcie_chipset$ sudo ./dma_benchmark -i xaxicdma -c h2c -s cmem -d 0x100000000 -m cdev:xaxicdma2@68a00000 -l 0x400000 -n 512
global params: 
  dma type: xaxicdma write 0x400000 bytes from 0xfdc00000 to 0x100000000
  cmd type: write
  poll time: 0
  test cycle: 512
  xaxicdma type: cdev name: /dev/xaxicdma2@68a00000
speed: 5007.33MB/s, cost times: 409ms
kylin@kylin-os:~/kylin_os_repo/software/pcie_chipset$ sudo ./dma_benchmark -i xaxicdma -c c2h -s cmem -d 0x100000000 -m cdev:xaxicdma2@68a00000 -l 0x400000 -n 512
global params: 
  dma type: xaxicdma read 0x400000 bytes from 0xfdc00000 to 0x100000000
  cmd type: read
  poll time: 0
  test cycle: 512
  xaxicdma type: cdev name: /dev/xaxicdma2@68a00000
speed: 5988.30MB/s, cost times: 342ms
kylin@kylin-os:~/kylin_os_repo/software/pcie_chipset$ 
kylin@kylin-os:~/kylin_os_repo/software/pcie_chipset$ sudo lspci -tv
-[0000:00]-+-00.0-[01]--+-00.0  Advanced Micro Devices, Inc. [AMD/ATI] Caicos XT [Radeon HD 7470/8470 / R5 235/310 OEM]
           |            \-00.1  Advanced Micro Devices, Inc. [AMD/ATI] Caicos HDMI Audio [Radeon HD 6400 Series]
           +-01.0-[02]----00.0  Xilinx Corporation Device 7038
           \-02.0-[03-0a]--+-00.0-[04-0a]--+-01.0-[05]--
                           |               +-02.0-[06]--
                           |               +-03.0-[07]----00.0  Intel Corporation 82574L Gigabit Network Connection
                           |               +-0b.0-[08]----00.0  Renesas Technology Corp. uPD720201 USB 3.0 Host Controller
                           |               +-0d.0-[09]----00.0  Renesas Technology Corp. uPD720201 USB 3.0 Host Controller
                           |               \-0f.0-[0a]----00.0  Marvell Technology Group Ltd. Device 9215
                           \-00.1  PLX Technology, Inc. PEX 8619 16-lane, 16-Port PCI Express Gen 2 (5.0 GT/s) Switch with DMA
```

# 时钟

# 复位
axi_areset可以用Xilinx IP核Processing System Reset来实现，将pcie slot的perstn管脚连入Aux_Reset_In，mmcm_lock连接到dcm_locked

# 中断
MSI Interrupt
When the msi_enable output pin indicates that the bridge has Endpoint MSI functionality
enabled (msi_enable = 1), the intx_msi_request input pin is defined as MSI_Request
and can be used to trigger a Message Signaled Interrupt through a special MemWr TLP to
an external Root Port for PCIe on the PCIe side of the Bridge. The intx_msi_request
input pin is positive-edge detected and synchronous to axi_aclk_out.

# 地址映射
```c
root：

PS -> GP -> S_AXI[AXI-PCIe Bridge] -> PCIe -> EP BAR
                                             |-> MIG
EP BMDMA -> PCIe -> M_AXI[AXI-PCIe Bridge] ->|
                                             |-> HP -> PS Mem

ep:

PS DMA -> GP ->| 
               |-> S_AXI[AXI-PCIe Bridge] -> PCIe -> Root Mem
      PL DMA ->|
Root -> M_AXI[AXI-PCIe Bridge] -> BAR
```
其中S_AXI上的地址转换可以通过寄存器配置，FPGA做EP时，这个值是由PCIe驱动写入的，低地址为高32字节，高地址为低32字节，在32位地址模式下，只使用低32字节。
![204](https://img-blog.csdnimg.cn/20190903101812693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 眼图
> [利用 IP 中的集成调试功能来调试 PCI Express 链接训练](https://forums.xilinx.com/t5/Xilinx-%E4%BA%A7%E5%93%81%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%8A%9F%E8%83%BD%E8%B0%83%E8%AF%95%E6%8A%80%E5%B7%A7/%E5%88%A9%E7%94%A8-IP-%E4%B8%AD%E7%9A%84%E9%9B%86%E6%88%90%E8%B0%83%E8%AF%95%E5%8A%9F%E8%83%BD%E6%9D%A5%E8%B0%83%E8%AF%95-PCI-Express-%E9%93%BE%E6%8E%A5%E8%AE%AD%E7%BB%83/ba-p/1123297)
