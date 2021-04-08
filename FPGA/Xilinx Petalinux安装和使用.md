# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [libgcc-xilinx和libgcc-xilinx-dev两个库是为了添加libgcc_s.so.1，来使用pthread_exit()函数](https://blog.csdn.net/songkai320/article/details/70317948)
> [开发者分享 | 如何给 u-boot 的源码生成 patch 并在 Petalinux 中编译](https://mp.weixin.qq.com/s/T1Y7mQV8UmYcrj5SeP_-1Q)

# Petalinux
安装依赖，
```bash
$ sudo apt install ssh make tftp-hpa tftpd-hpa dos2unix iproute2 gawk xvfb git make net-tools libncurses5-dev zlib1g-dev libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev screen pax gzip zlib1g:i386 minicom u-boot-tools mtd-utils
```
安装，
```bash
$ mkdir -p /opt/Xilinx/Petalinux/2019.2
$ ./petalinux-v2019.2-final-installer.run /opt/Xilinx/Petalinux/2019.2
$ sudo dpkg-reconfigure dash #选择no
```
建立工程，可以创建空白工程，也可以从bsp创建工程，
```bash
# 空白工程
$ petalinux-create -t project -n zynq-v2018.2 --template zynq
$ cp ~/project/fdk/pcierc-zc706/pcierc_wrapper.hdf ./zynq-v2018.2
$ cd zynq-v2018.2
$ petalinux-config --get-hw-description=.
# 从bsp创建工程
$ petalinux-create -t project -s ./zynq-v2018.2.bsp
```
系统配置，可配置生成的固件这些参数，Yocto相关参数配置都在这里，
```bash
$ petalinux-config
```
配置rootfs，
```bash
$ petalinux-config -c rootfs
```
配置内核，发现默认的内核路径为`build/tmp/work/plnx_zynq7-xilinx-linux-gnueabi/linux-xlnx/4.14-xilinx-v2018.2+gitAUTOINC+ad4cd988ba-r0/linux-plnx_zynq7-standard-build`，
```bash
$ petalinux-config -c kernel
```
编译，
```bash
$ petalinux-build
```
生成SDK，默认的SDK安装包在`<proj_proot>/images/linux/sdk.sh`，通过在rootfs中加入`packagegroup-petalinux-qt`，`packagegroup-petalinux-opencv`，可以制作出支持Qt和OpenCV的交叉编译工具链。
```bash
$ petalinux-build --sdk
```
安装SDK，默认的SDK安装路径在`<proj_proot>/images/linux/sdk/`，
```bash
$ petalinux-package --sysroot
PetaLinux SDK installer version 2018.2
======================================
You are about to install the SDK to "/home/qe/project/petalinux/zynq-v2018.2/images/linux/sdk". Proceed[Y/n]? Y
Extracting SDK................................done
Setting it up...done
SDK has been successfully set up and is ready to be used.
Each time you wish to use the SDK in a new shell session, you need to source the environment setup script e.g.
 $ . /home/qe/project/petalinux/zynq-v2018.2/images/linux/sdk/environment-setup-cortexa9hf-neon-xilinx-linux-gnueabi
```
使用Yocto工具的步骤，新建工程后需要使用`petalinux-config`或`petalinux-config --oldconfig`，完成基本环境创建。
```bash
$ source ~/program/petalinux-v2018.2-final/components/yocto/source/arm/environment-setup-cortexa9hf-neon-xilinx-linux-gnueabi 
SDK environment now set up; additionally you may now run devtool to perform development tasks.
Run devtool --help for further details.
qe@ubuntu:~/zynq-v2018.2$ source ~/program/petalinux-v2018.2-final/components/yocto/source/arm/layers/core/oe-init-build-env 

### Shell environment set up for builds. ###

You can now run 'bitbake <target>'

Common targets are:
    core-image-minimal
    core-image-sato
    meta-toolchain
    meta-ide-support

You can also run generated qemu images with a command like 'runqemu qemux86'
qe@ubuntu:~/zynq-v2018.2/build$ export PATH=~/program/petalinux-v2018.2-final/tools/hsm/bin:$PATH
qe@ubuntu:~/zynq-v2018.2/build$ export BB_ENV_EXTRAWHITE="BB_ENV_EXTRAWHITE PETALINUX"
# 测试bitbake命令
qe@ubuntu:~/project/petalinux/zynq-v2018.2/build$ bitbake strace
Loading cache: 100% |############################################| Time: 0:00:00
Loaded 3423 entries from dependency cache.
Parsing recipes: 100% |##########################################| Time: 0:00:03
Parsing of 2552 .bb files complete (2517 cached, 35 parsed). 3425 targets, 148 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
Initialising tasks: 100% |#######################################| Time: 0:00:03
Checking sstate mirror object availability: 100% |###############| Time: 0:02:33
NOTE: Executing SetScene Tasks
NOTE: Executing RunQueue Tasks
NOTE: Tasks Summary: Attempted 2206 tasks of which 2181 didn't need to be rerun and all succeeded.
```
下面就可以愉快的使用yocto的工具来开发了，修改内核`/home/qe/program/petalinux-v2018.2-final/components/yocto/source/arm/workspace/sources/linux-xlnx`测试一下，报错，找不到`openamp.scc`，但是这个文件在默认的内核路径为`build/tmp/work/plnx_zynq7-xilinx-linux-gnueabi/linux-xlnx/4.14-xilinx-v2018.2+gitAUTOINC+ad4cd988ba-r0`确实存在。进入这个目录来测试，还是不行，而且`petalinux-config -c kernel`也不能用了。把`recipe` reset之后`petalinux-config -c kernel`可以用了，所以，尝试在petalinux使用外部源代码来解决这个问题吧，当然需要`make mrproper`一下，刚看了官方好像是`bitbake virtual/kernel -c menuconfig`，算了，不测试了。
```bash
qe@ubuntu:~/zynq-v2018.2/build$ bitbake linux-xlnx -c menuconfig
Loading cache: 100% |########################################################################################################################################################################| Time: 0:00:01
Loaded 3423 entries from dependency cache.
Parsing recipes: 100% |######################################################################################################################################################################| Time: 0:00:04
Parsing of 2552 .bb files complete (2516 cached, 36 parsed). 3425 targets, 148 skipped, 0 masked, 0 errors.
NOTE: There are 1 recipes to be removed from sysroot plnx-zynq7, removing...
NOTE: Resolving any missing task queue dependencies
Initialising tasks: 100% |###################################################################################################################################################################| Time: 0:00:06
Checking sstate mirror object availability: 100% |###########################################################################################################################################| Time: 0:00:01
NOTE: Executing SetScene Tasks
NOTE: Executing RunQueue Tasks
ERROR: linux-xlnx-4.14-xilinx-v2018.2+git999-r0 do_kernel_metadata: Could not generate configuration queue for plnx-zynq7.
ERROR: linux-xlnx-4.14-xilinx-v2018.2+git999-r0 do_kernel_metadata: Function failed: do_kernel_metadata (log file is located at /home/qe/project/petalinux/zynq-v2018.2/build/tmp/work/plnx_zynq7-xilinx-linux-gnueabi/linux-xlnx/4.14-xilinx-v2018.2+git999-r0/temp/log.do_kernel_metadata.35873)
ERROR: Logfile of failure stored in: /home/qe/project/petalinux/zynq-v2018.2/build/tmp/work/plnx_zynq7-xilinx-linux-gnueabi/linux-xlnx/4.14-xilinx-v2018.2+git999-r0/temp/log.do_kernel_metadata.35873
Log data follows:
| DEBUG: Executing python function extend_recipe_sysroot
| NOTE: Direct dependencies are ['/home/qe/program/petalinux-v2018.2-final/components/yocto/source/arm/layers/core/meta/recipes-kernel/kern-tools/kern-tools-native_git.bb:do_populate_sysroot']
| NOTE: Installed into sysroot: ['kern-tools-native']
| NOTE: Skipping as already exists in sysroot: ['quilt-native']
| DEBUG: Python function extend_recipe_sysroot finished
| DEBUG: Executing shell function do_kernel_metadata
| ERROR: could not find kconf openamp.cfg, included from /home/qe/program/petalinux-v2018.2-final/components/yocto/source/arm/workspace/sources/linux-xlnx/oe-local-files/openamp.scc
| ERROR: could not process input files: /home/qe/project/petalinux/zynq-v2018.2/build/tmp/work/plnx_zynq7-xilinx-linux-gnueabi/linux-xlnx/4.14-xilinx-v2018.2+git999-r0/defconfig /home/qe/program/petalinux-v2018.2-final/components/yocto/source/arm/workspace/sources/linux-xlnx/oe-local-files/openamp.scc /home/qe/program/petalinux-v2018.2-final/components/yocto/source/arm/workspace/sources/linux-xlnx/oe-local-files/plnx_kernel.cfg /home/qe/program/petalinux-v2018.2-final/components/yocto/source/arm/workspace/sources/linux-xlnx/oe-local-files/user_2020-11-27-10-32-00.cfg
|        See /tmp/tmp.tGsI3j1qT6 for details
| ERROR: Could not generate configuration queue for plnx-zynq7.
| WARNING: /home/qe/project/petalinux/zynq-v2018.2/build/tmp/work/plnx_zynq7-xilinx-linux-gnueabi/linux-xlnx/4.14-xilinx-v2018.2+git999-r0/temp/run.do_kernel_metadata.35873:1 exit 1 from 'exit 1'
| ERROR: Function failed: do_kernel_metadata (log file is located at /home/qe/project/petalinux/zynq-v2018.2/build/tmp/work/plnx_zynq7-xilinx-linux-gnueabi/linux-xlnx/4.14-xilinx-v2018.2+git999-r0/temp/log.do_kernel_metadata.35873)
ERROR: Task (/home/qe/program/petalinux-v2018.2-final/components/yocto/source/arm/layers/meta-xilinx/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2018.2.bb:do_kernel_metadata) failed with exit code '1'
NOTE: Tasks Summary: Attempted 346 tasks of which 342 didn't need to be rerun and 1 failed.

Summary: 1 task failed:
  /home/qe/program/petalinux-v2018.2-final/components/yocto/source/arm/layers/meta-xilinx/meta-xilinx-bsp/recipes-kernel/linux/linux-xlnx_2018.2.bb:do_kernel_metadata
Summary: There were 2 ERROR messages shown, returning a non-zero exit code.
```
添加自己的固件文件到根文件系统，注意这个时候和APP不一样，你的`FILES_${PN}`一定要写，
```bash
$ petalinux-create -t apps --template install --name myfw --enable
```
petalinux-build卡住在source bitbake很久都没动，执行一下clean才能编，
```bash
petalinux-build -x distclean
```
创建BOOT.BIN，
```bash
# $ petalinux-package --boot --fsbl <FSBL_ELF> --fpga <BITSTREAM> --u-boot --pmufw <PMUFW_ELF>
#   It will generate a BOOT.BIN in your working directory with:
#     * specified <BITSTREAM>
#     * specified <FSBL_ELF>
#     * specified < PMUFW_ELF > *
#     * newly built u-boot image which is <PROJECT>/images/linux/u-boot.elf

# 3合1
$ petalinux-package --boot --fpga images/linux/pcierc_wrapper_gen2_x4.bit --u-boot --force
# 2合1
$ petalinux-package --boot --u-boot --force
```
配置自动登录，
```bash
# Select Yocto-settings > Enable debug-tweaks
$ petalinux-config
```
添加自启动程序，采用`install`类型的模板，只需要把自己的应用命名为`myapp`部署到`/usr/bin`就可以了，而petalinux的`${bindir}`就代表`/usr/bin`，
```bash
$ petalinux-create -t apps --template install -n myfw-init --enable
$ cat project-spec/meta-user/recipes-apps/myfw-init/myfw-init.bb
#
# This file is the myfw-init recipe.
#

SUMMARY = "Simple myfw-init application"
SECTION = "PETALINUX/apps"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://myfw-init \
	"

S = "${WORKDIR}"

inherit update-rc.d
INITSCRIPT_NAME = "myfw-init"
INITSCRIPT_PARAMS = "start 99 S ."

do_install() {
        install -d ${D}${sysconfdir}/init.d
        install -m 0755 ${S}/myfw-init ${D}${sysconfdir}/init.d/myfw-init
}
FILES_${PN} += "${sysconfdir}/*"
$ cat project-spec/meta-user/recipes-apps/myfw-init/files/myfw-init 
#!/bin/sh

DAEMON=/usr/bin/myapp
start ()
{
        echo " Starting myapp"
        start-stop-daemon -S -o --background -x $DAEMON
}
stop ()
{
        echo " Stoping myapp"
        start-stop-daemon -K -x $DAEMON
}
restart()
{
        stop
        start
}
[ -e $DAEMON ] || exit 1
        case "$1" in
                start)
                        start; ;;
                stop)
                        stop; ;;
                restart)
                        restart; ;;
                *)
                        echo "Usage: $0 {start|stop|restart}"
                        exit 1
        esac
exit $?
```
删除自己的应用，直接删除应用的文件夹，然后注释掉`IMAGE_INSTALL_append= " ***"`，
```bash
# <plnx-proj-root>/project-spec/meta-user/recipes-core/images/petalinux-image-full.bbappend
IMAGE_INSTALL_append= " ***"
```
发布bsp，
```bash
$ petalinux-package --bsp -o zynq-v2018.2.bsp -p .
$ petalinux-package --bsp -o zynq-v2018.2.bsp -p . --force
```
连着vivado工程一起打包，`clean`表示把vivado工程清除后再发布，减少文件体积，
```bash
$ petalinux-package --bsp -o zynq-v2018.2.bsp -p . --hwsource <PATH_TO_HARDWARE_PROJECT> --force
$ petalinux-package --bsp -o zynq-v2018.2.bsp -p . --hwsource <PATH_TO_HARDWARE_PROJECT> --force --clean
```

# rootfs配置
petalinux-v2018.2，开发用到的常用工具，使能如下选项，`libgcc-xilinx`和`libgcc-xilinx-dev`这两个软件包找不到。
```shell
Filesystem Packages  → base  → util-linux  util-linux-mkfs
Filesystem Packages  → base  → e2fsprogs 
	[*] e2fsprogs                  
	[ ] e2fsprogs-dev                                                                                                     
	[*] e2fsprogs-mke2fs                                                                                                  
	[ ] e2fsprogs-dbg                                                                                                     
	[*] e2fsprogs-resize2fs                                                                                               
	[*] e2fsprogs-tune2fs                                                                                                 
	[ ] libss                                                                                                             
	[ ] libcomerr                                                                                                         
	[ ] libext2fs                                                                                                         
	[ ] libe2p                                                                                                            
	[*] e2fsprogs-e2fsck                                                                                                  
	[*] e2fsprogs-badblocks    
Filesystem Packages  → base  → usbutils
Filesystem Packages  → base  → i2c-tools
Filesystem Packages  → net  → netcat
Filesystem Packages  → console  → network  → ethtool

```
关闭
```shell
Filesystem Packages  → misc  → tcf-agent 
```
默认已有
```shell
cantools pciutils microcom
```
# 增加软件包
petalinux采用yocto来制作跟文件系统，比如iperf3，在petalinux rootfs的menuconfig中是没有的，需要手动配置，iperf3 recipe位置在，
```shell
zc@ubuntu:~/program/petalinux-v2018.2-final/components/yocto/source/aarch64/layers/meta-openembedded/meta-oe/recipes-benchmark/iperf3$ ls -l
total 8
drwxr-xr-x 2 zc zc 4096 Jun  8  2018 iperf3
-rw-r--r-- 1 zc zc 1171 Jun  8  2018 iperf3_3.2.bb
```
在工程的meta-user里添加该软件包，双引号里记得加个空格，
```shell
zc@ubuntu:~/project/petalinux/zynqmp-v2018.2/project-spec/meta-user/recipes-core/images$ cat petalinux-image.bbappend 
#Note: Mention Each package in individual line
#      cascaded representation with line breaks are not valid in this file.
IMAGE_INSTALL_append = " peekpoke"
IMAGE_INSTALL_append = " gpio-demo"
IMAGE_INSTALL_append = " iperf3"
```
使能该软件包，选中iperf3，
```shell
zc@ubuntu:~/project/petalinux/zynqmp-v2018.2$ petalinux-config -c rootfs
```
编译，
```shell
zc@ubuntu:~/project/petalinux/zynqmp-v2018.2$ petalinux-build
```
对于petalinux-image-full中的recipes，有sstate locked，在project-spec/meta-user/conf/petalinuxbsp.conf文件中添加SIGGENE_UNLOCKED_RECIPES += "my-recipe"来unlock。
```shell
zc@ubuntu:~/project/petalinux/zynqmp-v2018.2$ cat project-spec/meta-user/conf/petalinuxbsp.conf 
#User Configuration

#OE_TERMINAL = "tmux"

# Add EXTRA_IMAGEDEPENDS default components
EXTRA_IMAGEDEPENDS_append_zynqmp = " virtual/fsbl virtual/pmu-firmware arm-trusted-firmware"
EXTRA_IMAGEDEPENDS_append_zynq = " virtual/fsbl"
EXTRA_IMAGEDEPENDS_append_microblaze = " virtual/fsboot virtual/elfrealloc"


#Remove all qemu contents
IMAGE_CLASSES_remove = "image-types-xilinx-qemu qemuboot-xilinx"
IMAGE_FSTYPES_remove = "wic.qemu-sd"

EXTRA_IMAGEDEPENDS_remove = "qemu-helper-native virtual/boot-bin"
```

# Petalinux生成的Makefile
petalinux 2015.2.1驱动Makefile，可参考学习一下，
```shell
# 
# Makefile template for out of tree kernel modules
#

# PetaLinux-related stuff
ifndef PETALINUX
$(error You must source the petalinux/settings.sh script before working with PetaLinux)
endif

-include modules.common.mk

ccflags-y += -Wno-error=date-time

KERNEL_BUILD:=$(PROOT)/build/$(LINUX_KERNEL)

LOCALPWD=$(shell pwd)
obj-m += pcie_sata_ep.o

all: build modules install

build:modules

.PHONY: build clean modules

clean:
	make INSTANCE=$(LINUX_KERNEL) -C $(KERNEL_BUILD) M=$(LOCALPWD) clean

modules:
	if [ ! -f "$(PROOT)/build/$(LINUX_KERNEL)/link-to-kernel-build/Module.symvers" ]; then \
		echo "ERROR: Failed to build module ${INSTANCE} because kernel hasn't been built."; \
		echo "ERROR: Please build kernel with petalinux-build -c kernel first."; \
		exit 255; \
	else \
		make INSTANCE=$(LINUX_KERNEL) -C $(KERNEL_BUILD) M=$(LOCALPWD) modules_only; \
	fi

install: $(addprefix $(DIR),$(subst .o,.ko,$(obj-m)))
	if [ ! -f "$(PROOT)/build/$(LINUX_KERNEL)/link-to-kernel-build/Module.symvers" ]; then \
		echo "ERROR: Failed to install module ${INSTANCE} because kernel hasn't been built."; \
		echo "ERROR: Please build kernel with petalinux-build -c kernel first."; \
		exit 255; \
	else \
		make INSTANCE=$(LINUX_KERNEL) -C $(KERNEL_BUILD) M=$(LOCALPWD) INSTALL_MOD_PATH=$(TARGETDIR) modules_install_only; \
	fi


help:
	@echo ""
	@echo "Quick reference for various supported build targets for $(INSTANCE)."
	@echo "----------------------------------------------------"
	@echo "  clean                  clean out build objects"
	@echo "  all                    build $(INSTANCE) and install to rootfs host copy"
	@echo "  build                  build subsystem"
	@echo "  install                install built objects to rootfs host copy"

```
