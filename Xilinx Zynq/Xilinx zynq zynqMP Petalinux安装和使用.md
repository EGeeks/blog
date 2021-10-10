# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 安装
petalinux升级了安装方式变化，2015.2.1会自动新建子文件夹，而2018.2需要手动设置，每次都折腾一下，记录一下。
```shell
cd ~/program
/mnt/hgfs/F/petalinux-v2018.2-final-installer.run petalinux-v2018.2-final
/mnt/hgfs/F/petalinux-v2015.2.1-final-installer.run .
```

# ubuntu切换到bash
```shell
qe@ubuntu:~/program$ ls -l /bin/sh
lrwxrwxrwx 1 root root 4 Jun 21  2019 /bin/sh -> dash
qe@ubuntu:~/program$ 
qe@ubuntu:~/program$ sudo dpkg-reconfigure dash
[sudo] password for qe: 
Removing 'diversion of /bin/sh to /bin/sh.distrib by dash'
Adding 'diversion of /bin/sh to /bin/sh.distrib by bash'
Removing 'diversion of /usr/share/man/man1/sh.1.gz to /usr/share/man/man1/sh.distrib.1.gz by dash'
Adding 'diversion of /usr/share/man/man1/sh.1.gz to /usr/share/man/man1/sh.distrib.1.gz by bash'
```

# 配置参数
配置镜像在flash或sd，对应u-boot环境变量变化，
```bash
default_bootcmd=run cp_kernel2ram && bootm ${netstart}
# flash
cp_kernel2ram=sf probe 0 && sf read ${netstart} ${kernelstart} ${kernelsize}
# sd
cp_kernel2ram=mmcinfo && fatload mmc ${sdbootdev} ${netstart} ${kernel_img}
```

