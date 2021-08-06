# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> 狗熊王的系列博客[一步一步制作yaffs/yaffs2根文件系统](https://blog.csdn.net/mybelief321/article/details/9995199)
> Xilinx Wiki [Build and Modify a Rootfs](http://www.wiki.xilinx.com/Build+and+Modify+a+Rootfs)
> [chroot，pivot_root和switch_root 区别](https://blog.csdn.net/u012385733/article/details/102565591)
> [pivot_root(8) - Linux man page](https://linux.die.net/man/8/pivot_root)
> [initrd和initramfs及根文件系统切换](https://blog.csdn.net/m0_38096844/article/details/99968912)
> [linux grub 引导启动过程详解](https://blog.csdn.net/sahusoft/article/details/6076371)

# busybox
在xilinx的SOC FPGA平台zynq（arm）上做验证。基于petalinux2015.2.1的根文件系统无法定制busybox，比如`ls`不支持2GB以上的大文件，需要手动重新制作并覆盖原来的一部分命令。
```bash
root@zynq:~# busybox --help
BusyBox v1.23.1 (2015-08-17 18:17:40 IST) multi-call binary.
BusyBox is copyrighted by many authors between 1998-2012.
Licensed under GPLv2. See source distribution for detailed
copyright notices.

Usage: busybox [function [arguments]...]
   or: busybox --list
   or: function [arguments]...

        BusyBox is a multi-call binary that combines many common Unix
        utilities into a single executable.  Most people will create a
        link to busybox for each function they wish to use and BusyBox
        will act like whatever it was invoked as.

Currently defined functions:
        [, [[, addgroup, adduser, ar, ash, awk, basename, cat, chattr, chgrp,
        chmod, chown, chroot, chvt, clear, cmp, cp, cpio, cut, date, dc, dd,
        deallocvt, delgroup, deluser, depmod, devmem, df, diff, dirname, dmesg,
        dnsdomainname, du, dumpkmap, dumpleases, echo, egrep, env, expr, false,
        fatattr, fbset, fdisk, fgrep, find, flock, free, fsck, fstrim, ftpd,
        ftpget, ftpput, fuser, getopt, getty, grep, groups, gunzip, gzip, halt,
        hd, head, hexdump, hostname, httpd, hwclock, id, ifconfig, ifdown,
        ifup, inetd, insmod, ip, kill, killall, klogd, less, ln, loadfont,
        loadkmap, logger, logname, logread, losetup, ls, lsmod, md5sum, mdev,
        mesg, microcom, mkdir, mkdosfs, mkfifo, mkfs.vfat, mknod, mkswap,
        mktemp, modprobe, more, mount, mv, netstat, nice, nohup, nslookup, od,
        openvt, patch, pidof, pivot_root, poweroff, printf, ps, pwd, rdate,
        readlink, realpath, reboot, renice, reset, rm, rmdir, rmmod, route,
        run-parts, sed, seq, setconsole, sh, sha1sum, sha256sum, sha3sum,
        sha512sum, shuf, sleep, sort, start-stop-daemon, stat, strings, stty,
        sulogin, swapoff, swapon, switch_root, sync, sysctl, syslogd, tail,
        tar, tee, telnet, telnetd, test, tftp, time, top, touch, tr, true, tty,
        udhcpc, udhcpd, umount, uname, uniq, unlink, unzip, uptime, users,
        usleep, vi, watch, watchdog, wc, wget, which, who, whoami, xargs, yes,
        zcat
```
解压busybox，
```shell
zc@ubuntu:~/xilinx/app$ tar -jxvf busybox-1.28.3.tar.bz2
```
![这里写图片描述](https://img-blog.csdn.net/20180519194723156?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
编译busybox，
```shell
zc@ubuntu:~/xilinx/app/busybox-1.28.3$ make menuconfig
```
![这里写图片描述](https://img-blog.csdn.net/20180519195004127?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 1. Settings>build options 设置CROSS_COMPILE zynq是arm-linux-gnueabihf-，zynqMP是aarch64-linux-gnu-老版本的编译器zynq为arm-xilinx-linux-gnueabi-
 2. Settings>install options 设置安装路径 /home/zc/xilinx/rootfs/arm64 **此处必须是绝对路径**
 3. [*] Fancy shell prompts  <—要选择这个选项:“Fancy shell prompts”，否则挂载文件系统后，无法正常显示命令提示符：“[\u@\h \W]#” 
 4. [*]vi-style line editing commands网上查资料大家都选了，目前不知为什么，估计是bash输入命令相关的

```shell
zc@ubuntu:~/xilinx/app/busybox-1.28.3$ cp .config zynqMP.config
zc@ubuntu:~/xilinx/app/busybox-1.28.3$ mkdir ~/xilinx/rootfs
zc@ubuntu:~/xilinx/app/busybox-1.28.3$ mkdir ~/xilinx/rootfs/arm64
```
现在准备工作完成，开始编译，执行，
```shell
zc@ubuntu:~/xilinx/app/busybox-1.28.3$ make
zc@ubuntu:~/xilinx/app/busybox-1.28.3$ make install
```
在配置的安装路径下生成busybox安装文件，
![这里写图片描述](https://img-blog.csdn.net/20180519223028179?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
可以看到，现在只有四个文件夹，一个完备的根文件系统不止这些，下面加入lib文件夹，lib下的glibc库可以从工具链的安装目录下找到，
```shell
zc@ubuntu:~/program/petalinux-v2017.4-final/tools/linux-i386/aarch64-linux-gnu/aarch64-linux-gnu/libc/lib$ ls -l
```
![这里写图片描述](https://img-blog.csdn.net/20180520104355576?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
复制库文件，必须的就是这几种，减小文件系统大小，使用`ldd`或者`arm-linux-readelf -a  hello2 | grep "Shared"`查看应用程序需要的动态链接库，按需再添加，
```shell
mkdir arm64/lib
cp -v -a $rootfsarm64lib/ld* arm64/lib
cp -v -a $rootfsarm64lib/libdl* arm64/lib
cp -v -a $rootfsarm64lib/libm-* arm64/lib
cp -v -a $rootfsarm64lib/libpthread* arm64/lib
```
添加`libgcc_s.so.1`，否则无法使用`pthread_cancel`
```shell
libgcc_s.so.1 must be installed for pthread_cancel to work
zynqmp:
cp -avf ~/program/petalinux-v2018.2-final/tools/linux-i386/aarch64-linux-gnu/aarch64-linux-gnu/lib64/libgcc* ~/program/fdk/bsp/zynqmp/package-v2018.2/libso/
zynq:
cp -avf ~/program/petalinux-v2015.2.1-final/tools/linux-i386/arm-xilinx-linux-gnueabi/arm-xilinx-linux-gnueabi/libc/lib/libgcc* ~/program/fdk/bsp/zynq/package-v2015.2.1/libso/
t2080：
cp ~/program/QorIQ-SDK-V2.0-20160527-yocto/fsl-qoriq/2.0/ppc64/sysroots/ppc64e6500-fsl-linux/lib64/libgcc* ~/program/fdk/bsp/t2080/package-v2.0/libso/
```
接下来我们构建etc目录，[etc目录详解](https://blog.csdn.net/awhip9/article/details/66969351)，etc不是什么缩写，是and so on的意思，来源于法语的et cetera，翻译成中文就是等等的意思。至于为什么在/etc下面存放配置文件， 按照原始的UNIX的说法(Linux文件结构参考UNIX的教学实现MINIX) 这下面放的都是一堆零零碎碎的东西，就叫etc，这其实是个历史遗留。
```shell
mkdir arm64/etc
```
etc目录比较复杂参考[1](https://blog.csdn.net/mybelief321/article/details/10027917)，[2](https://blog.csdn.net/mybelief321/article/details/10040939)
构建剩余目录
```shell
mkdir arm64/proc arm64/mnt arm64/tmp arm64/sys arm64/root arm64/home
```

# 打包根文件系统镜像
Petalinux也即yocto得到的是cpio.gz跟文件系统，打算将其部署在emmc中，所以需要制作ext4的镜像，首先解压cpio.gz根文件系统，可以自己添加新文件，
```shell
zc@ubuntu:~/xilinx/image/mwm178$ mkdir rootfs
zc@ubuntu:~/xilinx/image/mwm178$ gunzip -c rootfs.cpio.gz | sh -c 'cd rootfs/ && cpio -i'
25132 blocks
zc@ubuntu:~/xilinx/image/mwm178/rootfs$ cp ../image.ub boot/
zc@ubuntu:~/xilinx/image/mwm178/rootfs$ ls -l
total 60
drwxr-xr-x  2 zc zc 4096 Jul  3 20:58 bin
drwxr-xr-x  2 zc zc 4096 Jul  3 21:00 boot
drwxr-xr-x  2 zc zc 4096 Jul  3 20:58 dev
drwxr-xr-x 22 zc zc 4096 Jul  3 20:58 etc
drwxr-xr-x  3 zc zc 4096 Jul  3 20:58 home
lrwxrwxrwx  1 zc zc   10 Jul  3 20:58 init -> /sbin/init
drwxr-xr-x  4 zc zc 4096 Jul  3 20:58 lib
drwxr-xr-x  2 zc zc 4096 Jul  3 20:58 media
drwxr-xr-x  2 zc zc 4096 Jul  3 20:58 mnt
drwxr-xr-x  2 zc zc 4096 Jul  3 20:58 proc
drwxr-xr-x  2 zc zc 4096 Jul  3 20:58 run
drwxr-xr-x  2 zc zc 4096 Jul  3 20:58 sbin
drwxr-xr-x  2 zc zc 4096 Jul  3 20:58 sys
drwxrwxrwt  2 zc zc 4096 Jul  3 20:58 tmp
drwxr-xr-x 10 zc zc 4096 Jul  3 20:58 usr
drwxr-xr-x  7 zc zc 4096 Jul  3 20:58 var                          
zc@ubuntu:~/xilinx/image/mwm178/rootfs$ cd ..
```
重新打包cpio.gz的命令，
```shell
zc@ubuntu:~/xilinx/image/mwm178$ sh -c 'cd rootfs/ && find . | cpio -H newc -o' | gzip -9 > new_rootfs.cpio.gz
73524 blocks
```
打包成ext4的命令，
```shell
zc@ubuntu:~/xilinx/image/mwm178$ dd if=/dev/zero of=rootfs.ext4 bs=4K count=1572864
1572864+0 records in
1572864+0 records out
6442450944 bytes (6.4 GB, 6.0 GiB) copied, 32.7655 s, 197 MB/s 
zc@ubuntu:~/xilinx/image/mwm178$ mkfs.ext4 -F rootfs.ext4 
mke2fs 1.42.13 (17-May-2015)
Discarding device blocks: done                            
Creating filesystem with 1572864 4k blocks and 393216 inodes
Filesystem UUID: 115ac8d2-7423-4df5-b455-133abf62f1ce
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done 

zc@ubuntu:~/xilinx/image/mwm178$ mkdir rootfs_ext4
zc@ubuntu:~/xilinx/image/mwm178$ sudo mount rootfs.ext4 ./rootfs_ext4/
zc@ubuntu:~/xilinx/image/mwm178$ sudo chmod a+w rootfs_ext4/
zc@ubuntu:~/xilinx/image/mwm178$ cp -arf rootfs/* rootfs_ext4/
zc@ubuntu:~/xilinx/image/mwm178$ ls -l rootfs_ext4/
total 76
drwxr-xr-x  2 zc   zc    4096 Jul  3 20:58 bin
drwxr-xr-x  2 zc   zc    4096 Jul  3 21:00 boot
drwxr-xr-x  2 zc   zc    4096 Jul  3 20:58 dev
drwxr-xr-x 22 zc   zc    4096 Jul  3 20:58 etc
drwxr-xr-x  3 zc   zc    4096 Jul  3 20:58 home
lrwxrwxrwx  1 zc   zc      10 Jul  3 20:58 init -> /sbin/init
drwxr-xr-x  4 zc   zc    4096 Jul  3 20:58 lib
drwx------  2 root root 16384 Jul  3 21:36 lost+found
drwxr-xr-x  2 zc   zc    4096 Jul  3 20:58 media
drwxr-xr-x  2 zc   zc    4096 Jul  3 20:58 mnt
drwxr-xr-x  2 zc   zc    4096 Jul  3 20:58 proc
drwxr-xr-x  2 zc   zc    4096 Jul  3 20:58 run
drwxr-xr-x  2 zc   zc    4096 Jul  3 20:58 sbin
drwxr-xr-x  2 zc   zc    4096 Jul  3 20:58 sys
drwxrwxrwt  2 zc   zc    4096 Jul  3 20:58 tmp
drwxr-xr-x 10 zc   zc    4096 Jul  3 20:58 usr
drwxr-xr-x  7 zc   zc    4096 Jul  3 20:58 var
zc@ubuntu:~/xilinx/image/mwm178$ sudo chmod a-w rootfs_ext4/
zc@ubuntu:~/xilinx/image/mwm178$ sudo umount rootfs_ext4/
zc@ubuntu:~/xilinx/image/mwm178$ ls rootfs_ext4/
zc@ubuntu:~/xilinx/image/mwm178$ ls -l
total 243436
-rwxrwxr-x  1 zc zc        523 Jul  2 00:59 build.sh
-rw-rw-r--  1 zc zc       1423 Jul  2 01:16 fitImage.its
-rw-r--r--  1 zc zc   18915840 Jul  2 05:12 Image
-rw-rw-r--  1 zc zc   24776492 Jul  2 05:14 image.ub
-rw-rw-r--  1 zc zc      27319 Jul  1 23:29 plnx_aarch64-system.dtb
-rw-r--r--  1 zc zc      34575 Jul  2 00:58 plnx_aarch64-system.dts
-rw-rw-r--  1 zc zc      26619 Jul  2 01:09 plnx_aarch64-system-ps-only.dtb
-rw-r--r--  1 zc zc      33492 Jul  2 01:08 plnx_aarch64-system-ps-only.dts
drwxrwxr-x 17 zc zc       4096 Jul  3 21:00 rootfs
-rw-r--r--  1 zc zc    5804313 Jul  2 01:07 rootfs.cpio.gz
drwxrwxr-x  2 zc zc       4096 Jul  3 21:37 rootfs_ext4
-rw-rw-r--  1 zc zc 6442450944 Jul  3 21:47 rootfs.ext4
-rw-rw-r--  1 zc zc   23617775 Jul  3 21:06 rootfs-image.cpio.gz
zc@ubuntu:~/xilinx/image/mwm178$ gzip -9 rootfs.ext4 
zc@ubuntu:~/xilinx/image/mwm178$ ls -l
total 100776
-rwxrwxr-x  1 zc zc      523 Jul  2 00:59 build.sh
-rw-rw-r--  1 zc zc     1423 Jul  2 01:16 fitImage.its
-rw-r--r--  1 zc zc 18915840 Jul  2 05:12 Image
-rw-rw-r--  1 zc zc 24776492 Jul  2 05:14 image.ub
-rw-rw-r--  1 zc zc    27319 Jul  1 23:29 plnx_aarch64-system.dtb
-rw-r--r--  1 zc zc    34575 Jul  2 00:58 plnx_aarch64-system.dts
-rw-rw-r--  1 zc zc    26619 Jul  2 01:09 plnx_aarch64-system-ps-only.dtb
-rw-r--r--  1 zc zc    33492 Jul  2 01:08 plnx_aarch64-system-ps-only.dts
drwxrwxr-x 17 zc zc     4096 Jul  3 21:00 rootfs
-rw-r--r--  1 zc zc  5804313 Jul  2 01:07 rootfs.cpio.gz
drwxrwxr-x  2 zc zc     4096 Jul  3 21:37 rootfs_ext4
-rw-rw-r--  1 zc zc 29920634 Jul  3 21:47 rootfs.ext4.gz
-rw-rw-r--  1 zc zc 23617775 Jul  3 21:06 rootfs-image.cpio.gz
```
压缩完之后只有30MB，解压命令，
```shell
zc@ubuntu:~/xilinx/image/mwm178$ gzip -d rootfs.ext4.gz 
zc@ubuntu:~/xilinx/image/mwm178$ gzip -v9 rootfs.ext4
```
