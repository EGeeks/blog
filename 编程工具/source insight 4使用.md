# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [使用 SourceInsight 的第四大理由，关键中的关键](https://blog.csdn.net/weixin_42876465/article/details/108439775)

# 关系图
关系图中的函数可以锁定，只要你不主动刷新，关系图就不会变化。并且还可以多看一个（锁定后，首先找下一个需要查看的函数或变量，然后点击工具栏关系图按钮关闭再打开，即可看2个关系图。

# 输入卡顿
改大`Update recovery file`的值，取消`Background synchronization`，
![421](https://img-blog.csdnimg.cn/24c328bc217e44ffb8428a1cc48aacfb.PNG?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiJ6YGN54yq,size_20,color_FFFFFF,t_70,g_se,x_16)

## source insight checking for modified files
导致软件很卡，`optinos->preference->files`，将其中的`Reload externally modified files in background`去掉。
> [source insight checking for modified files 问题解决](https://blog.csdn.net/qq_33929118/article/details/53330957)

![340](https://img-blog.csdnimg.cn/20200922145919917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)

# 添加标准库头文件
菜单Project->Import External Symbols for Current Project
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215111115740.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# Search Project找不到
新编辑了，
```c
//#define CONFIG_ID_EEPROM
```
用Search Project查找`//#`竟然找不到，fuck，必须用Search Files，小心了，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190308154719167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

