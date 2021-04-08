# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [官网](http://www.stcmcu.com/)
> [学习笔记之－51单片机IO口详解](https://blog.csdn.net/gjxman1314/article/details/54985734)

# 功能和管脚介绍
下面是STC8A4K64S2A12的LQFP64封装的管脚图，来自芯片手册3.1.2小节，同样还有LQFP48和LQFP32封装的，这里以LQFP64介绍，LQFP64可提供最多的IO管脚，LQFP48和LQFP32只是LQFP64的一部分。STC8A8K64S2A12相比较STC8A4K64S2A12只是单片机RAM由4K增加到8K，视项目的复杂度可兼容切换。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619223624698.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619221137615.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018111114402049.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
上图中可以看到，比如64脚的标识是RxD2/PWM0_2/ADC0/P1.0，表示这个管脚既可以做串口2接收引脚，PWM0的输出，ADC0采集输入和通用IO口。另外，CCP标识的管脚可以测量外部信号的频率，此系列单片机最多能同时捕获4组外部输入CCP0~CCP4，SCLK、MISO、MOSI、SS这四个管脚是一组，实现SPI功能，比如如果项目中有SPI接口的液晶屏，可以连接到这个管脚上，I2CSDA、I2CSCL这两个管脚是一组，实现I2C功能，项目中有I2C接口的EEPROM可以接到这组管脚上，每个管脚的详细说明在手册的3.2小节。
另外有一点需要注意的是，这个系列的单片机提供了引脚功能切换功能，这里以串口RxD2介绍，图中可以看到，64脚有RxD2功能，在22脚出现了RxD_2标识，参考手册3.3.1小节，
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111144040897.png)
也就是说，串口2可以使用P1.0、P1.1或者P4.0、P4.2，但是不能RxD2使用P1.0，TxD2使用P1.1。同样CCP信号捕获接口，可以切换管脚，但不能分别切换，必须整体切换。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111144054343.PNG)
串口1，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519123652497.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
切换寄存器S1_S[1:0]，位于P_SW1，地址0xA2，EAXFR在访问高于0xFFF0地址的寄存器，比如PWM寄存器，需要置1，访问完后清0，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519123800327.PNG)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190523221838554.png)
STC8A的IO可配置输入输出模式，IO为ADC输入管脚时，配置为高阻输入，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190521221613730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
PCA模块可以当成定时器使用，可以输出PWM波，但通常用于测量外部信号频率，可对4路外部信号同时计算频率，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190521225305950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
PCA的计数时钟有如下选择，通常选择`100b`即系统时钟，
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019052123000035.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
PCA的中断源，
![7](https://img-blog.csdnimg.cn/20200923222131335.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
PWM在手册里指的是增强型PWM，PWM使用的原理和PCA差不多，一个统一的计数器PWMCH/L，这个计数器控制着PWM周期，对应8个通道，每个通道可以设置两个反转点，PWMxCR可设置初始电平，在遇到反转点的时候，IO电平反转，通过反转点可以控制占空比和相位，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190522232050843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
PWM可以在P1，P2，P6之间切换，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519222103638.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
STC8A部分中断向量号，C语言编程会用到，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190601180303609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
STC8A单片机IAP，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190616165645154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 应用电路
STC8A单片机需要提供系统电源和ADC参考电源，在不需要高精度ADC的情况下，可共用一组电源，我们的项目共用可满足需求，另外STC单片机提供了串口下载功能，不需要额外购买编程器，参考手册5.2.2小节电路图，
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111144110724.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
图中左方有一个Power On的上电开关，在下载的过程中需要断电再上电，所以需要有一个自锁开关，但项目批产的过程中，不需要次开关，可通过镊子短路或者外部电源开关实现上电过程，下载电路可将P3.0，P3.1，GND三个引脚用排针引出即可。红线右侧电路不需要。

# IO口准双向模式
![110](https://img-blog.csdnimg.cn/20190621233353796.png)
应用场景，开关按下5v，不按则悬空，需要完成的功能是，读1动作，读0不动作，这里为了能读到0，需要先对IO写0，否则是读不到0的，
![111](https://img-blog.csdnimg.cn/20190621233639233.png)
