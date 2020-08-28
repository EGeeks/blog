# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# FPGA设计
DMA原理框图如下，实际应用中把双口RAM换成自己的IP即可，首先使能了内部Descriptor Controller，那么BAR0默认连接到了Descriptor，若想通过PCIe BAR来访问寄存器，就必须得添加一个BAR4，其实可以不用使能内部的Descriptor Controller，自己添加DMA IP，手动在Qsys中连线，这样一个BAR就够了。Read Data Mover从PC读数据，这里Read Data Mover连接了两个设备，一个是Descriptor Controller内部的FIFO，用来存放DMA描述符的，另外的就是自己的数据源，Write Data Mover将数据发送到PC。
![这里写图片描述](https://img-blog.csdn.net/20180624193632519?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
Qsys连接图如下，其中wr_dts_slave和rd_dts_slave是Descriptor Controller内部的FIFO接口，这里分配的地址是0x08012000和0x08010000，这儿分配看实际需要，这儿的地址空间不能喝自己逻辑使用的冲突即可。另一个BAR4的空间设置了地址0x08000000，BAR4的空间占用不易过大，适当是最好的。另外wr_dcm_master和rd_dcm_master连接到了Txs端口，用来更新DMA描述符状态字和发送MSI消息。本来Descriptor Controller的Control有两组端口，一个接收端口连到内部的BAR0上，肯定是用于寄存器访问，发送端口连到了Txs上。
![这里写图片描述](https://img-blog.csdn.net/20180624194522725?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
关于BAR，参考手册，其中BAR的Size由软件根据你连接的元件自动计算，所以地址范围需要限定，不然就奔着32位总线的4GB去了，这是不可以的。
![这里写图片描述](https://img-blog.csdn.net/20180804140336800?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
如果是真的需要4GB bar，需修改主板bios above 4GB mmio选项，参考[Threadripper 1950X Max number of GPUs](https://community.amd.com/message/2863718)。

# 驱动设计
参考我的博客：[windows驱动开发-Altera PCIe DMA](https://blog.csdn.net/Zhu_Zhu_2009/article/details/80790252)
