# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [基于Z7010的EBAZ4205矿板改造](http://bbs.eeworld.com.cn/thread-1079271-1-1.html)
> [EBAZ4205 ZYNQ 7Z010 裸机程序NAND固化 JTAG调试方法](https://blog.csdn.net/z951573431/article/details/89739165)
> [ZYNQ--矿机通信控制板](https://blog.csdn.net/qq_22168673/article/details/103083544)
> [zynq7010之EBAZ4205入门改造](https://blog.csdn.net/weixin_42741023/article/details/103335948)

# 介绍
| Column 1 | Column 2      |
|:--------:|:-------------:|
| 主控 | XC7Z010CLG400-1 |
| 内存 | 256MB DDR3 EM6GD16EWKG/MT41K128M16 |
| nand | 128MB SLC Winbond W29N01HV |
| 以太网 | 百兆网Phy芯片IP101GA |
| 供电 | 12V，主板电源，兼容5V |
| 其他 | 1路串口PS UART1，2路PWM，14针标准jtag支持Xilinx下载器，3个20pin IO口 |

将R2578改到R2585，则进入jtag模式，将R2584焊到R2577上，则为SD卡启动。IP101GA phy地址0，mdc位于R2470，mdio位于R2469。
![275](https://img-blog.csdnimg.cn/20200217135256812.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 部署
## 板子有u-boot和内核
```bash
zynq-uboot> set ipaddr 192.168.6.195
zynq-uboot> setenv serverip 192.168.6.6
zynq-uboot> tftpboot 0x10000000 boot.bin
zynq-uboot> nand erase 0 0xc0000
zynq-uboot> nand write 0x10000000 0 0xc0000
zynq-uboot> reset
zynq-uboot> setenv Gem.e000b000_phyaddr 0
zynq-uboot> setenv serverip 192.168.6.6
zynq-uboot> setenv fpga_img ebit_wrapper.bit
zynq-uboot> env save
zynq-uboot> reset
```
下一步加载pl部分的bitstream，可通过串口或者TF卡，我这里使用TF卡，
```bash
zynq-uboot> mmc info && fatload mmc 0 $netstart /ebit_wrapper.bit && fpga loadb 0 ${netstart} ${fpgasize}
```
固化bitstream到nand，这一步完成后，以后再更新固件就不需要TF卡了，
```bash
zynq-uboot> run update_nand_fpga 
zynq-uboot> run nand_fpga_boot
```
开始部署ubifs文件系统，
```bash
zynq-uboot> run ubifs_bringup
```
重启。
## 板子无u-boot
更改zynq配置管脚至TF卡启动，进入我们自己的u-boot之后，设置环境变量，加载bitstream，使用EMIO百兆网，
```bash
zynq-uboot> setenv Gem.e000b000_phyaddr 0
zynq-uboot> setenv serverip 192.168.6.6
zynq-uboot> setenv fpga_img ebit_wrapper.bit
zynq-uboot> env save
zynq-uboot> reset
zynq-uboot> mmc info && fatload mmc 0 $netstart /ebit_wrapper.bit && fpga loadb 0 ${netstart} ${fpgasize}
```
更新boot固件，使用TF卡下载boot固件，是因为vivado下不支持Winbond的Flash，
```bash
zynq-uboot> run update_nand_boot
```
固化bitstream到nand，这一步完成后，以后再更新固件就不需要TF卡了，
```bash
zynq-uboot> run update_nand_fpga 
zynq-uboot> run nand_fpga_boot
```
开始部署ubifs文件系统，
```bash
zynq-uboot> run ubifs_bringup
```
重启，部署完可以将zynq改回nand启动，再次上电就不需要TF卡了。

# 调试记录
```bash
zynq-uboot> mmc info && fatload mmc 0 $netstart /ebit_wrapper.bit && fpga loadb 0 ${netstart} ${fpgasize}
zynq-uboot> mii info
PHY 0x00: OUI = 0x90C3, Model = 0x05, Rev = 0x04, 100baseT, FDX
zynq-uboot> mdio list
Gem.e000b000:
0 - IP101A/G <--> Gem.e000b000
root@zynq:~# cat start.sh 
#!/bin/sh

if [ -e "/dev/mmcblk0p1" ]; then
        if [ ! -d "/media/mmcblk0p1" ]; then
                mkdir -p /media/mmcblk0p1
        fi
        mount /dev/mmcblk0p1 /media/mmcblk0p1
fi
```
采用vivado s34ml01g1-nand-x8可以直接下载boot。

# 最后
ebit EBAZ4205 zynq矿机控制卡固件，逃堡和咸余大量有货，只需要40元，相比很多动辄几百上千的FPGA开发板实惠多了，确实40元你买不了上当买不了吃亏，我这也是给群友谋福利了，QQ群852283276。
