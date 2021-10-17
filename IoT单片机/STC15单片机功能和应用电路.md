# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [STC8单片机的低功耗详解](https://blog.csdn.net/l420ll/article/details/80517862)

# 功能
常用SOP28的管脚分布，
![58](https://img-blog.csdnimg.cn/20210328185631386.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
常用LQPF44的管脚分布，2021，芯片缺货，SOP28停产，只能移植到LQPF44上，
![87](https://img-blog.csdnimg.cn/895cf51214a14591bfb59831359c5396.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiJ6YGN54yq,size_20,color_FFFFFF,t_70,g_se,x_16)

IO配置，与STC8A保持一致，
![407](https://img-blog.csdnimg.cn/20210427152452166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
IAP功能的地址，STC15W4K48S4的扇区范围`0000h~27FFh`，
![57](https://img-blog.csdnimg.cn/20210328181147798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
STC15W4K48S4串口有4个，串口1，3，4默认定时器2做波特率发生器，可以选定时器1做波特率发生器，串口2只能用定时器2做波特率发生器，但是SOP28的封装上就只有两个串口，其中串口2只能固定在3，4脚，串口1可以切换3个位置，
![59](https://img-blog.csdnimg.cn/20210328185945285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
ADC配置，通过`P1ASF`使能通道，`ADC_CONTR`的位也和STC8A不一样，
![398](https://img-blog.csdnimg.cn/2021040617005223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
中断源，这个表很详细，方便编写中断函数，中断函数声明格式例如`void func(void) interrupt x`，
![63](https://img-blog.csdnimg.cn/20210421010707993.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
PWM，这地方相对于STC8A需要配的寄存器也是大不同，主要是`PWMCR`和`PWMxCR`，启停控制位变化，STC15无IO强制高低功能，需手动控制GPIO实现。如果需要配置PWM相关端口，需要将相关端口设置成准双向口或者强推挽输出口。
CCP，寄存器与STC8A保持一致，脚位寄存器P_SW1定义一致，但脚位选择有变化。
![406](https://img-blog.csdnimg.cn/20210427150311817.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)STC15系列单片机可以运行3种省电模式以降低功耗，它们分别是：低速模式，空闲模式和掉电模式。正常工作模式下，STC15系列单片机的典型功耗是2.7mA ~ 7mA，而掉电模式下的典型功耗是<0.1uA，空闲模式下的典型功耗是1.8mA。低速模式由时钟分频器CLK_DIV (PCON2)控制，而空闲模式和掉电模式的进入由电源控制寄存器PCON的相应位控制。

# RS485下载
当下载口用作RS485通信的时候，需要特殊设计和特殊操作，选择自动模式只需按下面电路，选择IO控制，需要用ISP软件在非RS485通信的板子上按正常模式下载一次程序的同时使能485控制，后面才可以使用485来下载，每次都需要勾选ISP软件配置，否则下一次就无法485下载了。
![408](https://img-blog.csdnimg.cn/20210427155545646.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)



