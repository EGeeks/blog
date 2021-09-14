# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [OpenBMC](https://www.jianshu.com/p/12139db32e49)
> [OpenBMC u-boot基于AST2400](https://blog.csdn.net/z190814412/article/details/89340036)
> [ColorsOfTheWorld OpenBMC专栏](https://blog.csdn.net/z190814412/category_8570218.html)
> [openBMC（todo）](https://www.cnblogs.com/soul-stone/p/7327879.html)
> [IPMI和BMC 通信的过程](https://blog.csdn.net/tiantao2012/article/details/72864286)
> [OpenBmc开发2：构建开发环境](https://blog.csdn.net/qq_34160841/article/details/104841318)
> [OpenBmc开发7：创建新layer（其他方法）](https://blog.csdn.net/qq_34160841/article/details/106203086)
> [BMC学习记录-smartfusion硬件架构](https://blog.csdn.net/zhaoxinfan/article/details/81089093)
> [服务器BMC技术调研](https://www.jianshu.com/p/e18de3800686)
> [使用 IPMI 远程为服务器安装操作系统教程](https://blog.csdn.net/shida_csdn/article/details/86134733)
> [产品知识中心：SOL（Serial Over LAN）](http://server.it168.com/a2009/0930/750/000000750632.shtml)
> [yocto的fetch问题](https://blog.csdn.net/chengbeng1745/article/details/83018238)
> [yocto的文件下载支持介绍](https://blog.csdn.net/groundhappy/article/details/55046166)
> [yocto 离线编译](https://www.cnblogs.com/zqb-all/p/9977924.html)
> [ubuntu下qemu使用：图文详解](https://blog.csdn.net/qq_34160841/article/details/104891169)

# 编译OpenBMC
> 参考OpenBMC文档cheatsheet.md

Git下载OpenBMC，使用`https://`不能用`git://`，
```bash
qe@ubuntu:~/program$ sudo apt-get install -y git build-essential libsdl1.2-dev texinfo gawk chrpath diffstat
qe@ubuntu:~/program$ git clone https://github.com/openbmc/openbmc.git
Cloning into 'openbmc'...
remote: Enumerating objects: 480, done.
remote: Counting objects: 100% (480/480), done.
remote: Compressing objects: 100% (306/306), done.
remote: Total 121605 (delta 242), reused 385 (delta 147), pack-reused 121125
Receiving objects: 100% (121605/121605), 53.98 MiB | 920.00 KiB/s, done.
Resolving deltas: 100% (64453/64453), done.
Checking connectivity... done.
```
查看支持的配置，
```bash
qe@ubuntu:~/program$ cd openbmc/
qe@ubuntu:~/program/openbmc$ find meta-* -name local.conf.sample
meta-aspeed/conf/local.conf.sample
meta-evb/meta-evb-nuvoton/meta-evb-npcm750/conf/local.conf.sample
meta-evb/meta-evb-aspeed/meta-evb-ast2500/conf/local.conf.sample
meta-evb/meta-evb-enclustra/meta-evb-zx3-pm3/conf/local.conf.sample
meta-evb/meta-evb-raspberrypi/conf/local.conf.sample
meta-facebook/meta-tiogapass/conf/local.conf.sample
meta-facebook/meta-yosemitev2/conf/local.conf.sample
meta-hxt/meta-stardragon4800-rep2/conf/local.conf.sample
meta-ibm/meta-romulus/conf/local.conf.sample
meta-ibm/meta-palmetto/conf/local.conf.sample
meta-ibm/meta-witherspoon/conf/local.conf.sample
meta-ingrasys/meta-zaius/conf/local.conf.sample
meta-inspur/meta-fp5280g2/conf/local.conf.sample
meta-inspur/meta-on5263m5/conf/local.conf.sample
meta-intel/meta-s2600wf/conf/local.conf.sample
meta-inventec/meta-lanyang/conf/local.conf.sample
meta-lenovo/meta-hr630/conf/local.conf.sample
meta-lenovo/meta-hr855xg2/conf/local.conf.sample
meta-mellanox/meta-msn/conf/local.conf.sample
meta-microsoft/meta-olympus/conf/local.conf.sample
meta-phosphor/conf/local.conf.sample
meta-portwell/meta-neptune/conf/local.conf.sample
meta-qualcomm/meta-centriq2400-rep/conf/local.conf.sample
meta-quanta/meta-gsj/conf/local.conf.sample
meta-quanta/meta-f0b/conf/local.conf.sample
meta-quanta/meta-olympus-nuvoton/conf/local.conf.sample
meta-quanta/meta-q71l/conf/local.conf.sample
meta-yadro/meta-vesnin/conf/local.conf.sample
meta-yadro/meta-nicole/conf/local.conf.sample
```
选个ast2400的配置测试一下，demo的`meta-ibm/meta-romulus/conf`是ast2500，后面再编译只需执行`. openbmc-env`即可，如果想换一个编译配置，删除原来的配置文件`build/conf`再`export`，
```bash
qe@ubuntu:~/program/openbmc$ export TEMPLATECONF=meta-ibm/meta-palmetto/conf
qe@ubuntu:~/program/openbmc$ . openbmc-env
### Initializing OE build env ###
You had no conf/local.conf file. This configuration file has therefore been
created for you with some default values. You may wish to edit it to, for
example, select a different MACHINE (target hardware). See conf/local.conf
for more information as common configuration options are commented.

You had no conf/bblayers.conf file. This configuration file has therefore been
created for you with some default values. To add additional metadata layers
into your configuration please add entries to conf/bblayers.conf.

The Yocto Project has extensive documentation about OE including a reference
manual which can be found at:
    http://yoctoproject.org/documentation

For more information about OpenEmbedded see their website:
    http://www.openembedded.org/

Common targets are:
     obmc-phosphor-image
```
生成conf文件夹，这时可以修改配置`local.conf`，
```bash
#DL_DIR ?= "${TOPDIR}/downloads"
DL_DIR = "/opt/openbmc/downloads"

BB_NUMBER_THREADS = ' 8 '
PARALLEL_MAKE     = ' -j 8 '
```
其他参数，
```bash
DL_DIR ——存放编译过程中下载后的数据，
BB_NUMBER_THREADS ——同时工作的最大任务数，一般给cpu核心数的两倍，我CPU核心数是4，故设置为8
PARALLEL_MAKE——每个任务使用的线程数，应该包含"-j"，如果希望8个线程一起运行，则设置为"-j 8"
BB_GENERATE_MIRROR_TARBALLS——在DL_DIR中产生源代码控制库（比如 GIT），包含元数据的tarball
INHERIT += “rm_work” ——命令BitBake在构建完包之后删除针对构建包的工作目录
RM_WORK_EXCLUDE += " core-image_minimal"  ——排除要被删除的对象
SSTATE_DIR = " "——存放共享状态缓存位置。
BB_NO_NETWORK = "1" ——如果你的环境不能联网需要此配置
```
编译，出错了之后重新执行这个命令，如果还是无法编译，需要仔细查阅log文件。
```bash
qe@ubuntu:~/program/openbmc/build$ bitbake obmc-phosphor-image
```
yocto在编译的时候会自动下载软件包，可以先到外网机器上下载好再编译，只下载不编译的命令，
```
qe@ubuntu:~/program/openbmc/build$ bitbake obmc-phosphor-image -c fetch
```
编译完成后，镜像在，
```bash
qe@qe-pc:/opt/openbmc/openbmc/build$ ls -l tmp/deploy/images/palmetto/
总用量 115572
-rw-r--r-- 2 qe qe    28332 9月   7 00:10 aspeed-bmc-opp-palmetto--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.dtb
lrwxrwxrwx 2 qe qe       77 9月   7 00:10 aspeed-bmc-opp-palmetto.dtb -> aspeed-bmc-opp-palmetto--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.dtb
lrwxrwxrwx 2 qe qe       77 9月   7 00:10 aspeed-bmc-opp-palmetto-palmetto.dtb -> aspeed-bmc-opp-palmetto--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.dtb
lrwxrwxrwx 2 qe qe       62 9月   7 00:10 fitImage -> fitImage--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.bin
-rw-r--r-- 2 qe qe  2649308 9月   7 00:10 fitImage--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.bin
-rw-r--r-- 2 qe qe     1609 9月   7 00:10 fitImage-its--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.its
-rw-r--r-- 2 qe qe     2214 9月   7 00:10 fitImage-its-obmc-phosphor-initramfs-palmetto--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.its
lrwxrwxrwx 2 qe qe       99 9月   7 00:10 fitImage-its-obmc-phosphor-initramfs-palmetto-palmetto -> fitImage-its-obmc-phosphor-initramfs-palmetto--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.its
lrwxrwxrwx 2 qe qe       66 9月   7 00:10 fitImage-its-palmetto -> fitImage-its--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.its
-rw-r--r-- 2 qe qe  2618976 9月   7 00:10 fitImage-linux.bin--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.bin
lrwxrwxrwx 2 qe qe       72 9月   7 00:10 fitImage-linux.bin-palmetto -> fitImage-linux.bin--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.bin
-rw-r--r-- 2 qe qe  3768476 9月   7 00:10 fitImage-obmc-phosphor-initramfs-palmetto--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.bin
lrwxrwxrwx 2 qe qe       95 9月   7 00:10 fitImage-obmc-phosphor-initramfs-palmetto-palmetto -> fitImage-obmc-phosphor-initramfs-palmetto--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.bin
lrwxrwxrwx 2 qe qe       62 9月   7 00:10 fitImage-palmetto.bin -> fitImage--5.8.5+git0+0dd0c8c492-r0-palmetto-20200906160647.bin
lrwxrwxrwx 2 qe qe       54 9月   7 00:11 flash-palmetto -> obmc-phosphor-image-palmetto-20200906160647.static.mtd
lrwxrwxrwx 2 qe qe       54 9月   7 00:11 image-bmc -> obmc-phosphor-image-palmetto-20200906160647.static.mtd
lrwxrwxrwx 2 qe qe       50 9月   7 00:11 image-kernel -> fitImage-obmc-phosphor-initramfs-palmetto-palmetto
lrwxrwxrwx 2 qe qe       40 9月   7 00:11 image-rofs -> obmc-phosphor-image-palmetto.squashfs-xz
lrwxrwxrwx 2 qe qe       34 9月   7 00:11 image-rwfs -> obmc-phosphor-image-palmetto.jffs2
-rw-r--r-- 2 qe qe   236420 9月   7 00:11 image-u-boot
-rw-r--r-- 2 qe qe     8468 9月   7 00:11 obmc-phosphor-image-palmetto-20200906160647.rootfs.manifest
-rw-r--r-- 2 qe qe 17895424 9月   7 00:11 obmc-phosphor-image-palmetto-20200906160647.rootfs.squashfs-xz
-rw-r--r-- 2 qe qe 33554432 9月   7 00:11 obmc-phosphor-image-palmetto-20200906160647.static.mtd
-rw-r--r-- 2 qe qe 33566720 9月   7 00:11 obmc-phosphor-image-palmetto-20200906160647.static.mtd.all.tar
-rw-r--r-- 2 qe qe 21913600 9月   7 00:11 obmc-phosphor-image-palmetto-20200906160647.static.mtd.tar
-rw-r--r-- 2 qe qe   332086 9月   7 00:11 obmc-phosphor-image-palmetto-20200906160647.testdata.json
-rw-r--r-- 2 qe qe        0 9月   7 00:11 obmc-phosphor-image-palmetto.jffs2
lrwxrwxrwx 2 qe qe       59 9月   7 00:11 obmc-phosphor-image-palmetto.manifest -> obmc-phosphor-image-palmetto-20200906160647.rootfs.manifest
lrwxrwxrwx 2 qe qe       62 9月   7 00:11 obmc-phosphor-image-palmetto.squashfs-xz -> obmc-phosphor-image-palmetto-20200906160647.rootfs.squashfs-xz
lrwxrwxrwx 2 qe qe       54 9月   7 00:11 obmc-phosphor-image-palmetto.static.mtd -> obmc-phosphor-image-palmetto-20200906160647.static.mtd
lrwxrwxrwx 2 qe qe       62 9月   7 00:11 obmc-phosphor-image-palmetto.static.mtd.all.tar -> obmc-phosphor-image-palmetto-20200906160647.static.mtd.all.tar
lrwxrwxrwx 2 qe qe       58 9月   7 00:11 obmc-phosphor-image-palmetto.static.mtd.tar -> obmc-phosphor-image-palmetto-20200906160647.static.mtd.tar
lrwxrwxrwx 2 qe qe       57 9月   7 00:11 obmc-phosphor-image-palmetto.testdata.json -> obmc-phosphor-image-palmetto-20200906160647.testdata.json
-rw-r--r-- 2 qe qe  1118948 9月   7 00:10 obmc-phosphor-initramfs-palmetto-20200906160647.rootfs.cpio.xz
-rw-r--r-- 2 qe qe      160 9月   7 00:10 obmc-phosphor-initramfs-palmetto-20200906160647.rootfs.manifest
-rw-r--r-- 2 qe qe   326529 9月   7 00:10 obmc-phosphor-initramfs-palmetto-20200906160647.testdata.json
lrwxrwxrwx 2 qe qe       62 9月   7 00:10 obmc-phosphor-initramfs-palmetto.cpio.xz -> obmc-phosphor-initramfs-palmetto-20200906160647.rootfs.cpio.xz
lrwxrwxrwx 2 qe qe       63 9月   7 00:10 obmc-phosphor-initramfs-palmetto.manifest -> obmc-phosphor-initramfs-palmetto-20200906160647.rootfs.manifest
lrwxrwxrwx 2 qe qe       61 9月   7 00:10 obmc-phosphor-initramfs-palmetto.testdata.json -> obmc-phosphor-initramfs-palmetto-20200906160647.testdata.json
lrwxrwxrwx 2 qe qe       62 9月   7 00:11 palmetto-20200906160647.all.tar -> obmc-phosphor-image-palmetto-20200906160647.static.mtd.all.tar
lrwxrwxrwx 2 qe qe       58 9月   7 00:11 palmetto-20200906160647.tar -> obmc-phosphor-image-palmetto-20200906160647.static.mtd.tar
lrwxrwxrwx 2 qe qe       53 9月   4 17:12 u-boot.bin -> u-boot-palmetto-v2016.07+gitAUTOINC+1ded9fa3a2-r0.bin
lrwxrwxrwx 2 qe qe       53 9月   4 17:12 u-boot-palmetto.bin -> u-boot-palmetto-v2016.07+gitAUTOINC+1ded9fa3a2-r0.bin
-rw-r--r-- 2 qe qe   236420 9月   4 17:12 u-boot-palmetto-v2016.07+gitAUTOINC+1ded9fa3a2-r0.bin
```

# qemu
> cheatsheet.md
> development/dev-environment.md

有三种方式安装qemu，使用软件源，
```bash
$ sudo apt install -y qemu
```
从OpenBMC官方github下载源码编译安装，需要安装`libpixman-1-dev`，编译生成的`qemu-system-arm`在路径`build/arm-softmmu`中，
```bash
$ git clone https://github.com/openbmc/qemu.git
$ qemu/
$ mkdir build
$ cd build/
$ ../configure --target-list=arm-softmmu
$ sudo apt install libpixman-1-dev
$ ../configure --target-list=arm-softmmu
$ make
```
从官网下载编译好的`qemu-system-arm`，
```bash
wget https://openpower.xyz/job/openbmc-qemu-build-merge-x86/lastSuccessfulBuild/artifact/qemu/arm-softmmu/qemu-system-arm
```
启动OpenBMC镜像，`root`登录密码`0penBmc`，按`Ctrl + a`再按`x`关闭qemu，
```bash
$ ./qemu-system-arm -m 256 -M palmetto-bmc -nographic -drive file=/opt/openbmc/openbmc/build/tmp/deploy/images/palmetto/obmc-phosphor-image-palmetto.static.mtd,format=raw,if=mtd -net nic -net user,hostfwd=:127.0.0.1:2222-:22,hostfwd=:127.0.0.1:2443-:443,hostname=qemu
```
ssh登录，
```bash
$ ssh -p 2222 root@127.0.0.1
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
RSA key fingerprint is SHA256:OVWKjpLMte28ET+IoVdKU/KDW2AZVVXWvTp6uO6NfzE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:2222' (RSA) to the list of known hosts.
root@127.0.0.1's password: 
```

# bmcweb
登录bmcweb，访问`https://localhost:2443`，如果是实际硬件则是`433`端口，忽略危险并继续，`root`登录密码`0penBmc`，
![2020-09-14 20-36-20屏幕截图](https://img-blog.csdnimg.cn/20200914204225586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
# 问题
## 下载软件包版本变化
之前下载的软件包是linux-5.4，过了两天downloads的缓存不能用了，发现刚好是OpenBMC的github更新了，导致下载的软件包是linux-5.8。

## 手动下载软件包
downloads的缓存不能用了，原先的是gcc-10.1.0，现在切换到gcc-10.2.0，70MB的包下载速度3KB/s，这就没法玩了，手动下载[gcc-10.2.0.tar.xz](http://mirrors.nju.edu.cn/gnu/gcc/gcc-10.2.0/)，放到缓存目录中，查看其他done文件，done文件由md5，sha等值组成，写个脚本来生成done文件。
```bash
qe@qe-pc:/opt/openbmc/downloads$ cat ~/gen_done.sh 
#!/bin/sh

src_file=$1
done_file=$2

echo -e -n "\x80\x02\x7d" > $done_file
echo -e -n "\x71\x00\x28\x58\x03\x00\x00\x00" >> $done_file
echo -e -n "md5" >> $done_file # 6d 64 35
echo -e -n "\x71\x01\x58\x20\x00\x00\x00" >> $done_file
str=$(md5sum $src_file)
str=${str%% *}
echo $str
echo -n $str >> $done_file
echo -e -n "\x71\x02\x58\x06\x00\x00\x00" >> $done_file
echo -e -n "sha512" >> $done_file # 73 68 61 35 31 32
echo -e -n "\x71\x03\x58\x80\x00\x00\x00" >> $done_file
str=$(sha512sum $src_file)
str=${str%% *}
echo $str
echo -n $str >> $done_file
echo -e -n "\x71\x04\x58\x06\x00\x00\x00" >> $done_file
echo -e -n "sha256" >> $done_file # 73 68 61 32 35 36
echo -e -n "\x71\x05\x58\x40\x00\x00\x00" >> $done_file
str=$(sha256sum $src_file)
str=${str%% *}
echo $str
echo -n $str >> $done_file
echo -e -n "\x71\x06\x58\x04\x00\x00\x00" >> $done_file
echo -e -n "sha1" >> $done_file # 73 68 61 31
echo -e -n "\x71\x07\x58\x28\x00\x00\x00" >> $done_file
str=$(sha1sum $src_file)
str=${str%% *}
echo $str
echo -n $str >> $done_file
echo -e -n "\x71\x08\x58\x06\x00\x00\x00" >> $done_file
echo -e -n "sha384" >> $done_file # 73 68 61 33 38 34
echo -e -n "\x71\x09\x58\x60\x00\x00\x00" >> $done_file
str=$(sha384sum $src_file)
str=${str%% *}
echo $str
echo -n $str >> $done_file
echo -e -n "\x71\x0a\x75\x2e" >> $done_file
```

# 编译OpenBMC文档
```bash
qe@ubuntu:~/program$ git clone https://github.com/openbmc/docs.git
Cloning into 'docs'...
remote: Enumerating objects: 1418, done.
remote: Total 1418 (delta 0), reused 0 (delta 0), pack-reused 1418
Receiving objects: 100% (1418/1418), 694.88 KiB | 362.00 KiB/s, done.
Resolving deltas: 100% (823/823), done.
Checking connectivity... done.qe@ubuntu:~/program/docs$ sudo apt-get install pandoc texlive-xetex
qe@ubuntu:~/program/docs$ fc-list #查看安装的字体
qe@ubuntu:~/program/docs$ cat ./userguide/userguide.tex 
\documentclass[]{article}

% fonts: libertine for roman text, inconsolata for monospace
\usepackage{fontspec}
% \setmainfont{Linux Libertine O}
\setmainfont{Ubuntu Condensed}
\setmonofont{Ubuntu Mono}
...
qe@ubuntu:~/program/docs$ make
```


# Facebook OpenBMC
| board | machine      |
|:--------|:-------------|
| Wedge | ast1250 |
| Wedge100 | ast1250 |
| Wedge400 | ast2520 |
| Yosemite | ast1250 |
| lightning | ast1250 |
| yamp | ast2520 |


