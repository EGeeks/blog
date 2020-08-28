# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# ubuntu16.04.6
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
安装petalinux，
```bash
$ mkdir -p /opt/Xilinx/Petalinux/2019.2
$ ./petalinux-v2019.2-final-installer.run /opt/Xilinx/Petalinux/2019.2
$ sudo dpkg-reconfigure dash #选择no
```

