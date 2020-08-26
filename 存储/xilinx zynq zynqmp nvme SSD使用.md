# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Connecting an SSD to an FPGA running PetaLinux](http://www.fpgadeveloper.com/2016/04/connecting-an-ssd-to-an-fpga-running-petalinux.html)

# zynq
板卡zc706，接口：PL端PCIe Gen2 x4，三星970pro 1TB，这基本是zynq上面PCIe Root的性能，也可能是zynq的DRAM性能，SSD的速度远没有跑满。
```bash
root@zynq:~# nvmel_benchmark -r /dev/nvme0n1 -p cmem -s 0 -l 0x400000 -n 128
cmem[nvmelbufc] type=2 va=0x3690a000[0x60100000] pa=0x20100000 size=0x400000 status=0x3
disk /dev/nvme0n1 cmd 0x2, speed 204.80MB/s, cost times 2500ms
root@zynq:~# nvmel_benchmark -w /dev/nvme0n1 -p cmem -s 0 -l 0x400000 -n 128
cmem[nvmelbufc] type=2 va=0x36939000[0x60100000] pa=0x20100000 size=0x400000 status=0x3
disk /dev/nvme0n1 cmd 0x4, speed 295.95MB/s, cost times 1730ms
root@zynq:~# blk_benchmark -r /dev/nvme0n1 -s 0 -l 0x400000 -n 128
speed: 177.35MB/s, cost times: 2887ms
root@zynq:~# blk_benchmark -w /dev/nvme0n1 -s 0 -l 0x400000 -n 128
speed: 203.34MB/s, cost times: 2518ms
```
读写zc706 pl端ddr，
```bash
root@zynq:~# nvmel_benchmark -r /dev/nvme0n1 -p 0x40000000 -s 0 -l 0x400000 -n 16
disk /dev/nvme0n1 cmd 0x2, speed 1230.77MB/s, cost times 52ms
root@zynq:~# nvmel_benchmark -r /dev/nvme0n1 -p 0x40000000 -s 0 -l 0x400000 -n 16
disk /dev/nvme0n1 cmd 0x2, speed 1230.77MB/s, cost times 52ms
root@zynq:~# nvmel_benchmark -w /dev/nvme0n1 -p 0x40000000 -s 0 -l 0x400000 -n 16
disk /dev/nvme0n1 cmd 0x4, speed 727.27MB/s, cost times 88ms
root@zynq:~# nvmel_benchmark -w /dev/nvme0n1 -p 0x40000000 -s 0 -l 0x400000 -n 16
disk /dev/nvme0n1 cmd 0x4, speed 735.63MB/s, cost times 87ms
root@zynq:~# nvmel_benchmark -r /dev/nvme0n1 -p 0x40000000 -s 0 -l 0x40000 -n 256
disk /dev/nvme0n1 cmd 0x2, speed 1084.75MB/s, cost times 59ms 
root@zynq:~# nvmel_benchmark -r /dev/nvme0n1 -p 0x40000000 -s 0 -l 0x40000 -n 512
disk /dev/nvme0n1 cmd 0x2, speed 1084.75MB/s, cost times 118ms
root@zynq:~# nvmel_benchmark -w /dev/nvme0n1 -p 0x40000000 -s 0 -l 0x40000 -n 512
disk /dev/nvme0n1 cmd 0x4, speed 670.16MB/s, cost times 191ms
root@zynq:~# nvmel_benchmark -w /dev/nvme0n1 -p 0x40000000 -s 0 -l 0x400000 -n 512
disk /dev/nvme0n1 cmd 0x4, speed 730.91MB/s, cost times 2802ms
```

# zynqmp
## 读测试
发现读长度为256MB时，不会报错（后文说明报错类型），
```shell
#!/bin/sh

date
dd if=/dev/nvme0n1p1 of=/dev/null bs=1M count=256
dd if=/dev/nvme0n1p1 of=/dev/null bs=1M count=256
dd if=/dev/nvme0n1p1 of=/dev/null bs=1M count=256
dd if=/dev/nvme0n1p1 of=/dev/null bs=1M count=256
date
```
测试结果，nvme到zu的ps端内存的读速度有500MB/s，这应该是内存带宽限制
```shell
root@zynqmp:~# ./testspeed.sh 
Wed Jan 20 00:45:30 UTC 2106
32MB read
256+0 records in
256+0 records out
256+0 records in
256+0 records out
256+0 records in
256+0 records out
256+0 records in
256+0 records out
Wed Jan 20 00:45:32 UTC 2106
```
## 写测试
128MB（256MB挂掉）可正常工作，
```shell
#!/bin/sh

date
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M count=128
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M count=128
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M count=128
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M count=128
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M count=128
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M count=128
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M count=128
dd if=/dev/zero of=/dev/nvme0n1p1 bs=1M count=128
date
```
测试结果，速度为256MB/s。
```shell
root@zynqmp:~# ./writetest.sh 
Wed Jan 20 00:43:03 UTC 2106
1GB write
128+0 records in
128+0 records out
128+0 records in
128+0 records out
128+0 records in
128+0 records out
128+0 records in
128+0 records out
128+0 records in
128+0 records out
128+0 records in
128+0 records out
128+0 records in
128+0 records out
128+0 records in
128+0 records out
Wed Jan 20 00:43:07 UTC 2106
```
## 读写PL端DDR4
测试数据来源于zynqmp PL的DDR4，单盘
```shell
write speed: 763.894MB/s, used times: 2681ms
read speed: 802.508MB/s, used times: 2552ms
```
三个盘，可见总速度并没有线性增长，两个盘就到顶了，有带宽限制，受限于pl逻辑（TLP strict/relax order）还是PEX8724性能，待定位，
```shell
write speed: 1411.765MB/s, used times: 4352ms
read speed: 1631.873MB/s, used times: 3765ms
```

