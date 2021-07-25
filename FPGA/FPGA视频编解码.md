 # 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [openasic官网 目前有开源H.264编V2.0解V1.0码 H.265编码V2.0](http://www.openasic.org/)
> [H.264视频编解码纯FPGA低延迟解决方案演示DEMO](http://www.openedv.com/forum.php?mod=viewthread&tid=274236&extra=page=1)
> [基于FPGA的H264视频解码器的研究与实现](https://wenku.baidu.com/view/024b8e512af90242a895e5c7.html)
> [h265_decoder.zip](https://download.csdn.net/download/tianqishi/19928474)
> [基于FPGA的h264编码系统设计与实现](https://download.csdn.net/download/xiaozhu19901990/10225668)
> [在FPGA上实现H_264AVC视频编码标准](https://download.csdn.net/download/ccwwff/3574966)
> [H.264视频编解码FPGA解决方案](https://blog.csdn.net/yinyidianzi/article/details/80227306)


# H.265实现
> [HEVC中变换（Transform）过程中的scaling操作的理解](https://blog.csdn.net/ftlisdcr/article/details/54345151)
> [HEVC 帧间预测](https://blog.csdn.net/strikedragon/article/details/82466002)
> [H.265/HEVC —— 帧内预测](https://blog.csdn.net/huangyifei_1111/article/details/84248118)
> [结合深度学习的视频编码方法--帧内预测](https://zhuanlan.zhihu.com/p/109723631)

# H.264实现
> [H264--1--编码原理以及I帧B帧P帧](https://blog.csdn.net/dxpqxb/article/details/7625652)
> [H.264视频编解码的FPGA源码分析（一）输入数据分析](https://blog.csdn.net/weixin_39584176/article/details/104809412)

在H264协议里定义了三种帧，完整编码的帧叫I帧，参考之前的I帧生成的只包含差异部分编码的帧叫P帧，还有一种参考前后的帧编码的帧叫B帧。

- I帧:帧内编码帧 ，I帧表示关键帧，你可以理解为这一帧画面的完整保留；解码时只需要本帧数据就可以完成（因为包含完整画面）
- P帧:前向预测编码帧。P帧表示的是这一帧跟之前的一个关键帧（或P帧）的差别，解码时需要用之前缓存的画面叠加上本帧定义的差别，生成最终画面。
- B帧:双向预测内插编码帧。B帧是双向差别帧，也就是B帧记录的是本帧与前后帧的差别（具体比较复杂，有4种情况，但我这样说简单些），换言之，要解码B帧，不仅要取得之前的缓存画面，还要解码之后的画面，通过前后画面的与本帧数据的叠加取得最终的画面。B帧压缩率高，但是解码时CPU会比较累。

I、B、P各帧是根据压缩算法的需要，是人为定义的,它们都是实实在在的物理帧。一般来说，I帧的压缩率是7（跟JPG差不多），P帧是20，B帧可以达到50。可见使用B帧能节省大量空间，节省出来的空间可以用来保存多一些I帧，这样在相同码率下，可以提供更好的画质。
