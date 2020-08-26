# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> 基于Xilinx Zynq Petalinux2015.2.1，linux3.19内核

AHCI（sata）相比nvme是老技术，快淘汰了，进入公司的时候，差不多已经做完了，没啥新东西给你做了，丧失了掌握AHCI（sata）的机会，现在我对nvme的熟悉程度甚至比AHCI（sata）高，那老技术还要不要学呢，不学的话，偶尔用到的时候还是感觉被束缚。

# 初始化
```c
//drivers\ata\ahci.c
ahci_init_one //PCIe驱动probe入口
  struct ata_port_info pi = ahci_port_info[board_id];
  hpriv->flags |= (unsigned long)pi.private_data; 
  ahci_pci_save_initial_config
    ahci_save_initial_config
      hpriv->cap = cap;
	  hpriv->cap2 = cap2;
	  hpriv->port_map = port_map;
	  hpriv->start_engine = ahci_start_engine;
  ahci_init_interrupts //共享中断或者每个port有自己的中断
  ata_host_alloc_pinfo //分配linux内核变量ata_host
    ata_host_alloc
      ata_port_alloc
   	ap->pio_mask = pi->pio_mask;
	ap->mwdma_mask = pi->mwdma_mask;
	ap->udma_mask = pi->udma_mask;
	ap->flags |= pi->flags;
	ap->link.flags |= pi->link_flags;
	ap->ops = pi->port_ops;
  ahci_reset_em //enclosure message
  ata_port_pbar_desc
    ata_port_desc //构建端口的描述信息
  ahci_configure_dma_masks
  ahci_pci_reset_controller //ahci协议，配置寄存器
  ahci_pci_init_controller
    ahci_init_controller //ahci协议，配置寄存器
      ahci_port_init //ahci协议，配置寄存器
  ahci_pci_print_info //打印controller的能力信息，sata版本等等
  ahci_host_activate //启动ahci，注册中断，注册ata_host
```
通过`board_id`找到`ata_port_info`，`pi.private_data`来自`AHCI_HFLAGS(flags)`初始化，并传给`hpriv->flags`，例如Marvell的88se9125，88se9230的`board_id`是`board_ahci_yes_fbs`，`hpriv->flags`就会被初始化为`AHCI_HFLAG_YES_FBS`，
```c
//drivers\ata\ahci.h line218
#define AHCI_HFLAGS(flags)		.private_data	= (void *)(flags)
//drivers\ata\ahci.c
static const struct ata_port_info ahci_port_info[] = {
	/* by features */
	[board_ahci] = {
		.flags		= AHCI_FLAG_COMMON,
		.pio_mask	= ATA_PIO4,
		.udma_mask	= ATA_UDMA6,
		.port_ops	= &ahci_ops,
	},
	[board_ahci_ign_iferr] = {
		AHCI_HFLAGS	(AHCI_HFLAG_IGN_IRQ_IF_ERR),
		.flags		= AHCI_FLAG_COMMON,
		.pio_mask	= ATA_PIO4,
		.udma_mask	= ATA_UDMA6,
		.port_ops	= &ahci_ops,
	},
...
	[board_ahci_yes_fbs] = {
		AHCI_HFLAGS	(AHCI_HFLAG_YES_FBS),
		.flags		= AHCI_FLAG_COMMON,
		.pio_mask	= ATA_PIO4,
		.udma_mask	= ATA_UDMA6,
		.port_ops	= &ahci_ops,
	},
...
}
```
其中`port_ops`采用`ahci_ops`，
```c
//drivers\ata\ahci.h
struct ata_port_operations ahci_ops = {
	.inherits		= &sata_pmp_port_ops,

	.qc_defer		= ahci_pmp_qc_defer,
	.qc_prep		= ahci_qc_prep,
	.qc_issue		= ahci_qc_issue,
	.qc_fill_rtf		= ahci_qc_fill_rtf,

	.freeze			= ahci_freeze,
	.thaw			= ahci_thaw,
	.softreset		= ahci_softreset,
	.hardreset		= ahci_hardreset,
	.postreset		= ahci_postreset,
	.pmp_softreset		= ahci_softreset,
	.error_handler		= ahci_error_handler,
	.post_internal_cmd	= ahci_post_internal_cmd,
	.dev_config		= ahci_dev_config,

	.scr_read		= ahci_scr_read,
	.scr_write		= ahci_scr_write,
	.pmp_attach		= ahci_pmp_attach,
	.pmp_detach		= ahci_pmp_detach,

	.set_lpm		= ahci_set_lpm,
	.em_show		= ahci_led_show,
	.em_store		= ahci_led_store,
	.sw_activity_show	= ahci_activity_show,
	.sw_activity_store	= ahci_activity_store,
	.transmit_led_message	= ahci_transmit_led_message,
#ifdef CONFIG_PM
	.port_suspend		= ahci_port_suspend,
	.port_resume		= ahci_port_resume,
#endif
	.port_start		= ahci_port_start,
	.port_stop		= ahci_port_stop,
};
EXPORT_SYMBOL_GPL(ahci_ops);

struct ata_port_operations ahci_pmp_retry_srst_ops = {
	.inherits		= &ahci_ops,
	.softreset		= ahci_pmp_retry_softreset,
};
EXPORT_SYMBOL_GPL(ahci_pmp_retry_srst_ops);
```
以`ahci_scr_read/ahci_scr_write`为例，
```c
static unsigned ahci_scr_offset(struct ata_port *ap, unsigned int sc_reg)
{
	static const int offset[] = {
		[SCR_STATUS]		= PORT_SCR_STAT,
		[SCR_CONTROL]		= PORT_SCR_CTL,
		[SCR_ERROR]		= PORT_SCR_ERR,
		[SCR_ACTIVE]		= PORT_SCR_ACT,
		[SCR_NOTIFICATION]	= PORT_SCR_NTF,
	};
	struct ahci_host_priv *hpriv = ap->host->private_data;

	if (sc_reg < ARRAY_SIZE(offset) &&
	    (sc_reg != SCR_NOTIFICATION || (hpriv->cap & HOST_CAP_SNTF)))
		return offset[sc_reg];
	return 0;
}

static int ahci_scr_read(struct ata_link *link, unsigned int sc_reg, u32 *val)
{
	void __iomem *port_mmio = ahci_port_base(link->ap);
	int offset = ahci_scr_offset(link->ap, sc_reg);

	if (offset) {
		*val = readl(port_mmio + offset);
		return 0;
	}
	return -EINVAL;
}

static int ahci_scr_write(struct ata_link *link, unsigned int sc_reg, u32 val)
{
	void __iomem *port_mmio = ahci_port_base(link->ap);
	int offset = ahci_scr_offset(link->ap, sc_reg);

	if (offset) {
		writel(val, port_mmio + offset);
		return 0;
	}
	return -EINVAL;
}
```
参考协议，`ahci_scr_read/ahci_scr_write`是读写`SCR0~SCR4`寄存器，函数`ahci_host_activate`是linux的ahci库函数，方便实现一个ahci驱动，
```c
ahci_host_activate
  ahci_host_activate_multi_irqs //每个port有自己的中断
    ata_host_start
    devm_request_threaded_irq
    ata_port_desc
    ata_host_register
  ata_host_activate //共享中断
    ata_host_start
      ata_finalize_port_ops
      ap->ops->port_start -> ahci_port_start
        ahci_port_resume
          ahci_power_up
          ahci_start_port
          ahci_pmp_attach
      ata_eh_freeze_port
        __ata_port_freeze
          ap->ops->freeze -> ahci_freeze
    ata_host_register//没有中断直接注册，自动进入轮询模式
    devm_request_irq
    ata_port_desc
    ata_host_register
```
注册ata_host结构体时传入`struct scsi_host_template`，
```c
//drivers\ata\ahci.h
static struct scsi_host_template ahci_sht = {
	AHCI_SHT("ahci"),
};
#define AHCI_SHT(drv_name)						\
	ATA_NCQ_SHT(drv_name),						\
	.can_queue		= AHCI_MAX_CMDS - 1,			\
	.sg_tablesize		= AHCI_MAX_SG,				\
	.dma_boundary		= AHCI_DMA_BOUNDARY,			\
	.shost_attrs		= ahci_shost_attrs,			\
	.sdev_attrs		= ahci_sdev_attrs
//drivers\ata\libahci.c
struct device_attribute *ahci_shost_attrs[] = {
	&dev_attr_link_power_management_policy,
	&dev_attr_em_message_type,
	&dev_attr_em_message,
	&dev_attr_ahci_host_caps,
	&dev_attr_ahci_host_cap2,
	&dev_attr_ahci_host_version,
	&dev_attr_ahci_port_cmd,
	&dev_attr_em_buffer,
	&dev_attr_em_message_supported,
	NULL
};
EXPORT_SYMBOL_GPL(ahci_shost_attrs);

struct device_attribute *ahci_sdev_attrs[] = {
	&dev_attr_sw_activity,
	&dev_attr_unload_heads,
	NULL
};
EXPORT_SYMBOL_GPL(ahci_sdev_attrs);
```
比如`dev_attr_ahci_host_caps`实现，sysfs接口调试使用，
```c
static ssize_t ahci_show_host_caps(struct device *dev,
				   struct device_attribute *attr, char *buf)
{
	struct Scsi_Host *shost = class_to_shost(dev);
	struct ata_port *ap = ata_shost_to_port(shost);
	struct ahci_host_priv *hpriv = ap->host->private_data;

	return sprintf(buf, "%x\n", hpriv->cap);
}
```
调用了`ap->ops->port_start`和`ap->ops->freeze`，对应
```c
//drivers\ata\libahci.c
static int ahci_port_start(struct ata_port *ap)
{
	struct ahci_host_priv *hpriv = ap->host->private_data;
	struct device *dev = ap->host->dev;
	struct ahci_port_priv *pp;
	void *mem;
	dma_addr_t mem_dma;
	size_t dma_sz, rx_fis_sz;

	pp = devm_kzalloc(dev, sizeof(*pp), GFP_KERNEL);
	if (!pp)
		return -ENOMEM;

	if (ap->host->n_ports > 1) {
		pp->irq_desc = devm_kzalloc(dev, 8, GFP_KERNEL);
		if (!pp->irq_desc) {
			devm_kfree(dev, pp);
			return -ENOMEM;
		}
		snprintf(pp->irq_desc, 8,
			 "%s%d", dev_driver_string(dev), ap->port_no);
	}

	/* check FBS capability */
	if ((hpriv->cap & HOST_CAP_FBS) && sata_pmp_supported(ap)) {
		void __iomem *port_mmio = ahci_port_base(ap);
		u32 cmd = readl(port_mmio + PORT_CMD);
		/* FIS-based Switching Capable Port (FBSCP): When set to ‘1’, indicates that this port 
supports Port Multiplier FIS-based switching.  When cleared to ‘0’, indicates that this 
port  does  not  support  FIS-based  switching.  This  bit  may  only  be  set  to  ‘1’  if  both 
CAP.SPM and CAP.FBSS are set to ‘1’. */
		if (cmd & PORT_CMD_FBSCP)
			pp->fbs_supported = true;
		else if (hpriv->flags & AHCI_HFLAG_YES_FBS) {
			dev_info(dev, "port %d can do FBS, forcing FBSCP\n",
				 ap->port_no);
			pp->fbs_supported = true;
		} else
			dev_warn(dev, "port %d is not capable of FBS\n",
				 ap->port_no);
	}

	if (pp->fbs_supported) {
		dma_sz = AHCI_PORT_PRIV_FBS_DMA_SZ;
		rx_fis_sz = AHCI_RX_FIS_SZ * 16;
	} else {
		dma_sz = AHCI_PORT_PRIV_DMA_SZ;
		rx_fis_sz = AHCI_RX_FIS_SZ;
	}

	mem = dmam_alloc_coherent(dev, dma_sz, &mem_dma, GFP_KERNEL);
	if (!mem)
		return -ENOMEM;
	memset(mem, 0, dma_sz);

	/*
	 * First item in chunk of DMA memory: 32-slot command table,
	 * 32 bytes each in size
	 */
	pp->cmd_slot = mem;
	pp->cmd_slot_dma = mem_dma;

	mem += AHCI_CMD_SLOT_SZ;
	mem_dma += AHCI_CMD_SLOT_SZ;

	/*
	 * Second item: Received-FIS area
	 */
	pp->rx_fis = mem;
	pp->rx_fis_dma = mem_dma;

	mem += rx_fis_sz;
	mem_dma += rx_fis_sz;

	/*
	 * Third item: data area for storing a single command
	 * and its scatter-gather table
	 */
	pp->cmd_tbl = mem;
	pp->cmd_tbl_dma = mem_dma;

	/*
	 * Save off initial list of interrupts to be enabled.
	 * This could be changed later
	 */
	pp->intr_mask = DEF_PORT_IRQ;

	/*
	 * Switch to per-port locking in case each port has its own MSI vector.
	 */
	if ((hpriv->flags & AHCI_HFLAG_MULTI_MSI)) {
		spin_lock_init(&pp->lock);
		ap->lock = &pp->lock;
	}

	ap->private_data = pp;

	/* engage engines, captain */
	return ahci_port_resume(ap);
}

static void ahci_port_stop(struct ata_port *ap)
{
	const char *emsg = NULL;
	int rc;

	/* de-initialize port */
	rc = ahci_deinit_port(ap, &emsg);
	if (rc)
		ata_port_warn(ap, "%s (%d)\n", emsg, rc);
}

static void ahci_freeze(struct ata_port *ap)
{
	void __iomem *port_mmio = ahci_port_base(ap);

	/* turn IRQ off */
	writel(0, port_mmio + PORT_IRQ_MASK);
}

static void ahci_thaw(struct ata_port *ap)
{
	struct ahci_host_priv *hpriv = ap->host->private_data;
	void __iomem *mmio = hpriv->mmio;
	void __iomem *port_mmio = ahci_port_base(ap);
	u32 tmp;
	struct ahci_port_priv *pp = ap->private_data;

	/* clear IRQ */
	tmp = readl(port_mmio + PORT_IRQ_STAT);
	writel(tmp, port_mmio + PORT_IRQ_STAT);
	writel(1 << ap->port_no, mmio + HOST_IRQ_STAT);

	/* turn IRQ back on */
	writel(pp->intr_mask, port_mmio + PORT_IRQ_MASK);
}
```
`ahci_port_start`使用到一些宏，其实是来自于枚举，并给ahci分配命令表内存，
```c
//drivers\ata\ahci.h
enum {
	AHCI_MAX_PORTS		= 32, //最大端口数
	AHCI_MAX_CLKS		= 5,
	AHCI_MAX_SG		= 168, /* hardware max is 64K */ //PRDT数量168，这应该是内核代码错误，64K这地方应该是16，作者多按了一个8
	AHCI_DMA_BOUNDARY	= 0xffffffff,
	AHCI_MAX_CMDS		= 32, //最大Command Header数量
	AHCI_CMD_SZ		= 32, //Command Header大小
	AHCI_CMD_SLOT_SZ	= AHCI_MAX_CMDS * AHCI_CMD_SZ,
	AHCI_RX_FIS_SZ		= 256, //Received FIS Structure大小
	AHCI_CMD_TBL_CDB	= 0x40,
	AHCI_CMD_TBL_HDR_SZ	= 0x80, //Command Table的CFIS + ACMD + Reserved
	AHCI_CMD_TBL_SZ		= AHCI_CMD_TBL_HDR_SZ + (AHCI_MAX_SG * 16),
	AHCI_CMD_TBL_AR_SZ	= AHCI_CMD_TBL_SZ * AHCI_MAX_CMDS,
	AHCI_PORT_PRIV_DMA_SZ	= AHCI_CMD_SLOT_SZ + AHCI_CMD_TBL_AR_SZ +
				  AHCI_RX_FIS_SZ,
	AHCI_PORT_PRIV_FBS_DMA_SZ	= AHCI_CMD_SLOT_SZ + //端口复用，最大复用数量16
					  AHCI_CMD_TBL_AR_SZ +
					  (AHCI_RX_FIS_SZ * 16),
...
}
```
