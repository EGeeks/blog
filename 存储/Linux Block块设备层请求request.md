# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 定义
不同版本的Linux，块层改动较大，增加了blk-mq之后，可谓是面目全非。

## Xilinx Petalinux2015.2.1 Linux3.19.0，
```c
//include\linux\blk_types.h
/*
 * Request flags.  For use in the cmd_flags field of struct request, and in
 * bi_rw of struct bio.  Note that some flags are only valid in either one.
 */
enum rq_flag_bits {
	/* common flags */
	__REQ_WRITE,		/* not set, read. set, write */
	__REQ_FAILFAST_DEV,	/* no driver retries of device errors */
	__REQ_FAILFAST_TRANSPORT, /* no driver retries of transport errors */
	__REQ_FAILFAST_DRIVER,	/* no driver retries of driver errors */

	__REQ_SYNC,		/* request is sync (sync write or read) */
	__REQ_META,		/* metadata io request */
	__REQ_PRIO,		/* boost priority in cfq */
	__REQ_DISCARD,		/* request to discard sectors */
	__REQ_SECURE,		/* secure discard (used with __REQ_DISCARD) */
	__REQ_WRITE_SAME,	/* write same block many times */

	__REQ_NOIDLE,		/* don't anticipate more IO after this one */
	__REQ_INTEGRITY,	/* I/O includes block integrity payload */
	__REQ_FUA,		/* forced unit access */
	__REQ_FLUSH,		/* request for cache flush */
...
};

#define REQ_WRITE		(1ULL << __REQ_WRITE)
#define REQ_FAILFAST_DEV	(1ULL << __REQ_FAILFAST_DEV)
#define REQ_FAILFAST_TRANSPORT	(1ULL << __REQ_FAILFAST_TRANSPORT)
#define REQ_FAILFAST_DRIVER	(1ULL << __REQ_FAILFAST_DRIVER)
#define REQ_SYNC		(1ULL << __REQ_SYNC)
#define REQ_META		(1ULL << __REQ_META)
#define REQ_PRIO		(1ULL << __REQ_PRIO)
#define REQ_DISCARD		(1ULL << __REQ_DISCARD)
#define REQ_WRITE_SAME		(1ULL << __REQ_WRITE_SAME)
#define REQ_NOIDLE		(1ULL << __REQ_NOIDLE)
#define REQ_INTEGRITY		(1ULL << __REQ_INTEGRITY)
...
```

