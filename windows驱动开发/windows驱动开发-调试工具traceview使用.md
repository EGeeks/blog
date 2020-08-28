# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 利用traceview对驱动进行调试
Debugview过时了，且不能在x64是跑，traceview是WDK安装包中附带的调试工具，是取代DbgView的单机开发驱动工具，位于安装路径C:\WinDDK\7600.16385.1\tools\tracing\amd64中。

# 驱动添加traceview调试代码
WDK自带的例子里大部分都有traceview调试接口代码。

# traceview软件使用
选择File，Create new Log Session，
![这里写图片描述](https://img-blog.csdn.net/2018052810583068?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在弹出的对话框中选择Add Provider，
![这里写图片描述](https://img-blog.csdn.net/20180528110137414?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
选择默认的PDB调试，定位到驱动编译生成的PDB路径，选择下一步，
![这里写图片描述](https://img-blog.csdn.net/20180528110147106?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在弹出的对话框中，可以重命名Log Session Name，选择Set Flags and Level，在Level中选择Verbose。
![这里写图片描述](https://img-blog.csdn.net/20180528110157413?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
现在可以测试用traceview调试驱动，打开设备管理器，安装驱动，就可以抓取到驱动中输出的消息。
![这里写图片描述](https://img-blog.csdn.net/20180528110228116?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在Log Session上右键菜单可以保存，方便下次使用，
![这里写图片描述](https://img-blog.csdn.net/2018052811055765?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
若程序发生更改，只需先Stop Trace，移除会话Remove Log Session，再重新打开工作空间即可，或者重启软件。
![这里写图片描述](https://img-blog.csdn.net/20180528110315868?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180528110326847?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在消息显示列表控价的表头右键菜单，可以控制显示的列，增加显示模块名，文件名，函数名，打印标志与等级等等。
![这里写图片描述](https://img-blog.csdn.net/20180717203235160?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
