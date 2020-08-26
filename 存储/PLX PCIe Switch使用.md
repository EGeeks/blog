# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Exploring the Complexities of PCIe Connectivity and Peer-to-Peer Communication](https://blog.exxactcorp.com/exploring-the-complexities-of-pcie-connectivity-and-peer-to-peer-communication/)
> [The mystery of peer-to-peer transfer](https://forums.xilinx.com/t5/PCI-Express/The-mystery-of-peer-to-peer-transfer/td-p/762545)
> [pcie-peer-to-peer-communication](https://stackoverflow.com/questions/37002703/pcie-peer-to-peer-communication)
> [PCIe Switch高级功能及应用](https://blog.csdn.net/qq275650963/article/details/82699478)
> [SOFTWARE DEVELOPMENT KITS](https://www.broadcom.com/products/pcie-switches-bridges/software-dev-kit)
> [PCI EXPRESS SWITCHES](https://www.broadcom.com/products/pcie-switches-bridges/pcie-switches/)

# P2P
CPU FT1500A/16，盘starblaze 1TB，FPGA1 PCIe1.0x8，FPGA2 PCIe3.0x8，
```bash
storage@kylin:~$ sudo ./nvmel_benchmark -i /dev/nvme0n1:/dev/nvme1n1:/dev/nvme2n1:/dev/nvme3n1
disk[0]: /dev/nvme0n1 size=[1022545821696Bytes 975175MB 952GB] max_hw_sectors=1024 online_queues=17 stripe_size=0 lba_shift=9 name=nvme0 serial=C1PSMDCP00038 model=S1200ITT1-T1M21T fw_rev=1.2.0.2
disk[1]: /dev/nvme1n1 size=[1022545821696Bytes 975175MB 952GB] max_hw_sectors=1024 online_queues=17 stripe_size=0 lba_shift=9 name=nvme1 serial=C1PSMDCP00015 model=S1200ITT1-T1M21T fw_rev=1.2.0.2
disk[2]: /dev/nvme2n1 size=[1022545821696Bytes 975175MB 952GB] max_hw_sectors=1024 online_queues=17 stripe_size=0 lba_shift=9 name=nvme2 serial=C1PSMDCP00049 model=S1200ITT1-T1M21T fw_rev=1.2.0.2
disk[3]: /dev/nvme3n1 size=[1022545821696Bytes 975175MB 952GB] max_hw_sectors=1024 online_queues=17 stripe_size=0 lba_shift=9 name=nvme3 serial=C1PSMDCP00047 model=S1200ITT1-T1M21T fw_rev=1.2.0.2
```
读写PC内存的速度，
```bash
storage@kylin:~$ sudo ./nvmel_benchmark -w /dev/nvme0n1:/dev/nvme1n1:/dev/nvme2n1:/dev/nvme3n1 -p malloc -s 0x4000000 -l auto -n 512 -m 16
auto rwLength=0x800000
disk /dev/nvme0n1 cmd 0x4, speed 885.43MB/s, cost times 4626ms
disk /dev/nvme1n1 cmd 0x4, speed 891.79MB/s, cost times 4593ms
disk /dev/nvme2n1 cmd 0x4, speed 892.76MB/s, cost times 4588ms
disk /dev/nvme3n1 cmd 0x4, speed 898.25MB/s, cost times 4560ms
storage@kylin:~$ sudo ./nvmel_benchmark -r /dev/nvme0n1:/dev/nvme1n1:/dev/nvme2n1:/dev/nvme3n1 -p malloc -s 0x4000000 -l auto -n 512 -m 16
auto rwLength=0x800000
disk /dev/nvme0n1 cmd 0x2, speed 2147.88MB/s, cost times 1907ms
disk /dev/nvme1n1 cmd 0x2, speed 2169.49MB/s, cost times 1888ms
disk /dev/nvme2n1 cmd 0x2, speed 2142.26MB/s, cost times 1912ms
disk /dev/nvme3n1 cmd 0x2, speed 2140.02MB/s, cost times 1914ms
```
单盘p2p速度，
```bash
storage@kylin:~$ sudo ./nvmel_benchmark -r /dev/nvme0n1:/dev/nvme1n1:/dev/nvme2n1:/dev/nvme3n1 -p 0x6c000000 -s 0x4000000 -l auto -n 512 -m 16
auto rwLength=0x800000
disk /dev/nvme0n1 cmd 0x2, speed 1478.17MB/s, cost times 2771ms
disk /dev/nvme1n1 cmd 0x2, speed 1481.91MB/s, cost times 2764ms
disk /dev/nvme2n1 cmd 0x2, speed 1463.38MB/s, cost times 2799ms
disk /dev/nvme3n1 cmd 0x2, speed 1465.47MB/s, cost times 2795ms
storage@kylin:~$ sudo ./nvmel_benchmark -r /dev/nvme0n1:/dev/nvme1n1:/dev/nvme2n1:/dev/nvme3n1 -p 0x74000000 -s 0x4000000 -l auto -n 512 -m 16
auto rwLength=0x800000
disk /dev/nvme0n1 cmd 0x2, speed 2036.80MB/s, cost times 2011ms
disk /dev/nvme1n1 cmd 0x2, speed 2016.74MB/s, cost times 2031ms
disk /dev/nvme2n1 cmd 0x2, speed 2054.16MB/s, cost times 1994ms
disk /dev/nvme3n1 cmd 0x2, speed 2039.84MB/s, cost times 2008ms
storage@kylin:~/nvmelite$ 
storage@kylin:~/nvmelite$ sudo ./nvmel_benchmark -w /dev/nvme0n1:/dev/nvme1n1:/dev/nvme2n1:/dev/nvme3n1 -p 0x6c000000 -s 0x4000000 -l auto -n 512 -m 16
auto rwLength=0x800000
disk /dev/nvme0n1 cmd 0x4, speed 885.05MB/s, cost times 4628ms
disk /dev/nvme1n1 cmd 0x4, speed 894.52MB/s, cost times 4579ms
disk /dev/nvme2n1 cmd 0x4, speed 902.40MB/s, cost times 4539ms
disk /dev/nvme3n1 cmd 0x4, speed 891.79MB/s, cost times 4593ms
storage@kylin:~$ sudo ./nvmel_benchmark -w /dev/nvme0n1:/dev/nvme1n1:/dev/nvme2n1:/dev/nvme3n1 -p 0x74000000 -s 0x4000000 -l auto -n 512 -m 16
auto rwLength=0x800000
disk /dev/nvme0n1 cmd 0x4, speed 892.18MB/s, cost times 4591ms
disk /dev/nvme1n1 cmd 0x4, speed 891.02MB/s, cost times 4597ms
disk /dev/nvme2n1 cmd 0x4, speed 899.23MB/s, cost times 4555ms
disk /dev/nvme3n1 cmd 0x4, speed 900.22MB/s, cost times 4550ms
```
并行p2p速度，
```bash
storage@kylin:~$ ./tbw.sh  # 1.2GB/s + 1.2GB/s
disk /dev/nvme0n1 cmd 0x4, speed 641.91MB/s, cost times 6381ms # FPGA1
disk /dev/nvme1n1 cmd 0x4, speed 640.80MB/s, cost times 6392ms # FPGA1
disk /dev/nvme2n1 cmd 0x4, speed 628.80MB/s, cost times 6514ms # FPGA2
disk /dev/nvme3n1 cmd 0x4, speed 626.59MB/s, cost times 6537ms # FPGA2
storage@kylin:~$ ./tbr.sh  # 2.1GB/s + 1.5GB/s
disk /dev/nvme2n1 cmd 0x2, speed 1069.73MB/s, cost times 3829ms # FPGA2
disk /dev/nvme3n1 cmd 0x2, speed 1068.06MB/s, cost times 3835ms # FPGA2
disk /dev/nvme1n1 cmd 0x2, speed 786.94MB/s, cost times 5205ms # FPGA1
disk /dev/nvme0n1 cmd 0x2, speed 787.24MB/s, cost times 5203ms # FPGA1
storage@kylin:~$ ./tbr_fpga2.sh # 2.2GB/s
disk /dev/nvme2n1 cmd 0x2, speed 562.95MB/s, cost times 7276ms # FPGA2
disk /dev/nvme3n1 cmd 0x2, speed 559.87MB/s, cost times 7316ms # FPGA2
disk /dev/nvme0n1 cmd 0x2, speed 548.62MB/s, cost times 7466ms # FPGA2
disk /dev/nvme1n1 cmd 0x2, speed 548.84MB/s, cost times 7463ms # FPGA2
```
X86平台，PEX8747，
```bash
$ sudo ./dma_benchmark -i xaxicdma -c c2h -s cmem -d 0x100000000 -m cdev:xaxicdma2@cca00000 -l 0x400000 -n 64
global params: 
  dma type: xaxicdma read 0x400000 bytes from 0x34400000 to 0x100000000
  cmd type: read
  poll time: 0
  test cycle: 64
  xaxicdma type: cdev name: /dev/xaxicdma2@cca00000
speed: 6095.24MB/s, cost times: 42ms
$ sudo ./dma_benchmark -i xaxicdma -c h2c -s cmem -d 0x100000000 -m cdev:xaxicdma2@cca00000 -l 0x400000 -n 64
global params: 
  dma type: xaxicdma write 0x400000 bytes from 0x34400000 to 0x100000000
  cmd type: write
  poll time: 0
  test cycle: 64
  xaxicdma type: cdev name: /dev/xaxicdma2@cca00000
speed: 5565.22MB/s, cost times: 46ms
qe@qe-pc:~/software/hw$ sudo ./dma_benchmark -i xaxicdma -c c2h -s 0xd0000000 -d 0x100000000 -m cdev:xaxicdma2@cca00000 -l 0x400000 -n 64
global params: 
  dma type: xaxicdma read 0x400000 bytes from 0xd0000000 to 0x100000000
  cmd type: read
  poll time: 0
  test cycle: 64
  xaxicdma type: cdev name: /dev/xaxicdma2@cca00000
speed: 4196.72MB/s, cost times: 61ms
qe@qe-pc:~/software/hw$ 
qe@qe-pc:~/software/hw$ 
qe@qe-pc:~/software/hw$ sudo ./dma_benchmark -i xaxicdma -c h2c -s 0xd0000000 -d 0x100000000 -m cdev:xaxicdma2@cca00000 -l 0x400000 -n 64
global params: 
  dma type: xaxicdma write 0x400000 bytes from 0xd0000000 to 0x100000000
  cmd type: write
  poll time: 0
  test cycle: 64
  xaxicdma type: cdev name: /dev/xaxicdma2@cca00000
speed: 4654.55MB/s, cost times: 55ms
```


# 软件架构
PCIe总线架构，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190401112951588.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
Port Numbering端口号和PCIe设备号的对应关系，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190523093132801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
PEX8724有2个Station，最多6+1个Port（Port 8是一个软件虚拟Port），PEX8764有4个Station，最多16个Port，对比PEX8724和PEX8764系统，对于系统上的软件拓扑和实际硬件的对应关系，首先确定上行端口在哪个Station，如果在Station 0，就很简单，比如下面的PEX8764系统，15个下行Port全预留，可以看到1，2下没有EP，对应Station 0 Port 1/2，没有6，7，因为4，5配置成x8。
```shell
root@t2080rdb:~# lspci -tv
-[0000:00]---00.0-[01-0f]----00.0-[02-0f]--+-01.0-[03]--
                                           +-02.0-[04]--
                                           +-03.0-[05]----00.0  Samsung Electronics Co Ltd Device a804
                                           +-04.0-[06]----00.0  Device 0731:8000
                                           +-05.0-[07]----00.0  Device 0731:8000
                                           +-08.0-[08]----00.0  Samsung Electronics Co Ltd Device a804
                                           +-09.0-[09]----00.0  Samsung Electronics Co Ltd Device a804
                                           +-0a.0-[0a]----00.0  Samsung Electronics Co Ltd Device a804
                                           +-0b.0-[0b]--
                                           +-0c.0-[0c]----00.0  Samsung Electronics Co Ltd Device a804
                                           +-0d.0-[0d]----00.0  Samsung Electronics Co Ltd Device a804
                                           +-0e.0-[0e]----00.0  Samsung Electronics Co Ltd Device a804
                                           \-0f.0-[0f]----00.0  Samsung Electronics Co Ltd Device a804
```
下面的PEX8724系统，上行端口接在Station 1 Port 9，x8模式，Station 1还有Port 8，Station 0有4个x4 Port，所以总线号还是0，1，2，3，8这几个值，但Port 8和上行端口在一个Station，它先被识别，占用0，还剩1，2，3，8四个号分给Station 0的4个Port，所以当上行端口不在Station 0时，一一对应的关系就不那么明显了，具体情况具体分析，
```shell
root@zynqmp:~# lspci -tv
-[0000:00]---00.0-[01-07]----00.0-[02-07]--+-00.0-[03]----00.0  PLX Technology, Inc. Device 87b1
                                           +-01.0-[04]----00.0  Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961
                                           +-02.0-[05]----00.0  Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961
                                           +-03.0-[06]----00.0  Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961
                                           \-08.0-[07]--
```
PCI内存映射配置空间，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190401112902605.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# Switch Routing
As you told Memory Write transactions requires "valid" Addr of recipient and Data and Memory read transactions requires "Vaild" Addr and data "size " required to creates write or read requests. For the requests from Downstream port, switch takes care of routing to any Upstream (Root) or Downstream (peer to peer), by comparing with its "Base and Limit" Registers. Switch Routing: First check the address on its own bars, if it matched, it will consume. Two bars are available in each Switches. If not, check its IO/P-MMIO/NP-MMIO Base and Limit Register pairs based on the request type. If a TLP travels to Upstream port and if it matches to its Base and Limit Registers it will be handled as "Unsupported Request" on secondary interface. ( again it will pass to the downstream port, other than one that it received, since it may be peer-to-peer communication). If not matches at any interface, it will forwarded to its primary interface as it not matches for the bridge and any function beneath this bridge.

# 硬件架构
4.4节Hardware Architecture，PEX8724包含两个Station，Station 0对应Port 0 ~ Port 3，Station 1对应Port 8 ~ Port 10，其中Port 8是软件虚拟的，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331145852415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 端口配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331150505570.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 芯片复位
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019033115081247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# I2C
寄存器访问方法，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331150933821.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
写命令格式，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331151552684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
读命令格式稍有不同，Command字段为100b，写数据流程，
```shell
start > addr > cmd > data > stop
```
读数据流程为，
```shell
start > addr > cmd > stop > start > addr > data > stop
```

# Serial EEPROM
SPI接口的EEPROM，刚开始是11，表示找到EEPROM但是没有有效的配置数据。
![322](https://img-blog.csdnimg.cn/20200710152243806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
REG BYTE COUNT是总字节数，不是寄存器个数。
![323](https://img-blog.csdnimg.cn/2020071517170712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# Read Pacing
Read Pacing是协调资源分配，在大流量设备工作时，防止小流量设备得不到仲裁被饿死，

# ACS Extended Capability Registers

# Egress/Ingress Control

