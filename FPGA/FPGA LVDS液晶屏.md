# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [分享 4.3寸 TFT LCD 驱动板【CPLD+SRAM】](https://www.amobbs.com/forum.php?mod=viewthread&tid=5671984&highlight=tft)
> [【正点原子FPGA连载】第二十二章 RGB TFT-LCD彩条显示实验](https://www.amobbs.com/forum.php?mod=viewthread&tid=5713048&highlight=tft)
> [LVDS，LCD调试总结（持续更新）](https://blog.csdn.net/a617996505/article/details/82386952)
> [VGA编程接口讲解](https://blog.csdn.net/qq_39148922/article/details/85005271)
> [VGA Signal Timing](http://tinyvga.com/vga-timing)
> [采用CPLD或者FPGA显示TFT液晶屏](https://blog.csdn.net/web_star/article/details/74935822)
> [LVDS高速ADC接口， xilinx fpga实现](https://blog.csdn.net/u010161493/article/details/76732970)

# LCD
液晶屏支持RGB和LVDS两种模式，RGB信号接口，
![184](https://img-blog.csdnimg.cn/20190819212940750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
RGB的输入时序参数，显示区域1280x800，
![183](https://img-blog.csdnimg.cn/20190819202257742.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
液晶屏的LVDS管脚，
![185](https://img-blog.csdnimg.cn/20190819213036498.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
液晶屏的LVDS数据输入格式，
![182](https://img-blog.csdnimg.cn/20190819202045542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 参数
LCD编程中需要用到参数，Porch=肩，可以从RGB的输入时序参数获取，
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018121419415226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5MTQ4OTIy,size_16,color_FFFFFF,t_70)

# Video Timing
vtc设置，
![198](https://img-blog.csdnimg.cn/20190826200928662.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