# Peer-to-Peer
方法是，将一个NVME SSD的BAR空间，4K字节，写入另一块SSD，读出作比较，
```shell
root@zynqmp:~# cat /proc/iomem
00000000-7fffffff : System RAM
  02080000-02cfffff : Kernel code
  02d80000-03040fff : Kernel data
80001000-80001fff : /amba_pl@0/i2c@80001000
a0000000-afffffff : /amba_pl@0/axi-pcie@a0000000
  a0000000-a05fffff : PCI Bus 0000:01
    a0000000-a05fffff : PCI Bus 0000:02
      a0000000-a01fffff : PCI Bus 0000:03
      a0200000-a03fffff : PCI Bus 0000:04
  a0700000-a0ffffff : PCI Bus 0000:01
    a0700000-a0efffff : PCI Bus 0000:02
      a0700000-a09fffff : PCI Bus 0000:03
        a0700000-a0703fff : 0000:03:00.0
          a0700000-a0703fff : nvme
      a0a00000-a0cfffff : PCI Bus 0000:04
        a0a00000-a0a03fff : 0000:04:00.0
          a0a00000-a0a03fff : nvme
      a0d00000-a0dfffff : PCI Bus 0000:05
        a0d00000-a0d03fff : 0000:05:00.0
          a0d00000-a0d03fff : nvme
      a0e00000-a0efffff : PCI Bus 0000:06
        a0e00000-a0e03fff : 0000:06:00.0
          a0e00000-a0e03fff : nvme
    a0f00000-a0f3ffff : 0000:01:00.0
      a0f00000-a0f3ffff : pcie_ep
root@zynqmp:~# memtool -32 0xa0d00000 20
Reading 0x20 count starting at address 0xA0D00000

0xA0D00000:  3C033FFF 00F00020 00010200 00000000
0xA0D00010:  00000000 00460001 00000000 00000001
0xA0D00020:  00000000 001F001F 70045000 00000000
0xA0D00030:  70042000 00000000 00000000 00000000
0xA0D00040:  00000000 00000000 00000000 00000000
0xA0D00050:  00000000 00000000 00000000 00000000
0xA0D00060:  00000000 00000000 00000000 00000000
0xA0D00070:  00000000 00000000 00000000 00000000

root@zynqmp:~# nvmed_benchmark -w /dev/nvme0n1 -d 0xa0d00000 -s 0 -l 0x4000
speed: 1.30MB/s, cost times: 12ms
root@zynqmp:~# dd if=/dev/nvme0n1 of=a.bin bs=4k count=1
1+0 records in
1+0 records out
root@zynqmp:~# hexdump -C a.bin 
00000000  ff 3f 03 3c 20 00 f0 00  00 02 01 00 00 00 00 00  |.?.< ...........|
00000010  00 00 00 00 01 00 46 00  00 00 00 00 01 00 00 00  |......F.........|
00000020  00 00 00 00 1f 00 1f 00  00 50 04 70 00 00 00 00  |.........P.p....|
00000030  00 20 04 70 00 00 00 00  00 00 00 00 00 00 00 00  |. .p............|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00001000
```
这里可以用FPGA抓信号，来确定是P2P还是RC做了转发。

# 问题
## zynqmp丢中断
读写都会发生丢中断，写比读多撑几次，
```shell
root@zynqmp:~# ./test.sh 
write test
speed: 716.08MB/s, cost times: 715ms
speed: 715.08MB/s, cost times: 716ms
speed: 718.09MB/s, cost times: 713ms
speed: 715.08MB/s, cost times: 716ms
write test end
sleep 4s
read test
speed: 2.09MB/s, cost times: 61099ms
speed: 2.09MB/s, cost times: 61130ms
speed: 4.23MB/s, cost times: 30277ms
speed: 4.24MB/s, cost times: 30220ms
read test end
```
内核打印，可见中断丢了，但是通过polled函数，这个数据最终是成功读写，
```shell
[  112.888882] nvme nvme3: I/O 149 QID 2 timeout, completion polled
[  112.895259] nvme nvme3: I/O 150 QID 2 timeout, completion polled
[  112.901593] nvme nvme3: I/O 151 QID 2 timeout, completion polled
[  112.907929] nvme nvme3: I/O 152 QID 2 timeout, completion polled
[  112.920898] nvme nvme0: I/O 726 QID 3 timeout, completion polled
[  112.920901] nvme nvme1: I/O 426 QID 4 timeout, completion polled
[  112.921406] nvme nvme1: I/O 427 QID 4 timeout, completion polled
[  112.921828] nvme nvme1: I/O 428 QID 4 timeout, completion polled
[  112.922239] nvme nvme1: I/O 429 QID 4 timeout, completion polled
[  112.951285] nvme nvme0: I/O 727 QID 3 timeout, completion polled
[  112.957720] nvme nvme0: I/O 728 QID 3 timeout, completion polled
[  112.964214] nvme nvme0: I/O 729 QID 3 timeout, completion polled
[  146.968883] nvme nvme3: I/O 539 QID 3 timeout, completion polled
[  146.968886] nvme nvme1: I/O 334 QID 1 timeout, completion polled
[  146.968889] nvme nvme2: I/O 76 QID 2 timeout, completion polled
[  146.969012] nvme nvme2: I/O 77 QID 2 timeout, completion polled
[  146.969023] nvme nvme1: I/O 335 QID 1 timeout, completion polled
[  146.969109] nvme nvme2: I/O 78 QID 2 timeout, completion polled
[  146.969132] nvme nvme1: I/O 336 QID 1 timeout, completion polled
[  147.010686] nvme nvme3: I/O 540 QID 3 timeout, completion polled
[  147.016751] nvme nvme3: I/O 541 QID 3 timeout, completion polled
[  173.080884] nvme nvme0: I/O 28 QID 0 timeout, completion polled
[  177.880874] nvme nvme2: I/O 78 QID 2 timeout, completion polled
[  177.880877] nvme nvme1: I/O 336 QID 1 timeout, completion polled
[  177.881021] nvme nvme1: I/O 337 QID 1 timeout, completion polled
[  177.898931] nvme nvme2: I/O 79 QID 2 timeout, completion polled
[  177.909098] nvme nvme3: I/O 152 QID 2 timeout, completion polled
[  203.864885] nvme nvme0: I/O 729 QID 3 timeout, completion polled
[  207.896877] nvme nvme1: I/O 337 QID 1 timeout, completion polled
[  208.888875] nvme nvme2: I/O 79 QID 2 timeout, completion polled
[  234.008872] nvme nvme0: I/O 110 QID 4 timeout, completion polled
[  234.014971] nvme nvme0: I/O 111 QID 4 timeout, completion polled
[  234.021053] nvme nvme0: I/O 112 QID 4 timeout, completion polled
```

