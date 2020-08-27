# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [GMII,RGMII,SGMII,TBI,RTBI接口信号及时序介绍](https://blog.csdn.net/davion_zhang/article/details/52789741)
> [GMII、SGMII和SerDes的区别和联系](https://blog.csdn.net/LIYUANNIAN/article/details/84574981)
> [求问怎么实现1000base-x光口](https://exp.newsmth.net/topic/article/dbdb342da777f16cc11032aefbcb63dc)
> [MII、RMII、GMII接口的详细介绍](https://blog.csdn.net/u013273161/article/details/88417300)

# PHY结构
以88e1111为例，Symbol encoder/decoder即PCS，
![200](https://img-blog.csdnimg.cn/20190827152004390.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# MAC的结构
以zynqmp为例，
![201](https://img-blog.csdnimg.cn/20190827152112514.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# GMII/RGMII
GMII/RGMII不经过MAC的PCS，所以需要PHY来实现PCS。 GMII采用8位接口数据，工作时钟125MHz，因此传输速率可达1000Mbps。同时兼容MII所规定的10/100 Mbps工作方式。GMII接口数据结构符合IEEE以太网标准。该接口定义见`IEEE 802.3-2000`。在千兆速率下，向PHY提供GTXCLK信号，TXD、TXEN、TXER信号与此时钟信号同步。否则，在10/100M速率下，PHY提供 TXCLK时钟信号，其它信号与此信号同步。其工作频率为25MHz（100M网络）或2.5MHz（10M网络）。管理配置接口控制PHY的特性。该接口有32个寄存器地址，每个地址16位。其中前16个已经在`IEEE 802.3,2000-22.2.4 Management Functions`中规定了用途，其余的则由各器件自己指定。
发送接口
- GTXCLK吉比特TX..信号的时钟信号（125MHz）
- TXCLK 10/100M信号时钟
- TXD[7:0]被发送数据
- TXEN发送器使能信号
- TXER发送器错误（用于破坏一个数据包）

 接收接口
- RXCLK接收时钟信号（从收到的数据中提取，因此与GTXCLK无关联）
- RXD[7:0]接收数据
- RXDV接收数据有效指示
- RXER接收数据出错指示
- COL冲突检测（仅用于半双工状态）
- CRS 载波监听

管理接口
- MDC配置接口时钟
- MDIO配置接口I/O

# SGMII和Serdes的区别
SGMII和1000BASE-X的区别在于自协商的包不一样，
SGMII和1000Base-X的电平标准一致，数据封装形式一致（SGMII引用的1000Base-X），数据里，链路状态的信息不一样，SGMII非对称，主要是PHY向MAC反馈PHY的链路状态（电口PHY），类似于RGMII的In Band Link Status，1000Base-X对称。二者不一致，1000Base-X的模块插SGMII上一般不能直接用。交换机上的解决方案是对联端口都配成强制速率，全双工，强制忽略状态信息，可用。SFP的SGMII-光口模块存在，但很少，要找，多数是到RJ45的电口模块。
# TBI
TBI即Ten Bit Interface的意思，接口数据位宽由GMII接口的8位增加到10位，在将数据发给PHY芯片之前进行了8B/10B变换(8B/10B变换本是在PHY芯片中完成的)
# 1000BASE-X
短波长光传输1000Base-SX、长波长光传输1000Base-LX，多模光纤有可以分为长波激光(称为1000BaseLX)、的短波激光(称为1000BaseSX)
1000Base-LX，是定义在 IEEE 802.3z 中的针对光纤布线吉比特以太网的一个物理层规范。LX 代表长波长，与 1000Base-SX 相反，1000Base-LX 使用长波长激光（1310nm）越过多模式和单模式光纤，1000Base-SX 使用短波长激光越过多模式光纤。多模式光纤的最大距离是 850m。
1000BASE-SX也对应于802.3z标准，只能使用多模光纤。1000BASE-SX所使用的光纤有：62.5μm多模光纤、50μm多模光纤。其中使用62.5μm多模光纤的最大传输距离为275m，使用50μm多模光纤的最大传输距离为550米。1000BASE-SX采用8B/10B编码方式。
