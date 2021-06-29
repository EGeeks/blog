# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [U-Boot USB Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841701/U-Boot+USB+Driver)
> [Zynq Linux USB Device Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842272/Zynq+Linux+USB+Device+Driver)
> [U-Boot USB Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841701/U-Boot+USB+Driver)
> [Zynq-7000 AP SoC USB Mass Storage Device Class Design Example Techtip](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842264/Zynq-7000+AP+SoC+USB+Mass+Storage+Device+Class+Design+Example+Techtip)
> [Zynq-7000 AP SoC USB CDC Device Class Design Example Techtip](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841624/Zynq-7000+AP+SoC+USB+CDC+Device+Class+Design+Example+Techtip)
> [Zynq Linux USB Device Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842272/Zynq+Linux+USB+Device+Driver)
> [Linux USB Gadget Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841925/Linux+USB+Gadget+Driver)
> [USB Host System Setup](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842562/USB+Host+System+Setup)
> [USB Host Controller Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841980/USB+Host+Controller+Driver)
> [AXI USB Device Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842386/AXI+USB+Device+Driver)
> [AXI USB gadget driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841620/AXI+USB+gadget+driver)
> [USB boot with Linux 2015.2 build](https://forums.xilinx.com/t5/Embedded-Linux/USB-boot-with-Linux-2015-2-build/td-p/643868)
> [zynq-7000学习笔记(八)——USB摄像头图像采集](https://blog.csdn.net/luotong86/article/details/52473867)
> [ZYNQ-ZedBoard USB HOST问题初探](https://www.cnblogs.com/qiantuo1234/p/6665361.html)
> [ZYNQ-ZedBoard USB HOST问题二探](http://www.cnblogs.com/qiantuo1234/p/6665638.html)
> [Zynq usb无法识别](https://blog.csdn.net/xfxiaofan123456/article/details/77102672)
> [USB Phy/ULPI (2-读写USB Phy寄存器)](https://blog.csdn.net/chdyuxin/article/details/18180059)
> [USB Mass Storage大容量存储的基本知识](https://blog.csdn.net/yu704645129/article/details/43229237)
> [怎么通过 /proc/scsi/usb-storage来确定u盘是/dev/sdb还是sdc](https://bbs.csdn.net/topics/390494997?list=17182426)
> [如何实现Linux下的U盘（USB Mass Storage）驱动](https://blog.csdn.net/mao0514/article/details/24996553)
> [Zynq Linux USB Driver Customization](https://forums.xilinx.com/t5/Embedded-Linux/Zynq-Linux-USB-Driver-Customization/td-p/239478)
> [Linux/DRA77P: ULPI USB interface](http://e2e.ti.com/support/processors/f/791/p/788993/2920361)

# 介绍
zynq上用的是chipidea的ip，ULPI 接口要求输入保持时间为1ns，TI TUSB1210 PHY 的最低 CTO 并未规定，但报告的最低 CTO 为 100 ps，无法满足 Zynq-7000 的要求（`AR# 53450`）。
`When the USB controller is configured in device mode, and Vbus voltage is varied such that it crosses the Vbus valid threshold multiple times, the ULPI interface becomes unresponsive.（AR# 61313）`。
# u-boot
u-boot下操作usb,
```shell
zynq-uboot> usb
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
zynq-uboot> usb start
starting USB...
USB0:   USB EHCI 1.00
scanning bus 0 for devices... 3 USB Device(s) found
USB1:   usb1 wrong num MIO: 0, Index 1
lowlevel init failed
       scanning usb for storage devices... 1 Storage Device(s) found
zynq-uboot> usb tree
USB device tree:
  1  Hub (480 Mb/s, 0mA)
  |  u-boot EHCI Host Controller 
  |
  +-2  Hub (480 Mb/s, 2mA)
    |
    +-3  Mass Storage (480 Mb/s, 300mA)
         Kingston DataTraveler 2.0 001A4D5F1A5CB0419945C2A1
       
zynq-uboot> usb info
1: Hub,  USB Revision 2.0
 - u-boot EHCI Host Controller 
 - Class: Hub
 - PacketSize: 64  Configurations: 1
 - Vendor: 0x0000  Product 0x0000 Version 1.0
   Configuration: 1
   - Interfaces: 1 Self Powered 0mA
     Interface: 0
     - Alternate Setting 0, Endpoints: 1
     - Class Hub
     - Endpoint 1 In Interrupt MaxPacket 8 Interval 255ms

2: Hub,  USB Revision 2.0
 - Class: Hub
 - PacketSize: 64  Configurations: 1
 - Vendor: 0x0424  Product 0x2514 Version 11.179
   Configuration: 1
   - Interfaces: 1 Self Powered Remote Wakeup 2mA
     Interface: 0
     - Alternate Setting 0, Endpoints: 1
     - Class Hub
     - Endpoint 1 In Interrupt MaxPacket 1 Interval 12ms
     - Endpoint 1 In Interrupt MaxPacket 1 Interval 12ms

3: Mass Storage,  USB Revision 2.0
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
```
# linux
petalinux 2018.2下的内核与设备树，
```c
Device Drivers
USB support
    <*> Support for Host-side USB
    <*> EHCI HCD (USB 2.0) support
    <*> USB Mass Storage support
    <*> ChipIdea Highspeed Dual Role Controller
    <*> ChipIdea host controller
        USB Physical Layer drivers --->
        <*> Generic ULPI Transceiver Driver
        
usb_0: usb@e0002000 {
     compatible = "xlnx,zynq-usb-2.20.a", "chipidea,usb2";
     clocks = <&clkc 28>
     dr_mode = "host";
     interrupt-parent = <&intc>;
     interrupts = <0 21 4>;
     reg = <0xe0002000 0x1000>;
     usb-phy = <&usb_phy0>;
 };
 
 usb_phy0: phy0 { //Z:\program\fdk\bsp\zynqmp\linux-xlnx-v2018.2\drivers\usb\phy\phy-ulpi.c
    compatible = "ulpi-phy";
    #phy-cells = <0>;
    reg = <0xe0002000 0x1000>;
    view-port = <0x170>;
    drv-vbus;
}
```
petalinux 2015.2.1也是`"xlnx,zynq-usb-2.20.a"`，参考`drivers\usb\chipidea\ci_hdrc_usb2.c`，petalinux 2015.2.1下的设备树，
```c
&usb0 {
	compatible = "xlnx,zynq-usb-2.20.a";
	dr_mode = "host";
	phy_type = "ulpi";
	status = "okay";
};
```
对内核打补丁，
```c
static int ci_hdrc_create_ulpi_phy(struct device *dev, struct ci_hdrc *ci)
{
        struct usb_phy *ulpi;
        int reset_gpio;
        int ret;

        reset_gpio = of_get_named_gpio(dev->parent->of_node, "xlnx,phy-reset-gpio", 0);
        if (gpio_is_valid(reset_gpio)) {
                ret = devm_gpio_request_one(dev, reset_gpio, GPIOF_INIT_LOW, "ulpi resetb");
                if (ret) {
                        dev_err(dev, "Failed to request ULPI reset gpio: %d\n", ret);
                        return ret;
                }
                msleep(5);
                gpio_set_value_cansleep(reset_gpio, 1);
                msleep(1);
        }

        ulpi = otg_ulpi_create(&ulpi_viewport_access_ops,
                ULPI_OTG_DRVVBUS | ULPI_OTG_DRVVBUS_EXT);
        if (ulpi) {
                ulpi->io_priv = ci->hw_bank.abs + 0x170;
                ci->usb_phy = ulpi;
        }

        return 0;
}
```
开机打印，
```shell
ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
ehci-pci: EHCI PCI platform driver
usbcore: registered new interface driver usb-storage
e0002000.usb supply vbus not found, using dummy regulator
ULPI transceiver vendor/product ID 0x0451/0x1507
Found TI TUSB1210 ULPI transceiver.
ULPI integrity check: passed.
ci_hdrc ci_hdrc.0: EHCI Host Controller
ci_hdrc ci_hdrc.0: new USB bus registered, assigned bus number 1
ci_hdrc ci_hdrc.0: USB 2.0 started, EHCI 1.00
hub 1-0:1.0: USB hub found
hub 1-0:1.0: 1 port detected
...
hub 1-1:1.0: USB hub found
hub 1-1:1.0: 4 ports detected
INIT: version 2.88 booting
usb 1-1.3: new high-speed USB device number 3 using ci_hdrc
mount: mounting devtmpfs on /dev failed: Device or resource busy
usb-storage 1-1.3:1.0: USB Mass Storage device detected
scsi host0: usb-storage 1-1.3:1.0
...
login[887]: root login on 'ttyPS0'
scsi 0:0:0:0: Direct-Access     Kingston DataTraveler 2.0 PMAP PQ: 0 ANSI: 6
sd 0:0:0:0: Attached scsi generic sg0 type 0
sd 0:0:0:0: [sda] 30240768 512-byte logical blocks: (15.4 GB/14.4 GiB)
sd 0:0:0:0: [sda] Write Protect is off
sd 0:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
 sda: sda1
sd 0:0:0:0: [sda] Attached SCSI removable disk
```
TUSB1210 OTG_CTRL寄存器，可控制CPEN管脚，配合硬件完成，OTG电源切换，USB3320也是一样，
![196](https://img-blog.csdnimg.cn/20190826151014922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
zc706 usb原理图，
![197](https://img-blog.csdnimg.cn/20190826151737960.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 读写ULPI PHY寄存器
zynq的usb0基地址在0xE0002000，usb1在0xE0003000，通过ULPI Viewport来访问寄存器，zynq芯片不能使用ULPI Viewport读写ULPI PHY的扩展寄存器集（地址 0x40 及更高）。写入操作将地址本身写为数据，读取操作返回不正确的数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513100644228.png)
Viewport寄存器，低8位是写入的寄存器地址，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513101642544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
唤醒，
```shell
memtool mw 0xE0002170 0xa0000000
```
读TUSB1210 ID，
```shell
root@zynq:~# memtool mw 0xE0002170 0x40000000
root@zynq:~# memtool md 0xE0002170+4
e0002170: 08005100                                           .Q..
root@zynq:~# memtool mw 0xE0002170 0x40010000
root@zynq:~# memtool md 0xE0002170+4
e0002170: 08010400                                           ....
root@zynq:~# memtool mw 0xE0002170 0x40020000
root@zynq:~# memtool md 0xE0002170+4
e0002170: 08020700                                           ....
root@zynq:~# memtool mw 0xE0002170 0x40030000
root@zynq:~# memtool md 0xE0002170+4
e0002170: 08031500 
```
写OTG_CTRL寄存器，
```shell
memtool mw 0xE0002170 0x400a0000
memtool md 0xE0002170+4
```

