# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考

> [_CM_PARTIAL_RESOURCE_DESCRIPTOR structure](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/ns-wdm-_cm_partial_resource_descriptor)

# PCM_PARTIAL_RESOURCE_DESCRIPTOR

今日开发altera dma驱动，获取资源的时候得到下面的打印，显示获取的资源类型为CmResourceTypeMemoryLarge，这个资源与普通的CmResourceTypeMemory不一样，查看官网，
```c
CmResourceTypeMemoryLarge	One of u.Memory40, u.Memory48, or u.Memory64.
The CM_RESOURCE_MEMORY_LARGE_XXX flags set in the Flags member determines which structure is used.
```
实际偏移和长度存储的变量由Flags位确定，
```c
CM_RESOURCE_MEMORY_LARGE_40	The memory descriptor uses the u.Memory40 member.
CM_RESOURCE_MEMORY_LARGE_48	The memory descriptor uses the u.Memory48 member.
CM_RESOURCE_MEMORY_LARGE_64	The memory descriptor uses the u.Memory64 member.
```
同时，此时的长度需要左移一个偏移才是实际长度，
```c
u.Memory40.Start

For raw resources: Specifies the bus-relative physical address of the lowest of a range of contiguous memory addresses that are allocated to the device.

For translated resources: Specifies the system physical address of the lowest of a range of contiguous memory addresses that are allocated to the device.

For more information about raw and translated resources, see Remarks.

u.Memory40.Length40

Contains the high 32 bits of the 40-bit length, in bytes, of the range of allocated memory addresses. The lowest 8 bits are treated as zero.

u.Memory48.Start

For raw resources: Specifies the bus-relative physical address of the lowest of a range of contiguous memory addresses that are allocated to the device.

For translated resources: Specifies the system physical address of the lowest of a range of contiguous memory addresses that are allocated to the device.

For more information about raw and translated resources, see Remarks.

u.Memory48.Length48

Contains the high 32 bits of the 48-bit length, in bytes, of the range of allocated memory addresses. The lowest 16 bits are treated as zero.

u.Memory64.Start

For raw resources: Specifies the bus-relative physical address of the lowest of a range of contiguous memory addresses that are allocated to the device.

For translated resources: Specifies the system physical address of the lowest of a range of contiguous memory addresses that are allocated to the device.

For more information about raw and translated resources, see Remarks.

u.Memory64.Length64

Contains the high 32 bits of the 64-bit length, in bytes, of the range of allocated memory addresses. The lowest 32 bits are treated as zero.
```
