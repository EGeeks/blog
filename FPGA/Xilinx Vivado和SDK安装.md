# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [官网下载](https://china.xilinx.com/support/download.html)
> [vivado 设置 多线程编译](https://blog.csdn.net/angelbosj/article/details/51596146)

# 方法
以vivado2015.2.1为例，先安装vivado2015.2，再安装vivado2015.2.1更新包，选下面两个都可以，看需求，
![160](https://img-blog.csdnimg.cn/20190726212422185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
选上sdk，按需选择，
![161](https://img-blog.csdnimg.cn/2019072621251276.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
添加license，
![162](https://img-blog.csdnimg.cn/20190726212605999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)安装完后卸载烦人的xic，xilinx information center，
![159](https://img-blog.csdnimg.cn/2019072621265123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 设置多线程
-----------------------------------Place Route
Windows默认----------------------2 2
Linux默认---------------------------4 4
Windows开启maxThreads=8--4 4
Linux开启maxThreads=8-------8 8
```tcl
set_param general.maxThreads 4
get_param general.maxThreads
```

# tcl
命令行使用tcl，`vivado -mode tcl`，使用前要先source一下settings64.sh文件，再执行`start_gui`可打开当前工程GUI。Linux和Windows下据可用GUI来使用tcl。

# 下载器驱动安装失败
Vivado无法扫描到FPGA，设备管理器中显示Xilinx Platform Cable USB Firmware Loader，可能是我安装过SDK2018.2，
![164](https://img-blog.csdnimg.cn/20190731173414183.png)
在设备上右键菜单卸载删除驱动，
![165](https://img-blog.csdnimg.cn/20190731173531289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
我本打算到安装包中手动安装驱动，但奇迹出现了，删除驱动后，又自动安装了一个驱动，变正常的了，由于异常过，这儿不能auto connect，选择手动发现就可以了。
![166](https://img-blog.csdnimg.cn/20190731174150612.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# Ubuntu
- 换了个i3-9100F的无法安装ubuntu16.04.6，换了最新的ubuntu20.04，安装vivado2018.2，卡在`Generating installed devices list`，换了ubuntu18.04，可以安装vivado2018.2，总是闪屏，更新显卡驱动，终于稳定了。
- 更改ubuntu的dash为bash

