# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [mmcblk0: error -110 transferring data, sector 266312, nr 240, cmd response 0x900, card status 0xb00](https://forums.xilinx.com/xlnx/board/crawl_message?board.id=ELINUX&message.id=17124)
> [Linux Kernel 4.9, emmc issue](https://forums.xilinx.com/xlnx/board/crawl_message?board.id=ELINUX&message.id=21374)

官网：
> [面向 Zynq-7000 SoC、eMMC 的设计咨询 - JEDEC 标准版 4.41 所需的输入保持时间为 3 纳秒](https://china.xilinx.com/support/answers/59999.html)
> [2015.3 SDK Zynq-7000 eMMC fails reading EXT_CSD reg to check for high speed mode support](https://china.xilinx.com/support/answers/65755.html)
> [](https://china.xilinx.com/support/answers/71019.html)
> [AR# 69995 2017.1-2017.4 Zynq UltraScale+ MPSoC: Linux mmcblk0 error -110 sending stop command, original cmd response 0x900, card status 0xe00 when using Swissbit SD card](https://china.xilinx.com/support/answers/69995.html)
> [AR# 67157 Zynq UltraScale+ MPSoC: eMMC Programming Solutions](https://china.xilinx.com/support/answers/67157.html)
> [AR# 71019 Zynq UltraScale+ MPSoC: eMMC Booting Checklist](https://china.xilinx.com/support/answers/71019.html)
> [AR# 65463 Zynq UltraScale+ MPSoC - What devices are supported for configuration?](https://china.xilinx.com/support/answers/65463.html)
> [AR# 71825 Zynq UltraScale+ MPSoC SD / eMMC clock has falling edge skew at 200 MHz (SDR104)](https://china.xilinx.com/support/answers/71825.html)
> [AR# 65676 Zynq UltraScale+ MPSoC, SDIO - SDIO Receiver Auto Tuning Fails In SD104/eMMC 200 Modes](https://china.xilinx.com/support/answers/65676.html)
> [AR# 69332 2017.1 Zynq UltraScale+ MPSoC：U-boot 需要一个补丁在 HS200 下运行 eMMC](https://china.xilinx.com/support/answers/69332.html)
> [AR# 69368 2017.1-2018.1 Zynq UltraScale+ MPSoC: How to slow down eMMC from HS200 to High Speed (HS) in FSBL, u-boot and Linux](https://china.xilinx.com/support/answers/69368.html)
> [AR# 69780 2016.4-2017.4 Zynq UltraScale+ MPSoC: PetaLinux does not correctly override the U-boot environment variables to set SD boot when both eMMC(SDIO0) and SD(SDIO1) are enabled in design](https://china.xilinx.com/support/answers/69780.html)
> [AR# 69978 Zynq UltraScale+ MPSoC: How to enable UHS (SD 3.0) support for ZCU102 and ZCU106 evaluation board PetaLinux BSPs](https://www.xilinx.com/support/answers/69978.html)
> [Kernel Boot Failing on Zynq in U-Boot](https://forums.xilinx.com/t5/Embedded-Linux/Kernel-Boot-Failing-on-Zynq-in-U-Boot/td-p/703136)
> [SD controller](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842090/SD+controller)

# Zynq SD boot
使用MIO40~MIO45，
![397](https://img-blog.csdnimg.cn/20210305095224355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 让设备树决定emmc的设备号
添加如下代码，
```c
struct mmc_host *mmc_alloc_host(int extra, struct device *dev)
{
	int err;
	struct mmc_host *host;
	int aliasesid;/*add by zhuce*/
...
	aliasesid = of_alias_get_id(dev->of_node, "mmc");
	if (aliasesid>= 0) {
		host->index = aliasesid;
	}
...
}
```
此时mmc的设备号变了，但是块设备的标号还是没变，先扫描到的为mmcblk0，所以只要在设备树里把要先发现的mmc放到前面就可以保证它的mmc的设备号和块设备号都是0。

# zu emmc降频到HS模式
发现ftp上传大文件到emmc内核会卡死，尝试降频到HS模式，在linux下只需要修改设备树，uboot和fsbl下降频比较复杂，参考AR69368。
```c
&sdhci0 {
  no-1-8-v;
};
```
降频后还是卡死，但是有改善，可以上传400MB文件，上传1GB文件还是卡死。
```shell
root@zynqmp:~# [   58.230411] random: crng init done
[  104.514173] init[1]: undefined instruction: pc=0000000000401bd0
[  104.520513] Code: 3e0ffadc 0749bf19 3e4db43a b0d3e231 (56ade825) 
[  104.526538] klogd[2051]: undefined instruction: pc=0000007f907a5160
[  104.527989] Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000004
[  104.527989] 
[  104.527995] CPU: 0 PID: 1 Comm: init Tainted: G           O    4.14.0-fdk-1.0.0-20181120.1856 #20
[  104.527996] Hardware name: xlnx,zynqmp (DT)
[  104.527998] Call trace:
[  104.528010] [<ffffff8008088ae8>] dump_backtrace+0x0/0x360
[  104.528014] [<ffffff8008088e5c>] show_stack+0x14/0x20
[  104.528019] [<ffffff80089d0320>] dump_stack+0x9c/0xbc
[  104.528024] [<ffffff800809af98>] panic+0x11c/0x274
[  104.528029] [<ffffff800809e1a8>] complete_and_exit+0x0/0x20
[  104.528032] [<ffffff800809ee70>] do_group_exit+0x38/0xa8
[  104.528037] [<ffffff80080a9108>] get_signal+0x130/0x470
[  104.528041] [<ffffff8008087df8>] do_signal+0x68/0x650
[  104.528045] [<ffffff80080887c0>] do_notify_resume+0xc0/0xf0
[  104.528048] Exception stack(0xffffff800803bec0 to 0xffffff800803c000)
[  104.528051] bec0: 0000007ffd8b6810 0000000000000000 0000000000000000 0000000000000000
[  104.528055] bee0: 0000007ffd8b6380 0000000000000000 0000000000000000 0000000000006028
[  104.528058] bf00: 0000000000000048 003b9aca00000000 00000000fff0334c 000000000001ac10
[  104.528061] bf20: 0000000000000018 00000003e8000000 00066ff2d7593574 0000182c9dd79931
[  104.528064] bf40: 0000000000418258 0000007fb589b938 0000000000000874 0000000000418678
[  104.528067] bf60: 0000000000418640 0000000000418308 0000000000406688 0000000000000000
[  104.528070] bf80: 0000000000406000 0000000000406000 0000000000406000 00000000ffffff9d
[  104.528073] bfa0: 0000000000000000 0000007ffd8b6790 0000000000405a58 0000007ffd8b6790
[  104.528077] bfc0: 0000000000401bd0 0000000060000000 000000000000000b 00000000ffffffff
[  104.528079] bfe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
[  104.528083] [<ffffff80080836c0>] work_pending+0x8/0x10
[  104.528087] SMP: stopping secondary CPUs
[  104.532762] Kernel Offset: disabled
[  104.532764] CPU features: 0x002004
[  104.532766] Memory Limit: none
[  104.707664] ---[ end Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000004
[  104.707664] 
```
ftp截图，这个问题在zynq平台没有出现过。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190222110327532.PNG)
开启UHS模式，更新FSBL，`project-spec/meta-user/recipes-bsp/fsbl/fsbl_%.bbappend`，
```bash
YAML_COMPILER_FLAGS_append = " -DUHS_MODE_ENABLE"
```
设备树，
```bash
& sdhci1 {
    /delete-property/ no-1-8-v;
};
```

# 不识别SD卡
SD卡识别不到都是因为更换了hdf没有更新fsbl的原因，但看了fsbl的源代码，没有相关代码，所以应该是hw project的系统初始化中的相关代码，更改后识别到ZC706 SD卡，
```shell
mmc0: new high speed SDHC card at address 59b4
mmcblk0: mmc0:59b4 SDU1  14.8 GiB 
 mmcblk0: p1 p2 p3
```

#  uboot sdhci_set_clock: Internal clock never stabilised
uboot下打印sdhci_set_clock: Internal clock never stabilised.
FPGA使用了SD1，uboot中为SD0

# uboot sdhci_send_command: MMC: 0 busy timeout increasing to: 200 ms.
uboot下访问emmc失败
```shell
zynq-uboot> mmcinfo
sdhci_send_command: MMC: 0 busy timeout increasing to: 200 ms.
sdhci_send_command: MMC: 0 busy timeout increasing to: 400 ms.
sdhci_send_command: MMC: 0 busy timeout increasing to: 800 ms.
sdhci_send_command: MMC: 0 busy timeout increasing to: 1600 ms.
sdhci_send_command: MMC: 0 busy timeout increasing to: 3200 ms.
sdhci_send_command: MMC: 0 busy timeout.
```
SD采用MIO，时钟50MHz，更新fsbl，正确显示，SD采用EMIO，则必须添加约束，
```shell
zynq-uboot> mmcinfo
Device: zynq_sdhci
Manufacturer ID: 45
OEM: 100
Name: DG400 
Tran Speed: 52000000
Rd Block Len: 512
MMC version 4.0
High Capacity: Yes
Capacity: 7.3 GiB
Bus Width: 4-bit
Erase Group Size: 512 KiB
```

# uboot 2015.04 petalinux2015.2.1 ext4write报错
```shell
Bytes transferred = 3652736 (37bc80 hex)
Device: zynq_sdhci
Manufacturer ID: 45
OEM: 100
Name: DG400 
Tran Speed: 52000000
Rd Block Len: 512
MMC version 4.0
High Capacity: Yes
Capacity: 7.3 GiB
Bus Width: 4-bit
Erase Group Size: 512 KiB
File System is consistent
sdhci_send_command: MMC: 0 busy timeout increasing to: 200 ms.
sdhci_send_command: MMC: 0 busy timeout increasing to: 400 ms.
sdhci_send_command: MMC: 0 busy timeout increasing to: 800 ms.
sdhci_send_command: MMC: 0 busy timeout increasing to: 1600 ms.
sdhci_send_command: MMC: 0 busy timeout increasing to: 3200 ms.
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
 ** ext4fs_devread read error - block
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
 ** ext4fs_devread read error - block
Error in getting the block group descriptor table
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
 ** ext2fs_devread() read error **
sdhci_send_command: MMC: 0 busy timeout.
 ** ext4fs_devread read error - block
sdhci_send_command: MMC: 0 busy timeout.
sdhci_send_command: MMC: 0 busy timeout.
 ** ext4fs_devread read error - block
sdhci_send_command: MMC: 0 busy timeout.
error in File System init
** Error ext4fs_write() **
** Unable to write file /boot/uImage **
```
更改u-boot，
```c
/*add_sdhci(host, 52000000, 52000000 >> 9);*/
add_sdhci(host, CONFIG_ZYNQ_SDHCI_WORK_FREQ, CONFIG_ZYNQ_SDHCI_WORK_FREQ >> 9);
```

# linux3.19  petalinux2015.2.1 内核正常启动
```shell
Driver 'mmcblk' needs updating - please use bus_type methods
sdhci: Secure Digital Host Controller Interface driver
sdhci: Copyright(c) Pierre Ossman
sdhci-pltfm: SDHCI platform and OF driver helper
sdhci-arasan e0100000.sdhci: No vmmc regulator found
sdhci-arasan e0100000.sdhci: No vqmmc regulator found
mmc0: SDHCI controller on e0100000.sdhci [e0100000.sdhci] using ADMA
...
mmc0: new high speed MMC card at address 0001
mmcblk0: mmc0:0001 DG4008 7.28 GiB 
mmcblk0boot0: mmc0:0001 DG4008 partition 1 4.00 MiB
mmcblk0boot1: mmc0:0001 DG4008 partition 2 4.00 MiB
mmcblk0rpmb: mmc0:0001 DG4008 partition 3 4.00 MiB
 mmcblk0: p1
EXT4-fs (mmcblk0p1): warning: mounting fs with errors, running e2fsck is recommended
EXT4-fs (mmcblk0p1): mounted filesystem with ordered data mode. Opts: (null)
VFS: Mounted root (ext4 filesystem) on device 179:1.
```

# petalinux2015.2.1 内核启动报错mmcblk0rpmb无法访问
报错mmcblk0rpmb无法访问，原因待定位，
```shell
mmc0: new high speed MMC card at address 0001
mmcblk0: mmc0:0001 DG4008 7.28 GiB 
mmcblk0boot0: mmc0:0001 DG4008 partition 1 4.00 MiB
mmcblk0boot1: mmc0:0001 DG4008 partition 2 4.00 MiB
mmcblk0rpmb: mmc0:0001 DG4008 partition 3 4.00 MiB
 mmcblk0: p1
EXT4-fs (ram0): mounted filesystem with ordered data mode. Opts: (null)
VFS: Mounted root (ext4 filesystem) on device 1:0.
devtmpfs: mounted
Freeing unused kernel memory: 224K (40686000 - 406be000)
INIT: version 2.88 booting
mount: mounting devtmpfs on /dev failed: Device or resource busy
EXT4-fs (mmcblk0p1): mounted filesystem with ordered data mode. Opts: (null)
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
blk_update_request: I/O error, dev mmcblk0rpmb, sector 0
FAT-fs (mmcblk0rpmb): unable to read boot sector
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
blk_update_request: I/O error, dev mmcblk0rpmb, sector 0
FAT-fs (mmcblk0rpmb): unable to read boot sector
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
mmcblk0rpmb: timed out sending r/w cmd command, card status 0x400900
blk_update_request: I/O error, dev mmcblk0rpmb, sector 0
FAT-fs (mmcblk0rpmb): unable to read boot sector
mount: mounting /dev/mmcblk0rpmb on /run/media/mmcblk0rpmb failed: Invalid argument
mount: mounting /dev/mmcblk0boot0 on /run/media/mmcblk0boot0 failed: Invalid argument
mount: mounting /dev/mmcblk0boot1 on /run/media/mmcblk0boot1 failed: Invalid argument
sh: /sys/block/mmcblk0/mmcblk0p1: unknown operand
mount: mounting /dev/mmcblk0 on /run/media/mmcblk0 failed: Device or resource busy
EXT4-fs (ram0): re-mounted. Opts: data=ordered
```
用不到该分区，所以修改mdev的脚本，屏蔽这个分区，
```shell
#!/bin/sh
MDEV_AUTOMOUNT=y
MDEV_AUTOMOUNT_ROOT=/run/media
[ -f /etc/default/mdev ] && . /etc/default/mdev
if [ "${MDEV_AUTOMOUNT}" = "n" ] ; then
	exit 0
fi

case "$ACTION" in
	add|"")
		ACTION="add"
		exit 0
		# check if already mounted
		if grep -q "^/dev/${MDEV} " /proc/mounts ; then
			# Already mounted
			exit 0
		fi
		DEVBASE=`expr substr $MDEV 1 3`
		if [ "${DEVBASE}" == "mmc" ] ; then
			DEVBASE=`expr substr $MDEV 1 7`
			DEVLEN=`expr length "$MDEV"`
			if [ $DEVLEN -gt 9 ]; then
				exit 0
			fi
		fi
		# check for "please don't mount it" file
		if [ -f "/dev/nomount.${DEVBASE}" ] ; then
			# blocked
			exit 0
		fi
		# check for full-disk partition
		if [ "${DEVBASE}" == "${MDEV}" ] ; then
			if [ -d "/sys/block/${DEVBASE}/${DEVBASE}*1" ] ; then
				# Partition detected, just quit
				exit 0
			fi
			if [ -d "/sys/block/${DEVBASE}/${DEVBASE}p1" ] ; then
				# Partition detected, just quit
				exit 0
			fi
			if [ -d "/sys/block/${DEVBASE}/${DEVBASE}1" ] ; then
				# Partition detected, just quit
				exit 0
			fi
			if [ ! -f "/sys/block/${DEVBASE}/size" ] ; then
				# No size at all
				exit 0
			fi
			if [ `cat /sys/block/${DEVBASE}/size` == 0 ] ; then
				# empty device, bail out
				exit 0
			fi
		fi
		# first allow fstab to determine the mountpoint
		if ! mount /dev/$MDEV > /dev/null 2>&1
		then
			if [ ! -d ${MDEV_AUTOMOUNT_ROOT} ]; then
				mkdir ${MDEV_AUTOMOUNT_ROOT}
			fi
			MOUNTPOINT="${MDEV_AUTOMOUNT_ROOT}/$MDEV"
			mkdir "$MOUNTPOINT"
			mount -t auto /dev/$MDEV "$MOUNTPOINT"
		fi
		;;
	remove)
		MOUNTPOINT=`grep "^/dev/$MDEV\s" /proc/mounts | cut -d' ' -f 2`
		if [ ! -z "$MOUNTPOINT" ]
		then
			umount "$MOUNTPOINT"
			rmdir "$MOUNTPOINT"
		else
			umount /dev/$MDEV
		fi
		;;
	*)
		# Unexpected keyword
		exit 1
		;;
esac
```

# linux3.19  petalinux2015.2.1 内核启动报错
原因定位为uboot MAPSZ配置错误。
```shell
Driver 'mmcblk' needs updating - please use bus_type methods
sdhci: Secure Digital Host Controller Interface driver
sdhci: Copyright(c) Pierre Ossman
sdhci-pltfm: SDHCI platform and OF driver helper
sdhci-arasan e0100000.sdhci: No vmmc regulator found
sdhci-arasan e0100000.sdhci: No vqmmc regulator found
mmc0: SDHCI controller on e0100000.sdhci [e0100000.sdhci] using ADMA
...
mmc0: new high speed MMC card at address 0001
Unable to handle kernel NULL pointer dereference at virtual address 00000220
pgd = 40004000
[00000220] *pgd=00000000
Internal error: Oops - BUG: 805 [#1] PREEMPT SMP ARM
Modules linked in:
CPU: 0 PID: 6 Comm: kworker/u4:0 Not tainted 3.19.0-1.0.0 #35
Hardware name: Xilinx Zynq Platform
Workqueue: kmmcd mmc_rescan
task: 7f02f080 ti: 7f062000 task.ti: 7f062000
PC is at mmc_blk_alloc_req+0x194/0x358
LR is at try_to_wake_up+0x270/0x288
pc : [<403aec68>]    lr : [<4003d570>]    psr: 60000113
sp : 7f063dc0  ip : 7f763ad0  fp : 406fc67c
r10: 7f2b3008  r9 : 7f26dc08  r8 : 00000000
r7 : 00000000  r6 : 00000000  r5 : 7f2b3000  r4 : 7f26dc00
r3 : 00000000  r2 : 404f5bf0  r1 : 60000113  r0 : 00000000
Flags: nZCv  IRQs on  FIQs on  Mode SVC_32  ISA ARM  Segment kernel
Control: 18c5387d  Table: 0000404a  DAC: 00000015
Process kworker/u4:0 (pid: 6, stack limit = 0x7f062238)
Stack: (0x7f063dc0 to 0x7f064000)
3dc0: 00000000 401d5cd0 00e90e80 00000000 000008f5 7f2b3008 4072c35c 4027aec4
3de0: 00000001 00000001 7f2b3000 4072c338 00000000 403af864 00000000 00000000
3e00: 00000001 7f0f6998 6ea4f538 40606009 6ea4f308 00000001 406fc324 4072c338
3e20: 00000000 40107fa0 7f2b3008 7f2b3008 4072c35c 4027aec4 406fc67c 00000001
3e40: 406fc324 4072c338 00000000 4027ad84 00000000 7f2b3008 4027aec4 7f2b3808
3e60: 00000000 40279764 7f2ab970 7f0cd4b8 7f2b3008 406fc27c 7f2b303c 4027acb8
3e80: 7f2b3008 406fc27c 7f2b3008 4027a428 7f2b3008 7f2b3010 00000000 40278cd8
3ea0: 7f2b3008 40611891 00000000 7f2b3ad8 7f2b3000 7f2b3008 405d58aa 00061a80
3ec0: 404f561c 00000000 7f02d860 403a5d94 405d58aa 405d58aa 4061193f 00000001
3ee0: 7f2b3800 00000000 00000000 403a85e0 7f2b3800 40ff8080 00000000 7f2b3a2c
3f00: 7f2b3800 403a562c 7f028880 7f2b3a2c 7f02d800 00000000 7f0a8d00 40032288
3f20: 7f028880 7f2b3a2c 00000088 7f028880 7f02d800 7f028898 406c2100 7f02d814
3f40: 7f02d994 4003273c 7f029680 00000000 7f028880 400325b4 00000000 00000000
3f60: 00000000 40035fc8 c1572f63 00000000 0a30e91e 7f028880 00000000 00000000
3f80: 7f063f80 7f063f80 00000000 00000000 7f063f90 7f063f90 7f063fac 7f029680
3fa0: 40035ef0 00000000 00000000 4000dd40 00000000 00000000 00000000 00000000
3fc0: 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
3fe0: 00000000 00000000 00000000 00000000 00000013 00000000 e0068ce0 14278861
[<403aec68>] (mmc_blk_alloc_req) from [<403af864>] (mmc_blk_probe+0x7c/0x2a0)
[<403af864>] (mmc_blk_probe) from [<4027ad84>] (driver_probe_device+0x8c/0x1cc)
[<4027ad84>] (driver_probe_device) from [<40279764>] (bus_for_each_drv+0x84/0x94)
[<40279764>] (bus_for_each_drv) from [<4027acb8>] (device_attach+0x64/0x88)
[<4027acb8>] (device_attach) from [<4027a428>] (bus_probe_device+0x28/0x80)
[<4027a428>] (bus_probe_device) from [<40278cd8>] (device_add+0x3bc/0x4c8)
[<40278cd8>] (device_add) from [<403a5d94>] (mmc_add_card+0x19c/0x20c)
[<403a5d94>] (mmc_add_card) from [<403a85e0>] (mmc_attach_mmc+0xb0/0x128)
[<403a85e0>] (mmc_attach_mmc) from [<403a562c>] (mmc_rescan+0x264/0x2b4)
[<403a562c>] (mmc_rescan) from [<40032288>] (process_one_work+0x13c/0x21c)
[<40032288>] (process_one_work) from [<4003273c>] (worker_thread+0x188/0x288)
[<4003273c>] (worker_thread) from [<40035fc8>] (kthread+0xd8/0xec)
[<40035fc8>] (kthread) from [<4000dd40>] (ret_from_fork+0x14/0x34)
Code: e0060693 e5826004 e59f21b8 e5943004 (e5832220) 
---[ end trace 834c0d842203f8e0 ]---
Unable to handle kernel paging request at virtual address ffffffec
pgd = 40004000
[ffffffec] *pgd=3ff7e821, *pte=00000000, *ppte=00000000
Internal error: Oops - BUG: 17 [#2] PREEMPT SMP ARM
Modules linked in:
CPU: 0 PID: 6 Comm: kworker/u4:0 Tainted: G      D        3.19.0-1.0.0 #35
Hardware name: Xilinx Zynq Platform
task: 7f02f080 ti: 7f062000 task.ti: 7f062000
PC is at kthread_data+0x4/0xc
LR is at wq_worker_sleeping+0xc/0x9c
pc : [<4003644c>]    lr : [<40032b08>]    psr: 00000193
sp : 7f063af8  ip : 7f02f0c8  fp : 7f063b84
r10: 406c8fc0  r9 : 00000000  r8 : 7f02f314
r7 : 406be600  r6 : 7f062000  r5 : 7f02f080  r4 : 00000000
r3 : 00000000  r2 : 7f75a600  r1 : 00000000  r0 : 7f02f080
Flags: nzcv  IRQs off  FIQs on  Mode SVC_32  ISA ARM  Segment user
Control: 18c5387d  Table: 0000404a  DAC: 00000015
Process kworker/u4:0 (pid: 6, stack limit = 0x7f062238)
Stack: (0x7f063af8 to 0x7f064000)
3ae0:                                                       7f75a600 404a7ef4
3b00: 0000001a 00000013 00000004 00000008 7f02f0f0 7f063b00 7f030040 7f030544
3b20: 7f02f290 7f02f080 00000001 4003458c 7f75abc0 40058450 7f02f080 406bd248
3b40: 00000000 7f030040 7f030544 40021c2c 00000005 00000000 00000000 00000001
3b60: 7f062000 7f02f080 7f06396c 7f02a9c0 7f063b98 00000001 7f02f290 403aec68
3b80: 7f063be3 400223f0 7f063be3 4004ee90 405ce445 7f063bb4 7f063b98 7f063b98
3ba0: 00000002 40704684 0000000b 403aec6a 00000000 7f063c12 406cc99c 403aec68
3bc0: 7f063be3 40011004 7f062238 0000000b 00000000 00000008 60000113 00000000
3be0: 65000000 30363030 20333936 32383565 34303036 39356520 62313266 35652038
3c00: 30333439 28203430 33383565 30323232 40002029 7f063c34 00000000 00000220
3c20: 00000805 7f063d78 00000000 7f02f080 00000000 00000805 406fc67c 404a55b4
3c40: 7f063d78 40019970 04208060 ffff8b49 00000000 406c2100 406c2080 00000009
3c60: 7f063d88 00000000 00000000 406bdab4 00000012 7f002400 00000010 404c8fbc
3c80: 7f063d88 40023a3c 7f112b00 7f112b00 00000000 00000000 ffffffff 406cfa80
3ca0: 7f063cfc 4003ba14 401dbdb0 00000113 00000000 00000805 406cd7cc 00000220
3cc0: 7f063d78 00000000 7f26dc08 7f2b3008 406fc67c 4000847c 00000000 7f02f080
3ce0: 406be600 40040fb8 7f763600 00000000 00000000 7f763600 7f763600 7f112ef4
3d00: 406be600 00000000 00000001 7f763600 7f063d24 4003a210 00000d13 7f112b00
3d20: 7f063d4c 4003a270 00000d13 7f112b00 7f763600 7f112ef4 406be600 00000000
3d40: 00000001 406c8fc0 7f063d7c 4003d3b0 00000000 60000113 7f063d58 7f26dc08
3d60: 00000000 403aec68 60000113 ffffffff 7f063dac 40011698 00000000 60000113
3d80: 404f5bf0 00000000 7f26dc00 7f2b3000 00000000 00000000 00000000 7f26dc08
3da0: 7f2b3008 406fc67c 7f763ad0 7f063dc0 4003d570 403aec68 60000113 ffffffff
3dc0: 00000000 401d5cd0 00e90e80 00000000 000008f5 7f2b3008 4072c35c 4027aec4
3de0: 00000001 00000001 7f2b3000 4072c338 00000000 403af864 00000000 00000000
3e00: 00000001 7f0f6998 6ea4f538 40606009 6ea4f308 00000001 406fc324 4072c338
3e20: 00000000 40107fa0 7f2b3008 7f2b3008 4072c35c 4027aec4 406fc67c 00000001
3e40: 406fc324 4072c338 00000000 4027ad84 00000000 7f2b3008 4027aec4 7f2b3808
3e60: 00000000 40279764 7f2ab970 7f0cd4b8 7f2b3008 406fc27c 7f2b303c 4027acb8
3e80: 7f2b3008 406fc27c 7f2b3008 4027a428 7f2b3008 7f2b3010 00000000 40278cd8
3ea0: 7f2b3008 40611891 00000000 7f2b3ad8 7f2b3000 7f2b3008 405d58aa 00061a80
3ec0: 404f561c 00000000 7f02d860 403a5d94 405d58aa 405d58aa 4061193f 00000001
3ee0: 7f2b3800 00000000 00000000 403a85e0 7f2b3800 40ff8080 00000000 7f2b3a2c
3f00: 7f2b3800 403a562c 7f028880 7f2b3a2c 7f02d800 00000000 7f0a8d00 40032288
3f20: 7f028880 7f2b3a2c 00000088 7f028880 7f02d800 7f028898 406c2100 7f02d814
3f40: 7f02d994 4003273c 7f029680 00000000 7f028880 400325b4 00000000 00000000
3f60: 00000000 40035fc8 c1572f63 00000000 0a30e91e 7f028880 00000000 00000000
3f80: 7f063f80 7f063f80 00000001 00010001 7f063f90 7f063f90 7f063fac 7f029680
3fa0: 40035ef0 00000000 00000000 4000dd40 00000000 00000000 00000000 00000000
3fc0: 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
3fe0: 00000000 00000000 00000000 00000000 00000013 00000000 e0068ce0 14278861
[<4003644c>] (kthread_data) from [<40032b08>] (wq_worker_sleeping+0xc/0x9c)
[<40032b08>] (wq_worker_sleeping) from [<404a7ef4>] (__schedule+0xf4/0x448)
[<404a7ef4>] (__schedule) from [<400223f0>] (do_exit+0x79c/0x7cc)
[<400223f0>] (do_exit) from [<40011004>] (die+0x2ac/0x3a4)
[<40011004>] (die) from [<404a55b4>] (__do_kernel_fault.part.11+0x54/0x74)
[<404a55b4>] (__do_kernel_fault.part.11) from [<40019970>] (do_page_fault+0x320/0x370)
[<40019970>] (do_page_fault) from [<4000847c>] (do_DataAbort+0x34/0x98)
[<4000847c>] (do_DataAbort) from [<40011698>] (__dabt_svc+0x38/0x60)
Exception stack(0x7f063d78 to 0x7f063dc0)
3d60:                                                       00000000 60000113
3d80: 404f5bf0 00000000 7f26dc00 7f2b3000 00000000 00000000 00000000 7f26dc08
3da0: 7f2b3008 406fc67c 7f763ad0 7f063dc0 4003d570 403aec68 60000113 ffffffff
[<40011698>] (__dabt_svc) from [<403aec68>] (mmc_blk_alloc_req+0x194/0x358)
[<403aec68>] (mmc_blk_alloc_req) from [<403af864>] (mmc_blk_probe+0x7c/0x2a0)
[<403af864>] (mmc_blk_probe) from [<4027ad84>] (driver_probe_device+0x8c/0x1cc)
[<4027ad84>] (driver_probe_device) from [<40279764>] (bus_for_each_drv+0x84/0x94)
[<40279764>] (bus_for_each_drv) from [<4027acb8>] (device_attach+0x64/0x88)
[<4027acb8>] (device_attach) from [<4027a428>] (bus_probe_device+0x28/0x80)
[<4027a428>] (bus_probe_device) from [<40278cd8>] (device_add+0x3bc/0x4c8)
[<40278cd8>] (device_add) from [<403a5d94>] (mmc_add_card+0x19c/0x20c)
[<403a5d94>] (mmc_add_card) from [<403a85e0>] (mmc_attach_mmc+0xb0/0x128)
[<403a85e0>] (mmc_attach_mmc) from [<403a562c>] (mmc_rescan+0x264/0x2b4)
[<403a562c>] (mmc_rescan) from [<40032288>] (process_one_work+0x13c/0x21c)
[<40032288>] (process_one_work) from [<4003273c>] (worker_thread+0x188/0x288)
[<4003273c>] (worker_thread) from [<40035fc8>] (kthread+0xd8/0xec)
[<40035fc8>] (kthread) from [<4000dd40>] (ret_from_fork+0x14/0x34)
Code: e513001c e7e00150 e12fff1e e5903268 (e5130014) 
---[ end trace 834c0d842203f8e1 ]---
Fixing recursive fault but reboot is needed!
```

