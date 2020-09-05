
# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [宏定义中的#,##](https://blog.csdn.net/jiangjingui2011/article/details/6706967)
> [关于宏##的使用注意一点](https://blog.csdn.net/lovewubo/article/details/37937455)
> [c语言 得到结构体成员偏移](https://blog.csdn.net/whatday/article/details/105000046)

# 计算结构体成员的偏移
3种方法，
```cpp
#include <stdio.h>
#include <stddef.h>
 
struct stru {
    char c;
    int  i;
};
 
int main(int argc, char *argv[])
{
    struct stru s;
 
    printf("Offset of stru.i:        %ld\n", (size_t)((char*)&s.i - (char*)&s));
    printf("&((struct stru *)0)->i:  %ld\n", (size_t)&((struct stru*)0)->i);
    printf("offsetof(struct stru,i): %ld\n", offsetof(struct stru, i));
 
    return 0;
}
```

# warning C206: missing function-prototype
单片机程序，引用DelayMs，
```c
  while (1) {
    //ShellMain();
    printf("hello world\r\n");
    DelayMs(1000);
  }
```
报错，
```shell
Build target 'Target 1'
compiling main.c...
..\src\main.c(26): warning C206: 'DelayUs': missing function-prototype
..\src\main.c(26): error C267: 'DelayUs': requires ANSI-style prototype
Target not created
```
明明已经实现了这个函数，后来发现是头文件的预编译宏和另外一个文件重复，
```c
#ifndef __PCA_H_
#define __PCA_H_

void DelayUs(unsigned long us);
void DelayMs(unsigned long ms);

#endif
```
更改为，
```c
#ifndef __DELAY_H_
#define __DELAY_H_

void DelayUs(unsigned long us);
void DelayMs(unsigned long ms);

#endif
```

# 宏##高级用法-给结构体赋值问题
编程时遇到一个需求，这是一个结构体，
```c
struct srioMaintenanceData
{
	unsigned int dstId;
	unsigned int hopCnt;
	unsigned int offset;
	unsigned int value;
};
```
在代码中，想给这个结构体赋值，
```c
maintData.dstId = 0xFF; maintData.hopCnt = 1;
maintData.offset = SRIO_REG_DEV_ID_CAR; maintData.value = 0;
```
想用一个宏来替代，类似于，
```c
SRIO_MAINT_DATA_SET(maintData, 0xFF, 1, SRIO_REG_DEV_ID_CAR, 0);
```
我这样定义，
```c
#define SRIO_MAINT_DATA_SET(n, a, b, c, d) n##.dstId=a;##n##.hopCnt=b;##n##.offset=c;##n##.value=d
```
报错，但这不是语法错误，是编译器不支持这个操作。
![这里写图片描述](https://img-blog.csdn.net/20180618194018891?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```c
Multiple markers at this line
	- pasting "maintData" and "." does not give a valid preprocessing 
	 token
	- Invalid use of macro pasting in macro: SRIO_MAINT_DATA_SET
	- in expansion of macro 'SRIO_MAINT_DATA_SET'
```
问题不知道怎么解决，搜了一下资料，据说vs中是不会报错的，[根据C标准，用##操作后的结果必须是一个已经预定义过的符号。否则是未定义的。所以gcc和vs对于这个未定义行为表示了不同的看法，前者是给出错误，后者一笑而过。那什么是已经预定过的符号呢? 它包含了这些：头文件名, 等式, 预处理数字, 字符常数, 字符串值, 标点符号, 单个非空字符](https://blog.csdn.net/lovewubo/article/details/37937455)。采取一种曲线救国策略，
```c
inline void SRIO_MAINT_DATA_SET(struct srioMaintenanceData* p, unsigned int dstId, unsigned int hopCnt, unsigned int offset, unsigned int value)
{
	p->dstId = dstId;
	p->hopCnt = hopCnt;
	p->offset = offset;
	p->value = value;
}
```
希望有大神可以在评论区告知怎样用宏完成结构体赋值，可能在linux内核代码里有类似操作。
2018-06-19补充：今天写windows驱动看到WDK的一个实现：
```c
WDF_DMA_ENABLER_CONFIG   dmaConfig;
WDF_DMA_ENABLER_CONFIG_INIT( &dmaConfig,
                           WdfDmaProfileScatterGather64Duplex,
                           DevExt->MaximumTransferLength );
```
WDK是这样实现的，也使用的inline。表示心里略平衡一点。
```c
VOID
FORCEINLINE
WDF_DMA_ENABLER_CONFIG_INIT(
    __out PWDF_DMA_ENABLER_CONFIG Config,
    __in  WDF_DMA_PROFILE    Profile,
    __in  size_t             MaximumLength
    )
{
    RtlZeroMemory(Config, sizeof(WDF_DMA_ENABLER_CONFIG));

    Config->Size = sizeof(WDF_DMA_ENABLER_CONFIG);
    Config->Profile = Profile;
    Config->MaximumLength = MaximumLength;
}
```

# C语言转义字符\x
反斜杠是转译符，`\x5c`就是说：ASCII码十六进制是`0x5c`的那个字符,`\x`可以用16进制的方式来初始化char数组，比如`char *s = "75\xA1\xE6"`表示GB2312编码的75℃。
