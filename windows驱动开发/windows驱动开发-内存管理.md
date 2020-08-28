# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

由于VS与WDK卸载，暂停，未完待续。。。

参考博客：[第二十七篇:Windows驱动中的PCI, DMA, ISR, DPC, ScatterGater, MapRegsiter, CommonBuffer, ConfigSpace](https://blog.csdn.net/u013140088/article/details/37742681)
系统空间的中虚拟内存与物理内存之间的联系通过IoAllocateMdl与MmBuildMdlForNonPagedPool建立特定的MDL来表示。其后，通过DMA_ADAPTER的DmaOperations中的GetScatterGatherList获取MDL所描述的虚拟地址内存的S/G列表，最后，在GetScatterGatherList的ExecutionRoutine 函数中, 将该列表填入Common buffer的TABLE(起始逻辑地址 与 长度)中， 以供DMA Controller所用。

2018-06-20 继续博客

# 参考

WDK帮助手册（WDK7600安装时自带本地帮助，高版本WDK可能没有，可以在微软官网的硬件开发者专区查看）
系列博客[PCIe学习笔记](https://blog.csdn.net/u013140088/article/category/7015733)

# windows系统内存空间

下图说明了windows NT-based系统虚拟内存和物理内存的关系，虚拟内存构建于分页的物理内存上，连续的虚拟内存有可能位于不连续的物理内存上，无论是用户空间还是系统空间，从Paged Pool中分配的内存随时都有可能被交换到第二存储上，包括该进程正在执行的时候。一个进程当前若是不正在执行，那么进程的虚拟空间是不可访问的。
![这里写图片描述](https://img-blog.csdn.net/20180620103903919?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 分配系统空间内存

驱动可能会分配系统空间内存用于IO缓冲，推荐用下面4个函数分配内存：

 - MmAllocateNonCachedMemory
 - MmAllocateContiguousMemorySpecifyCache
 - AllocateCommonBuffer(bus-master DMA or system DMA controller's auto-initialize mode)
 - ExAllocatePoolWithTag

非分页内存池（Nonpaged Pool）会越来越碎片化，所以如果需要长期使用，驱动最好在在DriverEntry中分配好内存，除了 ExAllocatePoolWithTag函数，其他调用分配的内存都是和处理器相关边界对齐（processor-specific boundary (determined by the processor's data-cache-line size)）来提供最好的性能。MmAllocateNonCachedMemory和MmAllocateContiguousMemorySpecifyCache分配的内存空间至少为一个page，请求少于一个page的内存，也将占用一个page，多余空间对驱动不可见，其他驱动也无法使用这段内存。AllocateCommonBuffer需要和adapter对象配合使用，adapter对象提供设备地址（LA）到物理地址（PA）的映射（map registers）。ExAllocatePoolWithTag需要提供一个POOL_TYPE参数。

 - NonPagedPool运行在IRQL > APC_LEVEL，如果请求大小size小于page，则分配size大小内存，如果size大于page，则向上取整，比如在x86上请求5KB内存，则会占用8KB，但是如果4KB+1KB分两次请求，则占用5KB。
 - PagedPool分配的内存只能在IRQL < APC_LEVEL上访问，同时is not in the file system's write path（？？？）

ExAllocatePoolWithTag返回NULL时，DriverEntry可以返回一个STATUS_INSUFFICIENT_RESOURCES失败状态。

# 映射寄存器组Map Registers

驱动发起DMA会用三种不同的地址空间，如下图所示，
![这里写图片描述](https://img-blog.csdn.net/20180620113559162?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
Windows平台上，驱动有权限访问任意虚拟空间，在32-bit处理器上，虚拟空间是4GB，CPU使用页表将虚拟地址匹配到系统物理空间，每一个页表项（PTE）映射一个page，MDL（memory descriptor list）对象为一个和DMA操作相关的缓冲区提供了相似映射。
设备使用的是逻辑（设备）地址空间，map registers用来把逻辑（设备）地址翻译到物理地址，提供和MDL是相同的功能。
硬件抽象层（HAL）创建支持各种设备和计算机总线的adapter对象，不是所有设备都可以访问处理器的全部地址空间，比如ISA DMA控制器，相反，一般PCI DMA设备可以访问全部地址空间，因此HAL必须提供设备可访问的逻辑地址空间到计算机物理地址空间的映射。下图展示了一个没有scatter/gather能力的ISA DMA设备的物理到逻辑地址映射关系。
![这里写图片描述](https://img-blog.csdn.net/20180620171655533?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 1. 每个map register将一个范围的物理地址映射到低端的逻辑地址。
 2. ISA设备用映射过的逻辑地址来访问系统空间内存。
 3. MDL的每一项将虚拟地址映射到物理地址。

每个map register和MDL的一项最多映射一个物理页。也可以少于一页，通过offset参数来表示，但最少为1字节。
在一个读或者写的IRP请求中，Irp->MdlAddress是用户空间缓冲区的MDL，表示虚拟到物理地址的映射。题外话，看一下IRP的结构（主角是map register），
```c
typedef struct _IRP {
  .
  .
  PMDL  MdlAddress;
  ULONG  Flags;
  union {
    struct _IRP  *MasterIrp;
    .
    .
    PVOID  SystemBuffer;
  } AssociatedIrp;
  .
  .
  IO_STATUS_BLOCK  IoStatus;
  KPROCESSOR_MODE  RequestorMode;
  BOOLEAN PendingReturned;
  .
  .
  BOOLEAN  Cancel;
  KIRQL  CancelIrql;
  .
  .
  PDRIVER_CANCEL  CancelRoutine;
  PVOID UserBuffer;
  union {
    struct {
    .
    .
    union {
      KDEVICE_QUEUE_ENTRY DeviceQueueEntry;
      struct {
        PVOID  DriverContext[4];
      };
    };
    .
    .
    PETHREAD  Thread;
    .
    .
    LIST_ENTRY  ListEntry;
    .
    .
    } Overlay;
  .
  .
  } Tail;
} IRP, *PIRP;
```
如果驱动使用direct I/O，且IRP的major function code是：

 - IRP_MJ_READ MDL
 - IRP_MJ_DEVICE_CONTROL，同时IOCTL code为METHOD_OUT_DIRECT
 - IRP_MJ_INTERNAL_DEVICE_CONTROL，同时IOCTL code为METHOD_OUT_DIRECT 

则MdlAddress表示一个空缓冲区，等待设备或者驱动写入。

 - IRP_MJ_WRITE MDL
 - IRP_MJ_DEVICE_CONTROL，同时IOCTL code为METHOD_IN_DIRECT
 - IRP_MJ_INTERNAL_DEVICE_CONTROL，同时IOCTL code为METHOD_IN_DIRECT 

则MdlAddress表示传输给设备或者驱动的缓冲区。
驱动如果不使用direct I/O，则MdlAddress为NULL。

# 映射总线相关地址到虚拟地址

在IRP_MN_START_DEVICE 处理函数中，将CM_RESOURCE_LIST结构体描述的I/O或者内存资源通过MmMapIoSpace映射到虚拟空间供驱动访问。当驱动收到来自PnP manager的IRP_MN_STOP_DEVICE或者IRP_MN_REMOVE_DEVICE请求时，调用MmUnmapIoSpace释放资源，如果IRP_MN_START_DEVICE处理失败，驱动也要调用MmUnmapIoSpace释放资源。
资源有下面这几种，
```c
#define CmResourceTypeNull                0   // ResType_All or ResType_None (0x0000)
#define CmResourceTypePort                1   // ResType_IO (0x0002)
#define CmResourceTypeInterrupt           2   // ResType_IRQ (0x0004)
#define CmResourceTypeMemory              3   // ResType_Mem (0x0001)
#define CmResourceTypeDma                 4   // ResType_DMA (0x0003)
#define CmResourceTypeDeviceSpecific      5   // ResType_ClassSpecific (0xFFFF)
#define CmResourceTypeBusNumber           6   // ResType_BusNumber (0x0006)
#define CmResourceTypeMemoryLarge         7   // ResType_MemLarge (0x0007)
#define CmResourceTypeNonArbitrated     128   // Not arbitrated if 0x80 bit set
#define CmResourceTypeConfigData        128   // ResType_Reserved (0x8000)
#define CmResourceTypeDevicePrivate     129   // ResType_DevicePrivate (0x8001)
#define CmResourceTypePcCardConfig      130   // ResType_PcCardConfig (0x8002)
#define CmResourceTypeMfCardConfig      131   // ResType_MfCardConfig (0x8003)
```
三种类型CmResourceTypePort，CmResourceTypeInterrupt和CmResourceTypeDma需要用READ_REGISTER_Xxx，WRITE_REGISTER_Xxx，READ_PORT_Xxx和WRITE_PORT_Xxx API来访问。

# 访问用户空间数据

有两种Buffered I/O和Direct I/O，对于IRP_MJ_READ和IRP_MJ_WRITE类型的IRP，由驱动初始化的时候决定时Buffered I/O活Direct I/O，对于 IRP_MJ_DEVICE_CONTROL和IRP_MJ_INTERNAL_DEVICE_CONTROL类型的IRP，由定义的I/O Control Code决定。
Buffered I/O，中间一个Nonpaged Pool做了一次copy。
![这里写图片描述](https://img-blog.csdn.net/20180621184027840?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
Direct I/O，如果是用于DMA，得到MDL就可以了，启动DMA传输了。
![这里写图片描述](https://img-blog.csdn.net/20180621184145549?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
Direct I/O，如果用于PIO，则需调用 MmGetSystemAddressForMdlSafe在系统空间再做一次映射。
![这里写图片描述](https://img-blog.csdn.net/2018062118471053?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
但这些过程在WDF里面都被接口API给实现了，驱动无需直接使用这些API。