# Petalinux给内核打补丁
> 参考 [PetaLinux Yocto Tips](http://www.wiki.xilinx.com/PetaLinux+Yocto+Tips)

1. Copy the patch to project file `<plnx-proj-root>/project-spec/meta-user/recipes-kernel/linux/linux-xlnx` directory.

2. Modify project file `<plnx-proj-root>/project-spec/meta-user/recipes-kernel/linux/linux-xlnx_%.bbappend` to use the patch file by adding the patch file name to the SRC_URI_append variable. If the variable does not exist in the file then add a new line with
```shell
SRC_URI_append = " file://0001-linux-driver-fix.patch"
FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}:"
```
3. Make sure the priority for the meta-user layer is 7 in the project file `<plnx-proj-root>/project-spec/meta-user/conf/layer.conf` .

# Petalinux给fsbl打补丁

> 参考 [PetaLinux Yocto Tips](http://www.wiki.xilinx.com/PetaLinux+Yocto+Tips)

注意：此方法不能用于2016.4版本打补丁，2016.4使用外部源代码编译fsbl，Yocto不支持对外部源代码打补丁。
在meta-user层创建fsbl文件夹
```shell
zc@ubuntu:~/project/mwm165$ mkdir -p project-spec/meta-user/recipes-bsp/fsbl/files
```
拷贝补丁文件到plnx-proj-root/project-spec/meta-user/recipes-bsp/fsbl/files
```shell
zc@ubuntu:~/project/mwm165$ cp /mnt/hgfs/F/xilinxlinux/doc/xapp1305-ps-pl-based-ethernet-solution/software/patches/0001-fsbl-si570-clk-config-on-A53.patch project-spec/meta-user/recipes-bsp/fsbl/files/
```
创建fsbl_%.bbappend文件，
```shell
zc@ubuntu:~/project/mwm165$ gedit project-spec/meta-user/recipes-bsp/fsbl/fsbl_%.bbapend
```
添加如下内容到文件中，
```shell
# Patch for FSBL

do_configure_prepend() {
    if [ -d "${S}/patches" ]; then
       rm -rf ${S}/patches
    fi
 
    if [ -d "${S}/.pc" ]; then
       rm -rf ${S}/.pc
    fi
}
 
SRC_URI_append = " \
        file://00001-fsbl-si570-clk-config-on-A53.patch \
        "
 
FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
 
#Add debug for FSBL(optional)
#XSCTH_BUILD_DEBUG = "1"
 
#Enable appropriate FSBL debug flags
#YAML_COMPILER_FLAGS_append = " -DXPS_BOARD_ZCU102"
 
# Note: This is not required if you are using Yocto
# EXTERNALXSCTSRC = ""
# EXTERNALXSCTSRC_BUILD = ""
```
删除plnx-proj-root/components/plnx_workspace，清除工程
```shell
zc@ubuntu:~/project/mwm165$ petalinux-build -x mrproper
zc@ubuntu:~/project/mwm165$ rm -rf components/plnx_workspace
```
重新编译FSBL
```shell
zc@ubuntu:~/project/mwm165$ petalinux-build
zc@ubuntu:~/project/mwm165$ petalinux-build -c bootloader
```
Xilinx对FSBL打补丁需要使用SDK新建FSBL工程，你看。。。还不如自己在SDK里建FSBL工程搞算了，Petalinux编译太慢，安装SDK，下载Windows或Linux下的web installer，运行之后选择下载到本地安装，这里选择下载Linux系统安装包。下载完之后安装，我这里安装命令行工具就算了，我不需要图形界面。
![这里写图片描述](https://img-blog.csdn.net/20180704000240185?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
下面编译就可以通过了，打印会有输出：
```shell
fsbl-2017.4+gitAUTOINC+77448ae629-r0 do_compile: 
NOTE: fsbl: compiling from external source tree /home/zc/program/petalinux-v2017.4-final/tools/hsm/data/embeddedsw
pmu-firmware-2017.4+gitAUTOINC+77448ae629-r0 do_compile: 
NOTE: pmu-firmware: compiling from external source tree /home/zc/program/petalinux-v2017.4-final/tools/hsm/data/embeddedsw
```

# 编译
新建的petalinux工程必须先编译，再改platform.h之类的，否则warning: backslash and newline separated by space，而且一旦坏掉就必须重建工程。编译完的镜像在`images/linux`下，zynqmp的linux编译出的原始镜像位于，
```bash
/home/qe/project/petalinux/zynqmp_vcu/build/tmp/work/plnx_zynqmp-xilinx-linux/linux-xlnx/4.19-xilinx-v2019.1+gitAUTOINC+9811303824-r0/image/boot/Image
# or
/home/qe/project/petalinux/zynqmp_vcu/build/tmp/work/plnx_zynqmp-xilinx-linux/linux-xlnx/4.19-xilinx-v2019.1+gitAUTOINC+9811303824-r0/package/boot/Image
```

# 打包BOOT镜像
包括fsbl，uboot等。
```shell
zc@ubuntu:~/project/mwm165$ petalinux-package --boot --fsbl ./images/linux/zynqmp_fsbl.elf --u-boot --pmufw ./images/linux/pmufw.elf --force
zc@ubuntu:~/project/mwm165$ cp BOOT.BIN /mnt/hgfs/F/xilinxlinux/boot/mwm165/
zc@ubuntu:~/project/mwm165$ cp images/linux/zynqmp_fsbl.elf /mnt/hgfs/F/xilinxlinux/boot/mwm165/
zc@ubuntu:~/project/mwm165$ cp images/linux/image.ub /mnt/hgfs/F/xilinxlinux/boot/mwm165/
```
# 添加已有驱动
![在这里插入图片描述](https://img-blog.csdn.net/20160630180234508?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
[如上图所示，"install"的操作需要修改以使得8812au.ko被包含在跟文件系统中/lib/modules/的合适子目录中。 -S是告诉打包工具不要做stripe操作。](https://blog.csdn.net/gdlituo/article/details/51789855)

# 内核单独编译
将2018.2 copy换了位置，编译zynqmp，设备树编译时出现错误，
```c
arch/arm64/boot/dts/xilinx/zynqmp-zc1751-xm015-dc1.dts:14:10: fatal error: dt-bindings/phy/phy.h: No such file or directory
 #include <dt-bindings/phy/phy.h>
          ^~~~~~~~~~~~~~~~~~~~~~~
compilation terminated.
```
解决办法，注释arch/arm64/boot/dts/xilinx/Makefile
```shell
# SPDX-License-Identifier: GPL-2.0
# dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1232-revA.dtb
dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1254-revA.dtb
dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1275-revA.dtb
dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1275-revB.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1751-xm015-dc1.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1751-xm016-dc2.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1751-xm017-dc3.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1751-xm018-dc4.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zc1751-xm019-dc5.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zcu100-revC.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zcu102-revA.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zcu102-revB.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zcu102-rev1.0.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zcu104-revA.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zcu104-revC.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zcu106-revA.dtb
#dtb-$(CONFIG_ARCH_ZYNQMP) += zynqmp-zcu111-revA.dtb

always		:= $(dtb-y)
subdir-y	:= $(dts-dirs)
clean-files	:= *.dtb
```
# 解决petalinux-build慢
build之前断网，则Checking sstate mirror object availability瞬间完成，推测肯定是联网，网速很慢导致吃屎，生命不可贵吗，熬夜到凌晨，现在编译时间可以控制到2min。如果我不熬夜，我估计也不会想着要快点，总之真的恶心。

# 常见问题
 1. device-tree串口需要匹配，否则没有输出很慌乱哦
 2. device-tree phy地址需要匹配，否则没有输出很慌乱哦
 3. device-tree不需要加compatible = "atheros, at803x";，这样由驱动自己匹配phy驱动，否则没有网卡很慌乱哦
 4. uboot配置中的环境变量出现不明字符导致编译错误

![这里写图片描述](https://img-blog.csdn.net/20180705001304392?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 用notepad打开之后，发现，
![这里写图片描述](https://img-blog.csdn.net/20180705001323606?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 删掉图中的字符，但是没有解决问题，陷入死结，全部删光，一点点重来，换下面这两种写法，
```c
#ifndef CONFIG_ZYNQ_GPIO
#define CONFIG_ZYNQ_GPIO
#endif

#ifdef CONFIG_BOOTDELAY
#undef CONFIG_BOOTDELAY
#endif
#define	CONFIG_BOOTDELAY	1
```
编译成功，另外添加环境变量字符串一定要复制粘贴来搞，不要自己敲空格回车编辑，之前应该是这里引入了未知的字符。下面的也会引入上面的错误，很无语啊
```c
#ifndef CONFIG_BOOTARGS
//#define CONFIG_BOOTARGS "earlycon clk_ignore_unused noinitrd console=ttyPS0,115200 root=/dev/mmcblk0 rw"
#endif
```

# 增加新软件包
更改`<plnx-proj-root>/project-spec/meta-user/recipes-core/images/petalinux-image.bbappend`，
```c
IMAGE_INSTALL_append = " iperf3"
IMAGE_INSTALL_append = " nginx_1.13.5"
```
在rootfs配置中选中，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515102659891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
