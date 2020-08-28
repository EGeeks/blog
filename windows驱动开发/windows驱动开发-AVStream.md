# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> 微软官网：[Docs\Windows Hardware\Windows Drivers\Device and Driver Technologies\Stream](https://docs.microsoft.com/en-us/windows-hardware/drivers/stream/avstream-overview)
> [Source Filter属性页一台电脑显示，一台不显示](https://bbs.csdn.net/topics/350140057)
> [AVStream ddk 翻译](https://blog.csdn.net/mao0514/article/details/39249949)
> [流Mini驱动开发(译自Microsoft DDK)](https://blog.csdn.net/cosmoslife/article/details/7947679)

# 硬件模型
> 参考我的博客：[Altera FPGA SDI VIP frame buffer control](https://blog.csdn.net/Zhu_Zhu_2009/article/details/80808803)

# 工具使用
点击菜单Graph->插入Filter，添加Video Capture Sources下的avsadma Source，添加DirectShow Filters下的Video Renderer，用鼠标将两者连接起来，如下图所示，点击工具栏上的三角开始捕获视频。
![这里写图片描述](https://img-blog.csdn.net/20180911223708463?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# AVStream架构
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9kb2NzLm1pY3Jvc29mdC5jb20vZW4tdXMvd2luZG93cy1oYXJkd2FyZS9kcml2ZXJzL3N0cmVhbS9pbWFnZXMvaGllcmFyY2h5LnBuZw?x-oss-process=image/format,png)

# avshws示例
DriverEntry中注册KSDEVICE_DESCRIPTOR，KSDEVICE_DESCRIPTOR包含KSDEVICE_DISPATCH，KSDEVICE_DISPATCH是用户需要实现的函数接口
```c
struct _KSDEVICE_DISPATCH {
    PFNKSDEVICECREATE Add;
    PFNKSDEVICEPNPSTART Start;
    PFNKSDEVICE PostStart;
    PFNKSDEVICEIRP QueryStop;
    PFNKSDEVICEIRPVOID CancelStop;
    PFNKSDEVICEIRPVOID Stop;
    PFNKSDEVICEIRP QueryRemove;
    PFNKSDEVICEIRPVOID CancelRemove;
    PFNKSDEVICEIRPVOID Remove;
    PFNKSDEVICEQUERYCAPABILITIES QueryCapabilities;
    PFNKSDEVICEIRPVOID SurpriseRemoval;
    PFNKSDEVICEQUERYPOWER QueryPower;
    PFNKSDEVICESETPOWER SetPower;
    PFNKSDEVICEIRP QueryInterface;  // added in version 0x100
};
```
示例中实现了三个接口：Add（DispatchCreate），Start（DispatchPnpStart）和Stop（DispatchPnpStop）。这和PCIe驱动开发差不多，在DispatchCreate中分配Device Context，
```c
KsAcquireDevice (Device);
Status = KsAddItemToObjectBag (
    Device -> Bag,
    reinterpret_cast <PVOID> (CapDevice),
    reinterpret_cast <PFNKSFREE> (CCaptureDevice::Cleanup)
    );
KsReleaseDevice (Device);

if (!NT_SUCCESS (Status)) {
    delete CapDevice;
} else {
    Device -> Context = reinterpret_cast <PVOID> (CapDevice);
}
```
DispatchPnpStart类似于EvtDevicePrepareHardware函数，提供了Resource List。在这里完成PCIe video采集卡设备资源（内存、IO）的申请。
```c
static NTSTATUS   DispatchPnpStart (
    IN PKSDEVICE Device,
    IN PIRP Irp,
    IN PCM_RESOURCE_LIST TranslatedResourceList,
    IN PCM_RESOURCE_LIST UntranslatedResourceList
    )
```
创建Pin对应的FilterFactory，KsCreateFilterFactory的第二个参数，传入FilterDescriptor，
```c
if (!m_Device -> Started) {
    // Create the Filter for the device
    KsAcquireDevice(m_Device);
    Status = KsCreateFilterFactory( m_Device->FunctionalDeviceObject,
                                    &CaptureFilterDescriptor,
                                    L"GLOBAL",
                                    NULL,
                                    KSCREATE_ITEM_FREEONSTOP,
                                    NULL,
                                    NULL,
                                    NULL );
    KsReleaseDevice(m_Device);

}
```
在Pin的DispatchCreate中分配图像帧缓存，
```c
Status = KsEdit (
    Pin, 
    &Pin -> Descriptor -> AllocatorFraming, 
    AVSHWS_POOLTAG);

if (NT_SUCCESS (Status)) {

    //
    // We've KsEdit'ed this...  I'm safe to cast away constness as
    // long as the edit succeeded.
    //
    PKSALLOCATOR_FRAMING_EX Framing =
        const_cast <PKSALLOCATOR_FRAMING_EX> (
            Pin -> Descriptor -> AllocatorFraming
            );

    Framing -> FramingItem [0].Frames = 2;

    //
    // The physical and optimal ranges must be biSizeImage.  We only
    // support one frame size, precisely the size of each capture
    // image.
    //
    Framing -> FramingItem [0].PhysicalRange.MinFrameSize =
        Framing -> FramingItem [0].PhysicalRange.MaxFrameSize =
        Framing -> FramingItem [0].FramingRange.Range.MinFrameSize =
        Framing -> FramingItem [0].FramingRange.Range.MaxFrameSize =
        VideoInfoHeader -> bmiHeader.biSizeImage;

    Framing -> FramingItem [0].PhysicalRange.Stepping = 
        Framing -> FramingItem [0].FramingRange.Range.Stepping =
        0;

}
```
在Pin的DispatchProcess中处理图像帧，参考[Stream Pointers](https://docs.microsoft.com/en-us/windows-hardware/drivers/stream/stream-pointers)，参考[Writing AVStream Minidrivers for Hardware](https://docs.microsoft.com/en-us/windows-hardware/drivers/stream/writing-avstream-minidrivers-for-hardware)
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9kb2NzLm1pY3Jvc29mdC5jb20vZW4tdXMvd2luZG93cy1oYXJkd2FyZS9kcml2ZXJzL3N0cmVhbS9pbWFnZXMvY25zdHJlYW00LnBuZw?x-oss-process=image/format,png)



# 结构体说明

[KSDEVICE](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksdevice)，在WDM驱动程序框架的AddDevice例程中，IoCreateDevice通过传入的PhysicalDeviceObject创建FunctionalDeviceObject。
```c
typedef struct _KSDEVICE {
  const KSDEVICE_DESCRIPTOR *Descriptor;
  KSOBJECT_BAG              Bag;
  PVOID                     Context;
  PDEVICE_OBJECT            FunctionalDeviceObject;
  PDEVICE_OBJECT            PhysicalDeviceObject;
  PDEVICE_OBJECT            NextDeviceObject;
  BOOLEAN                   Started;
  SYSTEM_POWER_STATE        SystemPowerState;
  DEVICE_POWER_STATE        DevicePowerState;
} KSDEVICE, *PKSDEVICE;
```
[KSPIN](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_kspin)
```c
typedef struct _KSPIN {
  const KSPIN_DESCRIPTOR_EX *Descriptor;
  KSOBJECT_BAG              Bag;
  PVOID                     Context;
  ULONG                     Id;
  KSPIN_COMMUNICATION       Communication;
  BOOLEAN                   ConnectionIsExternal;
  KSPIN_INTERFACE           ConnectionInterface;
  KSPIN_MEDIUM              ConnectionMedium;
  KSPRIORITY                ConnectionPriority;
  PKSDATAFORMAT             ConnectionFormat;
  PKSMULTIPLE_ITEM          AttributeList;
  ULONG                     StreamHeaderSize;
  KSPIN_DATAFLOW            DataFlow;
  KSSTATE                   DeviceState;
  KSRESET                   ResetState;
  KSSTATE                   ClientState;
} KSPIN, *PKSPIN;
```
[KSPIN](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_kspin) 中的[KSDATAFORMAT](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ksdataformat) ，注意[KSDATAFORMAT](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ksdataformat) 和[KSDATARANGE](https://msdn.microsoft.com/library/windows/hardware/ff561658) 是同一个结构体，
```c
#if !defined( _MSC_VER ) 
typedef struct {
    ULONG   FormatSize;
    ULONG   Flags;
    ULONG   SampleSize;
    ULONG   Reserved;
    GUID    MajorFormat;
    GUID    SubFormat;
    GUID    Specifier;
} KSDATAFORMAT, *PKSDATAFORMAT, KSDATARANGE, *PKSDATARANGE;
#else
typedef union {
    struct {
        ULONG   FormatSize;
        ULONG   Flags;
        ULONG   SampleSize;
        ULONG   Reserved;
        GUID    MajorFormat;
        GUID    SubFormat;
        GUID    Specifier;
    };
    LONGLONG    Alignment;
} KSDATAFORMAT, *PKSDATAFORMAT, KSDATARANGE, *PKSDATARANGE;
#endif
```
[KSDATAFORMAT](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ksdataformat) 指向一个具体的DATAFORMAT比如[KS_DATAFORMAT_VIDEOINFOHEADER](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-tagks_dataformat_videoinfoheader)
```c
typedef struct tagKS_DATAFORMAT_VIDEOINFOHEADER {
  KSDATAFORMAT       DataFormat;
  KS_VIDEOINFOHEADER VideoInfoHeader;
} KS_DATAFORMAT_VIDEOINFOHEADER, *PKS_DATAFORMAT_VIDEOINFOHEADER;
```
[KSFILTER_DESCRIPTOR](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksfilter_descriptor)
```c
typedef struct _KSFILTER_DESCRIPTOR {
  const KSFILTER_DISPATCH     *Dispatch;
  const KSAUTOMATION_TABLE    *AutomationTable;
  ULONG                       Version;
  ULONG                       Flags;
  const GUID                  *ReferenceGuid;
  ULONG                       PinDescriptorsCount;
  ULONG                       PinDescriptorSize;
  const KSPIN_DESCRIPTOR_EX   *PinDescriptors;
  ULONG                       CategoriesCount;
  const GUID                  *Categories;
  ULONG                       NodeDescriptorsCount;
  ULONG                       NodeDescriptorSize;
  const KSNODE_DESCRIPTOR     *NodeDescriptors;
  ULONG                       ConnectionsCount;
  const KSTOPOLOGY_CONNECTION *Connections;
  const KSCOMPONENTID         *ComponentId;
} KSFILTER_DESCRIPTOR, *PKSFILTER_DESCRIPTOR;
```
[KSFILTER_DESCRIPTOR](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksfilter_descriptor) 中的[KSFILTER_DISPATCH](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksfilter_dispatch) 是Filter的回调函数，在Create函数中创建Filter。
```c
typedef struct _KSFILTER_DISPATCH {
  PFNKSFILTERIRP     Create;
  PFNKSFILTERIRP     Close;
  PFNKSFILTERPROCESS Process;
  PFNKSFILTERVOID    Reset;
} KSFILTER_DISPATCH, *PKSFILTER_DISPATCH;
```
[KSFILTER_DESCRIPTOR](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksfilter_descriptor) 中的[KSPIN_DESCRIPTOR_EX](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_kspin_descriptor_ex)
```c
typedef struct _KSPIN_DESCRIPTOR_EX {
  const KSPIN_DISPATCH         *Dispatch;
  const KSAUTOMATION_TABLE     *AutomationTable;
  KSPIN_DESCRIPTOR             PinDescriptor;
  ULONG                        Flags;
  ULONG                        InstancesPossible;
  ULONG                        InstancesNecessary;
  const KSALLOCATOR_FRAMING_EX *AllocatorFraming;
  PFNKSINTERSECTHANDLEREX      IntersectHandler;
} KSPIN_DESCRIPTOR_EX, *PKSPIN_DESCRIPTOR_EX;
```
[KSPIN_DESCRIPTOR_EX](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_kspin_descriptor_ex) 中的[KSPIN_DISPATCH](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_kspin_dispatch)
```c
typedef struct _KSPIN_DISPATCH {
  PFNKSPINIRP                Create;
  PFNKSPINIRP                Close;
  PFNKSPIN                   Process;
  PFNKSPINVOID               Reset;
  PFNKSPINSETDATAFORMAT      SetDataFormat;
  PFNKSPINSETDEVICESTATE     SetDeviceState;
  PFNKSPIN                   Connect;
  PFNKSPINVOID               Disconnect;
  const KSCLOCK_DISPATCH     *Clock;
  const KSALLOCATOR_DISPATCH *Allocator;
} KSPIN_DISPATCH, *PKSPIN_DISPATCH;
```
[KSPIN_DESCRIPTOR_EX](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_kspin_descriptor_ex) 中的[KSPIN_DESCRIPTOR](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-kspin_descriptor)
```c
typedef struct KSPIN_DESCRIPTOR {
  ULONG                 InterfacesCount;
  const KSPIN_INTERFACE *Interfaces;
  ULONG                 MediumsCount;
  const KSPIN_MEDIUM    *Mediums;
  ULONG                 DataRangesCount;
  const PKSDATARANGE    *DataRanges;
  KSPIN_DATAFLOW        DataFlow;
  KSPIN_COMMUNICATION   Communication;
  const GUID            *Category;
  const GUID            *Name;
  union {
    LONGLONG Reserved;
    struct {
      ULONG        ConstrainedDataRangesCount;
      PKSDATARANGE *ConstrainedDataRanges;
    };
  };
}  *PKSPIN_DESCRIPTOR;
```
[KSPIN_DESCRIPTOR](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-kspin_descriptor) 中的[KSDATARANGE](https://msdn.microsoft.com/library/windows/hardware/ff561658)，关于MajorFormat，SubFormat（[Uncompressed RGB Video Subtypes](https://docs.microsoft.com/en-us/windows/desktop/directshow/uncompressed-rgb-video-subtypes)，[YUV Video Subtypes](https://docs.microsoft.com/en-us/windows/desktop/directshow/yuv-video-subtypes)），Specifier参考[Stream Categories](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/stream/stream-categories)
```c
typedef union {
  struct {
    ULONG FormatSize;
    ULONG Flags;
    ULONG SampleSize;
    ULONG Reserved;
    GUID  MajorFormat;
    GUID  SubFormat;
    GUID  Specifier;
  };
  LONGLONG Alignment;
} KSDATAFORMAT, *PKSDATAFORMAT, KSDATARANGE, *PKSDATARANGE;
```
SubFormat GUID和[FOURCC Codes](https://docs.microsoft.com/en-us/windows/desktop/directshow/fourcc-codes)，[Video FOURCCs](https://docs.microsoft.com/en-us/windows/desktop/medfound/video-fourccs) 有关，FOURCC Codes可以转换成Subtype GUIDs（参考[FOURCC Codes](https://docs.microsoft.com/en-us/windows/desktop/directshow/fourcc-codes)）
```c
MAKEFOURCC('Y','U','Y','2');
```
实际使用中，使用包含[KSDATARANGE](https://msdn.microsoft.com/library/windows/hardware/ff561658)的数据类型，比如[KS_DATARANGE_VIDEO](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-tagks_datarange_video) ，
```c
typedef struct tagKS_DATARANGE_VIDEO {
  KSDATARANGE                 DataRange;
  BOOL                        bFixedSizeSamples;
  BOOL                        bTemporalCompression;
  DWORD                       StreamDescriptionFlags;
  DWORD                       MemoryAllocationFlags;
  KS_VIDEO_STREAM_CONFIG_CAPS ConfigCaps;
  KS_VIDEOINFOHEADER          VideoInfoHeader;
} KS_DATARANGE_VIDEO, *PKS_DATARANGE_VIDEO;
```
通过DataRange中的MajorFormat GUID标志是什么类型的DataRange，比如[KS_DATARANGE_VIDEO](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-tagks_datarange_video) 的MajorFormat GUID是
```c
STATICGUIDOF (KSDATAFORMAT_TYPE_VIDEO)
```
[KS_DATARANGE_VIDEO](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-tagks_datarange_video) 中的[KS_VIDEO_STREAM_CONFIG_CAPS](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-_ks_video_stream_config_caps)
```c
typedef struct _KS_VIDEO_STREAM_CONFIG_CAPS {
  GUID     guid;
  ULONG    VideoStandard;
  SIZE     InputSize;
  SIZE     MinCroppingSize;
  SIZE     MaxCroppingSize;
  int      CropGranularityX;
  int      CropGranularityY;
  int      CropAlignX;
  int      CropAlignY;
  SIZE     MinOutputSize;
  SIZE     MaxOutputSize;
  int      OutputGranularityX;
  int      OutputGranularityY;
  int      StretchTapsX;
  int      StretchTapsY;
  int      ShrinkTapsX;
  int      ShrinkTapsY;
  LONGLONG MinFrameInterval;
  LONGLONG MaxFrameInterval;
  LONG     MinBitsPerSecond;
  LONG     MaxBitsPerSecond;
} KS_VIDEO_STREAM_CONFIG_CAPS, *PKS_VIDEO_STREAM_CONFIG_CAPS;
```
[KS_DATARANGE_VIDEO](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-tagks_datarange_video) 中的[KS_VIDEOINFOHEADER](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-tagks_videoinfoheader)，初始化之后从KSPIN可获得。
```c
typedef struct tagKS_VIDEOINFOHEADER {
  RECT                rcSource;
  RECT                rcTarget;
  DWORD               dwBitRate;
  DWORD               dwBitErrorRate;
  REFERENCE_TIME      AvgTimePerFrame;
  KS_BITMAPINFOHEADER bmiHeader;
} KS_VIDEOINFOHEADER, *PKS_VIDEOINFOHEADER;
```
[KS_VIDEOINFOHEADER](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-tagks_videoinfoheader) 中的[KS_BITMAPINFOHEADER](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ksmedia/ns-ksmedia-tagks_bitmapinfoheader)
```c
typedef struct tagKS_BITMAPINFOHEADER {
  DWORD biSize;
  LONG  biWidth;
  LONG  biHeight;
  WORD  biPlanes;
  WORD  biBitCount;
  DWORD biCompression;
  DWORD biSizeImage;
  LONG  biXPelsPerMeter;
  LONG  biYPelsPerMeter;
  DWORD biClrUsed;
  DWORD biClrImportant;
} KS_BITMAPINFOHEADER, *PKS_BITMAPINFOHEADER;
```
[DECLARE_SIMPLE_FRAMING_EX](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ksallocator_framing_ex)
```c
#define DECLARE_SIMPLE_FRAMING_EX(FramingExName, MemoryType, Flags, Frames, Alignment, MinFrameSize, MaxFrameSize) \
    const KSALLOCATOR_FRAMING_EX FramingExName = \
    {\
        1, \
        0, \
        {\
            1, \
            1, \
            0 \
        }, \
        0, \
        {\
            {\
                MemoryType, \
                STATIC_KS_TYPE_DONT_CARE, \
                0, \
                0, \
                Flags, \
                Frames, \
                Alignment, \
                0, \
                {\
                    0, \
                    (ULONG)-1, \
                    1 \
                }, \
                {\
                    {\
                        MinFrameSize, \
                        MaxFrameSize, \
                        1 \
                    }, \
                    0, \
                    0  \
                }\
            }\
        }\
    }
    

```
[KSALLOCATOR_FRAMING_EX](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ksallocator_framing_ex)
```c
typedef struct KSALLOCATOR_FRAMING_EX {
  ULONG           CountItems;
  ULONG           PinFlags;
  KS_COMPRESSION  OutputCompression;
  ULONG           PinWeight;
  KS_FRAMING_ITEM FramingItem[1];
}  *PKSALLOCATOR_FRAMING_EX;
```
[KSALLOCATOR_FRAMING_EX](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ksallocator_framing_ex) 中的[KS_FRAMING_ITEM](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ks_framing_item)
```c
typedef struct KS_FRAMING_ITEM {
  GUID                      MemoryType;
  GUID                      BusType;
  ULONG                     MemoryFlags;
  ULONG                     BusFlags;
  ULONG                     Flags;
  ULONG                     Frames;
  union {
    ULONG FileAlignment;
    LONG  FramePitch;
  };
  ULONG                     MemoryTypeWeight;
  KS_FRAMING_RANGE          PhysicalRange;
  KS_FRAMING_RANGE_WEIGHTED FramingRange;
}  *PKS_FRAMING_ITEM;
```
[KS_FRAMING_ITEM](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ks_framing_item) 中的[KS_FRAMING_RANGE](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ks_framing_range)
```c
typedef struct KS_FRAMING_RANGE {
  ULONG MinFrameSize;
  ULONG MaxFrameSize;
  ULONG Stepping;
}  *PKS_FRAMING_RANGE;
```
[KS_FRAMING_ITEM](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ks_framing_item) 中的[KS_FRAMING_RANGE_WEIGHTED](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ks_framing_range_weighted)
```c
typedef struct KS_FRAMING_RANGE_WEIGHTED {
  KS_FRAMING_RANGE Range;
  ULONG            InPlaceWeight;
  ULONG            NotInPlaceWeight;
}  *PKS_FRAMING_RANGE_WEIGHTED;
```
[KSSTREAM_POINTER](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksstream_pointer)
```c
typedef struct _KSSTREAM_POINTER {
  PVOID                    Context;
  PKSPIN                   Pin;
  PKSSTREAM_HEADER         StreamHeader;
  PKSSTREAM_POINTER_OFFSET Offset;
  KSSTREAM_POINTER_OFFSET  OffsetIn;
  KSSTREAM_POINTER_OFFSET  OffsetOut;
} KSSTREAM_POINTER, *PKSSTREAM_POINTER;
```
[KSSTREAM_POINTER](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksstream_pointer) 中的[KSSTREAM_HEADER](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-ksstream_header)
```c
typedef struct KSSTREAM_HEADER {
  ULONG    Size;
  ULONG    TypeSpecificFlags;
  KSTIME   PresentationTime;
  LONGLONG Duration;
  ULONG    FrameExtent;
  ULONG    DataUsed;
  PVOID    Data;
  ULONG    OptionsFlags;
  ULONG    Reserved;
}  *PKSSTREAM_HEADER;
```
[KSSTREAM_POINTER](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksstream_pointer) 中的[KSSTREAM_POINTER_OFFSET](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksstream_pointer_offset)，其中Data用于[Common Buffer DMA in AVStream](https://docs.microsoft.com/en-us/windows-hardware/drivers/stream/common-buffer-dma-in-avstream)，Mappings用于[Packet-based DMA in AVStream](https://docs.microsoft.com/en-us/windows-hardware/drivers/stream/packet-based-dma-in-avstream)
```c
typedef struct _KSSTREAM_POINTER_OFFSET {
  union {
    PUCHAR     Data;
    PKSMAPPING Mappings;
  };
  PUCHAR Data;
  PVOID  Alignment;
  ULONG  Count;
  ULONG  Remaining;
} KSSTREAM_POINTER_OFFSET, *PKSSTREAM_POINTER_OFFSET;
```
[KSSTREAM_POINTER_OFFSET](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksstream_pointer_offset) 中的[KSMAPPING](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/ns-ks-_ksmapping)
```c
typedef struct _KSMAPPING {
  PHYSICAL_ADDRESS PhysicalAddress;
  ULONG            ByteCount;
  ULONG            Alignment;
} KSMAPPING, *PKSMAPPING;

```

# 函数说明
[_KsEdit](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ks/nf-ks-_ksedit)
```c
#define KsEdit(Object,PointerToPointer,Tag)\
    _KsEdit(\
        (Object)->Bag,\
        (PVOID*)(PointerToPointer),\
        sizeof(**(PointerToPointer)),\
        sizeof(**(PointerToPointer)),\
        (Tag))
        
KSDDKAPI NTSTATUS _KsEdit(
  KSOBJECT_BAG ObjectBag,
  PVOID        *PointerToPointerToItem,
  ULONG        NewSize,
  ULONG        OldSize,
  ULONG        Tag
);
```
