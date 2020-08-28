# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 方法
ubuntu16.04.4，由于已经安装了Petalinux 2018.2，所以一些依赖软件包应该不需要再安装了，
```shell
$ cd /media/j2/xilinx/Xilinx_SDx_2018.3_1207_2324/
$ ./xsetup
```
安装下载器驱动，安装完重启，
```shell
$ cd ./Vivado/2018.3/data/xicom/cable_drivers/lin64/install_script/install_drivers
$ sudo ./install_drivers 
INFO: Installing cable drivers.
INFO: Script name = ./install_drivers
INFO: HostName = j2-pc
INFO: Current working dir = /media/j2/xilinx/Vivado/2018.3/data/xicom/cable_drivers/lin64/install_script/install_drivers
INFO: Kernel version = 4.13.0-36-generic.
INFO: Arch = x86_64.
Successfully installed Digilent Cable Drivers
--File /etc/udev/rules.d/52-xilinx-ftdi-usb.rules does not exist.
--File version of /etc/udev/rules.d/52-xilinx-ftdi-usb.rules = 0000.
--Updating rules file.
--File /etc/udev/rules.d/52-xilinx-pcusb.rules does not exist.
--File version of /etc/udev/rules.d/52-xilinx-pcusb.rules = 0000.
--Updating rules file.

INFO: Digilent Return code = 0
INFO: Xilinx Return code = 0
INFO: Xilinx FTDI Return code = 0
INFO: Return code = 0
INFO: Driver installation successful.
CRITICAL WARNING: Cable(s) on the system must be unplugged then plugged back in order for the driver scripts to update the cables.
```
SDx是全功能软件包，包含了Vivado 2018.3，开启Vivado，
```shell
$ source settings64.sh 
$ vivado
```

