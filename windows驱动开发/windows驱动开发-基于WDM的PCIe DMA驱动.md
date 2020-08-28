# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 访问MEM IO资源

[Mapping Bus-Relative Addresses to Virtual Addresses](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/mapping-bus-relative-addresses-to-virtual-addresses)

# 连接中断

[Servicing Interrupts](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/servicing-interrupts)

[Registering an ISR](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/registering-an-isr)

[Using Message-Signaled Interrupts](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/using-message-signaled-interrupts)

[Removing an ISR](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/removing-an-isr)

[IoConnectInterrupt](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nf-wdm-ioconnectinterrupt)
MSI必须使用 [IoConnectInterruptEx](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nf-wdm-ioconnectinterruptex)，参考[WdmlibIoConnectInterruptEx](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/iointex/nf-iointex-wdmlibioconnectinterruptex)
```c
#define IoConnectInterruptEx WdmlibIoConnectInterruptEx
NTSTATUS WdmlibIoConnectInterruptEx(
  PIO_CONNECT_INTERRUPT_PARAMETERS Parameters
);
```
[IO_CONNECT_INTERRUPT_PARAMETERS](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/ns-wdm-_io_connect_interrupt_parameters) 的构造参考[Using the CONNECT_MESSAGE_BASED Version of IoConnectInterruptEx](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/using-the-connect-message-based-version-of-ioconnectinterruptex)

[Using the CONNECT_LINE_BASED Version of IoConnectInterruptEx](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/using-the-connect-line-based-version-of-ioconnectinterruptex)
[Using the CONNECT_FULLY_SPECIFIED Version of IoConnectInterruptEx](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/using-the-connect-fully-specified-version-of-ioconnectinterruptex)

# DPC

[DPC Objects and DPCs](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/dpc-objects-and-dpcs)
有两种DPC，一种是WDM驱动集成的[DpcForIsr](https://msdn.microsoft.com/library/windows/hardware/ff544079)，使用方法参考[Registering and Queuing a DpcForIsr Routine](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/registering-and-queuing-a-dpcforisr-routine)，第二个是[CustomDpc](https://msdn.microsoft.com/library/windows/hardware/ff542972)，参考[Registering and Queuing a CustomDpc Routine](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/registering-and-queuing-a-customdpc-routine)，[CustomDpc](https://msdn.microsoft.com/library/windows/hardware/ff542972)又分为CustomThreadedDpc和CustomTimerDpc， CustomThreadedDpc参考[Threaded DPCs](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/threaded-dpcs)CustomTimerDpc参考[KeXxxTimer Routines, KTIMER Objects, and DPCs](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/timer-objects-and-dpcs)。
