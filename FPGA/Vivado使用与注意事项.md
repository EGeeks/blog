# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 仿真波形窗口Block Design中的IP信号位宽不对
更改IP参数后，需要重新Generate Output Products。

# [DRC RTRES-2] Global clock nets are using local routing resources: Global clock nets are using local routing resources. 1 net(s) have at least one node with COST_CODE_NAME equal to BOUNCEACROSS or INTENT_CODE_NAME equal to NODE_DOUBLE, NODE_HLONG, NODE_HQUAD, NODE_SINGLE, NODE_VLONG, or NODE_VQUAD. This situation occurs when a global clock net must enter a clock region where all global clock routing resources are occupied. Check the report from report_clock_utilization to determine which clock regions are traversed by the problem net and use floorplanning or other physical constraints to ensure a maximum of 24 global clock nets occupy each clock region. The problem net(s) are system_i/clk_wiz_0/inst/clk_out1.
因为ultrascale的xdma的时钟连接错误。

# ordered port connections cannot be mixed with named port connections
最后一个例化信号`.m_axis_mac_tvalid      (s_axis_tvalid)`后面不能有标点符号。
```verilog
  toe_core #(
    .ILA_EN(0)
    ) toe_core_i (
...
    .m_axis_mac_tready      (s_axis_tready),
    .m_axis_mac_tvalid      (s_axis_tvalid)
    );
```

# 仿真
> [AR# 64000 Vivado 仿真器 - 了解波形数据库文件 (WDB) 和波配置文件 (WCFG)](https://china.xilinx.com/support/answers/64000.html)
> [AR# 58880 Xilinx Simulation Solution Center - Design Assistant - Vivado Simulator - Waveform Database (.wcfg,.wdb etc).](https://china.xilinx.com/support/answers/58880.html)

先点击1，后点击2，更改代码则先点击3，再点击2，就这几板斧就够用了。
![55](https://img-blog.csdnimg.cn/2021020620000256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
保存sim的配置文件和波形，
```bash
save_wave_config D:/project/pcierc-zc706-axi-interconnect/tso.wcfg
open_wave_config D:/project/pcierc-zc706-axi-interconnect/tso.wcfg
```

# [Synth 8-1852] concurrent assignment to a non-net s_axis_ctrl_fifo_tvalid is not permitted
对一个reg变量进行assign会出现这个错误。

# Error while detecting SPI flash device - unrecognized JEDEC id bytes: 10, 00, 00
错误的使用了另外一个项目的fsbl导致的。

# dcp网表
dcp网表不需要附加一个`.v/.vhd`的文件，直接使用，ngc网表才需要。

# using cached ip status
在tcl console下输入，或者在`Project Setting > IP > Clear Cache`，
```tcl
config_ip_cache -clear_output_repo
```
然后Regenerate IP。

# tcl
```bash
write_cfgmem -force -format BIN -interface BPIx16 -size 128 -loadbit "up 0x0 E:/project/vivado2015.2.1/finace_xc7k325t/finace_xc7k325t.runs/impl_1/finace_wrapper.bit" E:/project/vivado2015.2.1/finace_xc7k325t/finace_xc7k325t.runs/impl_1/finace_wrapper.bin
copy /y E:\project\vivado2015.2.1\finace\finace.runs\impl_1\finace_wrapper.bit C:\project\boot\finace
copy /y E:\project\vivado2015.2.1\finace_1_10g_toe\finace.runs\impl_1\finace_wrapper.bit C:\project\boot\finace
```

# [Place 30-69] Instance mig_7series_0 ... is unplaced after IO placer
Block Design中MIG的DDR3信号管脚忘记从顶层模块导出了。

# [Shape Builder 18-140] Failed to build a LUTNM shape for instances
> [[Shape Builder 18-140] Failed to build a LUTNM shape](https://forums.xilinx.com/t5/Synthesis/Shape-Builder-18-140-Failed-to-build-a-LUTNM-shape/m-p/655672)

Do you have LUTNM constraints? I see that its a synthesis critical warning.
It just letting the user know about the decision that the synthesizer is making.
If you are not getting any error in implementation flow, its ok to ignore this.

# IP设置没法更改
生成IP后，无法立刻更改IP参数配置，此时IP配置窗口的OK键是无法点击的，因为刚生成的IP正在编译。

# 保存和显示波形文件
ila的波形，
```bash
write_hw_ila_data -force E:/project/vivado2015.2.1/finace/tcp_ila_data [upload_hw_ila_data hw_ila_1]
open_hw # 等同于点击按钮Hardware Manager
read_hw_ila_data E:/project/vivado2015.2.1/finace/tcp_ila_data.ila
display_hw_ila_data
```

# xdc
xdc文件的注释使用`#`，必须另起一行，不能在行末尾加注释。

# Verilog语法检查能力差
变量声明必须在处理之前，否则，vivado不报错，直接把`process reg a`部分优化掉，所以建议所以变量都声明在文件顶部，同理，`s_axis_tdata_csum`如果不声明，也不报错，被vivado当作`s_axis_tdata_csum = 0`处理，
```c
always @(...) begin
  process reg a
end
reg a;

assign s_axis_tdata_csum = s_axis_tdata[31:16] + s_axis_tdata[63:48];
```
不能重复声明变量，不报错，vivado会当成两个不同的变量，导致结果不对。

# axi stream data fifo
如果你不在block design中使用这个IP，那么GUI中设置的DATA宽度不会适配到生成的verilog中，总是512，可以手动更改生成的IP文件，手动更改，这个问题在vivado2015.2.1和vivado2017.4中都会出现。
```shell
xxx.srcs\sources_1\ip\axis_data_fifo_0\synth\axis_data_fifo_0.v
```

# [IP_Flow 19-4048] Interface 's_axi' may not be edited on IP 'finace_axi_10g_ethernet_0_0'.
vivado2015.2.1，reset synth，Delete Files，BD上reset generated outputs，重新编译。

# Processor System Reset
![218](https://img-blog.csdnimg.cn/20191104202144747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 外部管脚时钟域
可先查看其他的1处的时钟域，粘贴过去，
![219](https://img-blog.csdnimg.cn/20191104202622598.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 远程调试
> [远程共享和访问赛灵思器件](https://forums.xilinx.com/t5/Xilinx-%E4%BA%A7%E5%93%81%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%8A%9F%E8%83%BD%E8%B0%83%E8%AF%95%E6%8A%80%E5%B7%A7/%E8%BF%9C%E7%A8%8B%E5%85%B1%E4%BA%AB%E5%92%8C%E8%AE%BF%E9%97%AE%E8%B5%9B%E7%81%B5%E6%80%9D%E5%99%A8%E4%BB%B6/ba-p/1127489)
