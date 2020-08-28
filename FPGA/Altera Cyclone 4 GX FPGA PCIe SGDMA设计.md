# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考

> [PCI Express in Qsys Example Designs](https://fpgawiki.intel.com/wiki/PCI_Express_in_Qsys_Example_Designs)
> [https://fpgawiki.intel.com/wiki/Modular_SGDMA](https://fpgawiki.intel.com/wiki/Modular_SGDMA)
> [Modular SGDMA Video Frame Buffer](https://fpgawiki.intel.com/wiki/Modular_SGDMA_Video_Frame_Buffer)

# 设计

Cyclone IV GX不像Arria10的pcie-avmm带自带SGDMA，只能用altera提供的Modular SGDMA，这个模块还是以verilog源代码形式提供，其实是个学习verilog的好例子，整体架构如下，很清晰了，Modular SGDMA控制数据在DDR和PC直接互相传输。
![在这里插入图片描述](https://img-blog.csdn.net/20181007143304995?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
再看Modular SGDMA内部实现，将数据从一个MM端口，搬运到另一个MM端口，数据流单向流动，怎么搬运由Descriptors和CSR端口来控制，这两个端口为寄存器，可通过PCIe BAR来访问。
![在这里插入图片描述](https://img-blog.csdn.net/2018100714341177?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
Modular SGDMA的Dispatcher实现如下，
![在这里插入图片描述](https://img-blog.csdn.net/20181007144549897?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# PCI Express-to-Avalon-MM地址翻译

这里就是Bar的地址翻译，
![在这里插入图片描述](https://img-blog.csdn.net/20181009225049826?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
看下IP配置，这里的偏移都是0，altera设置死了，无法更改，手册说的是Hard-code，硬编码的。
![在这里插入图片描述](https://img-blog.csdn.net/20181009224953660?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# Avalon-MM-to-PCI Express地址翻译

For example, if the core is configured with an address translation table with the
following attributes:
■ Number of Address Pages—16
■ Size of Address Pages—1 MByte
■ PCI Express Address Size—64 bits
then the values in Figure 4–12 are:
■ N = 20 (due to the 1 MByte page size)
■ Q = 16 (number of pages)
■ M = 24 (20 + 4 bit page selection)
■ P = 64
In this case, the Avalon address is interpreted as follows:
■ Bits [31:24] select the TX slave module port from among other slaves connected to
the same master by the system interconnect fabric. The decode is based on the base
addresses assigned in Qsys.
■ Bits [23:20] select the address translation table entry.
■ Bits [63:20] of the address translation table entry become PCI Express address bits
[63:20].
■ Bits [19:0] are passed through and become PCI Express address bits [19:0].
看下IP核配置，首先选择翻译表项，最大支持512个，每个表项的长度也可设置。注意，翻译表的位置在CRA的0x1000处。截图来自quartus14.1。
![在这里插入图片描述](https://img-blog.csdn.net/20181009225213811?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 产生PCI Express中断

RX模块有16个Avalon-MM中断输入（RXmirq_irq[n:0]，n ≤ 15），设置这16个信号或者设置对应PCIe邮箱寄存器，会在中断寄存器相应位置1，中断使能寄存器在0x50处，中断状态寄存器在0x40处，中断发生后，软件必须清除相应的中断状态寄存器位。IP支持传统中断和MSI中断，
The PCI Express Avalon-MM bridge selects either MSI or legacy interrupts automatically based on the standard interrupt controls in the PCI Express configuration space registers. The Interrupt Disable bit, which is bit 10 of the
Command register (at configuration space offset 0x4) can be used to disable legacy interrupts. The MSI Enable bit, which is bit 0 of the MSI Control Status register in the MSI capability register (bit 16 at configuration space offset 0x50), can be used to enable MSI interrupts.
Only one type of interrupt can be enabled at a time. However, to change the selection of MSI or legacy interrupts during operation, software must ensure that no interrupt request is dropped. Therefore, software must first enable the new selection and then disable the old selection. To set up legacy interrupts, software must first clear the Interrupt Disable bit and then clear the MSI enable bit. To set up MSI interrupts, software must first set the MSI enable bit and then set the Interrupt Disable bit.
看下Qsys中配置，有一个Auto-enable PCIe interrupt的配置，需要测试一下。
![在这里插入图片描述](https://img-blog.csdn.net/2018101023040435?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
看下手册关于邮箱寄存器的描述，写和读在不同的位置，应该是为了避免信号冲突。
![在这里插入图片描述](https://img-blog.csdn.net/20181010231856334?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
中断处理函数中必须清除中断状态寄存器和SGDMA的中断状态寄存器

# 产生Avalon-MM中断

Generation of Avalon-MM interrupts requires the instantiation of the CRA slave module where the interrupt registers and control logic are implemented. The CRA slave port has an Avalon-MM Interrupt (CraIrq_irq in Qsys systems) output signal. A write access to an Avalon-MM mailbox register sets one of the P2A_MAILBOX_INTn bits in the “PCI Express to Avalon-MM Interrupt Status Register Address: 0x3060” on page 6–11 and asserts the CraIrq_o or CraIrq_irq output, if enabled. Software can enable the interrupt by writing to the “PCI Express to Avalon-MM Interrupt Enable Register Address: 0x3070” on page 6–11 through the CRA slave. After servicing the interrupt, software must clear the appropriate serviced interrupt status bit in the PCI-Express-to-Avalon-MM Interrupt Status register and ensure that there is no other interrupt pending.
