# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [内核线程同步之signal](https://www.jianshu.com/p/a89e2153fa31)
> [irq 29: nobody cared (try booting with the "irqpoll" option) 问题说明](https://blog.csdn.net/liuhuahan/article/details/26729547)
> [Linux 内核引导选项简介](http://www.jinbuguo.com/kernel/boot_parameters.html)
> [linux中断源码分析 - 中断发生(三)](http://www.cnblogs.com/tolimit/p/4444850.html#3971276)
> [linux中断源码分析 - 概述(一)](https://www.cnblogs.com/tolimit/p/4390724.html)
> [Linux中断机制](https://www.cnblogs.com/edver/p/7260696.html)

# 从设备树获取中断
常用函数如下，
```c
//drivers\base\platform.c
platform_get_irq
  //drivers\of\irq.c
  of_irq_get //CONFIG_OF_IRQ
    of_irq_parse_one
    irq_find_host
    irq_create_of_mapping
  platform_get_resource

//drivers\of\irq.c
irq_of_parse_and_map
  of_irq_parse_one
  irq_create_of_mapping
```
中断设备树编写，比如arm的gic中断，
- The 1st cell is the interrupt type; 0 for SPI interrupts, 1 for PPI interrupts.
- The 2nd cell contains the interrupt number for the interrupt type.
  SPI interrupts are in the range [0-987].  PPI interrupts are in the range [0-15].
- The 3rd cell is the flags, encoded as follows: bits[3:0] trigger type and level flags. 1 = low-to-high edge triggered 2 = high-to-low edge triggered 4 = active high level-sensitive 8 = active low level-sensitive bits[15:8] PPI interrupt cpu mask.  Each bit corresponds to each of the 8 possible cpus attached to the GIC.  A bit set to '1' indicated the interrupt is wired to that CPU.  Only valid for PPI interrupts.

# 中断函数编写
## tasklet
软中断中执行，当tasklet在执行的时候，不会重复进入。

## worker工作队列
可重复进入。

## 内核线程
内核线程函数体常用循环控制条件，其中`signal_pending`用来接收`kill -SIGKILL <pid>`，`kthread_should_stop`用来接收`kthread_stop`，
```cpp
#include <linux/module.h>
#include <linux/kthread.h>
#include <linux/delay.h>

static struct task_struct * slam_thread = NULL;
static int is_signal_exited = 0;

static int func(void *data) 
{
    allow_signal(SIGKILL);
    mdelay(1000);
	while(!signal_pending(current) && !kthread_should_stop()) {
	    set_current_state(TASK_INTERRUPTIBLE);
	    schedule_timeout(msecs_to_jiffies(5000));
	}
	is_signal_exited = 1;
	return 0;
}
static __init int kthread_signal_example_init(void)
{
        slam_thread = kthread_run(func, NULL, "slam");
        return 0;
}
static __exit void kthread_signal_example_exit(void)
{
    if(!is_signal_exited && !IS_ERR(slam_thread))
        kthread_stop(slam_thread);
}
module_init(kthread_signal_example_init);
module_exit(kthread_signal_example_exit);
```

# irq 18: nobody cared (try booting with the "irqpoll" option)
调试nvme存储发现，
```shell
irq 18: nobody cared (try booting with the "irqpoll" option)
CPU: 2 PID: 1477 Comm: kworker/2:3 Tainted: G           O    4.1.35-rt41-fdk-1.0.0-20190116.1935 #35
Workqueue: events .nvme_async_probe [nvme]
Call Trace:
[c0000000fffab520] [c000000000872e6c] .dump_stack+0xac/0xec (unreliable)
[c0000000fffab5b0] [c0000000000852d0] .__report_bad_irq+0x4c/0x138
[c0000000fffab650] [c000000000085a3c] .note_interrupt+0x2f0/0x34c
[c0000000fffab700] [c000000000082054] .handle_irq_event_percpu+0x150/0x200
[c0000000fffab7d0] [c00000000008215c] .handle_irq_event+0x58/0xa4
[c0000000fffab850] [c0000000000861bc] .handle_fasteoi_irq+0xd4/0x280
[c0000000fffab8d0] [c0000000000813b0] .generic_handle_irq+0x4c/0x70
[c0000000fffab950] [c000000000005ac8] .__do_irq+0x5c/0xa8
[c0000000fffab9c0] [c000000000005bfc] .do_IRQ+0xe8/0x118
[c0000000fffaba50] [c000000000000d28] restore_check_irq_replay+0x2c/0x70
--- interrupt: 501 at .arch_local_irq_restore+0x60/0x70
    LR = .arch_local_irq_restore+0x60/0x70
[c0000000fffabd40] [c000000000068900] .vtime_common_account_irq_enter+0x34/0x60 (unreliable)
[c0000000fffabdb0] [c00000000003d43c] .__do_softirq+0xd8/0x314
[c0000000fffabeb0] [c00000000003dba8] .irq_exit+0xb8/0xe4
[c0000000fffabf20] [c000000000005ad0] .__do_irq+0x64/0xa8
[c0000000fffabf90] [c000000000013018] .call_do_irq+0x14/0x24
[c0000000f1c033b0] [c000000000005b98] .do_IRQ+0x84/0x118
[c0000000f1c03440] [c00000000001793c] exc_0x500_common+0xfc/0x100
--- interrupt: 501 at .arch_local_irq_restore+0x60/0x70
    LR = .arch_local_irq_restore+0x60/0x70
[c0000000f1c03730] [c0000000f1c03820] 0xc0000000f1c03820 (unreliable)
[c0000000f1c037a0] [c00000000086ffb4] ._raw_spin_unlock_irqrestore+0x60/0x74
[c0000000f1c03810] [c000000000084514] .__setup_irq+0x478/0x7d4
[c0000000f1c038c0] [c000000000084a58] .request_threaded_irq+0x110/0x23c
[c0000000f1c03970] [80000000009211e0] .queue_request_irq+0x4c/0x8c [nvme]
[c0000000f1c039e0] [8000000000922ab8] .nvme_dev_start.part.45+0x1ac/0x4e4 [nvme]
[c0000000f1c03ab0] [80000000009231d0] .nvme_async_probe+0xec/0x670 [nvme]
[c0000000f1c03ba0] [c000000000053424] .process_one_work+0x1f8/0x438
[c0000000f1c03c40] [c0000000000537e4] .worker_thread+0x180/0x5b0
[c0000000f1c03d30] [c0000000000595a4] .kthread+0xf0/0x110
[c0000000f1c03e30] [c000000000000998] .ret_from_kernel_thread+0x58/0xc0
handlers:
[<800000000092b3d0>] .nvme_irq [nvme]
[<800000000092b3d0>] .nvme_irq [nvme]
Disabling IRQ #18
```
搜索发现，当一个中断号上有多个中断共享的时候，该中断来的时候，内核会依次调用共享该中断号的各个中断处理函数，如果中断处理函数检测到该中断不是自己的中断时就会返回IRQ_NONE,这时内核就会调用下一个中断处理函数，而这些中断处理函数中必须至少有一个返回IRQ_HANDLED告知内核该中断是自己的中断，已经正常处理，若内核依次调用完所有该中断号的中断处理函数仍未得到IRQ_HANDLED的返回值，内核就会报告上述错误，并在该中断出现一定次数后关闭该中断。即只有中断处理函数返回 IRQ_HANDLED ,这个中断才是被正确完成的的。
由于PCIe Switch上还有一个FPGA，怀疑是FPGA乱报中断导致。

# 内核中断相关引导参数
引导参数可通过u-boot bootargs传入。
[KNL]
threadirqs
强制线程化所有的中断处理器(明确标记为IRQF_NO_THREAD的除外)
[HW]
irqfixup
用于修复简单的中断问题：当一个中断没有被处理时搜索所有可用的中断处理器。用于解决某些简单的固件缺陷。
[HW]
irqpoll
用于修复高级的中断问题：当一个中断没有被处理时搜索所有可用的中断处理器，并且对每个时钟中断都进行搜索。用于解决某些严重的固件缺陷。

# Setting trigger mode 8 for irq 166 failed
报错来自`kernel\irq\manage.c`，
```c
int __irq_set_trigger(struct irq_desc *desc, unsigned int irq,
		      unsigned long flags)
{
	struct irq_chip *chip = desc->irq_data.chip;
	int ret, unmask = 0;

	if (!chip || !chip->irq_set_type) {
		/*
		 * IRQF_TRIGGER_* but the PIC does not support multiple
		 * flow-types?
		 */
		pr_debug("No set_type function for IRQ %d (%s)\n", irq,
			 chip ? (chip->name ? : "unknown") : "unknown");
		return 0;
	}

	flags &= IRQ_TYPE_SENSE_MASK;

	if (chip->flags & IRQCHIP_SET_TYPE_MASKED) {
		if (!irqd_irq_masked(&desc->irq_data))
			mask_irq(desc);
		if (!irqd_irq_disabled(&desc->irq_data))
			unmask = 1;
	}

	/* caller masked out all except trigger mode flags */
	ret = chip->irq_set_type(&desc->irq_data, flags);

	switch (ret) {
	case IRQ_SET_MASK_OK:
	case IRQ_SET_MASK_OK_DONE:
		irqd_clear(&desc->irq_data, IRQD_TRIGGER_MASK);
		irqd_set(&desc->irq_data, flags);

	case IRQ_SET_MASK_OK_NOCOPY:
		flags = irqd_get_trigger_type(&desc->irq_data);
		irq_settings_set_trigger_mask(desc, flags);
		irqd_clear(&desc->irq_data, IRQD_LEVEL);
		irq_settings_clr_level(desc);
		if (flags & IRQ_TYPE_LEVEL_MASK) {
			irq_settings_set_level(desc);
			irqd_set(&desc->irq_data, IRQD_LEVEL);
		}

		ret = 0;
		break;
	default:
		pr_err("Setting trigger mode %lu for irq %u failed (%pF)\n",
		       flags, irq, chip->irq_set_type);
	}
	if (unmask)
		unmask_irq(desc);
	return ret;
}
```
说明`irq_set_type`函数返回了错误值，查看`drivers\irqchip\irq-gic.c`，CPU是cortex-a9，zynq7000，
```c
static int gic_set_type(struct irq_data *d, unsigned int type)
{
	void __iomem *base = gic_dist_base(d);
	unsigned int gicirq = gic_irq(d);

	/* Interrupt configuration for SGIs can't be changed */
	if (gicirq < 16)
		return -EINVAL;

	if (type != IRQ_TYPE_LEVEL_HIGH && type != IRQ_TYPE_EDGE_RISING)
		return -EINVAL;

	raw_spin_lock(&irq_controller_lock);

	if (gic_arch_extn.irq_set_type)
		gic_arch_extn.irq_set_type(d, type);

	gic_configure_irq(gicirq, type, base, NULL);

	raw_spin_unlock(&irq_controller_lock);

	return 0;
}
```
驱动里只支持高电平和上升沿触发，和芯片手册是匹配的，我使用了低电平触发引发的这个错误。
![379](https://img-blog.csdnimg.cn/20201223183116144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)


