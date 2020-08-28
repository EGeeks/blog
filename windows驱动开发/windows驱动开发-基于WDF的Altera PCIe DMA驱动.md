# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# FPGA设计
> 参考我的博客：[Altera FPGA PCIe Avalon-MM DMA设计](https://blog.csdn.net/Zhu_Zhu_2009/article/details/80793919)

# Altera pcie-avmm dma IP寄存器

## DMA Descriptor Controller Registers
DMA控制器读写均支持最多128个描述符，读写操作是以FPGA视角来看，读操作是从PCIe地址空间到FPGA Avalon-MM地址空间，写操作是从FPGA Avalon-MM地址空间到PCIe地址空间。
在DMA控制器寄存器里设置描述符表位于在PCIe地址空间里的地址和大小，DMA控制器用Read Data Mover首先将描述符复制到自己内部的FIFO中，然后在根据描述符来开始DMA传输。描述符在RC内的地址必须是32字节对齐的。
DMA控制器有寄存器指示读写描述符的完成状态，读和写分别有自己的状态寄存器表，每个表有128个连续的DWORD项，对应128个描述符。状态字占用512字节，位置在RC Read Status and Descriptor Base指定的地址偏移0处，而实际的描述符在0x200偏移处，DMA控制器项状态字的done位写1表示传输成功，DMA控制器在完成最后一个描述符后会发送一个MSI中断，在接收到中断之后，主机host软件可以轮询done位来判断描述符状态，但是DMA控制器不会设置done位或者发送MSI在每一个描述符完成的时候，它根据RD_DMA_LAST PTR和WR_DMA_LAST_PTR寄存器存储的描述符ID来操作，由于描述符支持PCIe完成包的乱序传输，所以done位置位的时候，描述符可能还没有传输完成。例如想在128个描述符的传输中间时刻和完成时候获得通知：

 1. 写入RD_DMA_LAST_PTR值63。
 2. 写入RD_DMA_LAST_PTR值127。
 3. 轮询第63个状态字。
 4. 轮询第127个状态字。

## Read DMA Descriptor Controller Registers
| 地址偏移| 寄存器 | 访问权限 | 描述 |
| ------------- |:-------------:| -----:| -----:| 
| 0x00 | RC Read Status and Descriptor Base (Low) | R/W | 低32位，在设置高32位之后设置，地址必须32位对齐，在传输完成的时候才能改变这个寄存器 |
| 0x04 | RC Read Status and Descriptor Base (High) | R/W | 高32位 |
| 0x08 | EP Read Descriptor FIFO Base (Low) | RW | 低32位，指定存储描述符的FIFO地址，在设置高32位之后设置 |
| 0x0C | EP Read Descriptor FIFO Base (High) | RW | 高32位 |
| 0x10 | RD_DMA_LAST_PTR | RW | 读返回上次操作的描述符ID，如果没有DMA操作则返回0xFF，指定最后一个操作的描述符ID发起DMA，比如，读返回4，为了传输5个描述符，软件应该写入9 |
| 0x14 | RD_TABLE_SIZE | RW | 设置读描述符表的大小，值为描述符数量减1，默认为127 |
| 0x18 | RD_CONTROL | RW | 高31位保留，第0位设置上报每一个描述符的done位，但MSI都不会每次上报，否则根据RD_DMA_LAST_PTR来上报 |

## Write DMA Descriptor Controller Registers
和读一样，地址偏移在0x100。

# Read DMA and Write DMA Descriptor Format
每个描述符32字节，Read/Write Status and Descriptor Base + 0x200处，
| 地址偏移| 寄存器 | 描述 |
| ------------- |:-------------:| -----:| -----:| 
| 0x00 | RD_LOW_SRC_ADDR | 低32位，DMA源地址，PCIe地址空间 |
| 0x04 | RD_HIGH_SRC_ADDR | 高32位 |
| 0x08 | RD_CTRL_LOW_DEST_ADDR | 低32位，Avalon-MM地址空间 |
| 0x0C | RD_CTRL_HIGH_DEST_ADDR | 高32位 |
| 0x10 | CONTROL | [31:25] Reserved，必须为0<br>[24:18] ID，描述符ID，0-127<br>[17:00] SIZE，传输大小，以DWORD单位，最大传输大小是1MB-4bytes，超过最大传输大小，按最大值传输，否则传输设置值，这个字段最大值为0x40000-0x1 |
| 0x14-0x1C | Reserved | N/A |

# windows DMA编程
分配Descriptor内存，
```c
    NTSTATUS status = WdfCommonBufferCreate(engine->parentDevice->dmaEnabler, bufferSize,
                                            WDF_NO_OBJECT_ATTRIBUTES, &engine->descBuffer);
    if (!NT_SUCCESS(status)) {
        TraceError(DBG_INIT, "WdfCommonBufferCreate failed: %!STATUS!", status);
        return status;
    }

    PHYSICAL_ADDRESS descBufferLA = WdfCommonBufferGetAlignedLogicalAddress(engine->descBuffer);
    PUCHAR descBufferVA = (PUCHAR)WdfCommonBufferGetAlignedVirtualAddress(engine->descBuffer);
    RtlZeroMemory(descBufferVA, bufferSize);
```
申请MSI/MSI-X中断，
```c
    NTSTATUS status = WdfInterruptCreate(xdma->wdfDevice, &config, &attribs,
                                         &(xdma->channelInterrupts[index]));
    if (!NT_SUCCESS(status)) {
        TraceError(DBG_INIT, "WdfInterruptCreate failed: %!STATUS!", status);
    }
```
申请DMA，这里设置了ADMA_MAX_TRANSFER_SIZE，
```c
	WdfDeviceSetAlignmentRequirement(adma->wdfDevice, FILE_32_BYTE_ALIGNMENT); //add by zhuce 
    WDF_DMA_ENABLER_CONFIG dmaConfig;
    WDF_DMA_ENABLER_CONFIG_INIT(&dmaConfig, WdfDmaProfileScatterGather64Duplex, ADMA_MAX_TRANSFER_SIZE);
    status = WdfDmaEnablerCreate(adma->wdfDevice, &dmaConfig, WDF_NO_OBJECT_ATTRIBUTES, &adma->dmaEnabler);
    if (!NT_SUCCESS(status)) {
        TraceError(DBG_INIT, " WdfDmaEnablerCreate() failed: %!STATUS!", status);
        return status;
    }
```
申请读/写队列，
```c
	WDF_IO_QUEUE_CONFIG_INIT(&config, WdfIoQueueDispatchSequential);
	config.EvtIoWrite = EvtIoWriteDma;
	//config.EvtIoRead = EvtIoReadDma;
	WDF_OBJECT_ATTRIBUTES_INIT(&attribs);
	attribs.SynchronizationScope = WdfSynchronizationScopeQueue;
	WDF_OBJECT_ATTRIBUTES_SET_CONTEXT_TYPE(&attribs, QUEUE_CONTEXT);
	status = WdfIoQueueCreate(device, &config, &attribs, queue);
	if (!NT_SUCCESS(status)) {
		TraceError(DBG_INIT, "WdfIoQueueCreate failed %d", status);
		return status;
	}
```

发起DMA，ADMA_EngineProgramDma作为回调函数，超过ADMA_MAX_TRANSFER_SIZE，传输会被分多次，WDF为我们做了很多工作，这些我们都是看不到的。
```c
    status = WdfDmaTransactionInitializeUsingRequest(queue->engine->dmaTransaction, Request,
                                                     ADMA_EngineProgramDma,
                                                     WdfDmaDirectionReadFromDevice);
    if (!NT_SUCCESS(status)) {
        TraceError(DBG_IO, "WdfDmaTransactionInitializeUsingRequest failed: %!STATUS!",
                   status);
        goto ErrExit;
    }
    status = WdfDmaTransactionExecute(queue->engine->dmaTransaction, queue->engine);
    if (!NT_SUCCESS(status)) {
        TraceError(DBG_IO, "WdfDmaTransactionExecute failed: %!STATUS!", status);
        goto ErrExit;
    }
```
回调函数，填入描述符表。
```c
    for (ULONG i = 0; i < SgList->NumberOfElements; i++) {
        descriptor[i].control = XDMA_DESC_MAGIC;
        descriptor[i].numBytes = SgList->Elements[i].Length;
        ULONG hostAddrLo = SgList->Elements[i].Address.LowPart;
        LONG hostAddrHi = SgList->Elements[i].Address.HighPart;
        if (Direction == WdfDmaDirectionWriteToDevice) {
            // source is host memory
            descriptor[i].srcAddrLo = hostAddrLo;
            descriptor[i].srcAddrHi = hostAddrHi;
            descriptor[i].dstAddrLo = LIMIT_TO_32(deviceOffset);
            descriptor[i].dstAddrHi = LIMIT_TO_32(deviceOffset >> 32);
        } else {
            // destination is host memory
            descriptor[i].srcAddrLo = LIMIT_TO_32(deviceOffset);
            descriptor[i].srcAddrHi = LIMIT_TO_32(deviceOffset >> 32);
            descriptor[i].dstAddrLo = hostAddrLo;
            descriptor[i].dstAddrHi = hostAddrHi;
        }
    }
```
