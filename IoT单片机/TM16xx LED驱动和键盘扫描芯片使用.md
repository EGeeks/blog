# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [天微公司官网](http://www.titanmec.com/index.php/)
> [TM1637驱动程序](https://www.cnblogs.com/alanfeng/p/4845450.html)
> [TM1637数码管显示STC51单片机驱动程序](https://blog.csdn.net/farmanlinuxer/article/details/79008078)

# 管脚和功能介绍
数码管共阴接法，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602194414605.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
这种接发最常用，控制方便，程序简单，SEGx对应的显示寄存器位置1，数码管发光，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602194814430.png)
共阳接法，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602194845229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
这种方式控制一个数码管，需要将所有的显存都刷新一遍，为了不改变其它数码管的状态，MCU必须记住当前显存值，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602195510456.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
SEGx和GRIDx都是芯片管脚名，对于TM1620B，支持下面几种模式，TM1637/TM1638段数固定，不可切换，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602200204772.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
TM637按键，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190612201109305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
键值为，
```c
  0xe8, 0xe9, 0xea, 0xeb, 0xec, 0xed, 0xee, 0xef,
  0xf0, 0xf1, 0xf2, 0xf3, 0xf4, 0xf5, 0xf6, 0xf7,
```
芯片已经自带了消抖的功能，软件不需要实现，下面是针对按一下与长按处理，对于没有长按功能的按键，检测到键值相同就直接退出，
```c
void KeyProcess(void)
{
  unsigned char temp;
  keyEventLast = keyEvent;
  keyEvent = Tm16xxGetKey();
  
  if (keyEvent != keyEventLast || keyEvent == KEY_NONE) {
    pressTime = 0;
  }
  pressTime += KEY_PROCESS_CYCLE;
  
  if (keyEvent == keyCode[KEY_MOSHI_ZHUANHUAN]) {
    if (keyEvent == keyEventLast)
      return;
...
```
# 指令
共4种类型命令，TM1620B支持4种，TM1637/TM1638没有第一个显示模式命令，
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019060220065082.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
数据命令，设置普通模式，地址自动增加，写数据到显示寄存器，一次性把单片机里的数据写到显示寄存器，或者读键盘数据，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602201019850.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
显示控制，可控制数码管的亮度，亮度小寿命长，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602201351338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
地址命令，注意，显示地址从0xC0，对应地址其实是0，0xC0是因为高两位的命令类型是11b，不同芯片显存长度不一致，下面是TM1620B，TM1637只有6个字节，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602201626675.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
显存分布，以TM1620B说明，比如0xC1只有3/4/5位有用，对应点亮硬件原理图SEG12/SEG13/SEG14位置的LED，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602202014544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 通信接口
TM1620B/TM1638是3线制接口，STB下降沿为开始信号，STB高电平停止传输，TM1637是2线制接口，不同在于3线制多一个STB信号，可以表示数据起始停止信号，2线制中，这个信号通过CLK/DIO组合来实现，在输入数据时当CLK 是高电平时，DIO 上的信号必须保持不变；只有CLK 上的时钟信号为低电平时，DIO 上的信号才能改变。数据输入的开始条件是CLK 为高电平时，DIO 由高变低；结束条件是CLK 为高时，DIO 由低电平变为高电平。TM1637 的数据传输带有应答信号ACK，当传输数据正确时，会在第八个时钟的下降沿，芯片内部会产生一个应答信号ACK 将DIO 管脚拉低，在第九个时钟结束之后释放DIO 口线。TM1637读键盘数据，S0/S1/S2共3为对应8个SEGx上的按键，K1/K2是对应管脚上的按键，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190602210942964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 驱动
TM1637官方驱动，
```c
/*****************************************************************************
 *版权信息：深圳天微电子有限公司
 *文 件 名：TM1637-V1.0
 *当前版本：V1.0
 *MCU 型号：STC12C5608AD
 *开发环境：Keil uVision4
 *晶震频率：11.0592MHZ       
 *完成日期：2013-07-19
 *程序功能：数码管驱动和按键：驱动8段6位LED共阳数码管显示,当对应按键按下时显示1~7,原理图请参考TM1637规格书；          
 *免责声明：1.此程序为TM1637驱动共阳LED数码管和按键演示程序，仅作参考之用。
            2.如有直接使用本例程程序造成经济损失的，本公司不承担任何责任             
********************************************************************************/
#include <reg52.h>                                                //头文件
#include "intrins.h"                                                //包含_nop_()指令头文件

#define nop _nop_();_nop_();_nop_();_nop_();_nop_();                 //宏定义


/********************定义控制端口**********************/
sbit CLK=P2^2;                                                                 //定义CLK
sbit DIO=P2^3;                                                                 //定义DIO


/********************定义数据*************************/
unsigned char code CODE[10]={0xc0,0xf9,0xa4,0xb0,0x99,0x92,0x82,0xf8, 0x80,0x90,0x88,0x83,0xc6,0xa1,0x86,0x8e}; //共阳显示数据0-F
                                                        /* 0   1    2    3    4     5    6    7     8    9   a     b    c    d    e    f*/
unsigned char code TAB[10]={0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,};//数码管不显示

/********************延时函数，延时nms******************/
void delay_nms(unsigned int n)
{
 unsigned int i;
 while(n--)
   for(i=0;i<550;i++);
}

/********************Start函数*************************/
void I2CStart()
{
 DIO=1;
 CLK=1;
 nop;
 DIO=1;
 nop;
 DIO=0;
 nop;
 CLK=0;
}

/********************Stop函数*************************/
void I2CStop()
{
    CLK=0;
        nop;
        nop;
        DIO=0;
        nop;
        nop;
        CLK=1;
        nop;
        nop;
        nop;
        DIO=1;
        nop;
        CLK=0;
        DIO=0;
}

/***************发送8bit数据，从低位开始**************/
void I2CWritebyte(unsigned char oneByte)
{
  unsigned char i;
  for(i=0;i<8;i++)
  {
    CLK=0;
        if(oneByte&0x01) 
          DIO=1;
        else 
          DIO=0;
        nop;
    CLK=1;
    oneByte=oneByte>>1;
  }
                                                                                  //8位数据传送完                 
        CLK = 0;                                                                //判断芯片发过来的ACK应答信号
        nop;
        while(DIO==1);
        nop;
        CLK = 1;
        nop;
}

/***************读按键程序**************/
unsigned char read_key()
{
unsigned char rekey,i;
I2CStart();
I2CWritebyte(0x42);                                                         //写读键指令0x42
DIO=1;
for(i=0;i<8;i++)
{
        CLK=0;
        nop;
        nop;
        rekey=rekey>>1;                                                           //先读低位
        nop;
        nop;
        CLK=1;
        if(DIO) 
          rekey=rekey|0x80;
        else 
          rekey=rekey|0x00;
        nop;
}
        CLK = 0;                                                          //判断芯片发过来的ACK应答信号
        nop;
        nop;
        while(DIO==1);
        nop;
        nop;
        CLK = 1;
        nop;
        nop;
        I2CStop();
        
        return rekey;
}

/************显示函数，地址自加一************/
void disp0(unsigned char *p)                              
{
 unsigned char i;
 I2CStart();
 I2CWritebyte(0x40);                                  //数据命令设置：地址自动加1，写数据到显示寄存器
 I2CStop();

 I2CStart();
 I2CWritebyte(0xc0);                                  //地址命令设置：初始地址为00H
 for(i=0;i<6;i++)                                  //发送4字节数据到显存
 {
 I2CWritebyte(*p);
 p++;
 }
 I2CStop();

 I2CStart();
 I2CWritebyte(0x8C);                                 //显示控制命令：开显示，脉冲宽度为11/16.
 I2CStop();

}

/************显示函数，固定地址写数据************/
void disp(unsigned char add, unsigned char value)
{
 I2CStart();
 I2CWritebyte(0x44);                                 //数据命令设置：固定地址，写数据到显示寄存器
 I2CStop();

 I2CStart();
 I2CWritebyte(add);                                //地址命令设置：写入add对应地址

 I2CWritebyte(CODE[value]);                        //给add地址写数据
 I2CStop();

 I2CStart();
 I2CWritebyte(0x8C);                                //显示控制命令：开显示，脉冲宽度为11/16.
 I2CStop();

}

/************按键处理函数，按键数据低位在前高位在后************/
void key_process()
{
 unsigned char temp;
 temp=read_key();                                   //读取按键返回值
 if(temp!=0xff)
 {
  disp0(TAB);                                           //清屏
  switch(temp)
  {
    case 0xf7 : disp(0xc0,1);break;                  //K1与SG1对应按键按下,显示1
        case 0xf6 : disp(0xc1,2);break;                  //K1与SG2对应按键按下,显示2
        case 0xf5 : disp(0xc2,3);break;                  //K1与SG3对应按键按下,显示3
        case 0xf4 : disp(0xc3,4);break;                  //K1与SG4对应按键按下,显示4
        case 0xef : disp(0xc4,5);break;                  //K2与SG1对应按键按下,显示5
        case 0xee : disp(0xc5,6);break;                  //K2与SG2对应按键按下,显示6
        case 0xed : disp(0xc0,7);break;                  //K2与SG3对应按键按下,显示7
        default   : break;
  }
 }
}

void main()
{
　　disp0(CODE);                                      //上电数码管显示0~5
　　delay_nms(1);
　　while(1)                                     //按键后显示按键内容
　　{
　　　　key_process();
　　　　delay_nms(100);
　　}
}

```



