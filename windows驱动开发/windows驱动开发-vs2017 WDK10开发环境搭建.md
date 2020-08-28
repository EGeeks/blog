# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 下载安装包
微软官网下载安装包，然后下载vs2017离线安装包，网上说先安装证书，我忘记了，但是也没有问题，我是win7 x64，应该是我已经有了这些证书。
```shell
D:\vs_enterprise__111975078.1529511260.exe --layout D:\vs2017 --lang zh-CN en-US
```
![这里写图片描述](https://img-blog.csdn.net/20180623153407736?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
双击离线目录setup.exe安装vs2017，选择C++桌面开发。
配置下载缓存路径，
![这里写图片描述](https://img-blog.csdn.net/20180621123120447?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
安装完成之后到帮助菜单里注册。
Visual Studio 2017 Enterprise NJVYC-BMHX2-G77MM-4XJMR-6Q8QF
Visual Studio 2017 Professional KBJFW-NXHK6-W4WJM-CRMQB-G3CDH 
官网下载WDK10或EWDK10，按默认安装。

# 配置Git

由于我这里用了外部方法建了Git，所以这儿点击添加，来配置Git。
![这里写图片描述](https://img-blog.csdn.net/20180622111824708?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
从vs代码编辑窗口的底部有Git的提交等信息。
![这里写图片描述](https://img-blog.csdn.net/20180621234650789?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 项目设置

设置项目属性，设置输出可执行文件，
![这里写图片描述](https://img-blog.csdn.net/2018070616394334?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
设置头文件路径，
![这里写图片描述](https://img-blog.csdn.net/20180706164023487?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
设置库路径，
![这里写图片描述](https://img-blog.csdn.net/20180706164057899?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
设置依赖：同一个解决方案放下有很多项目，这里可以在解决方案资源管理器空白处，右键菜单->添加->新建解决方案文件夹，对项目进行分类管理。
如果有一个应用调用另一个库工程，这时候需要到工程属性中配置附加包含目录和附加链接库，vs提供了一个快捷方式建立项目依赖，简化配置过程，在项目引用里添加依赖的项目即可。
![这里写图片描述](https://img-blog.csdn.net/20180622152120724?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
Wpp Tracing此处配置打开，否则编译不过。
![这里写图片描述](https://img-blog.csdn.net/20180706164112461?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
其次配置Scan Configuration Data
![这里写图片描述](https://img-blog.csdn.net/2018070617100074?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
以上针对Debug/Release建立通用配置，下面可新建解决方案配置，分别适配win10和win7等。
![这里写图片描述](https://img-blog.csdn.net/20180706165143694?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
针对Win7驱动的特殊设置，这里只有Desktop类型，
![这里写图片描述](https://img-blog.csdn.net/20180706175634212?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
WDF版本设置为1.11，
![这里写图片描述](https://img-blog.csdn.net/20180706175706906?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
# 常见错误
编译错误，
```shell
Inf2Cat error -2: "Inf2Cat, signability test failed." Double click to see the tool output.
Inf2Cat Tool Output:
..........................
Signability test failed.

Errors:
22.9.7: DriverVer set to a date in the future (postdated DriverVer not allowed) in xxxx.inf.

Warnings:
None
```
刚好时间到了凌晨出了错误，参考博客[VS2012驱动项目时间戳验证失败](https://blog.csdn.net/blog_index/article/details/18084463)，解决办法很简单，在项目属性里设置，
![这里写图片描述](https://img-blog.csdn.net/20180708002015906?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


