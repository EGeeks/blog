# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [u-boot增加对jffs2分区的识别与加载](https://blog.csdn.net/lianyq1986/article/details/6878103)
> [u-boot访问jffs2文件系统](https://blog.csdn.net/voice_shen/article/details/16851097)
> [JFFS2 文件系统及新特性介绍](https://www.ibm.com/developerworks/cn/linux/l-jffs2/)
> [Uboot对jffs2的支持](http://blog.chinaunix.net/uid-24565138-id-2127552.html)
> [JFFS2文件系统挂载过程优化的分析报告](https://blog.csdn.net/lh2016rocky/article/details/54693248)
> [AR# 71439 2017.1-2018.2: Zynq UltraScale+ MPSoC: Linux kernel panic for JFFS2 filesystem on POR or reboot](https://www.xilinx.com/support/answers/71439.html)
> [AR# 71114 2017.1-2018.2 Zynq UltraScale+ MPSoC: Linux kernel boot failed while mounting a JFFS2 filesystem in QSPI boot mode](https://www.xilinx.com/support/answers/71114.html)
> [在Linux PC上挂载JFFS2文件系统](https://blog.csdn.net/benkaoya/article/details/52328815)
> [jffs2文件系统镜像挂载到Ubuntu](https://blog.csdn.net/wammaw1314/article/details/70306364)
> [ubuntu下挂载jffs2文件系统](https://blog.csdn.net/u014780165/article/details/43192663)

# uboot
uboot打开宏支持，
```c
#define CONFIG_CMD_JFFS2/*modify by zhuce*/
#define CONFIG_MTD_DEVICE
#define CONFIG_MTD_PARTITIONS
#define CONFIG_CMD_MTDPARTS
#define CONFIG_FLASH_CFI_MTD
```
在FreeScale T2080的u-boot下，编译错误，更改`jffs2.c`的命令列表，
```shell
cmd/jffs2.o:(.u_boot_list_2_cmd_2_ls+0x0): multiple definition of `_u_boot_list_2_cmd_2_ls'
cmd/fs.o:(.u_boot_list_2_cmd_2_ls+0x0): first defined here
scripts/Makefile.build:359: recipe for target 'cmd/built-in.o' failed
make[1]: *** [cmd/built-in.o] Error 1
Makefile:1223: recipe for target 'cmd' failed
make: *** [cmd] Error 2
```
更改环境变量，
```shell
partition=nor0,0
mtdids=nor0=fe0000000.nor
mtdparts=mtdparts=fe0000000.nor:768k(boot),128k@7f00000(fman),-(user)
```
`-(user)`的`-`表示剩余空间，`jffs2.c`官方的环境变量介绍，
```c
/*
 * Three environment variables are used by the parsing routines:
 *
 * 'partition' - keeps current partition identifier
 *
 * partition  := <part-id>
 * <part-id>  := <dev-id>,part_num
 *
 *
 * 'mtdids' - linux kernel mtd device id <-> u-boot device id mapping
 *
 * mtdids=<idmap>[,<idmap>,...]
 *
 * <idmap>    := <dev-id>=<mtd-id>
 * <dev-id>   := 'nand'|'nor'|'onenand'<dev-num>
 * <dev-num>  := mtd device number, 0...
 * <mtd-id>   := unique device tag used by linux kernel to find mtd device (mtd->name)
 *
 *
 * 'mtdparts' - partition list
 *
 * mtdparts=mtdparts=<mtd-def>[;<mtd-def>...]
 *
 * <mtd-def>  := <mtd-id>:<part-def>[,<part-def>...]
 * <mtd-id>   := unique device tag used by linux kernel to find mtd device (mtd->name)
 * <part-def> := <size>[@<offset>][<name>][<ro-flag>]
 * <size>     := standard linux memsize OR '-' to denote all remaining space
 * <offset>   := partition start offset within the device
 * <name>     := '(' NAME ')'
 * <ro-flag>  := when set to 'ro' makes partition read-only (not used, passed to kernel)
 *
 * Notes:
 * - each <mtd-id> used in mtdparts must albo exist in 'mtddis' mapping
 * - if the above variables are not set defaults for a given target are used
 *
 * Examples:
 *
 * 1 NOR Flash, with 1 single writable partition:
 * mtdids=nor0=edb7312-nor
 * mtdparts=mtdparts=edb7312-nor:-
 *
 * 1 NOR Flash with 2 partitions, 1 NAND with one
 * mtdids=nor0=edb7312-nor,nand0=edb7312-nand
 * mtdparts=mtdparts=edb7312-nor:256k(ARMboot)ro,-(root);edb7312-nand:-(home)
 *
 */
```
u-boot下切换分区，查看分区命令，
```shell
T2080> chpart nor0,1
partition changed to nor0,1
T2080> mtdparts

device nor0 <fe0000000.nor>, # parts = 7
 #: name                size            offset          mask_flags
 0: rcw                 0x00020000      0x00000000      0
 1: fs                  0x0f5e0000      0x00020000      0
 2: dtb                 0x00020000      0x0f600000      0
 3: kernel              0x008e0000      0x0f620000      0
 4: fman                0x00020000      0x0ff00000      0
 5: bootenv             0x00020000      0x0ff20000      0
 6: boot                0x000c0000      0x0ff40000      0

active partition: nor0,1 - (fs) 0x0f5e0000 @ 0x00020000

defaults:
mtdids  : nor0=fe0000000.nor
mtdparts: mtdparts=fe0000000.nor:128k(rcw),251776k(fs),128k(dtb),9088k(kernel),128k(fman),128k(bootenv),768k(boot)
```
jffs2相关命令，
```shell
T2080> jffs2info
### filesystem type is JFFS2
Scanning JFFS2 FS: ................................... done.
Compression: NONE
        frag count: 1522
        compressed sum: 379035
        uncompressed sum: 379035
Compression: ZERO
        frag count: 0
        compressed sum: 0
        uncompressed sum: 0
Compression: RTIME
        frag count: 11
        compressed sum: 68
        uncompressed sum: 174
Compression: RUBINMIPS
        frag count: 0
        compressed sum: 0
        uncompressed sum: 0
Compression: COPY
        frag count: 0
        compressed sum: 0
        uncompressed sum: 0
Compression: DYNRUBIN
        frag count: 0
        compressed sum: 0
        uncompressed sum: 0
Compression: ZLIB
        frag count: 32260
        compressed sum: 52613058
        uncompressed sum: 126003035
T2080> jffs2ls
 drwxr-xr-x        0 Wed Mar 13 07:24:22 2019 bin
 drwxr-xr-x        0 Sat Mar 17 13:20:29 2018 boot
 drwxr-xr-x        0 Sat Mar 17 13:20:29 2018 dev
 drwxr-xr-x        0 Wed Mar 13 07:24:27 2019 etc
 drwxr-xr-x        0 Sat Mar 17 13:20:29 2018 home
 drwxr-xr-x        0 Wed Mar 21 07:24:15 2018 lib
 drwxr-xr-x        0 Wed Mar 13 07:24:27 2019 lib64
 lrwxrwxrwx       12 Wed Mar 21 07:23:37 2018 linuxrc -> /bin/busybox
 drwx------        0 Wed Mar 21 07:24:22 2018 lost+found
 drwxr-xr-x        0 Sat Mar 17 13:20:29 2018 media
 drwxr-xr-x        0 Sat Mar 17 13:20:29 2018 mnt
 drwxr-xr-x        0 Sat Mar 17 13:20:29 2018 proc
 drwxr-xr-x        0 Wed Mar 21 07:24:11 2018 root
 drwxr-xr-x        0 Sat Mar 17 13:20:29 2018 run
 drwxr-xr-x        0 Wed Mar 21 07:24:15 2018 sbin
 drwxr-xr-x        0 Sat Mar 17 13:20:29 2018 sys
 drwxrwxrwt        0 Sat Mar 17 13:20:29 2018 tmp
 drwxr-xr-x        0 Sat Mar 17 13:43:18 2018 usr
 drwxr-xr-x        0 Wed Mar 21 07:24:15 2018 var
```
# 制作jffs2镜像
```shell
$ sudo apt install mtd-utils
# 没有网，离线装[mtd-utils](https://launchpad.net/ubuntu/+source/mtd-utils)，
$ sudo dpkg -i /mnt/hgfs/F/mtd-utils_1.5.2-1_amd64.deb
$ mkfs.jffs2 -h
Make a JFFS2 file system image from an existing directory tree

Options:
  -p, --pad[=SIZE]        Pad output to SIZE bytes with 0xFF. If SIZE is
                          not specified, the output is padded to the end of
                          the final erase block
  -r, -d, --root=DIR      Build file system from directory DIR (default: cwd)
  -s, --pagesize=SIZE     Use page size (max data node size) SIZE.
                          Set according to target system's memory management
                          page size (default: 4KiB)
  -e, --eraseblock=SIZE   Use erase block size SIZE (default: 64KiB)
  -c, --cleanmarker=SIZE  Size of cleanmarker (default 12)
  -m, --compr-mode=MODE   Select compression mode (default: priority)
  -x, --disable-compressor=COMPRESSOR_NAME
                          Disable a compressor
  -X, --enable-compressor=COMPRESSOR_NAME
                          Enable a compressor
  -y, --compressor-priority=PRIORITY:COMPRESSOR_NAME
                          Set the priority of a compressor
  -L, --list-compressors  Show the list of the available compressors
  -t, --test-compression  Call decompress and compare with the original (for test)
  -n, --no-cleanmarkers   Don't add a cleanmarker to every eraseblock
  -o, --output=FILE       Output to FILE (default: stdout)
  -l, --little-endian     Create a little-endian filesystem
  -b, --big-endian        Create a big-endian filesystem
  -D, --devtable=FILE     Use the named FILE as a device table file
  -f, --faketime          Change all file times to '0' for regression testing
  -q, --squash            Squash permissions and owners making all files be owned by root
  -U, --squash-uids       Squash owners making all files be owned by root
  -P, --squash-perms      Squash permissions on all files
      --with-xattr        stuff all xattr entries into image
      --with-selinux      stuff only SELinux Labels into jffs2 image
      --with-posix-acl    stuff only POSIX ACL entries into jffs2 image
  -h, --help              Display this help text
  -v, --verbose           Verbose operation
  -V, --version           Display version information
  -i, --incremental=FILE  Parse FILE and generate appendage output for it
$ sumtool -h
sumtool: error!: Usage: sumtool [OPTIONS] -i inputfile -o outputfile

Convert the input JFFS2 image to a summarized JFFS2 image
Summary makes mounting faster - if summary support enabled in your kernel

Options:
  -e, --eraseblock=SIZE     Use erase block size SIZE (default: 64KiB)
                            (usually 16KiB on NAND)
  -c, --cleanmarker=SIZE    Size of cleanmarker (default 12).
                            (usually 16 bytes on NAND, and will be set to
                            this value if left at the default 12). Will be
                            stored in OOB after each physical page composing
                            a physical eraseblock.
  -n, --no-cleanmarkers     Don't add a cleanmarker to every eraseblock
  -o, --output=FILE         Output to FILE 
  -i, --input=FILE          Input from FILE 
  -b, --bigendian           Image is big endian
  -l  --littleendian        Image is little endian
  -h, --help                Display this help text
  -v, --verbose             Verbose operation
  -V, --version             Display version information
  -p, --pad                 Pad the OUTPUT with 0xFF to the end of the final
                            eraseblock
```
其中`-s`指定Page size，这里是系统的Page，不是Flash的Page，
```shell
$ mkfs.jffs2 -n -e 0x20000 -p 0x3200000 -d rootfs/ -o rootfs.jffs2
$ sumtool -n -e 0x20000 -p -i rootfs.jffs2 -o rootfs.jffs2.sum

$ mkfs.jffs2 -b -n -e 0x20000 -p 0x3200000 -d rootfs/ -o rootfs.jffs2
$ sumtool -b -n -e 0x20000 -p -i rootfs.jffs2 -o rootfs.jffs2.sum
```
从jffs2启动，
```shell
bootargs=rootfstype=jffs2 noinitrd root=/dev/mtdblock1 rw console=ttyS0,115200\0
```

# PC挂载jffs2
使用`sudo ./jffs2_mount_mtdram.sh mtd7 ./jffs2 256`，
```bash
$ cat jffs2_mount_mtdram.sh 
#!/bin/bash

## Script to mount jffs2 filesystem using mtd kernel modules.
## EMAC, Inc. 2009

if [[ $# -lt 2 ]]
then
    echo "Usage: $0 FSNAME.JFFS2 MOUNTPOINT [ERASEBLOCK_SIZE]"
    exit 1
fi

if [ "$(whoami)" != "root" ]
then
    echo "$0 must be run as root!"
    exit 1
fi

if [[ ! -e $1 ]]
then
    echo "$1 does not exist"
    exit 1
fi

if [[ ! -d $2 ]]
then
    echo "$2 is not a valid mount point"
    exit 1
fi

if [[ "$3" == "" ]]
then
        esize="128"
else
        esize="$3"
fi

# cleanup if necessary
umount /dev/mtdblock0 &>/dev/null
modprobe -r mtdram &>/dev/null
modprobe -r mtdblock &>/dev/null

modprobe mtdram total_size=32768 erase_size=$esize || exit 1
modprobe mtdblock || exit 1
dd if="$1" of=/dev/mtdblock0 || exit 1
mount -t jffs2 /dev/mtdblock0 $2 || exit 1

echo "Successfully mounted $1 on $2"
exit 0
```

# 问题
操作小文件（两行字符串）之后，重启输出，再次重启，不输出错误，
```shell
jffs2: Empty flash at 0x036f2358 ends at 0x036f239c
jffs2: Empty flash at 0x036f23cc ends at 0x036f2400
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f2404: 0x0600 instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f2408: 0x998a instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f240c: 0x0e7d instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f2410: 0x785e instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f2414: 0x596f instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f2418: 0x0cf6 instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f241c: 0xa31f instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f2420: 0x063b instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f2424: 0xcfad instead
jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x036f2428: 0xb1db instead
jffs2: Further such events for this erase block will not be printed
```
操作大文件（8MB）之后，该问题是我的设备树配置错误。
```shell
root@t2080rdb:/boot# ls -l
jffs2: notice: (1745) jffs2_get_inode_nodes: Node header CRC failed at 0x4878630. {ffff,ffff,ffffffff,ffffffff}
jffs2: notice: (1745) jffs2_get_inode_nodes: Node header CRC failed at 0x485e108. {ffff,ffff,ffffffff,ffffffff}
jffs2: notice: (1745) jffs2_get_inode_nodes: Node header CRC failed at 0x484dcb0. {ffff,ffff,ffffffff,ffffffff}
```
下面问题是mkfs时参数设置错误，
```bash
fs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f6024: 0x858b instead
[   40.543687] jffs2: Further such events for this erase block will not be printed
[   40.554010] jffs2: Node at 0x000f6c80 with length 0x00000b40 would run over the end of the erase block
[   40.566171] jffs2: Perhaps the file system was created with the wrong erase size?
[   40.576693] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f7000: 0x61e3 instead
[   40.589138] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f7004: 0x5ce6 instead
[   40.601580] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f7008: 0x669e instead
[   40.614016] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f700c: 0xad53 instead
[   40.626457] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f7010: 0x05ab instead
[   40.638897] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f7014: 0x0cce instead
[   40.651335] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f7018: 0x26c1 instead
[   40.663772] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f701c: 0x887b instead
[   40.676211] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f7020: 0x5a36 instead
[   40.688603] jffs2: jffs2_scan_eraseblock(): Magic bitmask 0x1985 not found at 0x000f7024: 0xcf46 instead
```
开机打印，
```bash
[    4.849632] jffs2: notice: (1) jffs2_build_xattr_subsystem: complete building xattr subsystem, 0 of xdatum (0 unchecked, 0 orphan) and 0 of xref (0 dead, 0 orphan) found.
```