## PCIe Switch识别成1.0 x4
本来支持3.0，
```shell
root@zynqmp:~# lspci -vv
06:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961 (prog-if 02 [NVM Express])
        Subsystem: Samsung Electronics Co Ltd Device a801
        Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0
        Interrupt: pin A routed to IRQ 50
        Region 0: Memory at a0e00000 (64-bit, non-prefetchable) [size=16K]
        Capabilities: [40] Power Management version 3
                Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0-,D1-,D2-,D3hot-,D3cold-)
                Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
        Capabilities: [50] MSI: Enable+ Count=4/32 Maskable- 64bit+
                Address: 000000006d0bf000  Data: 0004
        Capabilities: [70] Express (v2) Endpoint, MSI 00
                DevCap: MaxPayload 256 bytes, PhantFunc 0, Latency L0s unlimited, L1 unlimited
                        ExtTag- AttnBtn- AttnInd- PwrInd- RBE+ FLReset+ SlotPowerLimit 25.000W
                DevCtl: Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
                        RlxdOrd+ ExtTag- PhantFunc- AuxPwr- NoSnoop+ FLReset-
                        MaxPayload 128 bytes, MaxReadReq 512 bytes
                DevSta: CorrErr- UncorrErr- FatalErr- UnsuppReq- AuxPwr+ TransPend-
                LnkCap: Port #0, Speed 8GT/s, Width x4, ASPM L1, Exit Latency L1 <64us
                        ClockPM+ Surprise- LLActRep- BwNot- ASPMOptComp+
                LnkCtl: ASPM Disabled; RCB 64 bytes Disabled- CommClk-
                        ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
                LnkSta: Speed 2.5GT/s, Width x4, TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
                DevCap2: Completion Timeout: Range ABCD, TimeoutDis+, LTR+, OBFF Not Supported
                         AtomicOpsCap: 32bit- 64bit- 128bitCAS-
                DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis-, LTR-, OBFF Disabled
                         AtomicOpsCtl: ReqEn-
                LnkCtl2: Target Link Speed: 8GT/s, EnterCompliance- SpeedDis-
                         Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
                         Compliance De-emphasis: -6dB
                LnkSta2: Current De-emphasis Level: -3.5dB, EqualizationComplete+, EqualizationPhase1+
                         EqualizationPhase2-, EqualizationPhase3-, LinkEqualizationRequest-
        Capabilities: [b0] MSI-X: Enable- Count=8 Masked-
                Vector table: BAR=0 offset=00003000
                PBA: BAR=0 offset=00002000
        Capabilities: [100 v2] Advanced Error Reporting
                UESta:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
                UEMsk:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
                UESvrt: DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
                CESta:  RxErr+ BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr-
                CEMsk:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
                AERCap: First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
                        MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-
                HeaderLog: 00000000 00000000 00000000 00000000
        Capabilities: [148 v1] Device Serial Number 00-00-00-00-00-00-00-00
        Capabilities: [158 v1] Power Budgeting <?>
        Capabilities: [168 v1] #19
        Capabilities: [188 v1] Latency Tolerance Reporting
                Max snoop latency: 0ns
                Max no snoop latency: 0ns
        Capabilities: [190 v1] L1 PM Substates
                L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2+ ASPM_L1.1+ L1_PM_Substates+
                          PortCommonModeRestoreTime=10us PortTPowerOnTime=10us
                L1SubCtl1: PCI-PM_L1.2- PCI-PM_L1.1- ASPM_L1.2- ASPM_L1.1-
                           T_CommonMode=0us LTR1.2_Threshold=0ns
                L1SubCtl2: T_PwrOn=10us
        Kernel driver in use: nvme
        Kernel modules: nvme
```

## nvme驱动set/get features接口取消
core.c增加下面接口，
```c
int nvme_get_features(struct nvme_ctrl *dev, unsigned fid, unsigned nsid,
			void *buffer, size_t buflen, u32 *result)/*add by aj for nvmed*/
{
	struct nvme_command c;
	union nvme_result res;
	int ret;

	memset(&c, 0, sizeof(c));
	c.features.opcode = nvme_admin_get_features;
	c.features.nsid = cpu_to_le32(nsid);
	c.features.fid = cpu_to_le32(fid);

	ret = __nvme_submit_sync_cmd(dev->admin_q, &c, &res, 
			buffer, buflen, 0, NVME_QID_ANY, 0, 0);
	if (ret >= 0 && result)
		*result = le32_to_cpu(res.u32);
	return ret;
}
EXPORT_SYMBOL_GPL(nvme_get_features);/*add by aj for nvmed*/

/*static(comment by aj for nvmed)*/
int nvme_set_features(struct nvme_ctrl *dev, unsigned fid, unsigned dword11,
		      void *buffer, size_t buflen, u32 *result)
{
	struct nvme_command c;
	union nvme_result res;
	int ret;

	memset(&c, 0, sizeof(c));
	c.features.opcode = nvme_admin_set_features;
	c.features.fid = cpu_to_le32(fid);
	c.features.dword11 = cpu_to_le32(dword11);

	ret = __nvme_submit_sync_cmd(dev->admin_q, &c, &res,
			buffer, buflen, 0, NVME_QID_ANY, 0, 0);
	if (ret >= 0 && result)
		*result = le32_to_cpu(res.u32);
	return ret;
}
EXPORT_SYMBOL_GPL(nvme_set_features);/*add by aj for nvmed*/
```

