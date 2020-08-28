# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

Finace-FPGA软件使用文档
# 硬件
## 版本
目前有两个版本的硬件，
- V1 Xilinx Zynq xc7z035 2路10g万兆网
- V2 Xilinx Kintex xc7k325t 4路10g万兆网

# FPGA
## 版本
目前有两个版本的FPGA，针对两个不同版本的硬件，尚未加入版本管理模块。
# 驱动
## 版本
目前有两个版本的驱动，针对两个不同版本的硬件，尚未加入版本管理模块。运行于Centos7.6/Redhat7.6上。

# 应用软件
## 版本
目前有两个版本的软件，针对两个不同版本的硬件。
（1）1.0.0 
初始版本。
（2）1.0.1
增加了内部shell，提供一些命令查看抓包信息。
# 使用
（1）硬件安装到服务器PCIe槽位。
（2）将软件包`finace_install.tar.gz`下载到服务器，解压。
```shell
# 解压
$ tar -zxvf finace_install.tar.gz
$ cd finace_install
$ chmod +x start.sh
$ chmod +x finace_pc
```
（3）查看当前网卡列表。
```shell
$ ifconfig -a
```
（4）安装驱动。
```shell
$ sudo ./start.sh
```
在Centos7.6/Redhat7.6上开机自动加载驱动，
```shell
sudo chmod +x /etc/rc.local
sudo chmod +x /etc/rc.d/rc.local
```
然后编辑`sudo vim /etc/rc.local`，
```shell
cd /home/qe/project/finace_install # 此处替换成你的路径
./start.sh
```
（5）安装驱动后，V1版本的硬件，系统会多出两块网卡，对应我们FPGA板卡上的两个SFP接口，对比一下多出的是哪两个网口，我这里是`enp1s0`和`eth0f1`，按照顺序，前面的那个对应板卡SFP-A，后者是SFP-B，在linux操作系统下，网卡名为，`eth0f*`，V1对应`eth0f0~eth0f1`，V2对应`eth0f0~eth0f3`，linux下udev会对网卡重命名，所以通常`eth0f0`会被重命名为`enp1s0`。V1 离金手指近的是第2个网口`eth0f1`，板卡丝印SFP-B，远的是第1个网口`eth0f0`，板卡丝印SFP-A。V2 离金手指近的是第4个网口`eth0f3`，远的是第1个网口`eth0f0`，依此类推。
```shell
$ ifconfig -a
```
（6）运行抓包程序，`sudo`表示以root模式执行，如果当前终端是root登录则去掉命令中的`sudo`，怎样判断是否处于root模式：如果终端提示符为`#`，则是root，如果终端提示符为`$`，则是普通用户，如果需要同时抓两个接口，则需要开启两个终端，分别运行下面的命令，
```shell
# 终端1
$ sudo ./finace_pc -i enp1s0 -s 0x100000 -f a.pcap
# 终端2
$ sudo ./finace_pc -i eth0f1 -s 0x100000 -f b.pcap
```
抓包程序`finace_pc`提供功能如下，不同版本显示信息可能变化，请以实际为准。
```shell
[qe@localhost project]$ ./finace_pc -h
Usage: finace [OPTION]...
pcap app

-i  [str],        nic name # 捕获的网卡接口
-s  [num],        min write size # 单次写入大小，如果外部包流量很大，这个值需要调大，0~2MB之间，通常设为1MB
-f  [str],        file # 写入的文件
-T  [num],        create new file when run [num] seconds # 每隔多少秒自动创建新文件
-L  [num],        create new file when file size > [num] # 文件达到指定大小之后自动创建新文件
-d       ,        show detail info, d = 0, 1, 2... # 调试使用，一般不要打开，默认为0
-v       ,        show version # 显示版本信息
-h       ,        show this help # 打开帮助

Example:
finace_pc -i enp1s0
```
内部shell提供的接口如下，在shell下按`enter`可弹出提示符`->`，输入`help`可弹出命令帮助。
```shell
[qe@localhost hw]$ sudo ./finace_pc -i enp1s0 -s 0x1000 -f ./a.pcap
create file time: 0s
create file size: 0Byte
set pcap hw timer to: 1573801716.970916000s
cmem[pcie_chipset0] type=3 va=0x7f22e3048000[0xffff9aa0cfc00000] pa=0x8fc00000 size=0x400000 status=0x3
    b=0x7f22e3048000 l=0x400000 r=0x0 w=0x0 a=0x400000 o=0x0
pcap process thread start
->help
help            show all cmd list # 显示命令列表
showbuf         show internal packet buf state # 内部包的循环缓冲区状态
showstat        show statistics of capture interface # 捕获状态统计信息
version         show version # 显示版本信息
exit            exit application # 退出应用
[result]: command executed successfully
->version
APP: finace-1.0.1-20191115.1424
[result]: command executed successfully
->showbuf
circbufDump b=0x7fb2e4ca7000 l=0x400000 r=0x0 w=0x0 a=0x400000 o=0x0 # 内部包的循环缓冲区为空
[result]: command executed successfully
->showbuf
circbufDump b=0x7fb2e4ca7000 l=0x400000 r=0x0 w=0x23a a=0x3ffdc6 o=0x23a # 此刻表明收到了0x23a字节的包
[result]: command executed successfully
->showstat
file count: 0 # 当设置定时新建文件或按文件大小自动新建文件才有数值，否则恒为0
current file name:  # 满0x1000字节才会执行写文件操作，所以此时无文件名
current file size: 0 # 当前文件大小
total received data size: 0 # 当前接收数据大小
[result]: command executed successfully
->showstat
file count: 0
current file name: ./a.pcap # 当前正在写入的文件名
current file size: 4120
total received data size: 4096 # 数据接收大小比文件大小少64字节，这是pcap的文件头header导致的
[result]: command executed successfully
->exit 
pcap process thread stop
```
# 运行收发包测试程序
```shell
[root@localhost DemoRecvSendServer]# ls
cfg-recv.xml  cfg-send.xml  dat_09  dat_09.zip  DemoRecvSendServer  DemoRecvSendServer.zip  SSEL2  SZL2
[root@localhost DemoRecvSendServer]# cat cfg-recv.xml 
<?xml version="1.0" encoding="utf-8"?>

<RecvSend use="SZ" desc="SH or SZ">
    <ShangHai>
        <Recv  use="1" ip="127.0.0.1" port="19149" rebuildport="19150" vdeid="shvdeid" compid="shselfid" version="1.00" heartbtint="5"/>
    </ShangHai>

    <ShenZhen>
                <Filter type="*" desc="need to decode or send"/>
                <Recv  use="1" ip="127.0.0.1" port="16662" rebuildport="16664" vdeid="W000633Q0005" compid="realtime" version="1.00" heartbtint="5" desc="yanshengping"/> 
    </ShenZhen>
</RecvSend>
[root@localhost DemoRecvSendServer]# cat cfg-send.xml 
<?xml version="1.0" encoding="utf-8"?>

<RecvSend use="SZ" desc="SH or SZ">
    <ShangHai>
        <Send  use="1" port="19149" interval="100" file="/root/DemoRecvSendServer/SSEL2/dat_09;"/>
    </ShangHai>

    <ShenZhen>
                <Filter type="*" desc="need to decode or send"/>
        <Send  use="1" port="16662" interval="20000" file="/root/DemoRecvSendServer/SZL2/dat_09;"/>
    </ShenZhen>
</RecvSend>
```

