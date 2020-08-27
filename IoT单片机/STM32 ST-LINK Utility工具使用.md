# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 下载
> [STM32 ST-LINK Utility v4.3.0](https://download.csdn.net/download/zhu_zhu_2009/10852990)
> [Jlink接口的Jtag和SWD接口定义](https://blog.csdn.net/u014124220/article/details/50829713)

# 安装
双击默认安装（一直点下一步）即可，若没有安装下载器驱动，则会弹出安装下载器驱动的窗口，也是按照默认安装。

# 下载固件
（1）连接将下载器连接板卡，连接关系如下，

> [正版ST-link/V2引脚定义和注意事项](https://blog.csdn.net/xinghuanmeiying/article/details/78026561)

下图是从淘宝上买的ST-LINK，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215200449220.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
但是这种下载器只有SWD下载接口，没有JTAG接口，官方的下载器，两种都支持，SWD和JTAG共用管脚，管脚的对应关系如下，对应关系通过第三列和第四列对比出来，SWD连接4根线，GND，SWDIO，SWCLK，NRST，VDD可以不用接，板卡采用外部供电。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215201219718.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
引脚位置图，
![320](https://img-blog.csdnimg.cn/20200629164031293.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
查看板卡上下载器接口，如下图所示，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215201936257.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
板卡上的信号有TMS，TCK，TDI，TDO，TRST，NRST，对应上表，SWDIO（TMS），SWCLK（TCK），NRST（NRST），GND（板卡地）。将所有SWD信号连接好之后，上电，将下载器插入电脑USB，进入下一步。
（2）从桌面打开STM32 ST-LINK Utility软件，点击菜单Target->Settings，如下图所示，确认是SWD接口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215202948988.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
点击下图1处按钮，如果配置正确，则会显示连接的板卡信息，如2所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215195817189.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
（3）点击2号按钮，打开编译好的hex文件，如果装载了hex，可以在2号上点击右键，在弹出的菜单里更换hex文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215203225828.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
（4）点击菜单Target->Program & Verify，弹出下面的窗口，点击1号按钮，开始下载。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181215203523115.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
（5）下载完之后，检测程序是否工作。
