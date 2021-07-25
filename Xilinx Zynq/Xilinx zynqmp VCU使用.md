# 参考
> [Zynq UltraScale+ MPSoC VCU TRD 2019.1](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/112230447/Zynq+UltraScale+MPSoC+VCU+TRD+2019.1)
> [HDMI Receiver Subsystem v3.1 - Patch Updates for the HDMI Receiver Subsystem v3.1 in Vivado 2018.1](https://www.xilinx.com/support/answers/71203.html)
> [音视频编解码: YUV存储格式中的YUV420P,YUV420SP,NV12, NV21理解(转)](https://www.cnblogs.com/yongdaimi/p/10696214.html)
> [常用视频像素格式NV12、NV21、I420、YV12、YUYV](https://blog.csdn.net/Fan0920/article/details/103710014)
> [对颜色空间YUV、RGB的理解](https://blog.csdn.net/asahinokawa/article/details/80596655)
> [ZCU106 VCU Linux驱动转裸机驱动篇（一）](https://blog.csdn.net/weixin_43873379/article/details/102875489)
> [hi3519v101　h265编码器编码出第一帧延迟是多少毫秒？](http://bbs.ebaina.com/forum.php?mod=viewthread&tid=23549)

# 基本概念
## RGB
RGB 模型是目前常用的一种彩色信息表达方式，它使用红、绿、蓝三原色的亮度来定量表示颜色。该模型也称为加色混色模型，是以RGB三色光互相叠加来实现混色的方法，因而适合于显示器等发光体的显示。

## YUV
Y表示亮度，CbCr表示颜色，可以粗浅地视YUV为YCbCr。怎么表示颜色，引用网上的一张图，
![72](https://img-blog.csdnimg.cn/2021070523372770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
YUV采样方式有三种，`Chroma Resampler`用于在这三个方式中进行转换，
- YUV 4:4:4采样，每一个Y对应一组UV分量。
- YUV 4:2:2采样，每两个Y共用一组UV分量。比如YUVY，UYVY，YUV422P。
- YUV 4:2:0采样，每四个Y共用一组UV分量。比如YU12，YV12，NV12，NV21。

YUV的数据组织格式有两种，planar和packed，
- planar的YUV格式，先连续存储所有像素点的Y，紧接着存储所有像素点的U，随后是所有像素点的V。比如YUV422P，YU12和YV12。
- packed的YUV格式，每个像素点的Y,U,V是连续交叉存储的。比如YUVY，UYVY。
- 融合了上面两种格式。比如NV12、NV21。

YUV每个分量占一个字节，一副图像的YUV444大小为`x*y*3`，YUV420大小为`x*y*3/2`。

RGB和YUV可通过`RGB to YCrCb Color-Space Converter`和`YCrCb to RGB Color-Space Converter`进行转换，

# VCU
Video Codec Unit (VCU) 核编码器块是采用 H.265 （ISO/IEC 23008-2 高效视频编码）和 H.264 （ISO/IEC 14496-10 高级视频编码）标准对视频流进行处理的视频编码器引擎。编码器块全面支持上述标准（这些标准包括支持 8 位和 10 颜色深度、4:2:0、4:2:2 和 4:0:0 Y-only（单色） 色度格式、以及高达 4K UHD @ 60 Hz 分辨率的性能）。

## 性能
速率为 @ 667 MHz(2) 时的性能，
32 个流 @ 720×480p @ 30 Hz
8 个流 @ 1920×1080p @ 30 Hz
4 个流 @ 1920×1080p @ 60 Hz
2 个流 @ 3840×2160p @ 30 Hz
1 个流 @ 3840×2160p @ 60 Hz
1 个流 @ 7680×4320p @ 15 Hz
采样位深度：8 bpc、10 bpc，
色度格式：YCbCr 4:2:0、YCbCr 4:2:2、Y-only（单色），

## 内存
CMA大小需求，
![73](https://img-blog.csdnimg.cn/20210707220558276.PNG)
解码器存储器的带宽取决于帧速率、分辨率、颜色深度、色度格式和解码器配置信息。LogiCORE™ IP 根据 GUI 中选择的视频参数来提供解码器的带宽估算值。赛灵思建议尽可能使用最快的 DDR4 存储器接口。具体来说， 8x8 位的存储器接口比 4x16 位的存储器接口更有效，因为 x8 模式有 4 个 bank 组，而 x16 模式只有 2 个 bank 组；而且 DDR4 允许同时访问多个 bank 组。如需了解更多信息，请参阅 AR# 71209。

## 编码器缓存
启用编码器缓存，请将 prefetch-buffer 参数传递到使用硬件的 GStreamer 流水线中。例如：
gst-launch-1.0 videotestsrc ! omxh265enc prefetch-buffer=true ! fakesink

## 时延
对于编解码，第一帧图像开始时会存在一个初始化延迟，但当图像流水起来之后，编解码器的总延迟是稳态延迟。

# Vivado打补丁
对Vivado 2018.2打补丁，`AR71203_Vivado_2018_1_preliminary_rev4.zip`，按照压缩包内的README.txt的提示打补丁即可，在Vivado 2019.1及以后的版本不需要，主要修复下面两个bug，
- (Xilinx Answer 71935)	Info frame packets corrupted when switching resolutions
- (Xilinx Answer 70870)	Link quality becomes bad after running for 15~45 minutes.

# 视频格式
2019.1的视频格式支持下4种，

# 使用
VCU的地址在0xA0000000，此时HPM0只有192MB可用空间，从0xA4000000开始，
![341](https://img-blog.csdnimg.cn/20200930155839516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)

```bash
$ cd ./rdf0428-zcu106-vcu-trd-2019-1_v2/pl/
$ source scripts/vcu_trd_proj.tcl
$ source scripts/vcu_hdmirx_proj.tcl
$ source scripts/vcu_hdmitx_proj.tcl
```

## vcu_hdmitx
`axi_ctrl`没有接，分辨率就固定了，没法动态配置，
![71](https://img-blog.csdnimg.cn/20210704235207811.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

