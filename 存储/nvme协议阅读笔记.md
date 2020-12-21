# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [nvme官网](https://nvmexpress.org/)
> [NVMe over Fabrics 概况](https://www.cnblogs.com/JamesLi/p/11511082.html)

# Controller Registers
##  CAP – Controller Capabilities
CAP的低16位，指示了主控支持的最大IO队列的深度，16位最大是65536，这就是64K最大队列深度的来源。
![327](https://img-blog.csdnimg.cn/20200818095405509.png#pic_center)

##  AQA – Admin Queue Attributes
Admin队列，最大深度是4K，
![329](https://img-blog.csdnimg.cn/20200818113027912.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)

## SQyTDBL – Submission Queue y Tail Doorbell
![330](https://img-blog.csdnimg.cn/20200818151016599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
由驱动写入通知主控Tail位置，在发起传输的时，提交SQ使用。

## CQyHDBL  –  Completion  Queue  y  Head Doorbell
![331](https://img-blog.csdnimg.cn/20200818151130226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
由驱动写入通知主控Head位置，在中断里，处理完CQ时使用。

# Identify
##  Identify Controller data structure (CNS 01h)
VWC，在controller的信息中有一个Volatile Write Cache (VWC)位，
![286](https://img-blog.csdnimg.cn/20200318151859451.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
MDTS，`max_hw_sectors = 1 << (id->mdts + page_shift - 9)`，单次传输的最大大小是有限制的，所以PRP的链表不支持无限长的，实际测试三星970pro的传输大小最大2MB，
![292](https://img-blog.csdnimg.cn/20200428182029508.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
NVMe over Fabrics使用`qualified naming`授权命名寻址约定。NVMe Qualified Name（NQN）用于识别远程NVMe存储目标。它类似于iSCSI限定名(IQN)。[2047:1792]是NVMEoF的协议内容，
![293](https://img-blog.csdnimg.cn/20200506143557761.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
参考NVMEoF协议，
![294](https://img-blog.csdnimg.cn/20200506145303349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
> mandatory: 强制的; 法定的; 义务的
> capsule: (装药物的)胶囊;(装物或装液体的)小塑料容器;太空舱;航天舱

例如`IOCCSZ`指示`nvme_command`的长度，
```c
//drivers\nvme\host\rdma.c
static int nvme_rdma_alloc_queue(struct nvme_rdma_ctrl *ctrl,
		int idx, size_t queue_size)
{
...
	if (idx > 0)
		queue->cmnd_capsule_len = ctrl->ctrl.ioccsz * 16;
	else
		queue->cmnd_capsule_len = sizeof(struct nvme_command);
...
}
```
## Identify Namespace data structure (CNS 00h) 
![296](https://img-blog.csdnimg.cn/20200506151519923.png)
![295](https://img-blog.csdnimg.cn/20200506151404979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
所以，`ns->ext`表示`set to ‘1’ indicates that the metadata is transferred at the end of the data LBA`，metadata和数据一起发送还是存在一个单独的缓存里。`ns->ms`表示主控支持的LBA格式，
```c
//drivers\nvme\host\core.c
#ifdef CONFIG_BLK_DEV_INTEGRITY
static void nvme_prep_integrity(struct gendisk *disk, struct nvme_id_ns *id,
		u16 bs)
{
...
	ns->ms = le16_to_cpu(id->lbaf[id->flbas & NVME_NS_FLBAS_LBA_MASK].ms);
	ns->ext = ns->ms && (id->flbas & NVME_NS_FLBAS_META_EXT);

	/* PI implementation requires metadata equal t10 pi tuple size */
	if (ns->ms == sizeof(struct t10_pi_tuple))
		pi_type = id->dps & NVME_NS_DPS_PI_MASK;
...
#endif /* CONFIG_BLK_DEV_INTEGRITY */
```
LBA格式，`RP`表示该种格式的性能，`LBADS`表示LBA大小，不得小于512，`MS`表示metadata的大小，
![297](https://img-blog.csdnimg.cn/20200506152623484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
在执行格式化命令`Format NVM command`的时候可以选择LBA格式，通常，盘加载后有且仅有一个ns，这个格式化命令没有从内核驱动中得到实现。
![298](https://img-blog.csdnimg.cn/20200506153637994.png)
# Completion Queue
Status位于DW3的高15位，
![314](https://img-blog.csdnimg.cn/20200615161357726.png)
## Status
![328](https://img-blog.csdnimg.cn/20200818111934166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
其中`Phase Tag`位在队列完成一次队列深度的传输之后取反，在驱动中处理完成队列会检查这个位，通过`Phase Tag`信息能判断中Tail的位置。
![315](https://img-blog.csdnimg.cn/20200615161712187.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
SCT，状态码的类型，
![316](https://img-blog.csdnimg.cn/20200615161742162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
状态码被分为三组，
- 00h to 7Fh: Applicable to Admin Command Set, or across multiple command sets; 
- 80h to BFh: I/O Command Set Specific status codes; and 
- C0h to FFh: Vendor Specific status codes. 

多个队列同时传输时三星970pro返回的status是`8194`，`0x2002`，返回`SC`为`Invalid Command Opcode: A reserved coded value or an unsupported value in the command opcode field. `，`More`标志置1，`More (M): If set to ‘1’, there is more status information for this command as part of the Error Information log that may be retrieved with the Get Log Page command. If cleared to ‘0’, there is no additional status information for this command. Refer to section 5.14.1.1.`

# DSM-Dataset Management 
> [NVMe SSD新功能Reservation从入门到精通](https://www.sohu.com/a/272502796_311575)
> [面壁UNH IOL NVMe一致性测试之9 – Dataset Management command ](https://www.sohu.com/a/270669832_505795)
> [NVMe之命令](https://blog.csdn.net/u010616442/article/details/70804470)

Dataset Management command可以通过设置Dword 11的Attribute-Deallocate（AD）字段deallocate一定范围的LBA，deallocate也就是通常所说的Trim。SSD收到AD字段为1的Dataset Management command后，会将相应范围的LBA Trim掉。如果Host针对被Trim的地址发送read命令，SSD应该返回全1，全0或者最后写入的数据。如果使能了deallocated 或 unwritten logical block error，当Host读取被deallocate区域时，SSD会返回该命令失败并且错误为unwritten或者deallocated logical block error，如果对一个被deallocate的LBA做写操作会导致deallocate状态消失，读操作则没有影响。

# HMB
> [东芝RC100 M.2 NVMe固态硬盘HMB特性解读](https://zhuanlan.zhihu.com/p/42649209)
> [4.7 Controller Memory Buffer](https://www.cnblogs.com/hswy/p/12669320.html)
> [How to enable host memory buffer in Windows10 by registry key?](https://social.technet.microsoft.com/Forums/lync/en-US/04699f35-834c-46dd-96f0-c845b07aaab9/how-to-enable-host-memory-buffer-in-windows10-by-registry-key?forum=win10itprosetup)
> [NVMe Host Memory Buffer主机内存缓冲效果](http://bbs.pceva.com.cn/thread-134884-1-1.html)
> [面壁UNH IOL NVMe一致性测试之17 – Host Memory Buffer](http://www.ssdfans.com/?p=106801)
> [StorPortAllocateHostMemoryBuffer function (storport.h)](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/ddi/storport/nf-storport-storportallocatehostmemorybuffer)

简言之，HMB是给DRAMLess SSD设计的用来存放FTL表的，肯定是比板载的DRAM延迟要大，毕竟经过了PCIe传输。
![hmb](https://img-blog.csdnimg.cn/20201219172209120.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)


