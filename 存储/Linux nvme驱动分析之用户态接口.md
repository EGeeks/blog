# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 使用
执行flush，read，write操作，
```c
#define NVME_IOCTL_SUBMIT_PIO	_IOW('N', 0x50, struct nvme_user_io)
#define NVME_IOCTL_SUBMIT_PIO_PMD	_IOW('N', 0x51, struct nvme_user_io)

struct nvme_user_io user_io;
struct nvme_passthru_cmd pt_cmd = {
	.opcode		= nvme_cmd_flush,
	.nsid		= 1,
};

ret = ioctl(fd[i], NVME_IOCTL_IO_CMD, &pt_cmd);
if (ret < 0) {
	printf("ioctl NVME_IOCTL_IO_CMD error, errno=%d\n", errno);
	return -1;
}

memset(&user_io, 0, sizeof(struct nvme_user_io));
user_io.opcode = (cmd == CMD_WRITE_DISK) ? nvme_cmd_write : nvme_cmd_read;
user_io.addr = sysStart;
user_io.slba = diskStart >> 9;
user_io.nblocks = rwLength >> 9;

ret = ioctl(fd[i], NVME_IOCTL_SUBMIT_IO, &user_io);
if (ret < 0) {
    printf("ioctl NVME_IOCTL_SUBMIT_IO error, errno=%d\n", errno);
    return -1;
}
```
# 字符设备用户态接口
内核版本4.1，在驱动初始化时注册字符设备类nvme_class，
```c
/*linux-qoriq-v2.0\drivers\block\nvme-core.c nvme_init*/
result = __register_chrdev(nvme_char_major, 0, NVME_MINORS, "nvme",
						&nvme_dev_fops);
if (result < 0)
	goto unregister_blkdev;
else if (result > 0)
	nvme_char_major = result;

nvme_class = class_create(THIS_MODULE, "nvme");
if (IS_ERR(nvme_class)) {
	result = PTR_ERR(nvme_class);
	goto unregister_chrdev;
}
```
内核版本4.1，在PCIe设备驱动probe中，创建字符设备，其中dev->instance是通过内核ida接口得到的从0开始的序号，
```c
/*linux-qoriq-v2.0\drivers\block\nvme-core.c nvme_probe*/
dev->device = device_create(nvme_class, &pdev->dev,
			MKDEV(nvme_char_major, dev->instance),
			dev, "nvme%d", dev->instance);
```
内核版本4.1，nvme驱动的字符设备`/dev/nvme%d`fops，
```c
/*linux-qoriq-v2.0\drivers\block\nvme-core.c*/
static const struct file_operations nvme_dev_fops = {
	.owner		= THIS_MODULE,
	.open		= nvme_dev_open,
	.release	= nvme_dev_release,
	.unlocked_ioctl	= nvme_dev_ioctl,
	.compat_ioctl	= nvme_dev_ioctl,
};
```
ioctl接口，内核4.14增加了3条命令，NVME_IOCTL_RESET等，这里的命令只支持一个namespace的情况，
```c
/*linux-qoriq-v2.0\drivers\block\nvme-core.c*/
static long nvme_dev_ioctl(struct file *f, unsigned int cmd, unsigned long arg)
{
	struct nvme_dev *dev = f->private_data;
	struct nvme_ns *ns;

	switch (cmd) {
	case NVME_IOCTL_ADMIN_CMD:
		return nvme_user_cmd(dev, NULL, (void __user *)arg);
	case NVME_IOCTL_IO_CMD:
		if (list_empty(&dev->namespaces))
			return -ENOTTY;
		ns = list_first_entry(&dev->namespaces, struct nvme_ns, list);
		return nvme_user_cmd(dev, ns, (void __user *)arg);
	default:
		return -ENOTTY;
	}
}
```
# 块设备用户态接口
内核版本4.1，在驱动初始化时注册块设备，
```c
/*linux-qoriq-v2.0\drivers\block\nvme-core.c nvme_init*/
result = register_blkdev(nvme_major, "nvme");
if (result < 0)
	goto kill_workq;
else if (result > 0)
	nvme_major = result;
```
内核版本4.1，在nvme_alloc_ns中，为每个namespace创建块设备，这里nvme%dn%d类似于，sata里的sd%c%d，
```c
/*linux-qoriq-v2.0\drivers\block\nvme-core.c*/
static void nvme_alloc_ns(struct nvme_dev *dev, unsigned nsid)
{
	struct nvme_ns *ns;
	struct gendisk *disk;
	int node = dev_to_node(&dev->pci_dev->dev);

	ns = kzalloc_node(sizeof(*ns), GFP_KERNEL, node);
	if (!ns)
		return;

	ns->queue = blk_mq_init_queue(&dev->tagset);
	if (IS_ERR(ns->queue))
		goto out_free_ns;
	queue_flag_set_unlocked(QUEUE_FLAG_NOMERGES, ns->queue);
	queue_flag_set_unlocked(QUEUE_FLAG_NONROT, ns->queue);
	queue_flag_set_unlocked(QUEUE_FLAG_SG_GAPS, ns->queue);
	ns->dev = dev;
	ns->queue->queuedata = ns;

	disk = alloc_disk_node(0, node);
	if (!disk)
		goto out_free_queue;

	ns->ns_id = nsid;
	ns->disk = disk;
	ns->lba_shift = 9; /* set to a default value for 512 until disk is validated */
	list_add_tail(&ns->list, &dev->namespaces);

	blk_queue_logical_block_size(ns->queue, 1 << ns->lba_shift);
	if (dev->max_hw_sectors)
		blk_queue_max_hw_sectors(ns->queue, dev->max_hw_sectors);
	if (dev->stripe_size)
		blk_queue_chunk_sectors(ns->queue, dev->stripe_size >> 9);
	if (dev->vwc & NVME_CTRL_VWC_PRESENT)
		blk_queue_flush(ns->queue, REQ_FLUSH | REQ_FUA);

	disk->major = nvme_major;
	disk->first_minor = 0;
	disk->fops = &nvme_fops;
	disk->private_data = ns;
	disk->queue = ns->queue;
	disk->driverfs_dev = dev->device;
	disk->flags = GENHD_FL_EXT_DEVT;
	sprintf(disk->disk_name, "nvme%dn%d", dev->instance, nsid);

	/*
	 * Initialize capacity to 0 until we establish the namespace format and
	 * setup integrity extentions if necessary. The revalidate_disk after
	 * add_disk allows the driver to register with integrity if the format
	 * requires it.
	 */
	set_capacity(disk, 0);
	nvme_revalidate_disk(ns->disk);
	add_disk(ns->disk);
	if (ns->ms)
		revalidate_disk(ns->disk);
	return;
 out_free_queue:
	blk_cleanup_queue(ns->queue);
 out_free_ns:
	kfree(ns);
}
```
内核版本4.1，块设备fops，
```c
/*linux-qoriq-v2.0\drivers\block\nvme-core.c*/
static const struct block_device_operations nvme_fops = {
	.owner		= THIS_MODULE,
	.ioctl		= nvme_ioctl,
	.compat_ioctl	= nvme_compat_ioctl,
	.open		= nvme_open,
	.release	= nvme_release,
	.getgeo		= nvme_getgeo,
	.revalidate_disk= nvme_revalidate_disk,
};
```
内核版本4.1，IOCTL接口，SG_IO为SCSI命令提供了一个兼容转换，
```c
/*linux-qoriq-v2.0\drivers\block\nvme-core.c*/
static int nvme_ioctl(struct block_device *bdev, fmode_t mode, unsigned int cmd,
							unsigned long arg)
{
	struct nvme_ns *ns = bdev->bd_disk->private_data;

	switch (cmd) {
	case NVME_IOCTL_ID:
		force_successful_syscall_return();
		return ns->ns_id;
	case NVME_IOCTL_ADMIN_CMD:
		return nvme_user_cmd(ns->dev, NULL, (void __user *)arg);
	case NVME_IOCTL_IO_CMD:
		return nvme_user_cmd(ns->dev, ns, (void __user *)arg);
	case NVME_IOCTL_SUBMIT_IO:
		return nvme_submit_io(ns, (void __user *)arg);
	case SG_GET_VERSION_NUM:
		return nvme_sg_get_version_num((void __user *)arg);
	case SG_IO:
		return nvme_sg_io(ns, (void __user *)arg);
	default:
		return -ENOTTY;
	}
}
```

