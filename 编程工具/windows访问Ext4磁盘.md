# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [在Windows下读取Ext4分区](https://blog.csdn.net/wdjhzw/article/details/38519695)
> [推荐一款Windows下读取 Linux文件系统Ext4的最佳软件 Paragon ExtFS](https://blog.csdn.net/xlin0208/article/details/40425807)
> [api-ms-win-crt-runtime-|1-1-0.dll丢失的两种解决方法](https://blog.csdn.net/csdnwr/article/details/80980335)

# Ext2Fsd
Ext2Fsd开源免费，最新版本Ext2Fsd-0.69，选中磁盘右键加载（F10），
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190225160543707.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
将F盘2GB文件拷贝，实测速度为41.5MB/s（i3-8100 + USB2.0）。

# 性能监测
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190225161145392.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# Paragon ExtFS
Win7首先安装KB2999226补丁程序，安装Paragon ExtFS 4.2，用法和Ext2Fsd差不多，也虚拟成了一个本地磁盘。
