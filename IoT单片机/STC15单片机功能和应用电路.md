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
STC15W4K48S4串口有4个，但是SOP28的封装上就只有两个串口，其中串口2只能固定在3，4脚，串口1可以切换3个位置，
![59](https://img-blog.csdnimg.cn/20210328185945285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
ADC配置，通过`P1ASF`使能通道，`ADC_CONTR`的位也和STC8A不一样，
![398](https://img-blog.csdnimg.cn/2021040617005223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)



