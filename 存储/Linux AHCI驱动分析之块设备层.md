# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

接上一篇博客[Linux AHCI驱动分析之设备初始化](https://blog.csdn.net/Zhu_Zhu_2009/article/details/105966212)

# 参考
> [ATA Disk在Linux中的驱动架构对比分析](https://blog.csdn.net/leino11121/article/details/6585017)
> [ata驱动框架及scsi请求处理流程](https://blog.csdn.net/qq_37403371/article/details/84567666)
> [ATA接口寄存器描述](https://blog.csdn.net/mao0514/article/details/32135815)
> [从ATA层向设备发送TRIM命令](https://blog.csdn.net/computerms/article/details/7868371)
> [使用硬盘ATA命令读取磁盘](https://blog.csdn.net/wrmsr/article/details/8744527)
> [scsi底层设备注册——如何一步步注册到block层](https://blog.csdn.net/qqqqqq999999/article/details/45506463)
> [Scsi命令队列转换为ata命令过程](https://blog.csdn.net/dianhuiren/article/details/7164099)
> [scsi设备的请求处理函数（request_fn)](https://blog.csdn.net/weixin_36145588/article/details/72878602)
> [libATA Developer's Guide](https://www.kernel.org/doc/htmldocs/libata/)
> [块设备读写流程](https://www.cnblogs.com/codestub/articles/2152532.html)
> [块设备读写流程](http://blog.chinaunix.net/uid-25052030-id-58337.html)
> [Linux Block Layer块设备层基于MultiQueue的部分源码分析](https://blog.csdn.net/g382112762/article/details/79606485)
> [为request的每一个bio创建DMA映射](https://blog.csdn.net/weixin_36145588/article/details/72897864)
> [Linux kernel scatterlist API介绍](http://www.wowotech.net/memory_management/scatterlist.html)

# 打开内核调试信息
定义`ATA_DEBUG`和`ATA_VERBOSE_DEBUG`，
```c
//include\linux\libata.h
/*
 * compile-time options: to be removed as soon as all the drivers are
 * converted to the new debugging mechanism
 */
#undef ATA_DEBUG		/* debugging output */
#undef ATA_VERBOSE_DEBUG	/* yet more debugging output */
#undef ATA_IRQ_TRAP		/* define to ack screaming irqs */
#undef ATA_NDEBUG		/* define to disable quick runtime checks */


/* note: prints function name for you */
#ifdef ATA_DEBUG
#define DPRINTK(fmt, args...) printk(KERN_ERR "%s: " fmt, __func__, ## args)
#ifdef ATA_VERBOSE_DEBUG
#define VPRINTK(fmt, args...) printk(KERN_ERR "%s: " fmt, __func__, ## args)
#else
#define VPRINTK(fmt, args...)
#endif	/* ATA_VERBOSE_DEBUG */
#else
#define DPRINTK(fmt, args...)
#define VPRINTK(fmt, args...)
#endif	/* ATA_DEBUG */
//drivers\ata\libata-core.c
struct ata_port *ata_port_alloc(struct ata_host *host)
{
...
#if defined(ATA_VERBOSE_DEBUG)
	/* turn on all debugging levels */
	ap->msg_enable = 0x00FF;
#elif defined(ATA_DEBUG)
	ap->msg_enable = ATA_MSG_DRV | ATA_MSG_INFO | ATA_MSG_CTL | ATA_MSG_WARN | ATA_MSG_ERR;
#else
	ap->msg_enable = ATA_MSG_DRV | ATA_MSG_ERR | ATA_MSG_WARN;
#endif
...
}
```

# 驱动模型
驱动模型如下图，通过LibATA驱动作为SCSI Middle Level与ATA Host之间的转换层，从而可以很好的将ATA Host直接融入到SCSI的驱动体系中来，可以直接将ATA设备驱成SCSI Device。
![scsi](https://img-blog.csdnimg.cn/20200508161911961.gif)

# 注册块设备
下面进入`ata_host_register`函数，
```c
ata_host_register //drivers\ata\libata-core.c
  host->n_tags = clamp(sht->can_queue, 1, ATA_MAX_QUEUE - 1); //can_queue = AHCI_MAX_CMDS - 1, n_tags=31, tag ATA_MAX_QUEUE - 1 is reserved for internal commands
  ata_tport_add
  ata_scsi_add_hosts //drivers\ata\libata-scsi.c
    scsi_host_alloc
	  shost->hostt = sht; // struct scsi_host_template *sht
      shost->can_queue = sht->can_queue; //struct scsi_host_template
	  shost->sg_tablesize = sht->sg_tablesize; //struct scsi_host_template
	  shost->use_blk_mq = scsi_use_blk_mq && !shost->hostt->disable_blk_mq; //scsi_use_blk_mq is from Kconfig
	shost->max_cmd_len = 16;
    scsi_add_host_with_dma //drivers\scsi\hosts.c
      scsi_use_blk_mq: scsi_mq_setup_tags //mq tagset
      scsi_setup_command_freelist
        scsi_get_host_cmd_pool
        scsi_host_alloc_command
      scsi_host_set_state(shost, SHOST_RUNNING)
      scsi_sysfs_add_host
      scsi_proc_host_add
  sata_link_init_spd
  ata_pack_xfermask
  async_schedule(async_port_probe, ap)
    ata_port_probe
      __ata_port_probe
      ata_port_wait_eh
    ata_scsi_scan_host //drivers\ata\libata-scsi.c
      __scsi_add_device //drivers\scsi\scsi_scan.c
        scsi_alloc_target
        scsi_probe_and_add_lun
          scsi_device_lookup_by_target
          scsi_alloc_sdev
            scsi_mq_alloc_queue/scsi_alloc_queue
            scsi_change_queue_depth
            scsi_sysfs_device_initialize
          scsi_probe_lun //探测lun
            scsi_execute_req: INQUIRY
              scsi_execute_req_flags //drivers\scsi\scsi_lib.c
                scsi_execute
                  blk_get_request
                  blk_rq_set_block_pc
                  blk_rq_map_kern
                  blk_execute_rq
          scsi_add_lun //drivers\scsi\scsi_scan.c
            scsi_sysfs_add_sdev //drivers\scsi\scsi_sysfs.c
              device_add //触发上层probe，
              bsg_register_queue //block\bsg.c  
        scsi_target_reap
```

# 块设备队列
队列创建，支持多队列和传统的单队列，Linux内核默认是单队列（3.19，4.14），请求分发函数为`scsi_queue_rq/scsi_request_fn`
```c
__scsi_init_queue //drivers\scsi\scsi_lib.c
  blk_queue_max_segments
  blk_queue_max_hw_sectors
  blk_queue_bounce_limit
  blk_queue_segment_boundary
  dma_set_seg_boundary
  blk_queue_max_segment_size
  blk_queue_dma_alignment

scsi_alloc_queue //传统单队列
  __scsi_alloc_queue
    blk_init_queue
    __scsi_init_queue
    blk_queue_prep_rq(q, scsi_prep_fn); //设置prep_rq函数
    blk_queue_unprep_rq(q, scsi_unprep_fn);
    blk_queue_softirq_done(q, scsi_softirq_done);
    blk_queue_rq_timed_out(q, scsi_times_out);
    blk_queue_lld_busy(q, scsi_lld_busy);

static struct blk_mq_ops scsi_mq_ops = {
	.map_queue	= blk_mq_map_queue,
	.queue_rq	= scsi_queue_rq,
	.complete	= scsi_softirq_done,
	.timeout	= scsi_timeout,
	.init_request	= scsi_init_request,
	.exit_request	= scsi_exit_request,
};

scsi_mq_alloc_queue //多队列mq
  blk_mq_init_queue
  __scsi_init_queue
```
以传统单队列为例，
```c
scsi_request_fn //drivers\scsi\scsi_lib.c
  blk_peek_request
    ret = q->prep_rq_fn(q, rq); //scsi_prep_fn
      scsi_prep_state_check
      scsi_get_cmd_from_req //构造struct scsi_cmnd
      scsi_setup_cmnd
        scsi_setup_fs_cmnd/scsi_setup_blk_pc_cmnd
      scsi_prep_return
  scsi_dev_queue_ready
  blk_start_request
  struct scsi_cmnd *cmd = req->special;
  scsi_target_queue_ready
  scsi_host_queue_ready
  scsi_init_cmd_errh
  cmd->scsi_done = scsi_done; //中断函数中会用到
  scsi_dispatch_cmd
    scsi_log_send
    host->hostt->queuecommand;
  scsi_queue_insert
```
其中`host->hostt->queuecommand`来自`struct scsi_host_template`，对应`ata_scsi_queuecmd`函数，函数`ata_sg_setup`中调用`dma_map_sg`，对于我们的异构系统，需要修改的就是这个函数，
```c
ata_scsi_queuecmd //drivers\ata\libata-scsi.c
  ata_shost_to_port
  ata_scsi_dump_cdb
  ata_scsi_find_dev
  __ata_scsi_queuecmd
    ata_get_xlat_func/atapi_xlat //check if SCSI to ATA translation is possible
    ata_scsi_translate/ata_scsi_simulate
    ata_scsi_translate
      ata_scsi_qc_new
      	qc->scsicmd = cmd;
		qc->scsidone = cmd->scsi_done;
		qc->sg = scsi_sglist(cmd);
		qc->n_elem = scsi_sg_count(cmd);
      ata_sg_init
      qc->complete_fn = ata_scsi_qc_complete;
      ata_get_xlat_func[ata_scsi_rw_xlat/ata_scsi_pass_thru]/atapi_xlat //translate
        struct ata_taskfile *tf //构建tf
      ap->ops->qc_defer //ahci_pmp_qc_defer， deferred：推迟; 延缓; 展期 drivers\ata\libahci.c
        ata_std_qc_defer/sata_pmp_qc_defer_cmd_switch
      ata_qc_issue
        ata_sg_setup
          dma_map_sg
        ap->ops->qc_prep //ahci_qc_prep drivers\ata\libahci.c
          ata_tf_to_fis //tf转fis，协议Command Table的CFIS
          memcpy(cmd_tbl + AHCI_CMD_TBL_CDB, qc->cdb, qc->dev->cdb_len) //协议Command Table的ACMD
          ahci_fill_sg
          ahci_fill_cmd_slot
        ap->ops->qc_issue //ahci_qc_issue drivers\ata\libahci.c
          writel(1 << qc->tag, port_mmio + PORT_SCR_ACT);
          writel(fbs, port_mmio + PORT_FBS);
          writel(1 << qc->tag, port_mmio + PORT_CMD_ISSUE);
```

# scsi驱动
scsi驱动位于`drivers\scsi\sd.c`，
```c
init_sd //drivers\scsi\sd.c
  register_blkdev
  blk_register_region
  class_register
  scsi_register_driver
```
其中`scsi_register_driver(&sd_template.gendrv)`，上面说的probe函数即`sd_probe`
```c
static struct scsi_driver sd_template = {
	.gendrv = {
		.name		= "sd",
		.owner		= THIS_MODULE,
		.probe		= sd_probe,
		.remove		= sd_remove,
		.shutdown	= sd_shutdown,
		.pm		= &sd_pm_ops,
	},
	.rescan			= sd_rescan,
	.init_command		= sd_init_command,
	.uninit_command		= sd_uninit_command,
	.done			= sd_done,
	.eh_action		= sd_eh_action,
};
```
`sd_probe`函数，磁盘的队列就是用的`scsi_device`里的队列，不需要新创建了，
```c
sd_probe //drivers\scsi\sd.c
  alloc_disk
  sd_format_disk_name
  sd_probe_async
  	gd->fops = &sd_fops;
	gd->queue = sdkp->device->request_queue; //scsi_disk->scsi_device->request_queue
    sd_revalidate_disk
    add_disk
    sd_dif_config_host
    sd_revalidate_disk
```
其中`sd_fops`如下，
```c
static const struct block_device_operations sd_fops = {
	.owner			= THIS_MODULE,
	.open			= sd_open,
	.release		= sd_release,
	.ioctl			= sd_ioctl,
	.getgeo			= sd_getgeo,
#ifdef CONFIG_COMPAT
	.compat_ioctl		= sd_compat_ioctl,
#endif
	.check_events		= sd_check_events,
	.revalidate_disk	= sd_revalidate_disk,
	.unlock_native_capacity	= sd_unlock_native_capacity,
};

sd_ioctl
  scsi_verify_blk_ioctl
  scsi_ioctl/scsi_cmd_blk_ioctl
```
其中，
```c
scsi_ioctl //block\scsi_ioctl.c
  sg_scsi_ioctl/sdev->host->hostt->ioctl //ata_scsi_ioctl
  
scsi_cmd_blk_ioctl
  scsi_verify_blk_ioctl
  scsi_cmd_ioctl
    sg_io
    
ata_scsi_ioctl //drivers\ata\libata-scsi.c
  ata_sas_scsi_ioctl
    ATA_IOC_GET_IO32
    ATA_IOC_SET_IO32
    HDIO_GET_IDENTITY ata_get_identity
    HDIO_DRIVE_CMD ata_cmd_ioctl
    HDIO_DRIVE_TASK ata_task_ioctl 
```

# 中断
以传统单队列为例，
```c
ahci_single_irq_intr //drivers\ata\libahci.c
  ahci_port_intr
    ahci_handle_port_interrupt
      ata_qc_complete_multiple //drivers\ata\libata-core.c
        ata_qc_from_tag
        ata_qc_complete
          __ata_qc_complete
          ata_sg_clean
            dma_unmap_sg
          qc->complete_fn //ata_scsi_qc_complete drivers\ata\libata-scsi.c
            ata_gen_passthru_sense/ata_gen_ata_sense
            ata_dump_status
            qc->scsidone //scsi_done drivers\scsi\scsi_lib.c
            ata_qc_free
```

