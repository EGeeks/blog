# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

Xilinx共提供三种类型的DMA IP，AXI DMA，AXI CDMA，AXI VDMA，分别适配于AXI-MM，AXI-Stream等相互搬运场合。

# AXI DMA
发送端通过`Start of Frame bit (TXSOF)`和`End of Frame bit (TXEOF)`来界定AXI-Stream上的包边界。`TXSOF`和`TXEOF`可以跨描述符，接收端也是类似，当包长度超过一个描述符长度时，会自动取下一个描述符来接收数据，通过`RXSOF`和`RXEOF`来界定一个包。
首先设置`DMACR.RS`为1，通过设置`TAILDESC_PTR`来指示开始发送或接收，可以动态修改`TAILDESC_PTR`寄存器，达到循环搬运的效果，类似于FIFO操作。

# SG描述符
地址偏移必须是16个32位数对齐，即64字节。 

# 中断
有三种类型的中断，

- IOC_Irq：SG模式下，每一个描述符完成都会触发，中断太频繁导致包处理性能太低，`coalesce`寄存器可用来调节中断频率。
- Dly_Irq：配合`coalesce`使用。
- Err_Irq：错误中断。


