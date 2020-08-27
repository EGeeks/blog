# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [STC官网](http://www.stcmcu.com/index.htm)

# 方法
首先下载安装Keil C51，百度下载即可，破解。到官网下载STC-ISP软件，现在2019-05-12，我下载的是`stc-isp-15xx-v6.86R`版本，下面在Keil中添加STC的器件库，点击图中Keil仿真设置，有一个按钮，按照提示操作，选中安装目录`C:\Keil`，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190512220620885.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
新建工程，`Project > New uVision Project`，选则`STC MCU Database`，保持工程文件，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190512221925264.png)
这里没有STC8A4K，选择STC8A8K近似，然后不选择加入默认的startup汇编，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190512222055714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
配置工程，点击1，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190512222959521.png)
选中生成HEX文件，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190512222603375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
获取STC8系列的头文件，点击保存文件，
![在这里插入图片描述](https://img-blog.csdnimg.cn/201905122228292.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
我的工程文件在c51中，源代码放在src，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190512223327916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
新建`main.c`，点击2，添加`main.c`到工程中，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190512222959521.png)
添加`main.c`到工程中，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190512223743536.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
编译即可。

# 下载
板子先保持断电状态，然后点击ISP下载，然后板子上电，会自动完成下载过程。在硬件选型中，有一个下载时是否擦除EEPROM选项，如果不想擦除保留的参数，则不勾选这个选项。

# 其他
STC的官网山寨风太浓了，开发工具也是很简单粗暴，但这么多年了，一直还有维护更新，说明人家活的还不错，至少技术人员还在一直维护，挺好。

