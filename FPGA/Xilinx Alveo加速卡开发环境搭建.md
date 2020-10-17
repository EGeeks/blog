# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [网页版帮助](https://www.xilinx.com/html_docs/accelerator_cards/alveo_doc/index.html)
> [官网Alveo U200 Data Center Accelerator Card首页](https://www.xilinx.com/products/boards-and-kits/alveo/u200.html#documentation)
> [Vitis Unified Software Development Platform 2020.1 Documentation](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/kme1569523964461.html)
> [Xilinx Runtime (XRT) Documentation](https://xilinx.github.io/XRT/)
> [在 Alveo 上的 Vitis 加速开发流程（中文）](https://china.xilinx.com/video/software/vitis-acceleration-development-flow-on-alveo.html)
> [Using Xilinx Alveo Cards to Accelerate Dynamic Workloads](https://china.xilinx.com/training/customer-training/using-xilinx-alveo-accelerate-workloads.html)
> [Get Moving with Alveo: Acceleration Basics](https://developer.xilinx.com/en/articles/acceleration-basics.html)
> [Alveo U280 数据中心加速器卡用户文档](https://china.xilinx.com/html_docs/accelerator_cards/alveo_doc_280/index.html)
> [Alveo 附件](https://china.xilinx.com/products/boards-and-kits/alveo/accessories.html)


# 介绍
QSFP28光模块的外形尺寸和QSFP光模块相同，但是它们的传输方式不一样，前者为4x10G或4x25G，后者是4x10G，所以100G QSFP28端口可以使用QSFP+光模块或QSFP28光模块。Alveo U200需要看ug1289和ug1301.

# XRT
下载编译安装,
```bash
# get XRT
$ git clone https://github.com/Xilinx/XRT.git
# install dependencies
$ sudo src/runtime_src/tools/scripts/xrtdeps.sh # warning, but just warning
The directory '/home/qe/.cache/pip/http' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
The directory '/home/qe/.cache/pip' or its parent directory is not owned by the current user and caching wheels has been disabled. check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
# build runtime
$ cd XRT/build
$ ./build.sh
# build deb for ubuntu
$ cd build/Release # release, auto built
$ make package
$ cd ../Debug # debug
$ make package
# install
$ sudo apt install --reinstall ./xrt_202020.2.8.0_18.04-amd64-xrt.deb
$ apt list --installed | grep xrt
xrt/now 2.8.0 amd64 [已安装，本地]
# build docs, build/Release/runtime_src/doc/html/index.html
cd build
./build.sh docs
xdg-open Release/runtime_src/doc/html/index.html # To browse the generated local documentation with a web browser
```
运行，添加`-dbg`开启调试，
```bash
<path>/XRT/build/run.sh ./host.exe kernel.xclbin
```

# Deployment Target Platform
下载安装，
```bash
$ sudo apt install ./xilinx-cmc-u50-1.0.17-2784148_18.04.deb 
$ sudo apt install ./xilinx-sc-fw-u50-5.0.27-2.e289be9_18.04.deb
$ sudo apt install ./xilinx-u50-gen3x16-xdma-201920.3-2784799_18.04.deb
$ sudo apt install ./xilinx-u50-gen3x16-xdma-dev-201920.3-2784799_18.04.deb
```
你以为这么顺利，然而我遇到来这个问题[Problems installing U50 deployment](https://forums.xilinx.com/t5/Alveo-Accelerator-Cards/Problems-installing-U50-deployment/m-p/1147095#M2081)，帖子里说重启就好了，我都没安装成功，我重启能有什么用，
```bash
This is create_xsabin.sh running from /usr/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b on Fri 04 Sep 2020 04:03:45 PM EDT
Run for install of partition xilinx-u50-gen3x16-xdma-blp version 1 release 2784799
Metadata file is partition_metadata.json
User install path is /opt/xilinx/firmware/u50/gen3x16-xdma/blp
Required ert firmware install directory not found at /opt/xilinx/xrt/share/fw. Unable to build xsabin files
```
然而老哥我手痒，打算卸载重装试一试，结果卸载失败，可能软件有残留没删干净，重新安装无法安装，我就按照提示手动`rm -rf`相关文件，再干这条命令，
![2020-10-10 22-15-42](https://img-blog.csdnimg.cn/20201010221600819.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)如此卸载删文件重装，搞了2个小时，夜已深，我能怎么办，我要去骂Xilinx垃圾？可能有点过了，我给CSDN提议增加一个功能：可删除不小心打开的付费订阅专栏，就是上面的《不要订阅这个专栏》，半年多了吧，改过没有，没有，Xilinx搞的还是挺好的，我骂它干嘛，对不！没用的。
第二天，老哥我拖着疲惫的身躯，又来干了，我放弃了干这条命令，打算曲线救国，先研究一下deb包是个什么东西，10分钟之后，我搞明白了，恩，不好意思，让各位久等了，我花了10min才把Xilinx的衣服给扒掉，我承认是老哥最近缺少练习，手生，我已经联系了楼下会所的老师，等干完这个命令，去找老师补习一下。少说废话，下面来欣赏一下Xilinx这新鲜的肉体，2019年的deb包，距今不过只1年而已，老哥我手动干掉这些文件，
```bash
/lib
/lib/firmware
/lib/firmware/xilinx
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/create_xsabin.log
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/create_xsabin.sh
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/license
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/license/COPYRIGHT
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/license/LICENSE
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/partition.mcs
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/partition.xsabin
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/partition_metadata.json
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/test
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/test/bandwidth.xclbin
/lib/firmware/xilinx/f465b0a3ae8c64f619bc150384ace69b/test/verify.xclbin
/opt
/opt/xilinx
/opt/xilinx/firmware/u50
/opt/xilinx/firmware/u50/gen3x16-xdma
/opt/xilinx/firmware/u50/gen3x16-xdma/blp
/opt/xilinx/firmware/u50/gen3x16-xdma/blp/firmware
/opt/xilinx/firmware/u50/gen3x16-xdma/blp/scripts
/usr
/usr/share
/usr/share/doc
/usr/share/doc/xilinx-u50-gen3x16-xdma-blp
/usr/share/doc/xilinx-u50-gen3x16-xdma-blp/changelog.Debian.gz
```
安装成功的状态，
```bash
$ apt list --installed | grep xrt
xrt/now 2.7.766 amd64 [已安装，本地]
$ apt list --installed | grep xilinx
xilinx-cmc-u50/now 1.0.17-2784148 all [已安装，本地]
xilinx-sc-fw-u50/now 5.0.27-2.e289be9 all [已安装，本地]
xilinx-u50-gen3x16-xdma-blp/now 1-2784799 all [已安装，本地]
xilinx-u50-gen3x16-xdma-dev/now 201920.3-2784799 all [已安装，本地]
```
安装mesa，有警告，
```bash
$ sudo add-apt-repository ppa:xorg-edgers/ppa 

 == Xorg packages fresh from git ==

Currently supported releases are Xenial/16.04 and Yakkety/16.10

* WARNING: Do not use this PPA with enabled HWE stack.

Be sure to revert this PPA before doing a release upgrade or the upgrade will not succeed. To revert to official packages, install the ppa-purge package and run "sudo ppa-purge xorg-edgers".

== Important notice ==

This PPA is currently meant to be used as a whole. Please do _not_ individually install packages from it, add it to your sources and let your package manager pull in every update. The packages here build against each other and compile different features based on whats available at build time. Do not assume that because it lets you install a DDX with just the driver and libdrm update that it will work.  These packages are made with scripts that use the the current packages as the base, so some dependencies can be wrong and your package manager will not resolve that for you.  If you want to individually install something from here, grab the source and rebuild it in your current environment instead.

** Please do not publish instructions for how to install from this archive without linking to this page! Anyone using packages from this archive is expected to read this page first and it is recommended to check back occasionally for notice on problems that may arise. **

** Please use ppa-purge to remove this PPA. It is *particularly* recommended to do this before upgrading to a new ubuntu release! **
 更多信息： https://launchpad.net/~xorg-edgers/+archive/ubuntu/ppa
按 [ENTER] 继续或 Ctrl-c 取消安装。

命中:1 http://mirrors.aliyun.com/ubuntu bionic InRelease
命中:2 http://mirrors.aliyun.com/ubuntu bionic-updates InRelease               
命中:3 http://mirrors.aliyun.com/ubuntu bionic-backports InRelease             
命中:4 http://mirrors.aliyun.com/ubuntu bionic-security InRelease                                 
命中:5 http://ppa.launchpad.net/wireshark-dev/stable/ubuntu bionic InRelease                      
获取:6 http://ppa.launchpad.net/xorg-edgers/ppa/ubuntu bionic InRelease [20.7 kB]
获取:7 http://ppa.launchpad.net/xorg-edgers/ppa/ubuntu bionic/main i386 Packages [6,116 B]
获取:8 http://ppa.launchpad.net/xorg-edgers/ppa/ubuntu bionic/main amd64 Packages [6,112 B]
获取:9 http://ppa.launchpad.net/xorg-edgers/ppa/ubuntu bionic/main Translation-en [4,376 B]
已下载 37.4 kB，耗时 5秒 (6,993 B/s)        
正在读取软件包列表... 完成
$ sudo apt-get update
$ sudo apt-get install libgl1-mesa-glx
$ sudo apt-get install libgl1-mesa-dri
$ sudo apt-get install libgl1-mesa-dev
$ sudo add-apt-repository --remove ppa:xorg-edgers/ppa
```

# Vitis
首先安装Vitis，参考我的博客[Xilinx Vitis Petalinux安装和使用](https://blog.csdn.net/Zhu_Zhu_2009/article/details/106443896)，注意部署目标平台与Vitis之间的兼容性，
![2020-10-09 22-56-32](https://img-blog.csdnimg.cn/20201009225830742.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
# 测试
插入Alveo加速卡，可以看到卡和驱动都装上了，
```bash
$ lspci -tv
-[0000:00]-+-00.0  Intel Corporation 8th Gen Core Processor Host Bridge/DRAM Registers
           +-01.0-[01]--+-00.0  Xilinx Corporation Device 5020
           |            \-00.1  Xilinx Corporation Device 5021
           +-08.0  Intel Corporation Xeon E3-1200 v5/v6 / E3-1500 v5 / 6th/7th Gen Core Processor Gaussian Mixture Model
           +-14.0  Intel Corporation 200 Series/Z370 Chipset Family USB 3.0 xHCI Controller
           +-16.0  Intel Corporation 200 Series PCH CSME HECI #1
           +-17.0  Intel Corporation 200 Series PCH SATA controller [AHCI mode]
           +-1c.0-[02-03]----00.0-[03]--
           +-1c.5-[04]--+-00.0  NVIDIA Corporation GP107GL [Quadro P1000]
           |            \-00.1  NVIDIA Corporation GP107GL High Definition Audio Controller
           +-1c.6-[05]--
           +-1c.7-[06]----00.0  Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller
           +-1d.0-[07]--
           +-1f.0  Intel Corporation Device a2ca
           +-1f.2  Intel Corporation 200 Series/Z370 Chipset Family Power Management Controller
           +-1f.3  Intel Corporation 200 Series PCH HD Audio
           \-1f.4  Intel Corporation 200 Series/Z370 Chipset Family SMBus Controller
$ sudo lspci -vv -s 01:00.0
01:00.0 Processing accelerators: Xilinx Corporation Device 5020
	Subsystem: Xilinx Corporation Device 000e
	Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
	Latency: 0, Cache Line Size: 64 bytes
	Region 0: Memory at d2000000 (64-bit, prefetchable) [size=32M]
	Region 2: Memory at d4020000 (64-bit, prefetchable) [size=128K]
	Capabilities: [40] Power Management version 3
		Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0-,D1-,D2-,D3hot-,D3cold-)
		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
	Capabilities: [60] MSI-X: Enable+ Count=32 Masked-
		Vector table: BAR=2 offset=00010000
		PBA: BAR=2 offset=00014000
	Capabilities: [70] Express (v2) Endpoint, MSI 00
		DevCap:	MaxPayload 1024 bytes, PhantFunc 0, Latency L0s <64ns, L1 <1us
			ExtTag+ AttnBtn- AttnInd- PwrInd- RBE+ FLReset- SlotPowerLimit 75.000W
		DevCtl:	Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
			RlxdOrd+ ExtTag+ PhantFunc- AuxPwr- NoSnoop+
			MaxPayload 256 bytes, MaxReadReq 512 bytes
		DevSta:	CorrErr+ UncorrErr- FatalErr- UnsuppReq+ AuxPwr- TransPend-
		LnkCap:	Port #0, Speed 8GT/s, Width x16, ASPM not supported, Exit Latency L0s unlimited, L1 unlimited
			ClockPM- Surprise- LLActRep- BwNot- ASPMOptComp+
		LnkCtl:	ASPM Disabled; RCB 64 bytes Disabled- CommClk+
			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
		LnkSta:	Speed 8GT/s, Width x16, TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
		DevCap2: Completion Timeout: Range BC, TimeoutDis+, LTR-, OBFF Not Supported
		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis-, LTR-, OBFF Disabled
		LnkCtl2: Target Link Speed: 8GT/s, EnterCompliance- SpeedDis-
			 Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
			 Compliance De-emphasis: -6dB
		LnkSta2: Current De-emphasis Level: -3.5dB, EqualizationComplete+, EqualizationPhase1+
			 EqualizationPhase2+, EqualizationPhase3+, LinkEqualizationRequest-
	Capabilities: [100 v1] Advanced Error Reporting
		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
		AERCap:	First Error Pointer: 00, GenCap- CGenEn- ChkCap- ChkEn-
	Capabilities: [1c0 v1] #19
	Capabilities: [e00 v1] Access Control Services
		ACSCap:	SrcValid- TransBlk- ReqRedir- CmpltRedir- UpstreamFwd- EgressCtrl+ DirectTrans-
		ACSCtl:	SrcValid- TransBlk- ReqRedir- CmpltRedir- UpstreamFwd- EgressCtrl- DirectTrans-
	Capabilities: [e10 v1] #15
	Capabilities: [e80 v1] Vendor Specific Information: ID=0020 Rev=0 Len=010 <?>
	Kernel driver in use: xclmgmt
	Kernel modules: xclmgmt

qe@qe-pc:~$ sudo lspci -vv -s 01:00.1
01:00.1 Processing accelerators: Xilinx Corporation Device 5021
	Subsystem: Xilinx Corporation Device 000e
	Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx-
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
	Latency: 0, Cache Line Size: 64 bytes
	Interrupt: pin A routed to IRQ 16
	Region 0: Memory at d0000000 (64-bit, prefetchable) [size=32M]
	Region 2: Memory at d4000000 (64-bit, prefetchable) [size=128K]
	Region 4: Memory at c0000000 (64-bit, prefetchable) [size=256M]
	Capabilities: [40] Power Management version 3
		Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0-,D1-,D2-,D3hot-,D3cold-)
		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
	Capabilities: [60] MSI-X: Enable+ Count=32 Masked-
		Vector table: BAR=2 offset=00010000
		PBA: BAR=2 offset=00014000
	Capabilities: [70] Express (v2) Endpoint, MSI 00
		DevCap:	MaxPayload 1024 bytes, PhantFunc 0, Latency L0s <64ns, L1 <1us
			ExtTag+ AttnBtn- AttnInd- PwrInd- RBE+ FLReset- SlotPowerLimit 75.000W
		DevCtl:	Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
			RlxdOrd+ ExtTag+ PhantFunc- AuxPwr- NoSnoop+
			MaxPayload 256 bytes, MaxReadReq 512 bytes
		DevSta:	CorrErr+ UncorrErr- FatalErr- UnsuppReq+ AuxPwr- TransPend-
		LnkCap:	Port #0, Speed 8GT/s, Width x16, ASPM not supported, Exit Latency L0s unlimited, L1 unlimited
			ClockPM- Surprise- LLActRep- BwNot- ASPMOptComp+
		LnkCtl:	ASPM Disabled; RCB 64 bytes Disabled- CommClk+
			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
		LnkSta:	Speed 8GT/s, Width x16, TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
		DevCap2: Completion Timeout: Range BC, TimeoutDis+, LTR-, OBFF Not Supported
		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis-, LTR-, OBFF Disabled
		LnkSta2: Current De-emphasis Level: -3.5dB, EqualizationComplete-, EqualizationPhase1-
			 EqualizationPhase2-, EqualizationPhase3-, LinkEqualizationRequest-
	Capabilities: [100 v1] Advanced Error Reporting
		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
		AERCap:	First Error Pointer: 00, GenCap- CGenEn- ChkCap- ChkEn-
	Capabilities: [e00 v1] Access Control Services
		ACSCap:	SrcValid- TransBlk- ReqRedir- CmpltRedir- UpstreamFwd- EgressCtrl+ DirectTrans-
		ACSCtl:	SrcValid- TransBlk- ReqRedir- CmpltRedir- UpstreamFwd- EgressCtrl- DirectTrans-
	Capabilities: [e10 v1] #15
	Capabilities: [e80 v1] Vendor Specific Information: ID=0020 Rev=0 Len=010 <?>
	Kernel driver in use: xocl
	Kernel modules: xocl
# 卡固件是最新，更新后冷重启方可使用
$ sudo /opt/xilinx/xrt/bin/xbmgmt flash --update --shell xilinx_u50_gen3x16_xdma_201920_3
	 Status: shell is up-to-date

Card(s) up-to-date and do not need to be flashed.
$ sudo lspci -vd 10ee:
01:00.0 Processing accelerators: Xilinx Corporation Device 5020
	Subsystem: Xilinx Corporation Device 000e
	Flags: bus master, fast devsel, latency 0
	Memory at d2000000 (64-bit, prefetchable) [size=32M]
	Memory at d4020000 (64-bit, prefetchable) [size=128K]
	Capabilities: [40] Power Management version 3
	Capabilities: [60] MSI-X: Enable+ Count=32 Masked-
	Capabilities: [70] Express Endpoint, MSI 00
	Capabilities: [100] Advanced Error Reporting
	Capabilities: [1c0] #19
	Capabilities: [e00] Access Control Services
	Capabilities: [e10] #15
	Capabilities: [e80] Vendor Specific Information: ID=0020 Rev=0 Len=010 <?>
	Kernel driver in use: xclmgmt
	Kernel modules: xclmgmt

01:00.1 Processing accelerators: Xilinx Corporation Device 5021
	Subsystem: Xilinx Corporation Device 000e
	Flags: bus master, fast devsel, latency 0, IRQ 16
	Memory at d0000000 (64-bit, prefetchable) [size=32M]
	Memory at d4000000 (64-bit, prefetchable) [size=128K]
	Memory at c0000000 (64-bit, prefetchable) [size=256M]
	Capabilities: [40] Power Management version 3
	Capabilities: [60] MSI-X: Enable+ Count=32 Masked-
	Capabilities: [70] Express Endpoint, MSI 00
	Capabilities: [100] Advanced Error Reporting
	Capabilities: [e00] Access Control Services
	Capabilities: [e10] #15
	Capabilities: [e80] Vendor Specific Information: ID=0020 Rev=0 Len=010 <?>
	Kernel driver in use: xocl
	Kernel modules: xocl	
# 确保版本号一致
$ sudo /opt/xilinx/xrt/bin/xbmgmt flash --scan
[sudo] qe 的密码： 
Card [0000:01:00.0]
    Card type:		u50
    Flash type:		SPI
    Flashable partition running on FPGA:
        xilinx_u50_gen3x16_xdma_201920_3,[ID=0xf465b0a3ae8c64f6],[SC=5.0.27]
    Flashable partitions installed in system:	
        xilinx_u50_gen3x16_xdma_201920_3,[ID=0xf465b0a3ae8c64f6],[SC=5.0.27]
$ sudo /opt/xilinx/xrt/bin/xbutil validate
INFO: Found 1 cards

INFO: Validating card[0]: xilinx_u50_gen3x16_xdma_201920_3
INFO: == Starting Kernel version check: 
INFO: == Kernel version check PASSED
INFO: == Starting AUX power connector check: 
AUX power connector not available. Skipping validation
INFO: == AUX power connector check SKIPPED
INFO: == Starting PCIE link check: 
INFO: == PCIE link check PASSED
INFO: == Starting SC firmware version check: 
INFO: == SC firmware version check PASSED
INFO: == Starting verify kernel test: 
INFO: == verify kernel test PASSED
INFO: == Starting DMA test: 
Host -> PCIe -> FPGA write bandwidth = 11427.041021 MB/s
Host <- PCIe <- FPGA read bandwidth = 11863.935490 MB/s
INFO: == DMA test PASSED
INFO: == Starting device memory bandwidth test: 
...........
Maximum throughput: 44035 MB/s
INFO: == device memory bandwidth test PASSED
INFO: == Starting PCIE peer-to-peer test: 
P2P BAR is not enabled. Skipping validation
INFO: == PCIE peer-to-peer test SKIPPED
INFO: == Starting memory-to-memory DMA test: 
M2M is not available. Skipping validation
INFO: == memory-to-memory DMA test SKIPPED
INFO: == Starting host memory bandwidth test: 
Host_mem is not available. Skipping validation
INFO: == host memory bandwidth test SKIPPED
INFO: Card[0] validated successfully.

INFO: All cards validated successfully.
```
