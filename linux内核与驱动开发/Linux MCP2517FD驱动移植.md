# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [MCP2517FD应用总结](https://blog.csdn.net/fengweibo112/article/details/90695305)
> [mcp2518驱动调试](https://blog.csdn.net/weixin_37723417/article/details/105484451)

# 过程
参考[MCP2517 CANFD an CM3](https://www.raspberrypi.org/forums/viewtopic.php?t=229580)，

- 第一个方案，[代码下载](https://www.spinics.net/lists/devicetree/msg202889.html)，单个文件，设备树采用misc/RPi-MCP2517，位于[misc/RPi-MCP2517](https://github.com/GBert/misc/tree/master/RPi-MCP2517)。

- 第二个方案，代码[mcp25xxfd-V6.11 kernel+driver compiled for raspberry pi 3](https://github.com/msperl/linux-rpi/releases)，驱动为单个文件，基于内核4.19，移植到3.19有些困难，但难度不大，移植过程参考[[1/2] dt-binding: can: mcp2517fd: document device tree bindings](https://patchwork.ozlabs.org/project/devicetree-bindings/patch/20171124183509.12810-2-kernel@martin.sperl.org/)。

- 第三个方案，参考[2 Channel CAN BUS FD Shield for Raspberry Pi](https://wiki.seeedstudio.com/2-Channel-CAN-BUS-FD-Shield-for-Raspberry-Pi/)，[Github地址](https://github.com/Seeed-Studio/seeed-linux-dtoverlays/tree/master/modules/CAN-HAT)，Seeed的代码来自于[marckleinebudde/linux](https://github.com/marckleinebudde/linux)的`branch v4.19-rpi/mcp25xxfd-20200429-46`，[marckleinebudde/linux](https://github.com/marckleinebudde/linux)里面针对mcp2517fd也有很多branch，驱动为6个c文件。

第一个方案在zynq，linux3.19上编译通过，轻松秒杀，但是收发数据失败，随后github上搜索了一下，发现两个方案，
- [tinhnn/CAN_HW](https://github.com/tinhnn/CAN_HW)类似于上面方案3，但驱动超10个文件。
- [Bytewerk/QuadCAN-FD](https://github.com/Bytewerk/QuadCAN-FD)类似于方案2，相比方案2`linux-rpi/blob/rpi-4.9.y-mcp2517fd-v4a/drivers/net/can/spi/mcp25xxfd.c`只是少了pinctrl，少数代码格式修改了一下移植到3.19，gpiochip和中断部分修改一下代码即可，发送正常，can模式接收会多固定的一帧`<0x004> [8] 00 00 00 00 08 00 00 00`，canfd+can混合模式，发送正常，接收时，CANFD测试工具直接显示发送失败。

硬件排查了两个问题，
- 板卡中断没接上拉电阻，打算用FPGA内部上拉
- 板卡没接终端电阻，CANFD测试工具接了终端电阻

修改了这两个问题，工作在CAN模式收发正常，工作在CANFD模式，板卡发送正常，接收会接收错误帧。随后用树莓派来交叉测试一下，定位问题，编写设备树，搞半天发现树莓派buster版本系统已经有了canfd驱动，且在CAN，CANFD模式下收发正常。

# 树莓派使用QuadCAN-FD
```c
/*
 * Device tree overlay for mcp2517fd/can0 on spi0.0
 */

/dts-v1/;
/plugin/;

/ {
    compatible = "brcm,bcm2835", "brcm,bcm2836", "brcm,bcm2708", "brcm,bcm2709";

    /* disable spi-dev for spi0 and spi1 */
    fragment@0 {
        target = <&spidev0>;
        __overlay__ {
            status = "disabled";
        };
    };
   
    /* MCP2517FD INT pins */
    fragment@2 {
        target = <&gpio>;
        __overlay__ {
            mcp2517fd_int_pins: mcp2517fd_int_pins {
                brcm,pins = <18>;
                brcm,function = <0>; /* input */
            };
        };
    };

    /* dedicated MCP2517FD clock @20MHz */
    fragment@3 {
        target-path = "/clocks";
        __overlay__ {
            mcp2517fd_osc: mcp2517fd_osc {
                compatible = "fixed-clock";
                #clock-cells = <0>;
                clock-frequency  = <40000000>;
            };
        };
    };

    /* CAN: SPI0 CE1 on GPIO7, INT pin on GPIO18 */
    fragment@4 {
        target = <&spi0>;
        __overlay__ {
            #address-cells = <1>;
            #size-cells = <0>;
            status = "okay";

            can0: can@0 {
                compatible = "microchip,mcp2517fd";
                reg = <1>;
                clocks = <&mcp2517fd_osc>;
                spi-max-frequency = <10000000>;
                interrupt-parent = <&gpio>;
                interrupts = <18 0x8>;
            };
        };
    };
};
```
编译驱动，
```bash
$ sudo apt install raspberrypi-kernel-headers
$ make -C /lib/modules/$(uname -r)/build M=$PWD
```
编译设备树，使能设备树，
```bash
$ dtc -@ -I dts -O dtb -o mcp2517fd.dtbo mcp2517fd.dts
$ sudo cp mcp2517fd.dtbo /boot/overlays
$ sudo sh -c 'echo "dtoverlay=mcp2517fd" >> /boot/config.txt'
```
加载驱动，
```bash
sudo insmod mcp25xxfd.ko
```
安装驱动，
```bash
sudo make -C /lib/modules/$(uname -r)/build M=$PWD modules_install
sudo depmod -a
```

# 测试
板卡配置，
```bash
# canfd
$ ip link set can0 up type can bitrate 500000 dbitrate 2000000 fd on
$ ip link set can0 up type can bitrate 1000000 dbitrate 4000000 fd on
# can
$ ip link set can1 up type can bitrate 500000
# 关闭
$ ip link set can0 down
```
打开CAN测试软件，
![380](https://img-blog.csdnimg.cn/20201223210726708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
发送测试，目前z7的can-util工具太老了，不支持canfd，新版本可以但是注意can帧的数据必须是偶数，而canfd必须是奇数，因为多了一个`canfd_frame.flags`，
```bash
$ cansend can0 0x01 0x00
# 新版本cansend工具
pi@raspberrypi:~$ cansend can0 123##a.11.22 # canfd
pi@raspberrypi:~$ cansend can0 123##1.11.22 # canfd
pi@raspberrypi:~$ cansend can0 123##2.11.22 # canfd
pi@raspberrypi:~$ cansend can0 123##21122   # canfd
pi@raspberrypi:~$ cansend can0 123#121122   # can
pi@raspberrypi:~$ cansend -help
Usage: cansend - simple command line tool to send CAN-frames via CAN_RAW sockets.
Usage: cansend <device> <can_frame>.
<can_frame>:
 <can_id>#{R|data}          for CAN 2.0 frames
 <can_id>##<flags>{data}    for CAN FD frames

<can_id>:
 can have 3 (SFF) or 8 (EFF) hex chars
{data}:
 has 0..8 (0..64 CAN FD) ASCII hex-values (optionally separated by '.')
<flags>:
 a single ASCII Hex value (0 .. F) which defines canfd_frame.flags

Examples:
  5A1#11.2233.44556677.88 / 123#DEADBEEF / 5AA# / 123##1 / 213##311
  1F334455#1122334455667788 / 123#R for remote transmission request.
pi@raspberrypi:~$ cansend can0 123##a
pi@raspberrypi:~$ cansend can0 123##a.11.22.33.44.55.66.77.88
pi@raspberrypi:~$ cansend can0 123##a.11.22.33.44.55.66.77.88.11.22.33.44.55.66.77.88
pi@raspberrypi:~$ cansend can0 123##0.11.22.33.44.55.66.77.88.11.22.33.44.55.66.77.88
pi@raspberrypi:~$ cansend can0 123##1.11.22.33.44.55.66.77.88.11.22.33.44.55.66.77.88
```
![381](https://img-blog.csdnimg.cn/20201223214704775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
接收测试，
```bash
$ candump can0
```


