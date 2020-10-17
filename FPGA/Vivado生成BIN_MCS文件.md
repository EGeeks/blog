# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> ug908 ug835
> [AR# 44635 7 Series - EMCCLK considerations to ensure the FPGA completes the startup sequence](https://www.xilinx.com/support/answers/44635.html)
> [AR# 62034 7 Series - 2014.2/2014.3 write_bitstream error - EMCCLK pin must be programmed as an input when generating a bitfile for configuration](https://www.xilinx.com/support/answers/62034.html)
> [FPGA BPI加载时间计算](https://blog.csdn.net/weixin_42564775/article/details/85084205)
> [7系列FPGA上电配置流程](https://blog.csdn.net/weixin_42564775/article/details/88360063)
> [ISE XILINX BPI EMCCLK 配置实现](https://www.amobbs.com/thread-5594364-1-1.html?_dsign=9ab4274d)
> [配置文件的自动化生成和管理](https://forums.xilinx.com/t5/Xilinx-%E4%BA%A7%E5%93%81%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%8A%9F%E8%83%BD%E8%B0%83%E8%AF%95%E6%8A%80%E5%B7%A7/%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E7%9A%84%E8%87%AA%E5%8A%A8%E5%8C%96%E7%94%9F%E6%88%90%E5%92%8C%E7%AE%A1%E7%90%86/ba-p/1076440)

# 方法
xdc约束，xc7k325t，PC28F00AP30TF，[如果使用CCLK，配置速率最大为66Mhz，所以在高速配置FPGA的需求下，需要外部EMCCLK来满足配置时间的要求。EMCCLK最大频率计算方法见下面的公式，并且不能超过DS181, DS182, 和 DS183文档中定义的最大值。在7系列中，常见的EMCCLK时钟频率为100Mhz。比如对于K7325T，通过查阅bitstream size的大小为91,548,896 bits（87.3Mb）](http://design.eccn.com/design_2016120213185161.htm)，如果VCCO0连接至2.5V或3.3V，CFGBVS连接至VCCO0，如果VCCO0连接至1.5V或1.8V，CFGBVS连接至GND。
```shell
set_property BITSTREAM.GENERAL.COMPRESS TRUE [current_design]
set_property BITSTREAM.CONFIG.CONFIGRATE 66 [current_design]
set_property CONFIG_MODE BPI16 [current_design]
set_property CONFIG_VOLTAGE 2.5 [current_design]
set_property CFGBVS VCCO [current_design]
# set_property BITSTREAM.CONFIG.BPI_1ST_READ_CYCLE 2 [current_design]
# set_property BITSTREAM.CONFIG.BPI_PAGE_SIZE 8 [current_design]
set_property BITSTREAM.CONFIG.BPI_SYNC_MODE TYPE2 [current_design]
```
![217](https://img-blog.csdnimg.cn/20191110170254335.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![216](https://img-blog.csdnimg.cn/20191110165957409.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
tcl命令，未压缩之前，bitstream大小11443725字节，10.9MB，压缩之后，7402859字节，7.05MB，这和工程内容有关。这样操作之后，可以在上电时找到FPGA PCIe设备。
```shell
write_cfgmem -force -format BIN -interface BPIx16 -size 128 -loadbit "up 0x0 E:/project/vivado2015.2.1/finace_xc7k325t/finace_xc7k325t.runs/impl_1/finace_wrapper.bit" E:/project/vivado2015.2.1/finace_xc7k325t/finace_xc7k325t.runs/impl_1/finace_wrapper.bin
write_cfgmem -force -format BIN -interface SPIx4 -size 128 -loadbit "up 0x0 E:/project/vivado2015.2.1/finace_xc7k325t/finace_xc7k325t.runs/impl_1/finace_wrapper.bit" E:/project/vivado2015.2.1/finace_xc7k325t/finace_xc7k325t.runs/impl_1/finace_wrapper.bin
copy /y E:\project\vivado2015.2.1\finace_1_10g_toe\finace.runs\impl_1\finace_wrapper.bit C:\project\boot\finace
```
# 其他
可选的时钟值，
![221](https://img-blog.csdnimg.cn/2019111316430971.png)
可添加时间戳，
![222](https://img-blog.csdnimg.cn/20191113164431399.png)
