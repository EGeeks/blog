# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)


# 参考
> [Vitis Unified Software Development Platform 2020.1 Documentation](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/index.html)
> [Vitis Application Acceleration Development Flow Documentation](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/kme1569523964461.html)
> [Vitis Embedded Software Development Flow Documentation](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/hly1569525384514.html)
> ug1144
> [how to add or modify petalinux 2016.4 yocto kernel source or devictree source?](https://forums.xilinx.com/t5/Embedded-Linux/how-to-add-or-modify-petalinux-2016-4-yocto-kernel-source-or/m-p/742861)
> [An example of using FILES_${PN}](https://stackoverflow.com/questions/46071039/an-example-of-using-files-pn)
> [Vitis Platform Out-of-Date after Update HW Specification](https://forums.xilinx.com/t5/Embedded-Development-Tools/Vitis-Platform-Out-of-Date-after-Update-HW-Specification/td-p/1157212)
> [如何使用 Git 在 Vitis IDE 中进行版本控制](https://mp.weixin.qq.com/s/aZKljCfXsZlK53EMzmGUeg)

# 安装
## Windows
Vivado打不开，Vitis闪退，通过命令行执行发现`Error: The file D:/Xilinx/Vivado/2020.1/lib/win64.o/librdi_device.dll is corrupt. Please re-install `，重新解压缩安装包安装后问题消失，文件在硬盘里放久了竟然损坏了？事事不顺，软件安了一天。

## Ubuntu16.04.6
安装Vitis，执行，
```bash
$ chmod -Rf 777 /opt
$ ./xsetup
```
按照默认设置，桌面快捷方式安装失败，应为我的是中文版的ubuntu，桌面快捷方式安装错误，把Desktop的快捷方式剪切到桌面即可，安装路径`/opt/Xilinx`，安装完之后，加载License， 从Dash Board里面可以卸载Information Center，我很讨厌这个，接着安装下载器驱动，参考UG973，
```bash
$ cd /opt/Xilinx/Vivado/2019.2/data/xicom/cable_drivers/lin64/install_script/install_drivers/
$ sudo ./install_drivers
```
命令行安装，
```bash
# ./xsetup -b ConfigGen
Running in batch mode...
Copyright (c) 1986-2021 Xilinx, Inc.  All rights reserved.

INFO : Log file location - /root/.Xilinx/xinstall/xinstall_1627208663081.log
Select a Product from the list:
1. Vitis
2. Vivado
3. On-Premises Install for Cloud Deployments
4. BootGen
5. Lab Edition
6. Hardware Server
7. Documentation Navigator (Standalone)

Please choose: 1

INFO : Config file available at /root/.Xilinx/install_config.txt. Please use -c <filename> to point to this install configuration.
# 修改安装路径 /opt/Xilinx
# vi /root/.Xilinx/install_config.txt 
# ./xsetup -a XilinxEULA,3rdPartyEULA,WebTalkTerms -b Install -c /root/.Xilinx/install_config.txt 
Running in batch mode...
Copyright (c) 1986-2021 Xilinx, Inc.  All rights reserved.

INFO : Log file location - /root/.Xilinx/xinstall/xinstall_1627208737680.log
INFO : Installing Edition: Vitis Unified Software Platform
INFO : Installation directory is /opt/Xilinx

Installing files, 99% completed. (Done)                         
It took 35 minutes to install files.

INFO : Log file is copied to : /opt/Xilinx/.xinstall/Vitis_2020.1/xinstall.log
INFO : Installation completed successfully.For the platforms: please visit xilinx.com and review the "Getting Started Guide" UG1301
```

# 使用
## 更改配色
因为我的ubuntu18.04 gnome装了黑色主题，vitis的主题必须调整，否则字体看不到了。
![2021-02-25 23-02-21](https://img-blog.csdnimg.cn/20210317224301848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

## 更新xsa
vitis没有像sdk那样自动检测hdf更新，重新生成hw，需要在项目窗口手动右键执行更新。vivado更改了一个地址，更新xsa发现vitis中地址没有更新，清除综合，重新跑一遍解决。中断在bd里面更新了，但是软件没有更新，这时候需要`Reset BSP`才可以。

## platform out-of-date
更新xsa文件后项目窗口显示out-of-date，右键执行编译可消除这个提示。

## bsp配置
打开项目的`*.spr`文件，
