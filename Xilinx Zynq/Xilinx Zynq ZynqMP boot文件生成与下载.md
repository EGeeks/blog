# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 使用Petalinux生成boot文件
3合1是指包含了fsbl，u-boot和FPGA bitstream，2合1只有fsbl，u-boot，一般采用3合1这种方式，
```bash
# 3合1
$ petalinux-package --boot --fpga images/linux/pcierc_wrapper_gen2_x4.bit --u-boot --force
# 2合1
$ petalinux-package --boot --u-boot --force
```

# 使用SDK生成boot文件


# 使用Vivado下载boot文件
**让芯片进入Jtag模式**，打开Vivado Hardware Manager，我这里直接使用Auto Connect，也可以从Recent Targets选择，或者用Open New Target手动添加，一路默认操作就可以添加完成。
![371](https://img-blog.csdnimg.cn/20201222215755229.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
连接上之后，显示如下界面，在红线处右键菜单添加配置Flash芯片，
![372](https://img-blog.csdnimg.cn/20201222220052467.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
根据硬件设计选择你的芯片，
![373](https://img-blog.csdnimg.cn/20201222220313258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
遇到这个对话框，选OK，
![374](https://img-blog.csdnimg.cn/20201222220400980.png)
选择生成的boot文件和fsbl文件，点击OK，
![375](https://img-blog.csdnimg.cn/20201222220450230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
等待下载完毕，切换回QSPI启动模式，从串口可以看到u-boot的打印，说明下载成功。
![376](https://img-blog.csdnimg.cn/20201222221200459.png)
如果下载失败，可尝试复位板卡，手动连接下载器，多尝试几次。

# 使用SDK下载boot文件
点击工具栏下载Flash按钮，
![377](https://img-blog.csdnimg.cn/202012231004264.png)
配置参数，点击Program开始下载，
![378](https://img-blog.csdnimg.cn/20201223100613467.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)







