# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> USB Type-C Spec R2.0
> [USB Type C规范详解](https://blog.csdn.net/weixin_42005993/article/details/100046714)
> [USB-C(USB Type-C)规范的简单介绍和分析](https://blog.csdn.net/u010538116/article/details/80420632)
> [意法半导体STM32G0生态系统扩展功能支持通用微控制器将USB-C用作标准接口](https://www.st.com/content/st_com/zh/about/media-center/press-item.html/n4139.html)
> AN4775

# 特点
- 外形纤薄，可翻转拔插方向：正反随便插
- USB Power Delivery提供100W电力
- 支持更多协议Display Port，HDMI，VGA，Ethernet
- USB3.1 Gen2 10Gbps

# 引脚功能
USB2.0规范的电缆长度小于4米，USB3.2 Gen1的长度小于2米，USB3.2Gen2的电缆长度小于1米。SDP屏蔽差分线的阻抗控制在90Ω±5Ω，单端同轴线控制在45Ω±3Ω。阻抗应该用200 ps(10%-90%)的上升时间来评估。电源VBUS和GND，电源的压降要小于500mV，Gnd上面的压降要小于250mV
![130](https://img-blog.csdnimg.cn/20200219231457243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
- 插座多出CC1和CC2管脚，插头只多出CC管脚来建立信号定位，另一个多出的管脚用作VCONN，为电子元器件供电，
- 另外USB2.0 D+/D-线只会实现一组
- VBUS电源，支持5V到20V，通过CC来进行协商
- SBU管脚，audio模式，display模式通过这个脚传送，用于USB拓展功能。


CC(Configuration Channel)
配置通道，这是USB Type-C里新增的关键通道。它的作用有检测正反插，检测USB连接识别可以提供多大的电压和电流，USB设备间数据与VBUS的连接建立与管理等。插座多出CC1和CC2管脚，插头只多出一个CC管脚。
- 检测USB设备是否接入；
- 检测USB插入方向，并以此建立USB 数据通道的路由；
- 插入后帮助建立USB设备角色（谁为HOST，谁为Device）；
- 发现并配置VUBS，配置USB PD供电模式；
- 配置Vconn；
- 发现和配置可选的备用和辅助模式；

VCONN（只有在插头上才会有该信号），当线缆里有芯片的时候，用来给线缆里的芯片供电（3.3V或5V）

# 配置处理
DFP(Downstream Facing Port)：
下行端口，可以理解为Host，DFP提供VBUS，可以提供数据。在协议规范中DFP特指数据的下行传输，笼统意义上指的是数据下行和对外提供电源的设备。典型的DFP设备是电源适配器。
UFP(Upstream Facing Port)：
上行端口，可以理解为Device，UFP从VBUS中取电，并可提供数据。典型设备是U盘，移动硬盘。
DRP(Dual Role Port)：
双角色端口，DRP既可以做DFP(Host)，也可以做UFP(Device)，也可以在DFP与UFP间动态切换。典型的DRP设备是笔记本电脑。

Source端CC脚有一个上拉电阻Rp，Sink端有一个下拉电阻Rd，通过的阻值来控制供电能力，需要USB PD电力传输时，使用Bi-phase Mark Coded（BMC）编码协议，通过CC管脚进行通信。在连接时，Source检测到CC管脚都为高电平，Sink端检测到CC管脚都未低电平。连接后，形成分压，电平为中间值。
![131](https://img-blog.csdnimg.cn/20200221215021336.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
Rp的阻值与对应的供电能力，
![132](https://img-blog.csdnimg.cn/20200221215226953.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
Rd的阻值是5.1K，精度为10%，否则不能发现电源的供电能力，
![133](https://img-blog.csdnimg.cn/20200221220749388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)


