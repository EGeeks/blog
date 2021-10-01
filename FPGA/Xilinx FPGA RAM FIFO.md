# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [FPGA block RAM和distributed RAM区别](http://blog.sina.com.cn/s/blog_6a8c8c750100roii.html)
> [【FPGA】关于Xilinx芯片中Block RAM和Distributed RAM 的区别](https://blog.csdn.net/u011327754/article/details/79741280)
> [浅谈XILINX FPGA CLB单元 汇总 (CLB、LUT、存储单元、Distributed RAM、移位寄存器、多路复用器、进位逻辑(Carry Logic))](https://blog.csdn.net/vivid117/article/details/102841135)
> [赛灵思（Xilinx）Block Ram预先存储数据及使用方法及地址定义](https://blog.csdn.net/sinat_25902709/article/details/87024730)

1. 物理上看，Bram是fpga中定制的ram资源，Dram就是用逻辑单元拼出来的。
2. 较大的存储应用，建议用Bram；零星的小ram，一般就用Dram。但这只是个一般原则，具体的使用得看整个设计中资源的冗余度和性能要求
3. Dram可以是纯组合逻辑，即给出地址马上出数据，也可以加上register变成有时钟的ram。而Bram一定是有时钟的。

# RAM
## Value  is out of the range for parameter 'Memory Depth(MEM_DEPTH)'
详细错误如下，
```bash
ERROR: [IP_Flow 19-3461] Value '134086656' is out of the range for parameter 'Memory Depth(MEM_DEPTH)' for BD Cell 'axi_bram_ctrl_0' . Valid values are - 512, 1024, 2048, 4096, 8192, 16384, 32768, 65536, 131072, 262144, 524288, 1048576, 2097152, 4194304, 8388608, 16777216, 33554432, 67108864, 134217728, 268435456, 536870912, 1073741824
INFO: [IP_Flow 19-3438] Customization errors found on 'axi_bram_ctrl_0'. Restoring to previous valid configuration.
ERROR: [Common 17-39] 'set_property' failed due to earlier errors.
ERROR: [BD 41-1273] Error running pre_propagate TCL procedure: ERROR: [Common 17-39] 'set_property' failed due to earlier errors.
    ::xilinx.com_ip_axi_bram_ctrl_4.1::pre_propagate Line 73
```

有两个master通过switch访问同一个bram ctrl，但是两个master的地址设置的不一样，是可以设置不同的地址的啊，但是总是出这个问题，搞的大哥快要疯了。

## coe初始化文件
bd中，需使用stand alone模式才可以添加初始化文件。

## Generating a 32-bit address interface
使能则地址按位宽递增，否则按1递增。
**从这点也看出来网上文章一大抄，大家有多浮躁，踏踏实实搞研究吧！**

# FIFO
## axis data fifo
注意packet mode这个参数，在有last信号的axis中，必须使能，否则会丢失包的边界。
![2021-03-13 22-51-17](https://img-blog.csdnimg.cn/20210317224550304.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

