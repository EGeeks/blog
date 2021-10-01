# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Intel ISDCT, SanDisk SCLI和NVMe-CLI三款SSD工具基本操作简介](https://www.chiphell.com/thread-1904828-1-1.html)
> [nvme-cli常用指令](https://blog.csdn.net/weixin_40343504/article/details/82386024)
> [nvme-cli官网](https://github.com/linux-nvme/nvme-cli)

# 显示系统内的NVMe SSD
采用list命令，
```shell
root@zynqmp:~# nvme list
Node             SN                   Model                                    Namespace Usage                      Format           FW Rev  
---------------- -------------------- ---------------------------------------- --------- -------------------------- ---------------- --------
/dev/nvme0n1     S3EVNX0K101880M      Samsung SSD 960 PRO 1TB                  1           4.54  GB /   1.02  TB    512   B +  0 B   2B6QCXP7
/dev/nvme1n1     S3EVNX0K101840H      Samsung SSD 960 PRO 1TB                  1           4.54  GB /   1.02  TB    512   B +  0 B   2B6QCXP7
/dev/nvme2n1     S3EVNX0K101883A      Samsung SSD 960 PRO 1TB                  1           4.54  GB /   1.02  TB    512   B +  0 B   2B6QCXP7
/dev/nvme3n1     S3EVNX0K101887V      Samsung SSD 960 PRO 1TB                  1         377.06  MB /   1.02  TB    512   B +  0 B   2B6QCXP7
```
# 读写NVMe
nvme-cli提供接口可直接读写扇区，这里以扇区大小为512，-c指定block个数，从0开始，即0表示一个block。
```shell
root@zynqmp:~# nvme write /dev/nvme0n1 -s 0 -c 1 -z 0x400 -d ./start.sh -t
 latency: write: 325680 us
write: Success
root@zynqmp:~# 
root@zynqmp:~# 
root@zynqmp:~# nvme read /dev/nvme0n1 -s 0 -c 1 -z 0x400 -d ./a.bin -t
 latency: read: 331882 us
read: Success
root@zynqmp:~# hexdump -C a.bin 
00000000  23 21 2f 62 69 6e 2f 73  68 0a 0a 6d 6f 64 70 72  |#!/bin/sh..modpr|
00000010  6f 62 65 20 70 63 69 65  5f 65 70 20 70 63 69 65  |obe pcie_ep pcie|
00000020  5f 65 70 5f 76 69 64 3d  30 78 31 30 62 35 20 70  |_ep_vid=0x10b5 p|
00000030  63 69 65 5f 65 70 5f 64  69 64 3d 30 78 38 37 32  |cie_ep_did=0x872|
00000040  34 0a 6d 6f 64 70 72 6f  62 65 20 6e 76 6d 65 64  |4.modprobe nvmed|
00000050  0a 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000060  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00000400
```
在T2080平台上测试，最大操作2MB的读写，否则报错，
```shell
root@t2080rdb:~# nvme write /dev/nvme0n1 -s 0 -c 2047 -z 0x100000 -d ./start.sh -t
 latency: write: 527 us
write: Success
root@t2080rdb:~# 
root@t2080rdb:~# 
root@t2080rdb:~# nvme write /dev/nvme0n1 -s 0 -c 4095 -z 0x200000 -d ./start.sh -t     
 latency: write: 995 us
write: Success
root@t2080rdb:~#   
root@t2080rdb:~# 
root@t2080rdb:~# nvme write /dev/nvme0n1 -s 0 -c 8191 -z 0x400000 -d ./start.sh -t     
 latency: write: 2574 us
write:INVALID_FIELD: A reserved coded value or an unsupported value in a defined field(2002)
```
实际比对数据，发现0x100000长度才能保证读写正确，
```shell
root@t2080rdb:~# nvmeqe_benchmark -w /dev/nvme0n1 -s 0x100000 -l 0x100000 -t 1   
speed: 200.00MB/s, cost times: 5ms
root@t2080rdb:~# 
root@t2080rdb:~# nvmeqe_benchmark -r /dev/nvme0n1 -s 0x100000 -l 0x100000 -t 1  
read: 
hello nvmeqe_benchmark...
speed: 166.67MB/s, cost times: 6ms
```
# 显示磁盘信息
identify命令，
```shell
root@zynqmp:~# nvme id-ctrl /dev/nvme0
NVME Identify Controller:
vid       : 0x144d
ssvid     : 0x144d
sn        : S3EVNX0K101880M     
mn        : Samsung SSD 960 PRO 1TB                 
fr        : 2B6QCXP7
rab       : 2
ieee      : 002538
cmic      : 0
mdts      : 9
cntlid    : 2
ver       : 10200
rtd3r     : 186a0
rtd3e     : 4c4b40
oaes      : 0
ctratt    : 0
rrls      : 0
oacs      : 0x7
acl       : 7
aerl      : 3
frmw      : 0x16
lpa       : 0x3
elpe      : 63
npss      : 4
avscc     : 0x1
apsta     : 0x1
wctemp    : 346
cctemp    : 349
mtfa      : 0
hmpre     : 0
hmmin     : 0
tnvmcap   : 1024209543168
unvmcap   : 0
rpmbs     : 0
edstt     : 0
dsto      : 0
fwug      : 0
kas       : 0
hctma     : 0
mntmt     : 0
mxtmt     : 0
sanicap   : 0
hmminds   : 0
hmmaxd    : 0
nsetidmax : 0
anatt     : 0
anacap    : 0
anagrpmax : 0
nanagrpid : 0
sqes      : 0x66
cqes      : 0x44
maxcmd    : 0
nn        : 1
oncs      : 0x1f
fuses     : 0
fna       : 0x5
vwc       : 0x1
awun      : 255
awupf     : 0
nvscc     : 1
nwpc      : 0
acwu      : 0
sgls      : 0
mnan      : 0
subnqn    : 
ioccsz    : 0
iorcsz    : 0
icdoff    : 0
ctrattr   : 0
msdbd     : 0
ps    0 : mp:6.90W operational enlat:0 exlat:0 rrt:0 rrl:0
          rwt:0 rwl:0 idle_power:- active_power:-
ps    1 : mp:5.50W operational enlat:0 exlat:0 rrt:1 rrl:1
          rwt:1 rwl:1 idle_power:- active_power:-
ps    2 : mp:5.10W operational enlat:0 exlat:0 rrt:2 rrl:2
          rwt:2 rwl:2 idle_power:- active_power:-
ps    3 : mp:0.0500W non-operational enlat:210 exlat:1200 rrt:3 rrl:3
          rwt:3 rwl:3 idle_power:- active_power:-
ps    4 : mp:0.0080W non-operational enlat:2000 exlat:6000 rrt:4 rrl:4
          rwt:4 rwl:4 idle_power:- active_power:-
```
# 显示namespace信息
identify命令，
```shell
root@zynqmp:~# nvme id-ns /dev/nvme0n1
NVME Identify Namespace 1:
nsze    : 0x773bd2b0
ncap    : 0x773bd2b0
nuse    : 0x873430
nsfeat  : 0
nlbaf   : 0
flbas   : 0
mc      : 0
dpc     : 0
dps     : 0
nmic    : 0
rescap  : 0
fpi     : 0x80
dlfeat  : 0
nawun   : 0
nawupf  : 0
nacwu   : 0
nabsn   : 0
nabo    : 0
nabspf  : 0
noiob   : 0
nvmcap  : 1024209543168
nsattr  : 0
nvmsetid: 0
anagrpid: 0
endgid  : 0
nguid   : 00000000000000000000000000000000
eui64   : 0025385181b14235
lbaf  0 : ms:0   lbads:9  rp:0 (in use)
```

# 查看寿命
nvme smart-log /dev/nvme*n1读取smart信息，其中percentage used表示寿命百分比，0表示全新，100表示寿命耗尽。

# help
```shell
root@zynqmp:~# nvme help
nvme-1.8.1
usage: nvme <command> [<device>] [<args>]

The '<device>' may be either an NVMe character device (ex: /dev/nvme0) or an
nvme block device (ex: /dev/nvme0n1).

The following are all implemented sub-commands:
  list                  List all NVMe devices and namespaces on machine
  list-subsys           List nvme subsystems
  id-ctrl               Send NVMe Identify Controller
  id-ns                 Send NVMe Identify Namespace, display structure
  list-ns               Send NVMe Identify List, display structure
  ns-descs              Send NVMe Namespace Descriptor List, display structure
  id-nvmset             Send NVMe Identify NVM Set List, display structure
  create-ns             Creates a namespace with the provided parameters
  delete-ns             Deletes a namespace from the controller
  attach-ns             Attaches a namespace to requested controller(s)
  detach-ns             Detaches a namespace from requested controller(s)
  list-ctrl             Send NVMe Identify Controller List, display structure
  get-ns-id             Retrieve the namespace ID of opened block device
  get-log               Generic NVMe get log, returns log in raw format
  telemetry-log         Retrieve FW Telemetry log write to file
  fw-log                Retrieve FW Log, show it
  changed-ns-list-log   Retrieve Changed Namespace List, show it
  smart-log             Retrieve SMART Log, show it
  ana-log               Retrieve ANA Log, show it
  error-log             Retrieve Error Log, show it
  effects-log           Retrieve Command Effects Log, show it
  endurance-log         Retrieve Endurance Group Log, show it
  get-feature           Get feature and show the resulting value
  device-self-test      Perform the necessary tests to observe the performance
  self-test-log         Retrieve the SELF-TEST Log, show it
  set-feature           Set a feature and show the resulting value
  set-property          Set a property and show the resulting value
  get-property          Get a property and show the resulting value
  format                Format namespace with new block format
  fw-commit             Verify and commit firmware to a specific slot (fw-activate in old version < 1.2)
  fw-download           Download new firmware
  admin-passthru        Submit an arbitrary admin command, return results
  io-passthru           Submit an arbitrary IO command, return results
  security-send         Submit a Security Send command, return results
  security-recv         Submit a Security Receive command, return results
  resv-acquire          Submit a Reservation Acquire, return results
  resv-register         Submit a Reservation Register, return results
  resv-release          Submit a Reservation Release, return results
  resv-report           Submit a Reservation Report, return results
  dsm                   Submit a Data Set Management command, return results
  flush                 Submit a Flush command, return results
  compare               Submit a Compare command, return results
  read                  Submit a read command, return results
  write                 Submit a write command, return results
  write-zeroes          Submit a write zeroes command, return results
  write-uncor           Submit a write uncorrectable command, return results
  sanitize              Submit a sanitize command
  sanitize-log          Retrieve sanitize log, show it
  reset                 Resets the controller
  subsystem-reset       Resets the subsystem
  ns-rescan             Rescans the NVME namespaces
  show-regs             Shows the controller registers or properties. Requires character device
  discover              Discover NVMeoF subsystems
  connect-all           Discover and Connect to NVMeoF subsystems
  connect               Connect to NVMeoF subsystem
  disconnect            Disconnect from NVMeoF subsystem
  disconnect-all        Disconnect from all connected NVMeoF subsystems
  gen-hostnqn           Generate NVMeoF host NQN
  dir-receive           Submit a Directive Receive command, return results
  dir-send              Submit a Directive Send command, return results
  virt-mgmt             Manage Flexible Resources between Primary and Secondary Controller 
  version               Shows the program version
  help                  Display this help

See 'nvme help <command>' for more information on a specific command

The following are all installed plugin extensions:
  intel           Intel vendor specific extensions
  lnvm            LightNVM specific extensions
  memblaze        Memblaze vendor specific extensions
  wdc             Western Digital vendor specific extensions
  huawei          Huawei vendor specific extensions
  netapp          NetApp vendor specific extensions
  toshiba         Toshiba NVME plugin
  micron          Micron vendor specific extensions
  seagate         Seagate vendor specific extensions

See 'nvme <plugin> help' for more information on a plugin
```

