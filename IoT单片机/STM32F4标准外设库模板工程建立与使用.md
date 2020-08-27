# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# SW4STM32安装
其实固件库安装过程很简单，在第一次新建工程时会提示选择使用Stdperiph 驱动还是Cube HAL，由于Stm32官方大力推行Cube HAL固件库，所以Cube HAL的固件库直接可以从网上直接一键下载安装。然而对于老的StdPeriph固件库不能一键式下载安装，会提示出错。所以，我们需要自己下载一个.zip固件包，放在C:\Users\LY\AppData\Roaming\Ac6\SW4STM32\firmwares文件夹下，其中的LY就是计算机的用户名。然后新建工程时在选择Stdperiph固件时会自动解压缩，这样就能使用该库进行编译了。界面如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125194709577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
 
# 工程配置

## 器件与时钟
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125194748320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
或者，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125194810564.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195014770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
修改晶振与时钟，根据注释可以算得sysclk为168MHz，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195336124.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195354839.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195410427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

## FSMC和FMC
STM32F4的某些系列是FSMC，有些是FMC
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195453468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195533372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

## 浮点处理器FPU
> [【MCU实战经验】+STM32F4 的FPU 的配置](http://www.stmcu.org/module/forum/thread-581903-1-1.html)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195750296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195805626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195818151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125195837691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
打开 option for target 选择 C/C++ 标签， 在define中添加：USE_HAL_DRIVER,STM32F407xx,__TARGET_FPU_VFP,ARM_MATH_MATRIX_CHECK,ARM_MATH_ROUNDING,ARM_MATH_CM4,__CC_ARM，由于我的是HAL的库，所以前面有USE_HAL_DRIVER的全局宏定义，如果你用的不是HAL库，而是使用固件库的话，一般会有固件库的一个全局宏定义USE_STDPERIPH_DRIVER和STM32F4XXxx在里面，这在固件库中的例子工程中都会有这个的。所以，我这里只需要添加
__TARGET_FPU_VFP,
ARM_MATH_MATRIX_CHECK,
ARM_MATH_ROUNDING,
ARM_MATH_CM4,
__CC_ARM
注意中间用英文逗号分开。其中ARM_MATH_MATRIX_CHECK是库函数的参数检查开关，这里添加后，就打开。ARM_MATH_ROUNDING这个是库函数在运算是是否开启四舍五入的功能，我这里添加，可以根据自己的需要进行配置。ARM_MATH_CM4这个就非常重要，必须要配置进去，否则在编译之后，会默认使用math.h的库函数，而不会用到硬件的FPU的。__CC_ARM是不同编译器的编译配置宏定义，__CC_ARM就是代表MDK开发环境。
打开工程中的 stm32f407xx.h 文件，注意不是 stm32f4xx.h 文件，是和你的芯片型号对应的头文件，比如我用的是STM32F407，所以我这里就选择打开stm32f407xx.h文件，找到     
#define __FPU_PRESENT            0       /*!< FPU present       这一句，将设置为 1
找到
#include "core_cm4.h"             /* Cortex-M4 processor and core peripherals */
#include "system_stm32f4xx.h"
#include <stdint.h>
这个地方，然后在下面添加 
#include "arm_math.h"
然后保存。

ARM_MATH_CM4
ARM_MATH_MATRIX_CHECK
ARM_MATH_ROUNDING
__FPU_PRESENT
__FPU_USED

## 代码优化
http://www.stmcu.org/module/forum/thread-603791-1-1.html


# 网线热插拔
http://blog.csdn.net/xukao5671927/article/details/77765464


# JTAG引脚复用

STM32f1 中JTAG 引脚作为普通IO口设置方法以及STM32f4中的方法的不同

在stm32f1中，我们对于不用的jtag引脚做io使用时，会使用以下步骤：（下面内容来自网络）
 GPIO_InitTypeDef GPIO_InitStructure;
 RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB|RCC_APB2Periph_GPIOC|RCC_APB2Periph_AFIO, ENABLE);
 
 /**********************
 1.执行端口重映射时,复用功能时钟得使能:RCC_APB2Periph_AFIO
 
 GPIO_Remap_SWJ_Disable
       : !< Full SWJ Disabled (JTAG-DP + SW-DP)
         此时PA13|PA14|PA15|PB3|PB4都可作为普通IO用了
 
 为了保存某些调试端口,GPIO_Remap_SWJ_Disable也可选择为下面两种模式：
  
GPIO_Remap_SWJ_JTAGDisable 
         :  !< JTAG-DP Disabled and SW-DP Enabled
         此时PA15|PB3|PB4可作为普通IO用了
  
GPIO_Remap_SWJ_NoJTRST
        : !< Full SWJ Enabled (JTAG-DP + SW-DP) but without JTRST
         此时只有PB4可作为普通IO用了 
 **********************/
 
 GPIO_PinRemapConfig(GPIO_Remap_SWJ_NoJTRST, ENABLE);  //使能禁止JTAG
 //初始化GPIOB  推挽输出
 GPIO_InitStructure.GPIO_Pin = (GPIO_Pin_3|GPIO_Pin_4);
 GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;  
 GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;  
 GPIO_Init(GPIOB, &GPIO_InitStructure); 
 GPIO_SetBits(GPIOB, GPIO_Pin_3|GPIO_Pin_4);
 //初始化GPIOA  推挽输出
 GPIO_InitStructure.GPIO_Pin = (GPIO_Pin_13|GPIO_Pin_14|GPIO_Pin_15);  
 GPIO_Init(GPIOA, &GPIO_InitStructure); 
 GPIO_SetBits(GPIOA, GPIO_Pin_13|GPIO_Pin_14|GPIO_Pin_15);

但是在stm32f4中不是这样的，STM32F4库函数中，已经取消了GPIO_PinRemapConfig()函数，对于复用功能，使用GPIO_PinAFConfig()函数了！
但是在GPIO_PinAFConfig()函数已经没有禁止JTAG/SW等选项了，而是复用到AF0~AF15线上，其中AF0是系统功能，STM32F4复位后JTAG对应的管脚的对应的功能就是AF0，（GPIO_AF_MCO=0） 所以这句可以不用：GPIO_PinAFConfig( , ,GPIO_AF_MCO);
直接配置GPIOx_MODER为输出，或输入模式即可，但是注意：STM32F4复位后JTAG对应的管脚的GPIOx_MODER值是0x02,即 复用功能！
所以直接配置GPIOx_MODER为所需的模式就可以了！

所以f4中，我们使用不用的jtag脚只需像平常使用其他io一样配置就好了。

# 使用

首先复制模板工程一份，重命名文件夹为新工程，打开工程，更新工程属性配置，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125200044667.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
编译工程，删除工程目录MDK目录下，原工程名开头的文件即可。

