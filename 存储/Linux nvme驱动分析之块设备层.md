# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Product Documentation Red Hat Enterprise Linux7 7.2 发行注记 第 14 章 存储](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/7.2_release_notes/storage)
> [块设备内核参数max_segments和max_sectors_kb解析](http://blog.chinaunix.net/uid-22954220-id-4813537.html)

# blk_mq
## 数据缓冲区转换成prp或者sg列表
用户态分配的内存使用`blk_rq_map_user`，内核态分配的内存使用`blk_rq_map_kern`，
```c
//xilinx petalinux-v2018.2
blk_rq_map_user
```
对比来看，
```c
//xilinx petalinux-v2018.2
blk_rq_map_kern
  bio_copy_kern //数据buf地址不对齐，硬件能力不支持，该分支应该很少进入，copy会带来性能降低
                //类似于bounce buffer回弹缓冲区
  bio_map_kern
    bio_kmalloc //分配一个bio
    bio_add_pc_page //把page循环加入bio
```
通过上面的操作把request的内存记录到bio中，通过`blk_rq_map_sg`，形成`struct scatterlist *sg`，prp或者sg列表是通过`nvme_setup_prps`基于`struct scatterlist *sg`构造的。

## IO请求如何下发到SSD
创建队列的时候注册`blk_mq_ops`，
```bash
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
块设备层通过`blk_execute_rq`或``blk_execute_rq_nowait``把请求加入队列，然后下发到硬件，
```c
//xilinx petalinux-v2018.2
nvme_queue_rq
  nvme_setup_cmd
  nvme_init_iod
  nvme_map_data
    blk_rq_map_sg
    dma_map_sg_attrs
    nvme_setup_prps
  blk_mq_start_request
  __nvme_submit_cmd
  nvme_process_cq
```

# legacy blk
传统的块设备只有一个队列，通过make_request分发io，内核在bio这块变动太多了，
```c
#if LINUX_VERSION_CODE <= KERNEL_VERSION(4,4,0)
#define bio_op(bio) ((bio)->bi_rw & REQ_OP_MASK)
#define bio_opf(bio) ((bio)->bi_rw)
#else
#define REQ_FLUSH REQ_PREFLUSH
#define bio_opf(bio) ((bio)->bi_opf)
#endif
```
内核在申请nvmeq时多申请了一片空间，
```cpp
struct nvme_cmd_info {
	nvme_completion_fn fn;
	void *ctx;
	unsigned long timeout;
	int aborted;
};

static struct nvme_cmd_info *nvme_cmd_info(struct nvme_queue *nvmeq)
{
	return (void *)&nvmeq->cmdid_data[BITS_TO_LONGS(nvmeq->q_depth)];
}

static unsigned nvme_queue_extra(int depth)
{
	return DIV_ROUND_UP(depth, 8) + (depth * sizeof(struct nvme_cmd_info));
}

/**
 * alloc_cmdid() - Allocate a Command ID
 * @nvmeq: The queue that will be used for this command
 * @ctx: A pointer that will be passed to the handler
 * @handler: The function to call on completion
 *
 * Allocate a Command ID for a queue.  The data passed in will
 * be passed to the completion handler.  This is implemented by using
 * the bottom two bits of the ctx pointer to store the handler ID.
 * Passing in a pointer that's not 4-byte aligned will cause a BUG.
 * We can change this if it becomes a problem.
 *
 * May be called with local interrupts disabled and the q_lock held,
 * or with interrupts enabled and no locks held.
 */
static int alloc_cmdid(struct nvme_queue *nvmeq, void *ctx,
				nvme_completion_fn handler, unsigned timeout)
{
	int depth = nvmeq->q_depth - 1;
	struct nvme_cmd_info *info = nvme_cmd_info(nvmeq);
	int cmdid;

	do {
		cmdid = find_first_zero_bit(nvmeq->cmdid_data, depth);
		if (cmdid >= depth)
			return -EBUSY;
	} while (test_and_set_bit(cmdid, nvmeq->cmdid_data));

	info[cmdid].fn = handler;
	info[cmdid].ctx = ctx;
	info[cmdid].timeout = jiffies + timeout;
	info[cmdid].aborted = 0;
	return cmdid;
}

static struct nvme_queue *nvme_alloc_queue(struct nvme_dev *lp, int qid,
							int depth, int vector)
{
	unsigned extra = nvme_queue_extra(depth);
	struct nvme_queue *nvmeq = kzalloc(sizeof(*nvmeq) + extra, GFP_KERNEL);
	if (!nvmeq)
		return NULL;
...
}
```
其中，如果q_depth取1024，`nvme_queue_extra = 1024 / 8 + 1024 * sizeof(struct nvme_cmd_info)`，`BITS_TO_LONGS(nvmeq->q_depth) = DIV_ROUND_UP(1024, 8 * sizeof(long)) = 1024 / 32(64)`，`DIV_ROUND_UP(n, 8)`可算出bit数总共占多少字节，`BITS_TO_LONGS`就是求一个数是几个long的长度。感觉这块代码有点问题，q_depth取1025不就溢出了。。。
```cpp
#define DIV_ROUND_UP(n,d) (((n) + (d) - 1) / (d))
#define BITS_TO_LONGS(nr)	DIV_ROUND_UP(nr, BITS_PER_BYTE * sizeof(long))
```

# 验证
写一个测试脚本，
```shell
root@t2080rdb:~# cat test.sh 
#!/bin/sh
echo "test read"
nvmeqe_benchmark -r /dev/nvme0n1 -p 0xe2000000 -s 0 -l 0x100000 -c 128 &
nvmeqe_benchmark -r /dev/nvme0n1 -p 0xe2100000 -s 0 -l 0x100000 -c 128 &
nvmeqe_benchmark -r /dev/nvme0n1 -p 0xe2200000 -s 0 -l 0x100000 -c 128 &
nvmeqe_benchmark -r /dev/nvme0n1 -p 0xe2300000 -s 0 -l 0x100000 -c 128
```
测试结果，对比测试前后的中断统计，内核把任务平均分给了各个CPU，
```shell
root@t2080rdb:~# cat /proc/interrupts | grep nvme0
 58:          0          0        403          0          0          0          0          0  fsl-msi-263  15 Edge      nvme0q0, nvme0q1
 59:          0          0          0        109          0          0          0          0  fsl-msi-224  16 Edge      nvme0q2
 60:          0          0          0          0        118          0          0          0  fsl-msi-225  17 Edge      nvme0q3
 61:          0          0          0          0          0        755          0          0  fsl-msi-226  18 Edge      nvme0q4
 62:          0          0          0          0          0          0        206          0  fsl-msi-227  19 Edge      nvme0q5
 63:          0          0          0          0          0          0          0        439  fsl-msi-228  20 Edge      nvme0q6
 64:       1038          0          0          0          0          0          0          0  fsl-msi-229  21 Edge      nvme0q7
 65:          0        353          0          0          0          0          0          0  fsl-msi-230  22 Edge      nvme0q8
root@t2080rdb:~# ./test.sh 
test read
speed: 306.95MB/s, cost times: 417ms
speed: 306.22MB/s, cost times: 418ms
speed: 305.49MB/s, cost times: 419ms
speed: 305.49MB/s, cost times: 419ms
root@t2080rdb:~# cat /proc/interrupts | grep nvme0
 58:          0          0        461          0          0          0          0          0  fsl-msi-263  15 Edge      nvme0q0, nvme0q1
 59:          0          0          0        142          0          0          0          0  fsl-msi-224  16 Edge      nvme0q2
 60:          0          0          0          0        243          0          0          0  fsl-msi-225  17 Edge      nvme0q3
 61:          0          0          0          0          0        938          0          0  fsl-msi-226  18 Edge      nvme0q4
 62:          0          0          0          0          0          0        260          0  fsl-msi-227  19 Edge      nvme0q5
 63:          0          0          0          0          0          0          0        471  fsl-msi-228  20 Edge      nvme0q6
 64:       1092          0          0          0          0          0          0          0  fsl-msi-229  21 Edge      nvme0q7
 65:          0        385          0          0          0          0          0          0  fsl-msi-230  22 Edge      nvme0q8
```

