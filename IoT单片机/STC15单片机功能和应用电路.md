# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 功能
常用SOP28的管脚分布，
![58](https://img-blog.csdnimg.cn/20210328185631386.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
IAP功能的地址，STC15W4K48S4的扇区范围`0000h~27FFh`，
![57](https://img-blog.csdnimg.cn/20210328181147798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
STC15W4K48S4串口有4个，串口1，3，4默认定时器2做波特率发生器，可以选定时器1做波特率发生器，串口2只能用定时器2做波特率发生器，但是SOP28的封装上就只有两个串口，其中串口2只能固定在3，4脚，串口1可以切换3个位置，
![59](https://img-blog.csdnimg.cn/20210328185945285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
ADC配置，通过`P1ASF`使能通道，`ADC_CONTR`的位也和STC8A不一样，
![398](https://img-blog.csdnimg.cn/2021040617005223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
中断源，这个表很详细，方便编写中断函数，中断函数声明格式例如`void func(void) interrupt x`，
![63](https://img-blog.csdnimg.cn/20210421010707993.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
PWM，这地方相对于STC8A需要配的寄存器也是大不同，主要是`PWMCR`和`PWMxCR`，启停控制位变化，STC15无IO强制高低功能，需手动控制GPIO实现。如果需要配置PWM相关端口，需要将相关端口设置成准双向口或者强推挽输出口。
CCP，与STC8A保持一致。


