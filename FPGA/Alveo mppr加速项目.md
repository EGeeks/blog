# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# mppr
> [Varint编码](https://www.cnblogs.com/jacksu-tencent/p/3389843.html)
> [整数压缩编码 ZigZag](https://www.cnblogs.com/en-heng/p/5570609.html)
> [ZigZag 与 反ZigZag编码](https://my.oschina.net/tigerBin/blog/1083549)

FPGA解压缩后整数要按照大端模式发送到网络上，
> [大小端及网络字节序](https://blog.csdn.net/z_ryan/article/details/79134980)
> [关于网络字节序](https://blog.csdn.net/zhaojiangwei102/article/details/4532184)

连线：
- 光纤1/12接enp1s0，靠近主板的那个，解码数据从这个接口发出，wireshark监控这个接口
- 光纤2/11接enp1s0d1，远离主板的那个，qt从这个接口发送测试包

问题：
- FIELD_INCREMENT_HEAD包含数据data12，根据data12找到是哪个静态数据，如果没有这个字段则后面的数据忽略，等待下一个UDP包
- FIELD_INCREMENT_VALU，同一个组合域里面可出现多次该数据类型，做Incrementdata42操作，首先得到st_data_calua数据结构，
- `FIELD_INCREMENT_TINFO`等需要FPGA计算浮点数乘法，浮点数的表示方法，目前来看都是int64*double，输出double，后发现Xilinx有相关转换IP，在Math Functions下面的Floating-point中，DSP48E可用于加减乘除运算，Floating-point还有直接使用DSP48E计算加减乘除的功能，采用两个Floating-point IP，一个定点转浮点，加上一个双精度浮点乘法即可

> [verilog中对浮点数的处理](https://blog.csdn.net/bleauchat/article/details/100024750)
> [FPGA浮点数定点数的处理](https://www.cnblogs.com/Dinging006/p/9450577.html)
> [verilog 浮点数转定点数_FPGA浮点数定点数的处理](https://blog.csdn.net/weixin_28912709/article/details/112934737)
> [verilog中的定点数、浮点数、定点小数、定点整数的表示及运算](https://blog.csdn.net/woshiyuzhoushizhe/article/details/105920861)
> [verilog 浮点转定点_定点数和浮点数](https://blog.csdn.net/weixin_39779537/article/details/111279559)
> [Xilinx怎么定点数转浮点数](http://www.elecfans.com/pld/703134.html)
> [Xilinx大神都懂的数字运算单元—DSP48E1](http://xilinx.eetrend.com/content/2020/100049181.html)
> [DSP48E1详解-2](https://blog.csdn.net/abcdef123456gg/article/details/103709480)

1. ST_DATA_HEAD等结构体没有加`__attribute__((packed))`，这个在不同编译器下会不会出问题？你们用的什么编译器，gcc？visual studio？
2. pcap文件第32314/45361个包的ST_DATA_HEAD->fieldSize为负值（是否是`__attribute__((packed))`的问题？），导致指针溢出。
3. Incrementdata42的作用是什么，函数写的比较复杂，我检索了整个600MB的pcap文件，没有调用这个函数。

20210219
varint解码必须等待上一个varint解码完成才可以，因为varint需要输入的数据是变长的，需要遍历数据解码后才能获知需要的数据长度，FIELD_INCREMENT_VALU类型的解码比较特殊，首先跳过了2个字节才开始varint解码，后面代码中`phead += data_head.fieldSize - nlen;`，即把数据消耗量强制对齐到data_head.fieldSize，而其他类型没有这个操作，varint解码出消耗多少字节就是多少，这里有特殊的设计吗？比如可以让data_head.fieldSize在每种类型的解码里都代码数据总消耗量。若想让解码性能达到最好，需要软硬件协同优化，同时，如果FPGA设计完成，软件就不能轻易改动了，可能会造成FPGA设计完全作废，架构需要重新设计。我这里先按照Demo软件的设计。
`pmkt->data10*pmkt->data7`可由驱动计算好写入静态数据包的最后。

调试机，`task=0x1`，
```bash
$ ssh test@192.168.2.227
$ sudo lspci -vv -s 01:00.0
$ sudo lspci -vv -d 1994:0828
$ sudo modprobe uio && sudo insmod pcie_chipset.ko
$ dmesg | grep pcie_chipset
[ 1002.151418] pcie_chipset: loading out-of-tree module taints kernel.
[ 1002.151472] pcie_chipset: module verification failed: signature and/or required key missing - tainting kernel
[ 1002.152038] pcie_chipset: Mar  1 2021 - 22:40:26
[ 1002.152067] pcie_chipset 0000:01:00.0: vendor[0x1994], device[0x0828], subvendor[0x1994], subdevice[0x1994], class[0x20000]
[ 1002.152070] pcie_chipset 0000:01:00.0: enabling device (0000 -> 0002)
[ 1002.152178] pcie_chipset 0000:01:00.0: dma_set_mask_and_coherent 32bit success
[ 1002.152179] pcie_chipset 0000:01:00.0: BAR0, start=0xf6000000, end=0xf6ffffff, len=0x1000000, flags=0x40200
[ 1002.152190] pcie_chipset 0000:01:00.0: auto find ucode, offset=0x0
[ 1002.152240] pcie_chipset 0000:01:00.0: ucode addr is 00000000d804518d[auto], origin addr=0000000066e3fb95
[ 1002.152241] pcie_chipset 0000:01:00.0: ucode version is 20210418.8064200
[ 1002.152242] pcie_chipset 0000:01:00.0: modules number is 6
[ 1002.152242] pcie_chipset 0000:01:00.0: find ucode device pmem
[ 1002.152243] pcie_chipset 0000:01:00.0: find ucode device cmem
[ 1002.152243] pcie_chipset 0000:01:00.0: find ucode device chipset_intc
[ 1002.152244] pcie_chipset 0000:01:00.0: find ucode device xintc
[ 1002.152244] pcie_chipset 0000:01:00.0: find ucode device xaxienet_dma
[ 1002.152245] pcie_chipset 0000:01:00.0: find ucode device xaxinet
[ 1002.152246] pcie_chipset 0000:01:00.0: probe ucode device pmem0@f6000000
[ 1002.152287] pcie_chipset 0000:01:00.0: probe ucode device cmem1@f6000000
[ 1002.152306] pcie_chipset 0000:01:00.0: probe ucode device csintc2@f6000000
[ 1002.152336] pcie_chipset 0000:01:00.0: chipset intc request 1 irqs, get 1 irqs, irq number:
[ 1002.152337] pcie_chipset 0000:01:00.0:   133
[ 1002.152337] pcie_chipset 0000:01:00.0: probe ucode device xintc3@f6200000
[ 1002.152346] pcie_chipset 0000:01:00.0: probe ucode device xaxienet5@f6300000
[ 1002.156855] pcie_chipset 0000:01:00.0 enp1s0d5: renamed from eth0
# 设置静态数据，否则没有静态数据，无法解码，发送端口显示没有任何解码包
$ sudo ./c 1 1
# 或
$ sudo ./mppr_pc -v
# 查看mppr控制寄存器，ctrl/status/ns_l/ns_h/max_ip_white_num/max_static_data_num
$ sudo memtool md 0xf6210000
# 查看IP白名单，16个u32
$ sudo memtool md 0xf6211000
# 查看静态数据data12，64个u32
$ sudo memtool md 0xf6212000
# 查看静态数据ram
$ sudo memtool md 0xf6800000
# 白名单，理论上只有前两个，
$ cat ip_white.txt 
239.3.42.71
239.4.42.72
239.4.41.72
# 开始捕获
$ sudo ./mppr_pc -p 0xf6000000 -l ip_white.txt -k ./20201203_1.dat -f a.bin
# 服务器
# ./mppr_pc -p 0x91000000 -l ip_white.txt -k ./20201203_1.dat -f a.bin
```
开发机，
```bash
# 生成调试激励，目前生成之后需要手动修改生成.v文件一点代码，module packet_ram_1_20210730，然后修改mppr_core_tb.v文件
$ /media/qe/SN550/au200/build-c-Desktop_Qt_5_9_6_GCC_64bit-Debug/c 2 8 /media/qe/SN550/project/au200/au200.srcs/sim_1/new/packet_ram.v //28069
$ /media/qe/SN550/au200/build-c-Desktop_Qt_5_9_6_GCC_64bit-Debug/c 2 8 /media/qe/SN550/project/au200/au200.srcs/sim_1/new/packet_ram_28158.v
$ /media/qe/SN550/au200/build-c-Desktop_Qt_5_9_6_GCC_64bit-Debug/c 2 8 /media/qe/SN550/project/au200/au200.srcs/sim_1/new/packet_ram_1_20210730.v
$ /media/qe/SN550/au200/build-c-Desktop_Qt_5_9_6_GCC_64bit-Debug/c 2 16 /media/qe/SN550/project/au200/au200.srcs/sim_1/new/st_data.coe
$ /media/qe/SN550/au200/build-c-Desktop_Qt_5_9_6_GCC_64bit-Debug/c 2 32 #生成寄存器verilog定义
# 上传静态数据文件
$ sftp test@192.168.2.227
sftp> put -rf /media/qe/SN550/au200/20201203.dat # 老版，没有按字节对齐，后修正。
sftp> put -rf /media/qe/SN550/au200/20201203_1.dat
# 抓包，目前enp1s0对应qspf0_0，用于验证axi dma万兆网卡，enp1s0d1对应qspf0_1，用于验证mppr
$ sudo tcpdump -i enp1s0d1
```
开发机chipset驱动，
```bash
$ source /opt/fdk/settings.sh -t x86
$ cd /opt/fdk_develop/package/chipset
$ ./build.sh -b x86 DRV_TYPE_PCIE
$ sftp test@192.168.2.227
sftp> put pcie_chipset.ko
```
开发机Qt，
```bash
# 传输代码
$ sftp test@192.168.2.227
sftp> put -rf /media/qe/SN550/au200/c
# 传输2进制
sftp> put -f /media/qe/SN550/au200/build-c-Desktop_Qt_5_9_6_GCC_64bit-Debug/c
```
开发机mppr_pc，
```bash
$ source /opt/fdk/settings.sh -t x86
$ cd /opt/fdk_develop
$ code package/hw
$ ./develop.sh -t x86 -b package hw
$ sftp test@192.168.2.227
sftp> put package/hw/mppr_pc
$ scp package/hw/mppr_pc test@192.168.2.227:/home/test
$ scp package/hw/mppr_pc test@192.168.2.227:~
$ scp package/hw/mppr_pc root@172.23.151.:~
```
开发机生成ucode，
```bash
$ sudo /opt/Qt5.9.6/Tools/QtCreator/bin/qtcreator
$ cp ~/project/qt/build-recorder-Desktop_Qt_5_9_6_GCC_64bit-Debug/cfg/mppr_ucode.coe /media/qe/SN550/project/au200/au200.srcs/sources_1/new/ # 更新ucode
```
远程登录，
```bash
$ sudo systemctl start EasyMonitor.service
# 打开EasyConnect UI界面，输入VPN网址，点击下一步，输入用户名密码
# 打开VNC Viewer UI界面，
```

20200418确认两个问题，
1. 授时，时间，万年历（年月日时分秒）
2. FIELD_INCREMENT_VALU的Incrementdata42函数实现

20210406，客户修改静态数据，按1字节对齐，
![2021-04-04 23-48-17](https://img-blog.csdnimg.cn/20210406210512840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)发送第28158个包会导致发送紊乱，一直处于发包状态，

**20210619** 尚未解决
mppr_pc问题，代码寄存器偏移填错，导致hcb vars avail不对，这个软件要错误处理，不能比size还大，
```bash
->showbuf
hcb regs begin**
base  =0xffffffffffffffff
size  =0xffffffffffffffff
head  =0xffffffffffffffff
tail  =0xffffffffffffffff
avail =0xffffffffffffffff
occupy=0xffffffffffffffff
ctrl  =0xffffffff
status=0xffffffff
ipc regs end****
hcb vars begin**
base  =0x000000003f400000
size  =0x0000000000400000
head  =0x0000000000300000
tail  =0xffffffffffffffff
avail =0x0000000725f00001
occupy=0xfffffff8da4fffff
ipc vars end****
```
shell不支持上下键调出历史，c语言添加历史命令buf，检测上下按键码做处理。
已解决：shell增加hexdump circbuf的命令

20200622，验证循环缓冲区收到的包和qsfp转发的包个数是否相等，参考对比mppr.pcap，
```bash
->showbuf #抓到9个包，tail是对的
hcb regs begin**
base           =0x000000003f400000
size           =0x0000000000400000
head           =0x0000000000000000
tail           =0x0000000000004800
avail          =0x00010000807fafee
occupy         =0x0000000000004800
addr_inc_preset=0x0000000000000800
ctrl           =0x00000002
status         =0x00000000
ipc regs end****
hcb vars begin**
base           =0x000000003f400000
size           =0x0000000000400000
head           =0x0000000000000000
tail           =0x0000000000004800
avail          =0x00000000003fb800
occupy         =0x0000000000004800
addr_inc_preset=0x0000000000000800
ipc vars end****
[result]: command executed successfully
->showbuf #抓到14个包，共23个包，tail是对的
hcb regs begin**
base           =0x000000003f400000
size           =0x0000000000400000
head           =0x0000000000000000
tail           =0x000000000000b800
avail          =0x00010000807e31d2
occupy         =0x000000000000b800
addr_inc_preset=0x0000000000000800
ctrl           =0x00000002
status         =0x00000000
ipc regs end****
hcb vars begin**
base           =0x000000003f400000
size           =0x0000000000400000
head           =0x0000000000000000
tail           =0x000000000000b800
avail          =0x00000000003f4800
occupy         =0x000000000000b800
addr_inc_preset=0x0000000000000800
ipc vars end****
[result]: command executed successfully
```
已解决：mppr应用不支持设置白名单IP，可以用配置文件的方式输入IP
静态数据目前只支持64个，扩大到1024个

下载的时候，下载到0x01002000偏移处
```bash
write_cfgmem  -format bin -size 128 -interface SPIx4 -loadbit {up 0x0 "/media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bit" } -file "/media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bin"
# write_cfgmem  -format bin -size 128 -interface SPIx4 -loadbit {up 0x01002000 "/media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bit" } -file "/media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bin"
Command: write_cfgmem -format bin -size 128 -interface SPIx4 -loadbit {up 0x01002000 "/media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bit" } -file /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bin
Creating config memory files...
Creating bitstream load up from address 0x01002000
Loading bitfile /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bit
Writing file /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bin
Writing log file /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.prm
===================================
Configuration Memory information
===================================
File Format        BIN
Interface          SPIX4
Size               128M
Start Address      0x00000000
End Address        0x07FFFFFF

Addr1         Addr2         Date                    File(s)
0x01002000    0x0328094B    Jul  3 21:31:28 2021    /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bit
0 Infos, 0 Warnings, 0 Critical Warnings and 0 Errors encountered.
write_cfgmem completed successfully
write_cfgmem: Time (s): cpu = 00:00:07 ; elapsed = 00:00:07 . Memory (MB): peak = 8768.953 ; gain = 0.000 ; free physical = 20138 ; free virtual = 27109
```
远程，
```bash
$ sudo systemctl start EasyMonitor.service
$ sftp root@172.23.151.220  
sftp> put chipset.tar.gz  
sftp> put hw.tar.gz
sftp> exit
# 编译
$ ssh root@172.23.151.220

# 下载FPGA
$ sftp root@172.23.151.105
sftp> lcd /media/qe/SN550/project/au200/au200.runs/impl_1
sftp> put system_wrapper.bin 
sftp> put system_wrapper.bit
sftp> put system_wrapper.ltx
```
服务器配置，CentOS 7.6
```bash
# uname -a
Linux localhost.localdomain 3.10.0-957.el7.x86_64 #1 SMP Thu Nov 8 23:39:32 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core)
```
开启远程桌面，
```bash
# 5901
# vncserver :1
```
1. 登录alveo卡所在服务器，进入`/root`目录，执行命令`./mppr_pc -p 0x91000000 -l ip_white.txt -k ./20201203_1.dat -f a.bin`，
`-l`指定IP白名单文件，`-k`指定静态数据文件，`-f`指定pcie接收的数据存储到本地文件。执行`./mppr_pc -h`可输出帮助说明。
2. 登录172.23.151.220，即alveo卡所在服务器，执行下面命令`./mppr_pc -p 0x91000000 -l ip_white.txt -k ./new0728.dat -d 1`，该命令不存盘，只监控FPGA状态，
3. 登录172.23.151.105，即USB调试线所在服务器，如下图，点击1处三角形按钮，观察下面2处的core status，绿灯转移到最右侧，同时文字从idle变为full，则说明网络上有命中IP白名单的数据包。
![请添加图片描述](https://img-blog.csdnimg.cn/d78da654baea4fc6835aebf9c6915b3e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
```bash
write_hw_ila_data -force /root/ila2_data_2 [upload_hw_ila_data hw_ila_2]
$ sftp root@172.23.151.105
sftp> put ila2_data_2
read_hw_ila_data E:/project/vivado2015.2.1/finace/tcp_ila_data.ila
display_hw_ila_data
```
20210802 前端待添加丢包统计,

# microblaze
```bash
$ exec updatemem -force -meminfo /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.mmi -bit /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.bit -data /media/qe/SN550/project/vitis/mppr/Debug/mppr.elf -proc system_i/microblaze_0 -out /media/qe/SN550/project/au200/au200.runs/impl_1/temp1.bit 
$ exec updatemem -force -meminfo /media/qe/SN550/project/au200/au200.runs/impl_1/system_wrapper.mmi -bit /media/qe/SN550/project/au200/au200.runs/impl_1/temp1.bit -data /media/qe/SN550/project/au200/au200.srcs/sources_1/bd/system/ip/system_cms_subsystem_0_0/fw/cms.elf -proc system_i/cms_subsystem_0/inst/shell_cmc_subsystem/inst/microblaze_cmc -out /media/qe/SN550/project/au200/au200.runs/impl_1/download.bit 
$ exec rm /media/qe/SN550/project/au200/au200.runs/impl_1/temp1.bit 
```

# 初始化脚本
au200所在机器，
```bash
[root@localhost ~]# vim init_au200.sh
[root@localhost ~]# cat init_au200.sh 
#!/bin/sh
echo "exec script with root"
modprobe uio && sudo insmod pcie_chipset.ko && dmesg | grep pcie_chipset
./mppr_pc -h
[root@localhost ~]# chmod +x init_au200.sh 
[root@localhost ~]# ./init_au200.sh
```
jtag所在机器，
```bash
$ vim download.tcl
$ cat download.tcl 
open_hw_manager
connect_hw_server -allow_non_jtag
current_hw_target [lindex [get_hw_targets] 0]
set_property PARAM.FREQUENCY 15000000 [lindex [get_hw_targets] 0]
open_hw_target
current_hw_device [get_hw_devices xcu200_0]
refresh_hw_device -update_hw_probes false [lindex [get_hw_devices xcu200_0] 0]
set_property PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
set_property FULL_PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
set_property PROGRAM.FILE {/root/download.bit} [get_hw_devices xcu200_0]
program_hw_devices [get_hw_devices xcu200_0]
close_hw_manager

$ chmod +x download.tcl 
$ vim prog_au200.sh
$ cat prog_au200.sh 
#!/bin/sh

source /opt/Xilinx/Vivado/2020.1/settings64.sh
echo "exit
" | vivado -mode tcl -source download.tcl

$ chmod +x prog_au200.sh 
$ ./prog_au200.sh 

****** Vivado v2020.1 (64-bit)
  **** SW Build 2902540 on Wed May 27 19:54:35 MDT 2020
  **** IP Build 2902112 on Wed May 27 22:43:36 MDT 2020
    ** Copyright 1986-2020 Xilinx, Inc. All Rights Reserved.

source download.tcl
# open_hw_manager
# connect_hw_server -allow_non_jtag
INFO: [Labtools 27-2285] Connecting to hw_server url TCP:localhost:3121
INFO: [Labtools 27-3415] Connecting to cs_server url TCP:localhost:3042
INFO: [Labtools 27-3414] Connected to existing cs_server.
# current_hw_target [lindex [get_hw_targets] 0]
# set_property PARAM.FREQUENCY 15000000 [lindex [get_hw_targets] 0]
# open_hw_target
INFO: [Labtoolstcl 44-466] Opening hw_target localhost:3121/xilinx_tcf/Xilinx/2130069AF04JA
# current_hw_device [get_hw_devices xcu200_0]
# refresh_hw_device -update_hw_probes false [lindex [get_hw_devices xcu200_0] 0]
INFO: [Labtools 27-2302] Device xcu200 (JTAG device index = 0) is programmed with a design that has 2 ILA core(s).
# set_property PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
# set_property FULL_PROBES.FILE {/root/system_wrapper.ltx} [get_hw_devices xcu200_0]
# set_property PROGRAM.FILE {/root/download.bit} [get_hw_devices xcu200_0]
# program_hw_devices [get_hw_devices xcu200_0]
INFO: [Labtools 27-3164] End of startup status: HIGH
program_hw_devices: Time (s): cpu = 00:00:18 ; elapsed = 00:00:18 . Memory (MB): peak = 2943.668 ; gain = 9.004 ; free physical = 56181 ; free virtual = 90974
# close_hw_manager
****** Webtalk v2020.1 (64-bit)
  **** SW Build 2902540 on Wed May 27 19:54:35 MDT 2020
  **** IP Build 2902112 on Wed May 27 22:43:36 MDT 2020
    ** Copyright 1986-2020 Xilinx, Inc. All Rights Reserved.

source /root/.Xil/Vivado-31363-localhost.localdomain/webtalk/labtool_webtalk.tcl -notrace
INFO: [Common 17-206] Exiting Webtalk at Fri Aug 13 21:23:51 2021...
exit
INFO: [Common 17-206] Exiting Vivado at Fri Aug 13 21:23:51 2021...
```