## Xilinx Petalinux2018.2 Linux4.14.0，
```c
//include\linux\blk_types.h
#define REQ_OP_BITS	8
#define REQ_OP_MASK	((1 << REQ_OP_BITS) - 1)
#define REQ_FLAG_BITS	24

enum req_opf {
	/* read sectors from the device */
	REQ_OP_READ		= 0,
	/* write sectors to the device */
	REQ_OP_WRITE		= 1,
	/* flush the volatile write cache */
	REQ_OP_FLUSH		= 2,
	/* discard sectors */
	REQ_OP_DISCARD		= 3,
	/* get zone information */
	REQ_OP_ZONE_REPORT	= 4,
	/* securely erase sectors */
	REQ_OP_SECURE_ERASE	= 5,
	/* seset a zone write pointer */
	REQ_OP_ZONE_RESET	= 6,
	/* write the same sector many times */
	REQ_OP_WRITE_SAME	= 7,
	/* write the zero filled sector many times */
	REQ_OP_WRITE_ZEROES	= 9,

	/* SCSI passthrough using struct scsi_request */
	REQ_OP_SCSI_IN		= 32,
	REQ_OP_SCSI_OUT		= 33,
	/* Driver private requests */
	REQ_OP_DRV_IN		= 34,
	REQ_OP_DRV_OUT		= 35,

	REQ_OP_LAST,
};

enum req_flag_bits {
	__REQ_FAILFAST_DEV =	/* no driver retries of device errors */
		REQ_OP_BITS,
	__REQ_FAILFAST_TRANSPORT, /* no driver retries of transport errors */
	__REQ_FAILFAST_DRIVER,	/* no driver retries of driver errors */
	__REQ_SYNC,		/* request is sync (sync write or read) */
	__REQ_META,		/* metadata io request */
	__REQ_PRIO,		/* boost priority in cfq */
	__REQ_NOMERGE,		/* don't touch this for merging */
	__REQ_IDLE,		/* anticipate more IO after this one */
	__REQ_INTEGRITY,	/* I/O includes block integrity payload */
	__REQ_FUA,		/* forced unit access */
	__REQ_PREFLUSH,		/* request for cache flush */
	__REQ_RAHEAD,		/* read ahead, can fail anytime */
	__REQ_BACKGROUND,	/* background IO */

	/* command specific flags for REQ_OP_WRITE_ZEROES: */
	__REQ_NOUNMAP,		/* do not free blocks when zeroing */

	__REQ_NOWAIT,           /* Don't wait if request will block */
	__REQ_NR_BITS,		/* stops here */
};

#define REQ_FAILFAST_DEV	(1ULL << __REQ_FAILFAST_DEV)
#define REQ_FAILFAST_TRANSPORT	(1ULL << __REQ_FAILFAST_TRANSPORT)
#define REQ_FAILFAST_DRIVER	(1ULL << __REQ_FAILFAST_DRIVER)
#define REQ_SYNC		(1ULL << __REQ_SYNC)
#define REQ_META		(1ULL << __REQ_META)
#define REQ_PRIO		(1ULL << __REQ_PRIO)
#define REQ_NOMERGE		(1ULL << __REQ_NOMERGE)
#define REQ_IDLE		(1ULL << __REQ_IDLE)
#define REQ_INTEGRITY		(1ULL << __REQ_INTEGRITY)
#define REQ_FUA			(1ULL << __REQ_FUA)
#define REQ_PREFLUSH		(1ULL << __REQ_PREFLUSH)
#define REQ_RAHEAD		(1ULL << __REQ_RAHEAD)
#define REQ_BACKGROUND		(1ULL << __REQ_BACKGROUND)
```
在REQ_OP这里，4.14.0的内核增加了`REQ_OP_SCSI`和`REQ_OP_DRV`，对于我们的异构系统，可添加几组宏来获得支持，
```c
/* SCSI passthrough using struct scsi_request */
REQ_OP_SCSI_IN		= 32,
REQ_OP_SCSI_OUT		= 33,
/* Driver private requests */
REQ_OP_DRV_IN		= 34,
REQ_OP_DRV_OUT		= 35,

//scsi驱动
static int sg_io(struct request_queue *q, struct gendisk *bd_disk,
		struct sg_io_hdr *hdr, fmode_t mode)
{
...
	rq = blk_get_request(q, writing ? REQ_OP_SCSI_OUT : REQ_OP_SCSI_IN,
			GFP_KERNEL);
	if (IS_ERR(rq))
		return PTR_ERR(rq);
	req = scsi_req(rq);
...
}

//nvme驱动
struct request *nvme_alloc_request(struct request_queue *q,
		struct nvme_command *cmd, unsigned int flags, int qid)
{
	unsigned op = nvme_is_write(cmd) ? REQ_OP_DRV_OUT : REQ_OP_DRV_IN;
	struct request *req;

	if (qid == NVME_QID_ANY) {
		req = blk_mq_alloc_request(q, op, flags);
	} else {
		req = blk_mq_alloc_request_hctx(q, op, flags,
				qid ? qid - 1 : 0);
	}
	if (IS_ERR(req))
		return req;

	req->cmd_flags |= REQ_FAILFAST_DRIVER;
	nvme_req(req)->cmd = cmd;

	return req;
}
```
两者都用到一个`blk_mq_rq_to_pdu`，来得到`request`的环境变量，
```bash
#define BLK_MAX_CDB	16

struct scsi_request {
	unsigned char	__cmd[BLK_MAX_CDB];
	unsigned char	*cmd;
	unsigned short	cmd_len;
	int		result;
	unsigned int	sense_len;
	unsigned int	resid_len;	/* residual count */
	int		retries;
	void		*sense;
};

static inline struct scsi_request *scsi_req(struct request *rq)
{
	return blk_mq_rq_to_pdu(rq);
}

/*
 * Common request structure for NVMe passthrough.  All drivers must have
 * this structure as the first member of their request-private data.
 */
struct nvme_request {
	struct nvme_command	*cmd;
	union nvme_result	result;
	u8			retries;
	u8			flags;
	u16			status;
};

enum {
	NVME_REQ_CANCELLED		= (1 << 0),
	NVME_REQ_PHY			= (1 << 6),
	NVME_REQ_PHY_PMD		= (1 << 7),
};

static inline struct nvme_request *nvme_req(struct request *req)
{
	return blk_mq_rq_to_pdu(req);
}
```
在队列的处理函数中，不管是传统单队列还是多队列，
```c
//传统单队列
static void scsi_request_fn(struct request_queue *q)
	__releases(q->queue_lock)
	__acquires(q->queue_lock)
{
...
	cmd = blk_mq_rq_to_pdu(req);
...
}

static blk_status_t scsi_queue_rq(struct blk_mq_hw_ctx *hctx,
			 const struct blk_mq_queue_data *bd)
{
...
	struct scsi_cmnd *cmd = blk_mq_rq_to_pdu(req);
...
}

```

