# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 数据流方向
```mermaid
flowchat
op0=>operation: Camera
op1=>operation: Video Data
op2=>operation: SDI
op3=>operation: VIP Frame Buffer IP Core（Writer）
op4=>operation: DDR2/3/4
op5=>operation: PCIe
op0(right)->op1(right)->op2->op3->op4->op5
```

# Altera VIP Frame Buffer II IP Core

## Frame Buffer II IP Core特性：

 - 缓存渐进（progressive）和隔行交错（interlaced）视频场（video field）
 - 支持双缓冲和三缓冲，取决于是否开启帧丢弃和帧重复选项
 - 可配置缓冲内部（inter-buffer）偏移，将输入放入DDR不同bank，以达到最大效率
 - 支持固化和动态配置缓冲延迟最大4095帧
 - 可配置的用户包处理行为

Frame Buffer II IP Core有两种基本模式，Reader，从内存读取帧送到输出，Writer，将帧存到内存。原理框图如下，由于Frame Buffer II IP Core同时只支持一种模式，所以需要两个IP才能完成下面的框图。
![这里写图片描述](https://img-blog.csdn.net/20180627000424467?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 重要的参数：

| Parameter | Value | Description |
| -- |:---:|:----:|
| Maximum frame width | 32–8192, Default =1920 | 设置最大帧宽，以像素为单位 |
| Maximum frame width | 32–8192, Default =1080 | 设置最大帧宽，以像素为单位 |
| Frame buffer memory base address | Any 32-bit value, Default = 0x00000000 | 设置帧缓存在外部memory的地址，UI界面下方的消息会显示具体占用的空间 |
| Enable use of inter-buffer offset | On or Off | 如果需要最大化DDR效率则打开 |
| Inter-buffer offset | Any 32-bit value, Default = 0x01000000 | 设置一个比单帧缓存大地址 |

## 寄存器

| Address | Register | Type | Description |
|:------------:|:------------:|:-------------------:|:-------------------:|
| 0 | Control | RW | 第0位是Go标志，设置为0则停止写入帧？如果UI界面使能run-time control，则这个位默认为0，否则为1 |
| 1 | Status | RO | 第0位是状态位，其他位保留 |
| 2 | Interrupt | RW | 当IP写入一帧到DDR，或从DDR读出一帧，IP通过中断线发出中断，设置第0位为1，写1清中断 |
| 3 | Frame Counter | RO | 对于writer，当帧没有被丢弃，计数+1，对于reader，当帧没有重复（读出），计数+1 |
| 4 | Drop/Repeat Counter | RO | 对于writer，当帧被丢弃，计数+1，对于reader，当帧重复（读出），计数+1 |
| 5 | Frame Information | RW | [31:31] available位，writer模式才有，0表示没有帧可供读取，在writer模式下，在接收下一帧之前必须确认MISC的acknowledge位<br>[30:30] 保留<br>[29:26] 包含上次写入帧的interlaced bits<br>[25:13] 包含上次写入帧的帧宽<br>[12:00] 包含上次写入帧的帧高 |
| 6 | Frame Start Address | RW | 这个寄存器存储上次写入帧在DDR中的开始地址，在Reader-only模式，必须设置开始地址到这个寄存器，对于writer，仅在Frame Information的available有效时，这个寄存器才有效 |
| 7 | Frame Reader | RO | [26:26] ready位，表示reader可以读取下一帧<br>[25:13] 表示读取的最大帧宽，在parameter editor配置的参数<br>[12:00] 表示读取的最大帧高，在parameter editor配置的参数 |
| 8 | Misc | RW | [27:16] 配置frame delay，默认值为1，可以配置2~4095<br>[15:02] 保留<br>[01:01] user packet affinity位，设置1表示丢弃/重复和user packets相关联的video packet（将被接收的下一包）？原文：Set this bit to 1 you want to drop and repeat user packets together with their associated video packet (this is the next video packet received). This mode allows for specific frame information that must be retained with each frame. — Set this bit to 0 if all user packets are to be produced as outputs in order, regardless of any dropping or repeating of associated video packets. This mode allows for audio or closed caption information.<br>[00:00] acknowledge位，仅在writer模式下有，设置1表示帧已经被处理，同时触发buffer进行复位，以备下次使用 |
| 9 | Locked Mode Enable | RW | 第0位控制locked mode，设置1时，Input Frame Rate和Output Frame Rate 寄存器严格控制丢弃和重复帧，设置0，则丢弃和重复仅仅有缓冲区空闲/使用状态决定 |
| 10 | Input Frame Rate | RW | [15:00] short型整数 |
| 11 | Output Frame Rate | RW | [15:00] short型整数 |

## Writer Only模式

在这个模式下，frame buffer将视频帧数据写入Frame buffer memory base address寄存器中的地址，然后自动跳转下一个地址，帧数据写入DDR之后，通过Frame Information寄存器的available位通知用户，该寄存器还包含帧的高度宽度等参数，帧数据所在地址在Frame Start Address寄存器，这些信息会一直在寄存器中，直到用户设置Misc寄存器的acknowledge位，表示帧已经被处理，缓冲区可以被再次使用，设置acknowledge位会清0 available位，但是如果此时正在写入新的一帧，available位不会清0，Frame Information会刷新为新的帧信息。
当帧有效时，Frame buffer II发起中断，设置Interrupt寄存器第0位为1，写1清中断。
当缓冲区满的时候，有新的一帧来到，如果打开了Frame dropping参数，帧将会被丢弃，否则输入将会被关闭。

## Reader Only模式

在这个模式下，设置Frame Information的帧尺寸信息和Frame Start Address寄存器，IP开始从DDR读取视频帧，设置Frame Start Address发起传输，如果设置了新的设置Frame Information的帧尺寸信息，则必须重新设置Frame Start Address寄存器，使参数生效。
Frame Buffer II不会判断一个帧时dirty还是clean，IP不停传输当前地址的数据，直到新的地址被设置。
一个3-buffer frame reader的使用过程：
 1. 等待Frame Reader寄存器的ready位有效
 2. 配置Frame Information的帧尺寸信息（包含4位interlace信息，比如progressive=0，interlaced F0=8，interlaced F=12）
 3. 向buffer N（位于DDR中）中写入帧数据
 4. 设置Frame Start Address寄存器
 5. 增加buffer编号N=（N + 1）% 3

## Frame Reader/Writer的内存使用布局

除了数据帧缓存之外，还保留一些Anc buffer缓存存储user packet，每个帧缓存对应字节的Anc buffer，每个用户包占1K字节（0x400）。
![这里写图片描述](https://img-blog.csdn.net/20180628141509631?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
帧缓存布局如下：
![这里写图片描述](https://img-blog.csdn.net/20180628141539568?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
