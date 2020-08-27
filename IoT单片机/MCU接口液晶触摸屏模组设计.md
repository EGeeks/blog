# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
资料我微信发送，下载比较慢
> [ATK-3.5' TFTLCD 模块V2版本资料](http://www.openedv.com/forum.php?mod=viewthread&tid=269343&highlight=lcd%2B%D7%CA%C1%CF)
> [ATK-4.3' TFTLCD电容触摸屏模块资料](http://www.openedv.com/forum.php?mod=viewthread&tid=34751&highlight=lcd%2B%D7%CA%C1%CF)
> [ATK-7寸TFTLCD V2版本模块资料](http://www.openedv.com/forum.php?mod=viewthread&tid=51287&highlight=lcd%2B%D7%CA%C1%CF)
> [正点原子STM32开发板+FPGA开发板+四轴+RT1052+各类模块最新资料](http://openedv.com/thread-13912-1-1.html)

# 液晶触摸屏
![TB25.DQaGzB9uJjSZFMXXXq4XXa_!!2997122261](https://img-blog.csdnimg.cn/20191002204209323.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
左边管脚较多的是液晶屏接口，一般都是RGB接口，通常包含8位R（Red红色），8位G（Green绿色），8位B（Blue蓝色），HSYNC（行同步），VSYNC（场同步），DCLK（像素时钟）等，通常DCLK的时钟为几十MHz。液晶屏的物理接口一般都采用FPC排线，FPC连接器。
右边管脚较少的是触摸接口，通常为SPI或I2C接口，这个接到单片机的SPI或I2C管脚即可，触摸分为两种：电阻触摸屏，电容触摸屏，电容触摸屏就是当下智能手机采用的屏幕，反应灵敏，但是带了手套就无法操控，电阻触摸屏是2010年前手机的触摸屏，触摸精度低，是按压原理，戴手套也是可以的。
![121](https://img-blog.csdnimg.cn/20191003182648256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# MCU接口
stm32f103系列是不自带RGB接口，而且DCLK频率很高，使用普通IO来驱动RGB接口也处理不过来，这就需要一个专用液晶驱动芯片来把RGB接口转换成MCU接口，而stm32f429和更高端的stm32f7带RGB LCD接口，这种单片机就不需要液晶驱动芯片了，但是价格也更贵，而且同时液晶驱动芯片提供了显存GRAM，否则绘图需要占用大量SRAM，这时就需要外扩SRAM，或者选择大容量SRAM的单片机，这也导致价格更贵。通常我们说的MCU接口就是Intel 8086并行接口，包含读写时钟信号，16/18/24位不等的数据线，一般，我们在单片机上采用16位数据线。
将电阻屏或者电容屏的触摸信息转换为单片机可读的触摸位置，也需要专用芯片，通常电阻屏选用XPT2046，XPT2046为SPI接口，电容屏选择FT5426/FT5206/GT9147等，FT5426/FT5206/GT9147为I2C接口。通常液晶触摸芯片被放在液晶屏上，如上图所示，触摸排线上方的软PCB里有一个FT5426芯片。
# 方案
物料：
- 单片机：stm32f103zet6
- 液晶驱动芯片：ssd1963
- 液晶触摸屏：7寸电容触摸屏，RGB接口，FT5426触摸驱动
- 电源：MP3302/AMS1117/TPS61040

液晶部分的原理图参考[ATK-7寸TFTLCD V2版本模块资料](http://www.openedv.com/forum.php?mod=viewthread&tid=51287&highlight=lcd%2B%D7%CA%C1%CF)，这里就不重复造轮子了，需要注意的是7寸屏的原理图里，放了一个XPT2046电阻触摸屏芯片，这样做是可以一块电路板同时支持电容屏和电阻屏，通过焊接不同位置的电阻来切换，在项目的实际应用中可只选其一。
![119](https://img-blog.csdnimg.cn/20191003174450893.png)
这里看一下液晶和单片机制接如何连接，根据上面所述，主要是连接液晶的并行接口和触摸的I2C接口，参考正点原子的战舰V3开发板资料，液晶屏并行接口连接到了stm32的FSMC管脚，stm32的FSMC总线相当于51单片机访问外部内存的总线，是专门用来访问并行接口的外设的。FSMC管脚位于单片机的`PD0~15`，`PE0~15`，`PF0~15`，`PG0~15`管脚上，具体参考战舰V3的原理图。触摸接口接到了普通IO上，没有使用单片机的专用SPI/I2C管脚，所以是用软件模拟的SPI/I2C。
![120](https://img-blog.csdnimg.cn/2019100317572079.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)


