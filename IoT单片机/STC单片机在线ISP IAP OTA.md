# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [STC15单片机功能和应用电路](https://blog.csdn.net/Zhu_Zhu_2009/article/details/93248561)
> [STC8A单片机功能和应用电路](https://blog.csdn.net/Zhu_Zhu_2009/article/details/83958478)
> [STC51单片机实现IAP远程升级过程分享](https://blog.csdn.net/v__star/article/details/86626494)

# ISP
在线ISP，主要依靠`IAP_CONTR`寄存器，使用`IAP_CONTR=0x60`触发单片机进入ISP模式，而不需要冷启动，减少调试时的麻烦，可配合自定义串口命令，可使用官方提供的ISP软件，或者利用官方提供的`Upgrade.exe`自行开发软件。

# IAP
一般将全部64K都设置成EEPROM，让用户程序空间和EEPROM空间完全重合，这样就能实现用户对自己程序空间进行修改和控制。然后用户需要开发ISP程序和AP程序，由ISP程序负责AP程序的更新和引导。ISP程序默认在0地址，AP程序由于地址不是从0开始的，在使用Keil开发时需要在`BL51 Locate`中配置`Code Range`，在`Target`中配置`Off-chip Code memory`，


# OTA
基于lora，配合IAP功能，完成OTA功能。

