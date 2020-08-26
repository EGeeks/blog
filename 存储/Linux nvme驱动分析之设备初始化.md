# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [独家发布 | Linux NVMe Driver学习笔记大合集](https://blog.csdn.net/zhuzongpeng/article/details/76136164)
> [linux内核源码分析 - nvme设备的初始化](https://www.cnblogs.com/tolimit/p/8779876.html)
> [强势回归，Linux blk用实力证明自己并不弱！](https://blog.csdn.net/yedushu/article/details/82050520)
> [详解linux io flush](https://www.cnblogs.com/kungf/p/11995054.html)

# 驱动初始化
驱动`nvme_probe`过程，
```c
nvme_probe
  dev = kzalloc
  dev->entry = kzalloc //msix entry
  nvme_set_instance
  nvme_setup_prp_pools
  nvme_dev_start
    nvme_dev_map
      pci_enable_device_mem
      pci_set_master
      ioremap
      pci_enable_msix_range
      readq(&dev->bar->cap);
    nvme_configure_admin_queue
      nvme_disable_ctrl
      nvme_alloc_queue
      nvme_enable_ctrl
      queue_request_irq
      nvme_init_queue
    nvme_setup_io_queues
      nr_io_queues = num_possible_cpus();
      set_queue_count
        nvme_set_features NVME_FEAT_NUM_QUEUES
      iounmap(dev->bar);
      ioremap(pci_resource_start(pdev, 0), size);
      dev->dbs = ((void __iomem *)dev->bar) + 4096;
      adminq->q_db = dev->dbs;
      free_irq(dev->entry[0].vector, adminq);
      pci_disable_msix(pdev);
      pci_enable_msix_range(pdev, dev->entry, 1, nr_io_queues);
      pci_enable_msi_range(pdev, 1, min(nr_io_queues, 32));
      queue_request_irq(dev, adminq, adminq->irqname);
	  nvme_free_queues(dev, nr_io_queues + 1);
      nvme_create_io_queues(dev);
  nvme_dev_add
    nvme_identify
    nvme_alloc_ns
      blk_alloc_queue
      queue_flag_set_unlocked
      blk_queue_make_request
      blk_queue_logical_block_size
      blk_queue_max_hw_sectors
      blk_queue_flush
      set_capacity

```

# 中断与轮询
> [schedule_timeout与mdelay的区别](https://www.cnblogs.com/muryo/p/4106208.html)
> [schedule_timeout函数](https://blog.csdn.net/u011046042/article/details/79013490)
> [linux时间---延迟和定时](http://blog.chinaunix.net/uid-27189249-id-3943964.html)
> [内核定时机制API之__round_jiffies_relative](https://blog.csdn.net/tiantao2012/article/details/79294890)

在4.1版本的内核，有一个内核线程nvme_kthread，负责1s轮询一下所以NVMe盘，看有没有队列完成，在4.14版本的内核中，blk mq增加了poll接口，该功能由函数`nvme_timeout`和`nvme_poll`取代，`nvme_timeout`调用`nvme_poll`会查看当前是否有没有处理的完成队列，4.1版本的`nvme_timeout`直接取消当前的request。
```c
static const struct blk_mq_ops nvme_mq_admin_ops = {
	.queue_rq	= nvme_queue_rq,
	.complete	= nvme_pci_complete_rq,
	.init_hctx	= nvme_admin_init_hctx,
	.exit_hctx      = nvme_admin_exit_hctx,
	.init_request	= nvme_init_request,
	.timeout	= nvme_timeout,
};

static const struct blk_mq_ops nvme_mq_ops = {
	.queue_rq	= nvme_queue_rq,
	.complete	= nvme_pci_complete_rq,
	.init_hctx	= nvme_init_hctx,
	.init_request	= nvme_init_request,
	.map_queues	= nvme_pci_map_queues,
	.timeout	= nvme_timeout,
	.poll		= nvme_poll,
};
```