## oops bug
```shell
	result = nvme_pci_enable(dev);
	if (result)
		goto out;
	/* add msleep to solve this oops
Starting udev
[    3.957187] udevd[1670]: starting version 3.2.2
[    4.054765] udevd[1671]: starting eudev-3.2.2
[    4.739681] nvme nvme0: pci function 0000:04:00.0
[    4.744435] pci 0000:00:00.0: enabling device (0000 -> 0002)
[    4.750027] pci 0000:01:00.0: enabling device (0000 -> 0002)
[    4.750184] nvme nvme1: pci function 0000:05:00.0
[    4.750347] pci 0000:02:02.0: enabling device (0000 -> 0002)
[    4.750360] nvme 0000:05:00.0: enabling device (0000 -> 0002)
[    4.750375] Synchronous External Abort: synchronous external abort (0x96000010) at 0xffffff800bcd801c
[    4.750378] Internal error: : 96000010 [#1] SMP
[    4.750381] Modules linked in: nvme(+) nvme_core uio_pdrv_genirq
[    4.750391] CPU: 3 PID: 50 Comm: kworker/u8:3 Not tainted 4.14.0-xilinx-v2018.2 #5
[    4.750392] Hardware name: xlnx,zynqmp (DT)
[    4.750410] Workqueue: nvme-wq nvme_reset_work [nvme]
[    4.750413] task: ffffffc06daa0600 task.stack: ffffff80091c8000
[    4.750425] xilinx-pcie 1000000000.axi-pcie: Slave unsupported request
[    4.750426] PC is at nvme_reset_work+0xd4/0x1478 [nvme]
[    4.750436] LR is at nvme_reset_work+0xbc/0x1478 [nvme]
[    4.750438] pc : [<ffffff8000aaec5c>] lr : [<ffffff8000aaec44>] pstate: 60000145
[    4.750439] sp : ffffff80091cbcd0
[    4.750440] x29: ffffff80091cbcd0 x28: 0000000000000000 
[    4.750443] x27: 0000000000000000 x26: ffffff8008bdf968 
[    4.750446] x25: ffffffc06ccd5098 x24: ffffff8008a07000 
[    4.750450] x23: ffffffc06ccd5000 x22: ffffffc06bddf000 
[    4.750453] x21: ffffffc06bddf000 x20: ffffffc06bddee30 
[    4.750456] x19: ffffffc06bddf230 x18: 0000000000010000 
[    4.750459] x17: ffffff8008a08630 x16: 0000000000000078 
[    4.750462] x15: 00000000fffffff0 x14: ffffff8008dac5b8 
[    4.750465] x13: ffffff8008e37d38 x12: ffffff8008dac000 
[    4.750468] x11: 0000000000000000 x10: ffffff8008e37000 
[    4.750471] x9 : 0000000000000000 x8 : ffffff8008e429ab 
[    4.750473] x7 : 0000000000000000 x6 : 000000000d8931d2 
[    4.750476] x5 : ffffffc06ccf7398 x4 : 0000000000000886 
[    4.750479] x3 : 0000000000500004 x2 : 0000000000000000 
[    4.750482] x1 : ffffffffffffffff x0 : ffffff800bcd801c 
[    4.750486] Process kworker/u8:3 (pid: 50, stack limit = 0xffffff80091c8000)
[    4.750487] Call trace:
[    4.750490] Exception stack(0xffffff80091cbb90 to 0xffffff80091cbcd0)
[    4.750493] bb80:                                   ffffff800bcd801c ffffffffffffffff
[    4.750496] bba0: 0000000000000000 0000000000500004 0000000000000886 ffffffc06ccf7398
[    4.750500] bbc0: 000000000d8931d2 0000000000000000 ffffff8008e429ab 0000000000000000
[    4.750503] bbe0: ffffff8008e37000 0000000000000000 ffffff8008dac000 ffffff8008e37d38
[    4.750506] bc00: ffffff8008dac5b8 00000000fffffff0 0000000000000078 ffffff8008a08630
[    4.750509] bc20: 0000000000010000 ffffffc06bddf230 ffffffc06bddee30 ffffffc06bddf000
[    4.750513] bc40: ffffffc06bddf000 ffffffc06ccd5000 ffffff8008a07000 ffffffc06ccd5098
[    4.750516] bc60: ffffff8008bdf968 0000000000000000 0000000000000000 ffffff80091cbcd0
[    4.750519] bc80: ffffff8000aaec44 ffffff80091cbcd0 ffffff8000aaec5c 0000000060000145
[    4.750522] bca0: ffffffc06bddf230 0002ffc06bddee30 ffffffffffffffff ffffff8000aaebe4
[    4.750525] bcc0: ffffff80091cbcd0 ffffff8000aaec5c
[    4.750535] [<ffffff8000aaec5c>] nvme_reset_work+0xd4/0x1478 [nvme]
[    4.750543] [<ffffff80080b1a14>] process_one_work+0x1d4/0x338
[    4.750547] [<ffffff80080b1bbc>] worker_thread+0x44/0x488
[    4.750551] [<ffffff80080b7754>] kthread+0x12c/0x130
[    4.750555] [<ffffff8008084a60>] ret_from_fork+0x10/0x18
[    4.750560] Code: f90112c0 91074296 f9408ec0 91007000 (b9400000) 
[    4.750563] ---[ end trace 9fc6470ab8262428 ]---
	*/
	/*1->10->50->100*/
	msleep(100);
```

