# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [GB2312简体中文编码表](https://blog.csdn.net/halchan/article/details/78353947)
> [GB2312、Unicode编码等](https://www.cnblogs.com/predator-wang/p/4859860.html)
> [RGB565颜色表，附RGB888转RGB565工具，RGB24转RGB565工具](https://blog.csdn.net/liquanfeng9227/article/details/74895446)
> [图解RGB565、RGB555、RGB16、RGB24、RGB32、ARGB32等格式的区别](https://blog.csdn.net/byhook/article/details/84262330)

# 更新图片资源
目的是改掉开机画面，首先制作自己屏幕尺寸相同的图片，480x272，打开迪文资料带的jpgconvert，运行，win10需要先安装`.NET3.5`，我的图片大小超了32KB，用软件可以生成32KB以下的图片，在windows格式化的时候必须手动选择4KB扇区，不能默认。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190627232034415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
根目录
![112](https://img-blog.csdnimg.cn/20190703222610729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 方法
（1）确定型号DMT48270C050_04WN，官网下载手册，
（2） 确定屏幕驱动芯片类型，T5UIC1，下载该芯片的使用手册，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190606000249495.PNG)
（3）目前只需要显示汉字和数值，所以只需要这两天命令，对于传输的16进制数，为了显示浮点数，设置了，Num_I和Num_F来设置小数点前和小数点后的位数，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190606000710156.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
（4）在键盘设置参数时，需要数值的某一位闪烁，可利用下面的指令，XOR方式，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190607105248472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 乱码
Keil编辑器设置Ascii编码，字符串中输入中文直接显示，发现汉字待和过乱码，这两个字均包含0xfd，在字符串中添加0xfd即可，
```c
  "待\xfd机            ", 
```
后续发现这是Keil的bug，需要打[补丁](https://blog.csdn.net/u010443760/article/details/80441183)，
# 测试
![微信图片_20191007211837](https://img-blog.csdnimg.cn/20191007211909930.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 代码
c语言，
```c
#include "uart.h"
#include "disp.h"

/*480*272*/
char dispBuf[256];

#define COLOR_BLACK   0x0000
#define COLOR_RED     0xf800
#define COLOR_GREEN   0x07e0
#define COLOR_BLUE    0x001f
#define COLOR_WHITE   0xffff

#define BACKGROUND_COLOR    COLOR_BLACK

#define CHAR_WIDTH    24
#define CHAR_HEIGHT   48

void DispClear(void)
{
  unsigned char i = 0;
  disp[i] = 0xaa;i++;
  disp[i] = 0x01;i++;/*cmd*/
  disp[i] = BACKGROUND_COLOR >> 8;i++;/*MSB first*/
  disp[i] = BACKGROUND_COLOR;i++;/*MSB first*/
  disp[i] = 0xcc;i++;
  disp[i] = 0x33;i++;
  disp[i] = 0xc3;i++;
  disp[i] = 0x3c;i++;
  Uart2SendBuf(dispBuf, i);
}

void DispShowString(unsigned short color, unsigned short x, unsigned short y, char* p)
{
  unsigned char i = 0;
  disp[i] = 0xaa;i++;
  disp[i] = 0x11;i++;/*cmd*/
  disp[i] = 0x07;i++;/*mode: without background color, 24x48*/
  disp[i] = color >> 8;i++;/*MSB first*/
  disp[i] = color;i++;/*MSB first*/
  disp[i] = BACKGROUND_COLOR >> 8;i++;/*MSB first*/
  disp[i] = BACKGROUND_COLOR;i++;/*MSB first*/
  disp[i] = x >> 8;i++;/*MSB first*/
  disp[i] = x;i++;/*MSB first*/
  disp[i] = y >> 8;i++;/*MSB first*/
  disp[i] = y;i++;/*MSB first*/
  while (*p != 0) {
    disp[i] = *p;
    i++;
    p++;
  }
  disp[i] = 0xcc;i++;
  disp[i] = 0x33;i++;
  disp[i] = 0xc3;i++;
  disp[i] = 0x3c;i++;
  Uart2SendBuf(dispBuf, i);
}
/*
n: disp num
i, f: fraction, radix point, n = 169 i = 2 f = 1 -> 16.9
*/
void DispShowNum(unsigned short color, unsigned char i, unsigned char f, unsigned short x, unsigned short y, unsigned long n)
{
  unsigned char i = 0;
  disp[i] = 0xaa;i++;
  disp[i] = 0x14;i++;/*cmd*/
  disp[i] = 0x07;i++;/*mode: without background color, 24x48*/
  disp[i] = color >> 8;i++;/*MSB first*/
  disp[i] = color;i++;/*MSB first*/
  disp[i] = BACKGROUND_COLOR >> 8;i++;/*MSB first*/
  disp[i] = BACKGROUND_COLOR;i++;/*MSB first*/
  disp[i] = x >> 8;i++;/*MSB first*/
  disp[i] = x;i++;/*MSB first*/
  disp[i] = y >> 8;i++;/*MSB first*/
  disp[i] = y;i++;/*MSB first*/
  disp[i] = n >> 24;i++;
  disp[i] = n >> 16;i++;
  disp[i] = n >> 8;i++;
  disp[i] = n;i++;
  disp[i] = 0xcc;i++;
  disp[i] = 0x33;i++;
  disp[i] = 0xc3;i++;
  disp[i] = 0x3c;i++;
  Uart2SendBuf(dispBuf, i);
}

void DispSelectChar(unsigned short color, unsigned short x, unsigned short y)
{
  unsigned char i = 0;
  disp[i] = 0xaa;i++;
  disp[i] = 0x05;i++;/*cmd*/
  disp[i] = 0x02;i++;/*mode: color XOR*/
  disp[i] = color >> 8;i++;/*MSB first*/
  disp[i] = color;i++;/*MSB first*/
  disp[i] = x >> 8;i++;/*MSB first*/
  disp[i] = x;i++;/*MSB first*/
  disp[i] = y >> 8;i++;/*MSB first*/
  disp[i] = y;i++;/*MSB first*/
  disp[i] = (x + CHAR_WIDTH) >> 8;i++;/*MSB first*/
  disp[i] = (x + CHAR_WIDTH);i++;/*MSB first*/
  disp[i] = (y + CHAR_HEIGHT); >> 8;i++;/*MSB first*/
  disp[i] = (y + CHAR_HEIGHT);i++;/*MSB first*/
  disp[i] = 0xcc;i++;
  disp[i] = 0x33;i++;
  disp[i] = 0xc3;i++;
  disp[i] = 0x3c;i++;
  Uart2SendBuf(dispBuf, i);  
}
```

