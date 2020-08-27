# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [51单片机实现scanf和printf函数](https://blog.csdn.net/jipingyuan/article/details/20056989)
> [通过串口实现printf和scanf函数](https://blog.csdn.net/fillthesky/article/details/52199877)
> [适用于单片机的小型类shell的命令行软件](https://blog.csdn.net/qq_38901733/article/details/88114837)

# 目标
如何从头开始一个单片机项目，结合STC8A来说一说我的做法，因为这也是我第一次使用STC8A这个芯片，**我这里不想谈什么面向对象设计，什么代码解耦和，什么代码复用，小项目，一期项目搞完，后期修修补补也够了，改几行代码搞定**，拿到项目说明，首先总结一下用到的所有外设，打好工程的基本框架，比如，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190521225430122.png)
app是应用，hw是用到的外设驱动，util是工具类代码，app将调用hw驱动和util工具来完成项目功能，我这里不强调代码复用，所以来了一个新项目，hw下的代码都要看情况小改，这点肯定比不过STM32那种函数库，不过这种方式软件效率必然是最高的，强调通用性也就必然会损失性能。在这个项目里，

- `main.c`是主函数
- `shellcmd.c`是调试命令函数
- `adc.c`是模拟数据采集，电压和温度
- `iap.c`是操作STC8A内部EEPROM
- `pin.c`是IO控制，在main函数开始初始化所有管脚状态，包括设置管脚是uart，spi等特殊功能管脚
- `pwm.c`是STC8A内部高性能PWM模块控制代码，驱动电机或者蜂鸣器等
- `timer.c`是STC8A定时器控制代码，定时器可提供时间基准，完成延时和任务切换
- `uart.c`是串口函数
- `disp.c`是液晶屏显示函数
- `key.c`是键盘扫描函数
- `pca.c`是PCA模块控制代码，测量外部信号频率
- `shell.c`是调试终端工具，配合uart使用

根据项目需求，完成驱动设计，比如在`adc.c`中完成15路adc采集，`pwm.c`中完成2路pwm控制，下面介绍我的开发，
# ADC
初始化函数`adcInit`配置管脚`P1.0~7 P0.0~6`为ADC模式，配置速度模式为512个CPU时钟，这也是最慢的速度，配置采样数据模式为左对齐，左对齐的目的是将12位精度的ADC当成8位用，最后将将ADC电源打开，`ADCCFG = 0x0f`时ADC将采用最慢的速度采集，由于没加入RTOS，这会卡住很长的时间，注意场合。
```c
void PinInit(void)
{
  /*P1.0~7 ADC0~7 P0.0~6 ADC8~14 P0.7 push-pull out*/
  P1M0 = 0x00;
  P1M1 = 0xff;
  P0M0 = 0x80;
  P0M1 = 0x7f;  
}
void adcInit()
{
  ADCCFG = 0x0f;                              
  ADC_CONTR = 0x80;                           
}
```
获取每个通道的电压，
```c
unsigned char adcGetCh(unsigned char ch)
{
  unsigned char c;
  ADC_CONTR |= (0x40 + ch);
  _nop_();
  _nop_();
  while (!(ADC_CONTR & 0x20));
  ADC_CONTR &= ~0x20;
  c = ADC_RES;
  return c;
}
```
# PWM
初始化PWM，没有使用任何PWM中断，相应位置0，通道初始电平为0，第一个反转点为0，则PWM波从高开始，24MHz的系统时钟，周期计数为240，产生100KHz的PWM波，访问PWM寄存器需要控制P_SW2的EAXFR位，
```c
#include "config.h"
#include "stc8.h"
#include <intrins.h>
#include "pwm.h"
/*
PWM0, P20, 1Hz, %50, beep
PWM6, P26, 100KHz, control by P00/P01, fixed-power
PWM7, P27, 100KHz, control by P00/P01, fixed-current
*/
void PwmInit(void)
{
  /*EAXFR=1 when reg addr>0xFFF0*/
  P_SW2 |= 0x80;
  PWMCKS = 0x00;//system clk FOSC=24MHz
  PWMC = 240;//100KHz 
  PWM6T1= 0;
  PWM6T2= 120;
  PWM6CR= 0x80;
  PWM7T1= 0;
  PWM7T2= 120;
  PWM7CR= 0x80;
  P_SW2 &= 0x7f;
  PWMCR = 0x80;
}
```
强制输出高或低，很多场景下会用到，
```c
/*
force high/low
*/
void PwmSetPin(unsigned char ch, unsigned char out)
{
  if (out)
    (*(unsigned char volatile xdata *)(0xff05 + ch << 0x10)) = 0x2;
  else
    (*(unsigned char volatile xdata *)(0xff05 + ch << 0x10)) = 0x1;
}
```
# PCA
初始化PCA在P2口，
```c
  /*Uart1 P30 P31 CCP P22~P26*/
  P_SW1 = 0x1f;
```
初始化PCA为外部脉冲捕获功能，CCP0为P23，CCP1为P24，CCP2为P25，上升沿捕获，这样两个上升沿之间的宽度就是周期，可计算出频率，
```c
void PcaInit(void)
{
  /*clear interrupt flag*/
  CCON = 0x00;
  /*system clk enable overflow interrupt*/
  CMOD = 0x09;
  CL = 0x00;
  CH = 0x00;
  /*capture on rising edge*/
  CCAPM0 = 0x21;
  CCAP0L = 0x00;
  CCAP0H = 0x00;
  CCAPM1 = 0x21;
  CCAP1L = 0x00;
  CCAP1H = 0x00;
  CCAPM2 = 0x21;
  CCAP2L = 0x00;
  CCAP2H = 0x00;
  /*start*/
  CR = 1;
}
```
中断处理，计数器只有16位，对于50Hz这样的波形，24MHz系统时钟，会发生溢出，通过检测CF溢出中断才能正确计算频率，其中`ccpCfCnt`也需要定期清零，否则仍然有溢出风险，
```c
void PCA_Isr() interrupt 7
{
  if (CF)
  {
    CF = 0;
    ccpCfCnt++;                                  
  }
  for (ccpi = 0; ccpi < CCP_NUM; ccpi++) {
    if (CCON & (1 << ccpi)) {
      CCON &= (1 << ccpi);
      ccpCount0[i] = ccpCount1[i];
      ccpCount1[i] = (ccpCfCnt << 16) + (CCAP0H << 8) + CCAP0L;
      ccpTick[i] = ccpCount1[i] - ccpCount0[i];
      /*
      if (ccpCount1[i] > ccpCount0[i])
        ccpTick[i] = ccpCount1[i] - ccpCount0[i];
      else {
        ccpCfCnt = 0;
        ccpCount0[i] = 0;
        ccpCount1[i] = (ccpCfCnt << 16) + (CCAP0H << 8) + CCAP0L;
      }
      */
    }
  }
}
```
# Timer
定时器用来调度任务，没有RTOS，没有任务的时候，CPU可以进入低功耗状态，定时器默认工作在12T模式，即计数时钟是CPU时钟的`1/12`，sysTick存储当前时刻的开机时间，在溢出的时候累加，单位毫秒，最大为49.7天，之后就溢出了，需要注意，定时器工作在模式0，16位自动重载模式，T0_CNTS计算法则是控制定时器距离溢出的时间为10ms，
```c
/*overflow time 10ms*/
#define T0_OF_TIME      10
#define T0_CNTS         (65536 - ((FOSC / 12) * 10) / 1000)
unsigned long sysTickMs;

void Timer0Isr() interrupt 1
{
  sysTickMs += T0_OF_TIME;                             
}

void Timer0Init(void)
{
  sysTickMs = 0;
  /*16-bit auto reload*/
  TMOD = 0x00;
  TL0 = T0_CNTS;
  TH0 = (T0_CNTS >> 8);
  /*start T0*/
  TR0 = 1;
  /*IE enable T0 overflow interrupt*/
  ET0 = 1;
}
```
# LVD低压检测
本打算在低压检测时存储掉电数据，后发现EEPROM总是丢数，坑啊
```c
void LvdIsr() interrupt 6
{
    PCON &= ~LVDF;     
    IapSave();
}
```

