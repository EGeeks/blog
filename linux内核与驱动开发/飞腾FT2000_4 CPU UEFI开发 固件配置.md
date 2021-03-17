# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [UEFI启动分析 UEFI设置启动项](https://www.jianshu.com/p/9166e8c00ca2)

# CPU结构指标
- 集成 4 个 FTC663 核； 
- L2 Cache：每个 Cluster 内有 2MB，共 4MB； 
- L3 Cache：分为 8 个 Bank，共 4MB； 
- 集成 2 个 DDR4-3200 通道，支持对 DDR 存储数据进行实时加密； 
- 集成 34 Lanes PCIe 3.0 接口：2 个 X16（每个可拆分成 2 个 X8），2个 X1； 
- 集成 2 个千兆 Ethernet 接口(RGMII)，支持 10/100/1000Mbps 自适应； 
- 集成 1 个 SD 卡控制器，兼容 SD 2.0 规范； 
- 集成 1 个 HDA (HD-Audio)，支持音频输出，可同时支持最多 4 个Codec； 
- 集成 SM2/SM3/SM4/SM9 密码加速引擎； 
- 集成 4 个 UART，1 个 LPC Master，32 个 GPIO，4 个 I2C，1 个 QSPI，2 个通用 SPI，2 个 WDT，1 个 RTC，16 个外部中断（和 GPIO 共用IO）； 
- 集成温度传感器； 
- 集成 128KB On Chip Memory； 
- 集成 ROM 作为片内可信启动根。

# 参数配置
UEFI里面自带了一个参数文件，根据文档，看下默认的参数配置情况，

- CPU主频2600MHz
- DDR频率0MHz，**文档错了？？？**
- 使能4个核，CPU架构是2个Cluster，每个Cluster 2个Core，这个格式可推算出每个Cluster最多4个Core
- PEU1为x8模式，PCIE0初始化为x16模式，
- PCIE0控制器0/1/2，都初始化为RC模式，**强制为GEN3**，均衡值为0x48484848（根据结构指标，共6个PCIE）
- PCIE1控制器0/1/2，都初始化为RC模式，**强制为GEN3**，均衡值为0x48484848（根据结构指标，共6个PCIE）
- S3标志由外部电源管理模块（ CPLD ）通过GPIO0-PORTA1提供，详见硬件设计参考文档
- 使能双通道内存，使能ECC（具体是不是ECC还得看内存条是否支持），DDR时序信息从内存条SPD读取

使用飞腾打包工具的时候，dump生成的文件，多了很多0x0d 0x0a回车换行符，不知道什么意思，略过。

# 引脚复用
