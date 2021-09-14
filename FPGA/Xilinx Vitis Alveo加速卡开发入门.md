# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Vitis Unified Software Development Platform 2020.1 Documentation](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/index.html)
> [Vitis Application Acceleration Development Flow Documentation](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/kme1569523964461.html)
> [Vitis Embedded Software Development Flow Documentation](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/hly1569525384514.html)
> [Using Vitis HLS](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/irn1582730075765.html)
> [Methodology for Accelerating Applications with the Vitis Software Platform](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/methodologyacceleratingapplications.html#wgb1568690490380)
> [Vitis libraries](https://xilinx.github.io/Vitis_Libraries/)
> [Xilinx Runtime (XRT) Documentation](https://xilinx.github.io/XRT/)
> [Xilinx Runtime (XRT) Documentation Master](https://xilinx.github.io/XRT/master/html/index.html)
> [创建 Vitis 加速平台第 1 部分：在 Vivado 中为加速平台创建硬件工程](https://forums.xilinx.com/t5/Xilinx-%E4%BA%A7%E5%93%81%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%8A%9F%E8%83%BD%E8%B0%83%E8%AF%95%E6%8A%80%E5%B7%A7/%E5%88%9B%E5%BB%BA-Vitis-%E5%8A%A0%E9%80%9F%E5%B9%B3%E5%8F%B0%E7%AC%AC-1-%E9%83%A8%E5%88%86-%E5%9C%A8-Vivado-%E4%B8%AD%E4%B8%BA%E5%8A%A0%E9%80%9F%E5%B9%B3%E5%8F%B0%E5%88%9B%E5%BB%BA%E7%A1%AC%E4%BB%B6%E5%B7%A5%E7%A8%8B/ba-p/1147504)
> [2020.1 Vitis™ Application Acceleration Development Flow Tutorials](https://github.com/Xilinx/Vitis-Tutorials)
> [Vitis Accel Examples' Repository](https://github.com/Xilinx/Vitis_Accel_Examples)
> [Vitis Embedded Platform Source](https://github.com/Xilinx/Vitis_Embedded_Platform_Source)
> [开发者分享 | Alveo加速卡上管理子系统 CMC 介绍](https://mp.weixin.qq.com/s/cNzFy_kRvvsnKO5_c4XGtg)
> [开发者分享 | 如何在Vitis中设定Kernel 的频率](https://mp.weixin.qq.com/s/10zN5G_vJW8gDiKFtS8Y6Q)
> [AR# 71754 Alveo 数据中心加速卡 — 定制流程 — 一般类信息和已知问题](https://china.xilinx.com/support/answers/71754.html)
> [AR# 71757 Alveo Data Center Accelerator Card - Reverting Card to Factory image](https://china.xilinx.com/support/answers/71757.html)

# Vitis
老哥现在在Ubuntu18.04上，打开Vitis，platform可以是官方的，比如Alveo或者官方开发板或者第三方开发板，也可以是自己创建的，Windows上直接双击桌面图标打开即可，
```bash
# setup XILINX_VITIS and XILINX_VIVADO variables
$ source /opt/Xilinx/Vitis/2020.1/settings64.sh
# setup XILINX_XRT
$ source /opt/xilinx/xrt/setup.sh
# specify the location of platform, /opt/xilinx/platforms/xilinx_u50_gen3x16_xdma_201920_3
# 不添加这个路径也可以，可能默认路径就是/opt/xilinx/platforms
$ export PLATFORM_REPO_PATHS=/opt/xilinx/platforms
```
Vitis加速框架，这个应该在各种地方都看到过了，使用OpenCL™ API来和加速器交互，
![vitis acceleration](https://img-blog.csdnimg.cn/20201014203420731.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)Vitis加速的软件架构，Host可以是X86 PC，也可以是MPSOC中的ARM，思维不要局限了，Host Processor和Programmable Logic之间通过PCIe或者AXI通信，
![vitis execution model](https://img-blog.csdnimg.cn/20201014220253392.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)Vitis编译流程，一个应用有两个部分，Host Application和FPGA Kernels，老哥我的理解，前者负责数据的调度、控制，后者是实际硬件加速的部分，
![vitis build process](https://img-blog.csdnimg.cn/20201014221210426.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)基本流程已经清楚，现在按照官网的例子，跑个Demo，看看Vitis怎样使用，
```bash
$ git clone https://github.com/Xilinx/Vitis-Tutorials
```
初始化环境，然后进入工程目录，这是一个深度学习里面脉动阵列的矩阵乘法程序，Xilinx已经写好了Makefile，当然也可以按照文档自己敲`g++`，`v++`命令，修改`design.cfg`适配自己的platform，
```bash
$ cd ~/project/vitis/Vitis-Tutorials/docs/Pathway3/reference-files/run
$ ls -l
总用量 8
-rw-r--r-- 1 qe qe   91 10月 16 14:18 design.cfg
-rw-r--r-- 1 qe qe 1944 10月 16 14:18 Makefile
$ make help
Makefile Usage:
  make build TARGET=<sw_emu/hw_emu/hw> PLATFORM=<FPGA platform>
      Command to generate the design for specified Target and Device.

  make run TARGET=<sw_emu/hw_emu/hw> PLATFORM=<FPGA platform>
      Command to run for specified Target.

  make exe 
      Command to generate host.

  make xclbin 
      Command to generate hardware platform files(xo,xclbin).

  make clean 
      Command to remove the generated files.
```
编译，老哥的电脑i3-9100F，4核4.0GHz的CPU，内存16GB，得1小时，感觉很久，我一个综艺都要看完了，编译阶段vivado的CPU占用率基本跑满，内存占了8GB，
```bash
$ source /opt/Xilinx/Vitis/2020.1/settings64.sh && source /opt/xilinx/xrt/setup.sh && export LIBRARY_PATH=/usr/lib/x86_64-linux-gnu
$ make build TARGET=hw PLATFORM=xilinx_u50_gen3x16_xdma_201920_3
# 使用图形化的方式查看结果
$ vitis_analyzer mmult.hw.xilinx_u50_gen3x16_xdma_201920_3.xclbin.link_summary
```
在硬件上执行一下，
```bash
$ ./host mmult.hw.xilinx_u50_gen3x16_xdma_201920_3.xclbin
Found Platform
Platform Name: Xilinx
INFO: Reading mmult.hw.xilinx_u50_gen3x16_xdma_201920_3.xclbin
Loading: 'mmult.hw.xilinx_u50_gen3x16_xdma_201920_3.xclbin'
TEST PASSED 
# or
$ make run TARGET=hw PLATFORM=xilinx_u50_gen3x16_xdma_201920_3
```
仿真，创建`xrt.ini`文件，
```bash
$ cat xrt.ini 
[Debug]
profile=true
timeline_trace=true
data_transfer_trace=fine
```
更改`design.cfg`，添加`profile_kernel=data:all:all:all`，重新编译，查看结果，
```bash
$ source /opt/Xilinx/Vitis/2020.1/settings64.sh && source /opt/xilinx/xrt/setup.sh && export LIBRARY_PATH=/usr/lib/x86_64-linux-gnu
$ make build TARGET=hw_emu PLATFORM=xilinx_u50_gen3x16_xdma_201920_3
$ vitis_analyzer mmult.hw_emu.xilinx_u50_gen3x16_xdma_201920_3.xclbin.run_summary
```
![2020-10-19 19-50-20.png](https://img-blog.csdnimg.cn/20201019195100670.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
# Vivado
如果需要用传统方法开发Alveo程序，需要下载一些文件，但是要申请，回去让公司直接找FAE要吧，Xilinx是不推荐这样玩的，还是乖乖的用Vitis吧。
![19](https://img-blog.csdnimg.cn/20201008152402765.PNG#pic_center)为确保客户映像在运行时成功加载到 Alveo 数据中心加速卡上，必须使用以下设置创建 MCS 文件，
- 内存部件：mt25qu01g-spi-x1_x2_x4
- 起始地址：0x01002000

DDR，以U200为例，3个SLR，4个DDR，QSFP所在SLR2，对应DDR[3]，所以网络数据处理的时候优先使用DDR[3]，除非SLR2资源不够了，那就另当别论了。
![2021-08-30 23-11-40](https://img-blog.csdnimg.cn/f829392d52c84582a708d75c94ae529e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiJ6YGN54yq,size_20,color_FFFFFF,t_70,g_se,x_16)![2021-08-30 22-35-41](https://img-blog.csdnimg.cn/785038efc61b486da0a30a6860bcceb4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiJ6YGN54yq,size_20,color_FFFFFF,t_70,g_se,x_16)
# 问题
## AR# 73269
试图在 Alveo U50 电路板上对配置内存器件进行编程时，如果将非配置分配信息 I/O 引脚设置为 "Pull-up"状态，则在对 MCS 进行编程时会出现以下错误：
```bash
ERROR: [Labtools 27-2149] File /opt/Xilinx/Vivado/2019.2/data/xicom/cfgmem/bitfile/spi_xcu50_pullup.bit not found.Check file name and file permissions.
ERROR: [Labtools 27-2149] File C:/Xilinx/Vivado/2019.2/data/xicom/cfgmem/bitfile/spi_xcu50_pullup.bit not found.Check file name and file permissions.
```
这是 2019.2 版中的一个已知问题。下载 bitfile.zip 文件，将 `C:\Xilinx\Vivado\2019.2\data\xicom\cfgmem` 或 `/data/xicom/cfgmem` 中的 bitfiles.zip 文件替换为本答复记录中提供的版本。

## ./elaborate.sh: 行 20: LIBRARY_PATH: 未绑定的变量
```bash
export LIBRARY_PATH=/usr/lib/x86_64-linux-gnu
```

