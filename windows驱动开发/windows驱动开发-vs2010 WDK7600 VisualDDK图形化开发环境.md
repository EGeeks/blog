# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 安装
 1. 安装vs2010，双击安装即可。
 2. 安装WDK7600，双击安装，中间会联网下载安装.NET3.5等，保持联网。进入ISO，进入Debugger目录安装x86 32bit的windbg，VisualDDK用32bit的windbg。WDK7600的wdf版本是1.9。
 3. 安装VisualDDK，下载1.5.6版本，最新的我这测试有bug。

# 新建工程
按照vs2010的新建工程的方式新建，选择VisualDDK选项卡。
![这里写图片描述](https://img-blog.csdn.net/20180614190539711?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
弹出工程配置窗口，默认即可，注意DDK/WDK version和Debug/Release flags。
![这里写图片描述](https://img-blog.csdn.net/20180614190856798?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 开发
添加或者删除源文件。
![这里写图片描述](https://img-blog.csdn.net/20180614192549676?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
编译设置，在工程目录下就生成sys驱动文件。
![这里写图片描述](https://img-blog.csdn.net/20180614192511199?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
# 工程源文件中有红色下滑波浪线
在工程属性中添加包含目录。
![这里写图片描述](https://img-blog.csdn.net/20180616184919885?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
这种方式更通用，
![这里写图片描述](https://img-blog.csdn.net/2018071200064232?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# vs2010本地帮助
vs2010之后的就没有了，开始菜单打开帮助管理器，选择优先使用本地帮助
![153](https://img-blog.csdnimg.cn/20190711145831138.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