## mkfs的时候出现内核bug
该问题需要查看mkfs源码（e2fsprogs）来定位错误。
> [编译e2fsprogs源码](https://blog.csdn.net/passerbyyuan/article/details/79000671)

Creating journal的时候，触发异常。
```shell
root@zynqmp:~# mkfs -t ext4 /dev/nvme
nvme0      nvme0n1    nvme0n1p1  nvme1      nvme1n1    nvme1n1p1  nvme2      nvme2n1    nvme2n1p1
root@zynqmp:~# mkfs -t ext4 /dev/nvme0n1p1 
mke2fs 1.43.5 (04-Aug-2017)
Discarding device blocks: done                            
Creating filesystem with 250050560 4k blocks and 62513152 inodes
Filesystem UUID: 394d7270-513c-4593-9d4c-8508d7f958ef
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
        102400000, 214990848

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): 
[   27.555490] mkfs.ext4[2167]: undefined instruction: pc=0000007f939a0700
[   27.562052] Code: 00000000 00000000 00000000 00000000 (00000000) 
[   27.569683] udevd[1673]: unhandled level 2 translation fault (11) at 0x00000018, esr 0x92000006, in libc-2.26.so[7f9860d000+138000]
[   27.581446] CPU: 3 PID: 1673 Comm: udevd Not tainted 4.14.0-xilinx-v2018.2 #5
[   27.588552] Hardware name: xlnx,zynqmp (DT)
[   27.592714] task: ffffffc06d129080 task.stack: ffffff800bb08000
[   27.598616] PC is at 0x7f9867eb10
[   27.601907] LR is at 0x7f98681b20
[   27.605211] pc : [<0000007f9867eb10>] lr : [<0000007f98681b20>] pstate: 60000000
[   27.612588] sp : 0000007fc50c0b50
[   27.615886] x29: 0000007fc50c0b50 x28: 0000007f98758a20 
[   27.621182] x27: 0000007f98758a78 x26: 0000007f98758a28 
[   27.626480] x25: 0000007f98758a78 x24: 0000000000000000 
[   27.631772] x23: 0000007f987580d8 x22: 0000000000000000 
[   27.637068] x21: 0000000000000000 x20: 00000000005aecf0 
[   27.637104] klogd[2130]: undefined instruction: pc=0000007f84b83b00
[   27.637109] Code: 52800042 2a1703e0 1a82a362 17fffddd (00000000) 
[   27.637479] init[1]: undefined instruction: pc=0000007f83ed8040
[   27.637483] Code: a9bd7bfd 90000583 93407c42 910003fd (00001000) 
[   27.637633] Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000004
[   27.637633] 
[   27.637638] CPU: 0 PID: 1 Comm: init Not tainted 4.14.0-xilinx-v2018.2 #5
[   27.637639] Hardware name: xlnx,zynqmp (DT)
[   27.637640] Call trace:
[   27.637653] [<ffffff8008088ae8>] dump_backtrace+0x0/0x360
[   27.637657] [<ffffff8008088e5c>] show_stack+0x14/0x20
[   27.637663] [<ffffff80089de320>] dump_stack+0x9c/0xbc
[   27.637668] [<ffffff800809af98>] panic+0x11c/0x274
[   27.637673] [<ffffff800809e1a8>] complete_and_exit+0x0/0x20
[   27.637677] [<ffffff800809ee70>] do_group_exit+0x38/0xa8
[   27.637682] [<ffffff80080a9108>] get_signal+0x130/0x470
[   27.637686] [<ffffff8008087df8>] do_signal+0x68/0x650
[   27.637690] [<ffffff80080887c0>] do_notify_resume+0xc0/0xf0
[   27.637693] Exception stack(0xffffff800803bec0 to 0xffffff800803c000)
[   27.637697] bec0: 00000000ffffffff 0000007fd2515c5c 0000000000000001 0000007f83f88000
[   27.637700] bee0: 0000007fd2516f00 0000000000000000 0000000000000000 00000000000011b0
[   27.637703] bf00: 0000000000000048 003b9aca00000000 00000000ffe7f3e7 00000000000090e1
[   27.637706] bf20: 0000000000000018 00000003e8000000 00022550f76fa534 0000109f00e4709d
[   27.637710] bf40: 00000000004182d0 0000007f83ed8030 0000000000000874 0000007f83fb2780
[   27.637713] bf60: 0000000000418640 0000000000418678 0000000000000002 0000000000000000
[   27.637716] bf80: 0000000000406000 0000000000406000 0000000000406000 0000007fd25171a0
[   27.637719] bfa0: 0000000000000000 0000007fd2515bf0 0000000000402b1c 0000007fd2515bf0
[   27.637722] bfc0: 0000007f83ed8040 0000000080000000 000000000000000b 00000000ffffffff
[   27.637725] bfe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
[   27.637728] [<ffffff80080836c0>] work_pending+0x8/0x10
[   27.637732] SMP: stopping secondary CPUs
[   27.642357] Kernel Offset: disabled
[   27.642360] CPU features: 0x002004
[   27.642361] Memory Limit: none
[   27.839481] ---[ end Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000004
[   27.839481] 
```

## dd测试长度太长挂掉
```shell
date
dd if=/dev/nvme0n1p1 of=/dev/null bs=1M count=1024
date
```
挂掉
```shell
[  121.301225] dd[2177]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  121.312599] CPU: 3 PID: 2177 Comm: dd Not tainted 4.14.0-xilinx-v2018.2 #5
[  121.319394] Hardware name: xlnx,zynqmp (DT)
[  121.319494] klogd[2129]: undefined instruction: pc=000000000043a934
[  121.319499] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  121.335879] task: ffffffc06db7f280 task.stack: ffffff800b5f8000
[  121.341782] PC is at 0x0
[  121.344298] LR is at 0x0
[  121.346816] pc : [<0000000000000000>] lr : [<0000000000000000>] pstate: 00000000
[  121.354193] sp : 0000007fe3205f90
[  121.357490] x29: 0000007fe3206160 x28: 0000000000000000 
[  121.362786] x27: 0000000000000000 x26: 00000000004a5000 
[  121.368081] x25: 0000000000000000 x24: 0000000000000018 
[  121.373376] x23: 0000007fe3206ef8 x22: 0000007faa74a010 
[  121.378670] x21: 0000007fe3206ef8 x20: 0000000000100000 
[  121.383966] x19: 00000000004a6028 x18: 000000000000085a 
[  121.389260] x17: 0000007faa90a718 x16: 00000000004a57c8 
[  121.394555] x15: 0000007faa84ed00 x14: 0000007faa85bd98 
[  121.399850] x13: 0000000000000000 x12: 0000000000000000 
[  121.405145] x11: 0000000000000000 x10: 0101010101010101 
[  121.410442] x9 : 0000000000000000 x8 : 000000000000003f 
[  121.415735] x7 : 0000000000000000 x6 : 0000000000000000 
[  121.421030] x5 : 0000007fe3205e48 x4 : 0000007fe3205f80 
[  121.426324] x3 : 0000007faa99b000 x2 : 0000000000100000 
[  121.431619] x1 : 0000007faa74a010 x0 : 0000000000100000 
[  121.556508] testspeed.sh[2175]: undefined instruction: pc=000000000042f8b4
[  121.563316] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  121.569781] sh[2138]: undefined instruction: pc=000000000042f8b4
[  121.575706] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  121.582656] syslogd[2126]: undefined instruction: pc=000000000043b818
[  121.583048] login[2137]: undefined instruction: pc=0000007fa2acf8e0
[  121.583052] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  121.589348] start_getty[2134]: undefined instruction: pc=000000000042f8b4
[  121.589352] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  121.614182] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  121.621416] start_getty[2178]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  121.633431] CPU: 0 PID: 2178 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  121.641052] Hardware name: xlnx,zynqmp (DT)
[  121.645219] task: ffffffc06d1ee280 task.stack: ffffff800c270000
[  121.651121] PC is at 0x0
[  121.653634] LR is at 0x406d2c
[  121.656589] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  121.663966] sp : 0000007ff5ae8a90
[  121.667265] x29: 0000007ff5ae8b60 x28: 0000000000000000 
[  121.672559] x27: 0000000000000000 x26: 00000000004a5000 
[  121.677854] x25: 0000000000000000 x24: 0000000000000085 
[  121.683149] x23: 0000007ff5ae8c98 x22: 0000000000000005 
[  121.688444] x21: 0000000000000340 x20: 0000000000000085 
[  121.693740] x19: 00000000004a5000 x18: 0000000000000824 
[  121.699034] x17: 0000007fb8bedd80 x16: 00000000004a55c0 
[  121.704328] x15: 0000007fb8b77d00 x14: 0000007fb8b84d98 
[  121.709623] x13: 0000000000005b76 x12: 0000000000000018 
[  121.714918] x11: 0000000000000034 x10: 0101010101010101 
[  121.720213] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  121.725508] x7 : 0000007ff5e669c5 x6 : 0000000000000df4 
[  121.730802] x5 : 740d000000000000 x4 : 0000000000000001 
[  121.736097] x3 : 00000000004a6000 x2 : 0000000000000000 
[  121.741392] x1 : 0000007ff5ae8c98 x0 : 0000000000000005 
[  121.747981] start_getty[2179]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  121.759985] CPU: 2 PID: 2179 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  121.767617] Hardware name: xlnx,zynqmp (DT)
[  121.771783] task: ffffffc06ce90400 task.stack: ffffff800b558000
[  121.777689] PC is at 0x0
[  121.780203] LR is at 0x406d2c
[  121.783155] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  121.790531] sp : 0000007fd6653ec0
[  121.793831] x29: 0000007fd6653f90 x28: 0000000000000000 
[  121.799125] x27: 0000000000000000 x26: 00000000004a5000 
[  121.804420] x25: 0000000000000000 x24: 0000000000000085 
[  121.809715] x23: 0000007fd66540c8 x22: 0000000000000005 
[  121.815010] x21: 0000000000000340 x20: 0000000000000085 
[  121.820304] x19: 00000000004a5000 x18: 0000000000000824 
[  121.825600] x17: 0000007f92768d80 x16: 00000000004a55c0 
[  121.830895] x15: 0000007f926f2d00 x14: 0000007f926ffd98 
[  121.836189] x13: 0000000000005b76 x12: 0000000000000018 
[  121.841484] x11: 0000000000000034 x10: 0101010101010101 
[  121.846779] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  121.852073] x7 : 0000007fd62da9c5 x6 : 0000000000000df4 
[  121.857369] x5 : 740d000000000000 x4 : 0000000000000001 
[  121.862664] x3 : 00000000004a6000 x2 : 0000000000000000 
[  121.867958] x1 : 0000007fd66540c8 x0 : 0000000000000005 
[  121.874540] start_getty[2180]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  121.886556] CPU: 1 PID: 2180 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  121.894182] Hardware name: xlnx,zynqmp (DT)
[  121.898348] task: ffffffc06c9d2e80 task.stack: ffffff800c350000
[  121.904250] PC is at 0x0
[  121.906766] LR is at 0x406d2c
[  121.909714] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  121.917095] sp : 0000007fd8e22cd0
[  121.920394] x29: 0000007fd8e22da0 x28: 0000000000000000 
[  121.925691] x27: 0000000000000000 x26: 00000000004a5000 
[  121.930984] x25: 0000000000000000 x24: 0000000000000085 
[  121.936280] x23: 0000007fd8e22ed8 x22: 0000000000000005 
[  121.941573] x21: 0000000000000340 x20: 0000000000000085 
[  121.946868] x19: 00000000004a5000 x18: 0000000000000824 
[  121.952163] x17: 0000007f9a2b4d80 x16: 00000000004a55c0 
[  121.957458] x15: 0000007f9a23ed00 x14: 0000007f9a24bd98 
[  121.962753] x13: 0000000000005b76 x12: 0000000000000018 
[  121.968048] x11: 0000000000000034 x10: 0101010101010101 
[  121.973342] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  121.978637] x7 : 0000007fd8aad9c5 x6 : 0000000000000df4 
[  121.983932] x5 : 740d000000000000 x4 : 0000000000000001 
[  121.989227] x3 : 00000000004a6000 x2 : 0000000000000000 
[  121.994522] x1 : 0000007fd8e22ed8 x0 : 0000000000000005 
[  122.001052] start_getty[2181]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  122.013063] CPU: 3 PID: 2181 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  122.020693] Hardware name: xlnx,zynqmp (DT)
[  122.024860] task: ffffffc06d1ee280 task.stack: ffffff800c350000
[  122.030761] PC is at 0x0
[  122.033274] LR is at 0x406d2c
[  122.036230] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  122.043607] sp : 0000007ff7b33300
[  122.046906] x29: 0000007ff7b333d0 x28: 0000000000000000 
[  122.052204] x27: 0000000000000000 x26: 00000000004a5000 
[  122.057496] x25: 0000000000000000 x24: 0000000000000085 
[  122.062792] x23: 0000007ff7b33508 x22: 0000000000000005 
[  122.068085] x21: 0000000000000340 x20: 0000000000000085 
[  122.073380] x19: 00000000004a5000 x18: 0000000000000824 
[  122.078676] x17: 0000007fb2033d80 x16: 00000000004a55c0 
[  122.083970] x15: 0000007fb1fbdd00 x14: 0000007fb1fcad98 
[  122.089265] x13: 0000000000005b76 x12: 0000000000000018 
[  122.094560] x11: 0000000000000034 x10: 0101010101010101 
[  122.099855] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  122.105149] x7 : 0000007ff7fbd9c5 x6 : 0000000000000df4 
[  122.110444] x5 : 740d000000000000 x4 : 0000000000000001 
[  122.115739] x3 : 00000000004a6000 x2 : 0000000000000000 
[  122.121034] x1 : 0000007ff7b33508 x0 : 0000000000000005 
[  122.127562] start_getty[2182]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  122.139565] CPU: 2 PID: 2182 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  122.147197] Hardware name: xlnx,zynqmp (DT)
[  122.151364] task: ffffffc06ce91080 task.stack: ffffff800c350000
[  122.157265] PC is at 0x0
[  122.159783] LR is at 0x406d2c
[  122.162733] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  122.170111] sp : 0000007fc770bee0
[  122.173410] x29: 0000007fc770bfb0 x28: 0000000000000000 
[  122.178705] x27: 0000000000000000 x26: 00000000004a5000 
[  122.184002] x25: 0000000000000000 x24: 0000000000000085 
[  122.189295] x23: 0000007fc770c0e8 x22: 0000000000000005 
[  122.194592] x21: 0000000000000340 x20: 0000000000000085 
[  122.199885] x19: 00000000004a5000 x18: 0000000000000824 
[  122.205179] x17: 0000007f9909ad80 x16: 00000000004a55c0 
[  122.210476] x15: 0000007f99024d00 x14: 0000007f99031d98 
[  122.215769] x13: 0000000000005b76 x12: 0000000000000018 
[  122.221064] x11: 0000000000000034 x10: 0101010101010101 
[  122.226359] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  122.231654] x7 : 0000007fc73829c5 x6 : 0000000000000df4 
[  122.236949] x5 : 740d000000000000 x4 : 0000000000000001 
[  122.242243] x3 : 00000000004a6000 x2 : 0000000000000000 
[  122.247538] x1 : 0000007fc770c0e8 x0 : 0000000000000005 
[  122.254106] start_getty[2183]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  122.266126] CPU: 0 PID: 2183 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  122.273746] Hardware name: xlnx,zynqmp (DT)
[  122.277911] task: ffffffc06c9d2e80 task.stack: ffffff800c358000
[  122.283813] PC is at 0x0
[  122.286325] LR is at 0x406d2c
[  122.289281] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  122.296658] sp : 0000007ff9246d50
[  122.299957] x29: 0000007ff9246e20 x28: 0000000000000000 
[  122.305252] x27: 0000000000000000 x26: 00000000004a5000 
[  122.310546] x25: 0000000000000000 x24: 0000000000000085 
[  122.315843] x23: 0000007ff9246f58 x22: 0000000000000005 
[  122.321136] x21: 0000000000000340 x20: 0000000000000085 
[  122.326431] x19: 00000000004a5000 x18: 0000000000000824 
[  122.331726] x17: 0000007fa91f2d80 x16: 00000000004a55c0 
[  122.337021] x15: 0000007fa917cd00 x14: 0000007fa9189d98 
[  122.342316] x13: 0000000000005b76 x12: 0000000000000018 
[  122.347610] x11: 0000000000000034 x10: 0101010101010101 
[  122.352905] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  122.358200] x7 : 0000007ff96c99c5 x6 : 0000000000000df4 
[  122.363495] x5 : 740d000000000000 x4 : 0000000000000001 
[  122.368790] x3 : 00000000004a6000 x2 : 0000000000000000 
[  122.374085] x1 : 0000007ff9246f58 x0 : 0000000000000005 
[  122.380660] start_getty[2184]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  122.392668] CPU: 1 PID: 2184 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  122.400299] Hardware name: xlnx,zynqmp (DT)
[  122.404468] task: ffffffc06d1ee280 task.stack: ffffff800c360000
[  122.410368] PC is at 0x0
[  122.412886] LR is at 0x406d2c
[  122.415840] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  122.423215] sp : 0000007fe1ea53f0
[  122.426508] x29: 0000007fe1ea54c0 x28: 0000000000000000 
[  122.431808] x27: 0000000000000000 x26: 00000000004a5000 
[  122.437103] x25: 0000000000000000 x24: 0000000000000085 
[  122.442397] x23: 0000007fe1ea55f8 x22: 0000000000000005 
[  122.447692] x21: 0000000000000340 x20: 0000000000000085 
[  122.452988] x19: 00000000004a5000 x18: 0000000000000824 
[  122.458282] x17: 0000007f907f6d80 x16: 00000000004a55c0 
[  122.463577] x15: 0000007f90780d00 x14: 0000007f9078dd98 
[  122.468872] x13: 0000000000005b76 x12: 0000000000000018 
[  122.474166] x11: 0000000000000034 x10: 0101010101010101 
[  122.479461] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  122.484757] x7 : 0000007fe1a2b9c5 x6 : 0000000000000df4 
[  122.490051] x5 : 740d000000000000 x4 : 0000000000000001 
[  122.495346] x3 : 00000000004a6000 x2 : 0000000000000000 
[  122.500641] x1 : 0000007fe1ea55f8 x0 : 0000000000000005 
[  122.507219] start_getty[2185]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  122.519225] CPU: 3 PID: 2185 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  122.526855] Hardware name: xlnx,zynqmp (DT)
[  122.531022] task: ffffffc06ce90400 task.stack: ffffff800c368000
[  122.536923] PC is at 0x0
[  122.539440] LR is at 0x406d2c
[  122.542388] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  122.549772] sp : 0000007ffdd283e0
[  122.553069] x29: 0000007ffdd284b0 x28: 0000000000000000 
[  122.558363] x27: 0000000000000000 x26: 00000000004a5000 
[  122.563658] x25: 0000000000000000 x24: 0000000000000085 
[  122.568954] x23: 0000007ffdd285e8 x22: 0000000000000005 
[  122.574247] x21: 0000000000000340 x20: 0000000000000085 
[  122.579543] x19: 00000000004a5000 x18: 0000000000000824 
[  122.584837] x17: 0000007f808c2d80 x16: 00000000004a55c0 
[  122.590132] x15: 0000007f8084cd00 x14: 0000007f80859d98 
[  122.595427] x13: 0000000000005b76 x12: 0000000000000018 
[  122.600722] x11: 0000000000000034 x10: 0101010101010101 
[  122.606016] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  122.611312] x7 : 0000007ffd9a69c5 x6 : 0000000000000df4 
[  122.616606] x5 : 740d000000000000 x4 : 0000000000000001 
[  122.621901] x3 : 00000000004a6000 x2 : 0000000000000000 
[  122.627196] x1 : 0000007ffdd285e8 x0 : 0000000000000005 
[  122.633718] start_getty[2186]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  122.645730] CPU: 2 PID: 2186 Comm: start_getty Not tainted 4.14.0-xilinx-v2018.2 #5
[  122.653358] Hardware name: xlnx,zynqmp (DT)
[  122.657525] task: ffffffc06c9d2e80 task.stack: ffffff800c368000
[  122.663427] PC is at 0x0
[  122.665940] LR is at 0x406d2c
[  122.668896] pc : [<0000000000000000>] lr : [<0000000000406d2c>] pstate: 20000000
[  122.676275] sp : 0000007ff50fd4c0
[  122.679572] x29: 0000007ff50fd590 x28: 0000000000000000 
[  122.684870] x27: 0000000000000000 x26: 00000000004a5000 
[  122.690161] x25: 0000000000000000 x24: 0000000000000085 
[  122.695457] x23: 0000007ff50fd6c8 x22: 0000000000000005 
[  122.700751] x21: 0000000000000340 x20: 0000000000000085 
[  122.706046] x19: 00000000004a5000 x18: 0000000000000824 
[  122.711341] x17: 0000007f8bc59d80 x16: 00000000004a55c0 
[  122.716635] x15: 0000007f8bbe3d00 x14: 0000007f8bbf0d98 
[  122.721930] x13: 0000000000005b76 x12: 0000000000000018 
[  122.727225] x11: 0000000000000034 x10: 0101010101010101 
[  122.732520] x9 : 0000ffffffffffff x8 : 7f7f7fffffffffff 
[  122.737814] x7 : 0000007ff54739c5 x6 : 0000000000000df4 
[  122.743110] x5 : 740d000000000000 x4 : 0000000000000001 
[  122.748404] x3 : 00000000004a6000 x2 : 0000000000000000 
[  122.753699] x1 : 0000007ff50fd6c8 x0 : 0000000000000005 
INIT: Id "PS0" respawning too fast: disabled for 5 minutes
```
或者，
```shell
[  256.314662] dd[2196]: undefined instruction: pc=0000007f80cc8740
[  256.320713] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  256.326823] klogd[2128]: undefined instruction: pc=0000007f9ccb4d88
[  256.333014] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  256.339411] init[1]: undefined instruction: pc=0000007faacf9990
[  256.345248] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  256.351504] Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000004
[  256.351504] 
[  256.360545] CPU: 2 PID: 1 Comm: init Not tainted 4.14.0-xilinx-v2018.2 #5
[  256.367311] Hardware name: xlnx,zynqmp (DT)
[  256.371478] Call trace:
[  256.373920] [<ffffff8008088ae8>] dump_backtrace+0x0/0x360
[  256.379293] [<ffffff8008088e5c>] show_stack+0x14/0x20
[  256.384328] [<ffffff80089de320>] dump_stack+0x9c/0xbc
[  256.389362] [<ffffff800809af98>] panic+0x11c/0x274
[  256.394135] [<ffffff800809e1a8>] complete_and_exit+0x0/0x20
[  256.399690] [<ffffff800809ee70>] do_group_exit+0x38/0xa8
[  256.404986] [<ffffff80080a9108>] get_signal+0x130/0x470
[  256.410194] [<ffffff8008087df8>] do_signal+0x68/0x650
[  256.415228] [<ffffff80080887c0>] do_notify_resume+0xc0/0xf0
[  256.420782] Exception stack(0xffffff800803bec0 to 0xffffff800803c000)
[  256.427207] bec0: fffffffffffffffc 0000007fd63d1fb0 0000000000000000 0000000000000000
[  256.435018] bee0: 0000007fd63d1e20 0000000000000000 0000000000000000 0000000000006f56
[  256.442830] bf00: 0000000000000048 003b9aca00000000 00000000ffe7f4cd 00000000000412f8
[  256.450641] bf20: 0000000000000018 00000003e8000000 000fbc519507eb38 00002dca9e0234c0
[  256.451522] testspeed.sh[2194]: unhandled level 2 translation fault (11) at 0x00000000, esr 0x82000006, in busybox.nosuid[400000+95000]
[  256.451539] CPU: 3 PID: 2194 Comm: testspeed.sh Not tainted 4.14.0-xilinx-v2018.2 #5
[  256.451541] Hardware name: xlnx,zynqmp (DT)
[  256.451543] task: ffffffc06c95e400 task.stack: ffffff800b6d0000
[  256.451546] PC is at 0x0
[  256.451547] LR is at 0x0
[  256.451549] pc : [<0000000000000000>] lr : [<0000000000000000>] pstate: 00000000
[  256.451550] sp : 0000007fcec9e040
[  256.451552] x29: 0000000000000000 x28: 0000007fcec9ef85 
[  256.451555] x27: 00000000ffffffff x26: 0000000000000000 
[  256.451558] x25: 0000000022fc8680 x24: 00000000004a8000 
[  256.451561] x23: 0000000000000000 x22: 0000000000000000 
[  256.451564] x21: 0000000022fc9400 x20: 00000000004a8000 
[  256.451567] x19: 0000000000000000 x18: 0000000000000030 
[  256.451570] x17: 0000007f9cf2f030 x16: 00000000004a59d0 
[  256.451573] x15: 0000000000000010 x14: 0000000000000010 
[  256.451576] x13: 3170316e30656d76 x12: 0101010101010101 
[  256.451579] x11: 0000000000000004 x10: 0000007f9cfda0d8 
[  256.451582] x9 : 0000000000484b30 x8 : 0000000000000104 
[  256.451585] x7 : 0000007fcec9ef85 x6 : 0000000000484a60 
[  256.451588] x5 : 0000007f9d0b7010 x4 : 0000000000000000 
[  256.451590] x3 : 0000000000000000 x2 : 0000000000000000 
[  256.451593] x1 : 0000007fcec9e064 x0 : 0000000000000894 
[  256.451877] sh[2141]: undefined instruction: pc=0000007f91f52ee0
[  256.451882] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  256.452140] login[2136]: undefined instruction: pc=0000007f9f96cee0
[  256.452144] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  256.458829] start_getty[2133]: undefined instruction: pc=0000007f82b44ee0
[  256.458832] Code: 00000000 00000000 00000000 00000000 (00000000) 
[  256.620771] bf40: 0000000000418258 0000007faacf9938 0000000000000874 0000007fd63d1ea0
[  256.628583] bf60: 000000000000000b 0000000000418308 0000000000406de0 0000000000000000
[  256.636395] bf80: 0000000000406000 0000000000406000 0000000000406000 0000007fd63d20c0
[  256.644207] bfa0: 0000000000000000 0000007fd63d1de0 000000000040506c 0000007fd63d1de0
[  256.652020] bfc0: 0000007faacf9990 0000000080000000 0000000000000000 00000000ffffffff
[  256.659831] bfe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
[  256.667644] [<ffffff80080836c0>] work_pending+0x8/0x10
[  256.672765] SMP: stopping secondary CPUs
[  256.676672] Kernel Offset: disabled
[  256.680142] CPU features: 0x002004
[  256.683526] Memory Limit: none
[  256.686568] ---[ end Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000004
[  256.686568] 
```
这个初步定位为根文件系统问题，应该和内核无关。
