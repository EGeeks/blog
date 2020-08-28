# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> RapidIO规范《RapidIO_Rev_2.2_Specification》
> 书籍《RapidIO The Embedded System Interconnect》
> Xilinx手册pg007《Serial RapidIO Gen2 Endpoint v4.1 LogiCORE IP Product Guide》
> IDT CPS1848手册《CPS-1848™ User Manual》
> 博客[Zynq-Linux移植学习笔记之14-RapidIO驱动开发](https://blog.csdn.net/zhaoxinfan/article/details/72774363)
> 博客[基于Xilinx的RapidIO核配置和AXI-SRIO核设计](https://blog.csdn.net/web_star/article/details/79709540)
> 系列博客[SRIO学习](https://blog.csdn.net/haiyonghao/article/category/6403188)
> 系列博客[RapidIO](https://blog.csdn.net/shanghaiqianlun/article/category/6066627)（还介绍了TSI721）
> [Zynq—Linux移植学习笔记（十三）](http://www.openhw.org/module/forum/thread-657701-1-1.html)
> [Xilinx Srio详解&IP核使用](https://blog.csdn.net/zhipao6108/article/details/81570667)
> [SRIO系统初始化过程和路由配置](https://blog.csdn.net/Zhu_Zhu_2009/article/details/91524160)

# Xilinx Srio IP
Xilinx Srio IP的寄存器定义如下：
```c
typedef struct/*total 16M*/
{
	/*Capability Register Space 0x00~0x3C*/
	unsigned int deviceIdentityCar;
	unsigned int deviceInfomationCar;
	unsigned int assemblyIdentityCar;
	unsigned int assemblyInfomationCar;
	unsigned int processingElementFeaturesCar;
	unsigned int switchPortInformationCar;
	unsigned int sourceOperationsCar;
	unsigned int destinationOperationsCar;
	unsigned int reservedCar[8];
	/*Command and Status Register Space 0x40~0xFC*/
	unsigned int reservedCsr1[3];
	unsigned int processingElementLogicalLayerCsr;
	unsigned int reservedCsr2[2];
	unsigned int localConfigurationSpaceBaseAddress0Csr;
	unsigned int localConfigurationSpaceBaseAddress1Csr;
	unsigned int baseDeviceIdCsr;
	unsigned int reservedCsr3;
	unsigned int hostBaseDeviceIdLockCsr;
	unsigned int componentTagCsr;
	unsigned int reservedCsr4[36];
	/*Extended Features Space default 0x100 ~ 0xFFFC*/
	/*    LP-Serial Extended Features block 0x100 ~ 0x1FC*/
	unsigned int lpSerialRegisterBlockHeader;
	unsigned int reservedEfs1[7];
	unsigned int portLinkTimeoutCsr;
	unsigned int portResponseTimeoutCsr;
	unsigned int reservedEfs2[5];
	unsigned int portGeneralControlCsr;
	unsigned int portNMaintenanceRequestCsr;
	unsigned int portNMaintenanceResponseCsr;
	unsigned int portNLocalAckIdCsr;
	unsigned int reservedEfs3[2];
	unsigned int portNControl2Csr;
	unsigned int portNErrorAndStatusCsr;
	unsigned int portNControlCsr;
	unsigned int reservedEfs4[40];
	/*    reserved 0x200~0x3FC*/
	unsigned int reservedEfs5[192];
	/*    LP-Serial Lane Extended Features block 0x400~0x4FC*/
	unsigned int lpSerialLaneRegisterBlockHeader;
	unsigned int reservedEfs6[3];
	unsigned int lane0Status0Csr;
	unsigned int lane0Status1Csr;
	unsigned int reservedEfs7[6];
	unsigned int lane1Status0Csr;
	unsigned int lane1Status1Csr;
	unsigned int reservedEfs8[6];
	unsigned int lane2Status0Csr;
	unsigned int lane2Status1Csr;
	unsigned int reservedEfs9[6];
	unsigned int lane3Status0Csr;
	unsigned int lane3Status1Csr;
	unsigned int reservedEfs10[34];
	/*reserved 0x500~0xFFFC*/
	unsigned int reserved1[16064];
	/*Implementation-defined Space 0x10000~0xFFFFFC*/
	unsigned int watermarksCsr;
	unsigned int bufferControlCsr;
	unsigned int reservedIds[62];
	unsigned int maintenanceRequestInfomationRegister;
	/*reserved */
	unsigned int reserved2[0];
} __attribute__((packed, aligned(4))) SRIOREG, *PSRIOREG;
```

列举用到的4个寄存器，详细参考pg007。

 1. Port General Control CSR，使能Master Enable。
![这里写图片描述](https://img-blog.csdn.net/20180609211016478?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 2. Base Device ID CSR，设置器件ID，host可设为1，默认值在vivado里设置。
![这里写图片描述](https://img-blog.csdn.net/20180609212345289?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 3. Host Base Device ID Lock CSR，器件锁定寄存器，复位之后，这个寄存器只能被写一次（之后被锁定），配置之后如果写入值和寄存器值相等，则寄存器值被复位为0xFFFF，向该寄存器写入0xFFFF不会锁定寄存器。
![这里写图片描述](https://img-blog.csdn.net/20180609212355794?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 4. Maintenance Request Information Register，维护包配置寄存器，地址在0x10100，低16位用于配置目的ID，当用IP发维护包之前，需要配置这个寄存器。
![这里写图片描述](https://img-blog.csdn.net/2018061010434629?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 5. Processing Elements Features CAR，表示这个设备提供的功能，可以是Bridge，Memory，Processor，Switch 4种，SRIO IP支持前3种（endpoint），支持16位地址模式，可在vivado中通过GUI设置。
![这里写图片描述](https://img-blog.csdn.net/2018061222304694?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
