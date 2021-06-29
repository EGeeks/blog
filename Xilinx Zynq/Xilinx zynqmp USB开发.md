# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Linux USB DWC3 Host/Peripheral Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842069/USB)
> [Zynq Ultrascale MPSOC Linux USB device driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841729/Zynq+Ultrascale+MPSOC+Linux+USB+device+driver)
> [U-Boot USB Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841701/U-Boot+USB+Driver)
> [Zynq UltraScale＋ MPSoC USB 3.0 Mass Storage Device Class Design](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842410/Zynq+UltraScale+MPSoC+USB+3.0+Mass+Storage+Device+Class+Design)
> [Zynq UltraScale＋ MPSoC USB 3.0 CDC Device Class Design](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842045/Zynq+UltraScale+MPSoC+USB+3.0+CDC+Device+Class+Design)
> [Linux USB Gadget Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841925/Linux+USB+Gadget+Driver)
> [USB Host System Setup](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842562/USB+Host+System+Setup)
> [USB Host Controller Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841980/USB+Host+Controller+Driver)
> [AXI USB Device Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842386/AXI+USB+Device+Driver)
> [AXI USB gadget driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841620/AXI+USB+gadget+driver)
> [USB boot with Linux 2015.2 build](https://forums.xilinx.com/t5/Embedded-Linux/USB-boot-with-Linux-2015-2-build/td-p/643868)
> [[U-Boot,3/4] ARM64: zynqmp: Add support for USB ulpi phy reset via mode pins](http://patchwork.ozlabs.org/patch/664935/)
> [USB Phy/ULPI (2-读写USB Phy寄存器)](https://blog.csdn.net/chdyuxin/article/details/18180059)
> [USB2 UAS NULL pointer dereference](https://forums.xilinx.com/t5/Embedded-Linux/USB2-UAS-NULL-pointer-dereference/m-p/921559)
> [USB Mass Storage大容量存储的基本知识](https://blog.csdn.net/yu704645129/article/details/43229237)
> [怎么通过 /proc/scsi/usb-storage来确定u盘是/dev/sdb还是sdc](https://bbs.csdn.net/topics/390494997?list=17182426)
> [如何实现Linux下的U盘（USB Mass Storage）驱动](https://blog.csdn.net/mao0514/article/details/24996553)
> [USB3.0 HDMI输入方案之FT601Q](https://my.oschina.net/msxbo/blog/3106017)
> [米联客(MSXBO)USB3.0 FT600/FT601/FT602测试图集](https://my.oschina.net/msxbo/blog/3108619)
> [米联客(MSXBO)USB3.0 UVC摄像头实现基于FT602Q芯片方案](https://my.oschina.net/msxbo/blog/3106273)
> [【工程师分享】MPSoC设计中USB Phy的复位信号](https://mp.weixin.qq.com/s/0tmqhz0CvclyFltH1s2_lg)

# 介绍
zynqmp上用的是synopsis的ip，dwc3。

# u-boot
基于zcu102开发板的基本配置，配错了编译都无法通过，
```bash
CONFIG_USB=y
CONFIG_USB_ULPI_VIEWPORT=y
CONFIG_USB_ULPI=y
CONFIG_USB_STORAGE=y
CONFIG_USB_GADGET=y
CONFIG_USB_GADGET_MANUFACTURER="Xilinx"
CONFIG_USB_GADGET_VENDOR_NUM=0x03fd
CONFIG_USB_GADGET_PRODUCT_NUM=0x0300
CONFIG_CMD_USB=y
CONFIG_USB_DWC3=y
CONFIG_USB_XHCI_DWC3=y
CONFIG_USB_XHCI_ZYNQMP=y
CONFIG_USB_DWC3_GADGET=y
CONFIG_USB_HOST=y
CONFIG_USB_DWC3_GENERIC=y
CONFIG_ZYNQMP_USB=y
CONFIG_USB_STORAGE=y
CONFIG_USB_XHCI_HCD=y
CONFIG_USB_GADGET_DOWNLOAD=y
```
代码，dwc3 usb驱动入口在`drivers\usb\dwc3\dwc3-generic.c`，`dwc3_generic_bind`函数遍历设备树子节点，绑定host，peripheral或otg的驱动，子节点驱动不使用设备树来获取，对于host类型，绑定的子节点驱动为`drivers\usb\host\xhci-zynqmp.c`，
```c
//drivers\usb\dwc3\dwc3-generic.c
dwc3_generic_bind (trigger)->>
//drivers\usb\host\xhci-zynqmp.c
xhci_usb_probe
  zynqmp_xhci_core_init
```
u-boot下操作usb,
```shell
ZynqMP> usb
usb - USB sub-system

Usage:
usb start - start (scan) USB controller
usb reset - reset (rescan) USB controller
usb stop [f] - stop USB [f]=force stop
usb tree - show USB device tree
usb info [dev] - show available USB devices
usb test [dev] [port] [mode] - set USB 2.0 test mode
    (specify port 0 to indicate the device's upstream port)
    Available modes: J, K, S[E0_NAK], P[acket], F[orce_Enable]
usb storage - show details of USB storage devices
usb dev [dev] - show or set current USB storage device
usb part [dev] - print partition table of one or all USB storage    devices
usb read addr blk# cnt - read `cnt' blocks starting at block `blk#'
    to memory address `addr'
usb write addr blk# cnt - write `cnt' blocks starting at block `blk#'
    from memory address `addr'
    
ZynqMP> usb start
starting USB...
USB0:   Register 2000440 NbrPorts 2
Starting the controller
USB XHCI 1.00
scanning bus 0 for devices... 1 USB Device(s) found
       scanning usb for storage devices... 0 Storage Device(s) found
       
ZynqMP> usb tree
USB device tree:
  1  Hub (5 Gb/s, 0mA)
     U-Boot XHCI Host Controller 
   
ZynqMP> usb reset
resetting USB...
USB0:   Register 2000440 NbrPorts 2
Starting the controller
USB XHCI 1.00
scanning bus 0 for devices... 2 USB Device(s) found
       scanning usb for storage devices... 1 Storage Device(s) found
ZynqMP> usb tree
USB device tree:
  1  Hub (5 Gb/s, 0mA)
  |  U-Boot XHCI Host Controller 
  |
  +-2  Mass Storage (480 Mb/s, 300mA)
       Kingston DataTraveler 2.0 001A4D5F1A5CB0419945C2A1
     
ZynqMP> usb info
1: Hub,  USB Revision 3.0
 - U-Boot XHCI Host Controller 
 - Class: Hub
 - PacketSize: 512  Configurations: 1
 - Vendor: 0x0000  Product 0x0000 Version 1.0
   Configuration: 1
   - Interfaces: 1 Self Powered 0mA
     Interface: 0
     - Alternate Setting 0, Endpoints: 1
     - Class Hub
     - Endpoint 1 In Interrupt MaxPacket 8 Interval 255ms

2: Mass Storage,  USB Revision 2.0
 - Kingston DataTraveler 2.0 001A4D5F1A5CB0419945C2A1
 - Class: (from Interface) Mass Storage
 - PacketSize: 64  Configurations: 1
 - Vendor: 0x0930  Product 0x6545 Version 1.16
   Configuration: 1
   - Interfaces: 1 Bus Powered 300mA
     Interface: 0
     - Alternate Setting 0, Endpoints: 2
     - Class Mass Storage, Transp. SCSI, Bulk only
     - Endpoint 1 In Bulk MaxPacket 512
     - Endpoint 2 Out Bulk MaxPacket 512

ZynqMP> usb storage
  Device 0: Vendor: Kingston Rev: PMAP Prod: DataTraveler 2.0
            Type: Removable Hard Disk
            Capacity: 14766.0 MB = 14.4 GB (30240768 x 512)

ZynqMP> usb dev

IDE device 0: Vendor: Kingston Rev: PMAP Prod: DataTraveler 2.0
            Type: Removable Hard Disk
            Capacity: 14766.0 MB = 14.4 GB (30240768 x 512)
            
ZynqMP> usb part

Partition Map for USB device 0  --   Partition Type: DOS

Part    Start Sector    Num Sectors     UUID            Type
  1     16128           30224640        8d9140c4-01     0c Boot

```

# linux
## 驱动分析
xilinx驱动入口代码`drivers\usb\dwc3\dwc3-of-simple.c`，dwc3驱动入口代码`drivers\usb\dwc3\core.c`，
```c
dwc3_probe
  dwc3_get_properties //获取设备树属性
    usb_get_maximum_speed //USB_SPEED_SUPER "super-speed"
    usb_get_dr_mode
    of_usb_get_phy_mode dwc->hsphy_mode //nothing to be done when hsphy_mode == ulpi
    snps,hsphy_interface //used when DWC3_GHWPARAMS3 == DWC3_GHWPARAMS3_HSPHY_IFC_UTMI_ULPI
    dwc3_simple_check_quirks //针对Xilinx的芯片修正，drivers\usb\dwc3\dwc3-of-simple.c
  dwc3_cache_hwparams
    DWC3_GHWPARAMS3 //DWC_USB3_HSPHY_INTERFACE = 0x2
  dma_set_coherent_mask
  dwc3_get_dr_mode //根据上面设备树和内核驱动配置设置芯片是host还是peripheral
  dwc3_core_init
    dwc3_core_get_phy
      dwc->usb2_phy/dwc->usb3_phy
      dwc->usb2_generic_phy/dwc->usb3_generic_phy
    dwc3_core_soft_reset
      usb_phy_init usb2_phy/usb3_phy/usb2_generic_phy/usb3_generic_phy
    dwc3_config_soc_bus
    dwc3_phy_setup
      dwc3_ulpi_init //DWC3_GHWPARAMS3 == DWC3_GHWPARAMS3_HSPHY_IFC_ULPI
        ulpi_register_interface
          ulpi_register
            
    ...
    usb_phy_set_suspend dwc->usb2_phy
	usb_phy_set_suspend dwc->usb3_phy
	phy_power_on dwc->usb2_generic_phy
    phy_power_on dwc->usb3_generic_phy
    ...
  dwc3_check_params
  dwc3_core_init_mode
    otg_set_vbus(usb2_phy)/phy_set_mode(usb2_generic_phy)
  dwc3_debugfs_init
...
```

## 设备树
设备树节点含义，
```c
 - usb-phy : array of phandle for the PHY device.  The first element in the array is expected to be a handle to the USB2/HS PHY and the second element is expected to be a handle to the USB3/SS PHY
 - phys: from the *Generic PHY* bindings
 - phy-names: from the *Generic PHY* bindings; supported names are "usb2-phy" or "usb3-phy". 
```
例子，
```c
&pinctrl0 {
	pinctrl_usb0_default: usb0-default {
		mux {
			groups = "usb0_0_grp";
			function = "usb0";
		};

		conf {
			groups = "usb0_0_grp";
			slew-rate = <1>;/*SLEW_RATE_SLOW*/
			io-standard = <1>;/*IO_STANDARD_LVCMOS18*/
		};

		conf-rx {
			pins = "MIO52", "MIO53", "MIO55";
			bias-high-impedance;
		};

		conf-tx {
			pins = "MIO54", "MIO56", "MIO57", "MIO58", "MIO59",
			       "MIO60", "MIO61", "MIO62", "MIO63";
			bias-disable;
		};
	};
};

&usb0 {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_usb0_default>;
};

&dwc3_0 {
	dr_mode = "host";
	snps,usb3_lpm_capable;
	phy-names = "usb3-phy";
	phys = <&lane0 4 0 2 100000000>;/*2->PHY_TYPE_USB3=4*/
	maximum-speed = "super-speed";
};
```
开机打印，
```shell
[    7.690943] xilinx-psgtr fd400000.zynqmp_phy: Lane:2 type:0 protocol:3 pll_locked:yes
[    7.698965] xhci-hcd xhci-hcd.0.auto: xHCI Host Controller
[    7.704383] xhci-hcd xhci-hcd.0.auto: new USB bus registered, assigned bus number 1
[    7.712216] xhci-hcd xhci-hcd.0.auto: hcc params 0x0238f625 hci version 0x100 quirks 0x22010010
[    7.720864] xhci-hcd xhci-hcd.0.auto: irq 48, io mem 0xfe200000
[    7.726853] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002
[    7.733562] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    7.740765] usb usb1: Product: xHCI Host Controller
[    7.745626] usb usb1: Manufacturer: Linux 4.14.0-xilinx-v2018.2 xhci-hcd
[    7.752308] usb usb1: SerialNumber: xhci-hcd.0.auto
[    7.757438] hub 1-0:1.0: USB hub found
[    7.761132] hub 1-0:1.0: 1 port detected
[    7.765189] xhci-hcd xhci-hcd.0.auto: xHCI Host Controller
[    7.770607] xhci-hcd xhci-hcd.0.auto: new USB bus registered, assigned bus number 2
[    7.778286] usb usb2: We don't know the algorithms for LPM for this host, disabling LPM.
[    7.786385] usb usb2: New USB device found, idVendor=1d6b, idProduct=0003
[    7.793098] usb usb2: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    7.800301] usb usb2: Product: xHCI Host Controller
[    7.805159] usb usb2: Manufacturer: Linux 4.14.0-xilinx-v2018.2 xhci-hcd
[    7.811843] usb usb2: SerialNumber: xhci-hcd.0.auto
[    7.816926] hub 2-0:1.0: USB hub found
[    7.820617] hub 2-0:1.0: 1 port detected
```
大概测了一个SSD的速度，读写都可以达到292.57MB/s
```shell
root@xilinx-zcu104-2018_2:~# cat test.sh 
#!/bin/sh
date
dd if=/dev/zero of=/dev/sda1 bs=4M count=512
date

date
dd if=/dev/sda1 of=/dev/null bs=4M count=512
date
root@xilinx-zcu104-2018_2:~# ./test.sh 
Fri Jul  6 04:20:40 UTC 2018
512+0 records in
512+0 records out
Fri Jul  6 04:20:46 UTC 2018
Fri Jul  6 04:20:46 UTC 2018
512+0 records in
512+0 records out
Fri Jul  6 04:20:52 UTC 2018
```

# 读写ULPI PHY寄存器
zynqmp通过GUSB2PHYACC_ULPI寄存器来访问ULPI PHY，地址，
- 0xFE20C280 (USB3_0_XHCI)
- 0xFE30C280 (USB3_1_XHCI)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513105744447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
读PHY芯片 USB3315 ID，
```shell
ZynqMP> mw 0xFE20C280 0x2000000
ZynqMP> md 0xFE20C280 1
fe20c280: 01000024                               $...
```

#  SMSC 3320
在 USB 控制器针对 USB 2.0 配置以及启用待机／重启状态时，该控制器会通过发送 ULPI PHY 单命令来选择终止和收发器。在 PHY 要求这些通过两个单独的命令发送时，这会导致系统挂起。GUCTL1（0x0000C11C）寄存器的位 10 需要设置为 1，以便实现与 USB 2.0 PHY 设备（SMSC 3320）的更佳互操作性，以及预防高速设备的挂起/恢复有任何问题（`06/13/2017 AR# 67667`）。 

# UAS
UAS在2018.2还不稳定，等待改进，建议使用usb-storage (BOT) driver。

