# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [VS2012 ddk驱动编译与虚拟机联机调试设置（vs调试驱动）](https://blog.csdn.net/mao0514/article/details/90603510)
> [用USB 3.0 + WinDbg开始你的调试之旅](https://mp.weixin.qq.com/s/rN_N2ie4mVU00zcSiGMQxg)

# 方法
通过网线调试的优点有只要插上网线，电脑可以随便放了，我在研一用无线网卡貌似也可以配置调试环境，可以用一台调试计算机可以调试多台网络内的目标计算机，而且网卡非常常见，现在的计算机已经很少有串口和1394 fire wire。至于usb，我还没研究怎么搞到适当的连接线连接两台电脑，微软的网页也懒得看了，配一种方式够用即可。
防火墙先关掉，出错了还不知道原因，网线直连不行的请搞个交换机或者路由器过来，进行双机互ping测试，貌似是两个网卡都得有自动交换功能才能直连。微软官网里有支持调试的网卡列表，网线标准网上说是CAT5或更好的线。

# 配置目标计算机

 1. 管理员模式运行cmd 开启调试模式 bcdedit /debug on 
 2. 设置网络调试接口bcdedit /dbgsettings net    hostip:192.168.1.6 port:54321 端口号Port   
 3. Number取值范围为49152到65535，此端口号在调试计算机上会用到   
 4. 将cmd中产生的口令Key拷贝出来，在调试计算机上配置WinDbg调试环境时会用到   
 5. 对于有多块网卡的目标计算机，还要设置调试网卡所在的PCI总线、设备、功能号 重启目标计算机
![这里写图片描述](https://img-blog.csdn.net/20180528103808487?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
操作记录
```shell
Microsoft Windows [版本 6.3.9600]
(c) 2013 Microsoft Corporation。保留所有权利。
C:\Windows\system32>bcdedit /debug on
操作成功完成。
C:\Windows\system32>bcdedit /dbgsettings net hostip:192.168.1.6 port:54321
Key=se79bf657uf1.335dplfrhcali.3f7us0hrxvlx9.2reuein5hf0t4
C:\Windows\system32>bcdedit /set "{dbgsettings}" busparams 2.0.0
操作成功完成。
C:\Windows\system32>
```
![这里写图片描述](https://img-blog.csdn.net/20180528103901891?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 配置调试计算机
我下载的是针对Win7的WDK7600.16385.1，这是以前MSDN疯狂找到的ISO，大四的时候瞎折腾搞到的，现在下载都去Windows硬件开发人员中心，WDK7600安装包中带的WinDbg版本太低，好像对NET调试不太支持，虽然Kernel Debug中NET选项卡，但是只有端口号Port Number输入编辑框，并没有Key的输入编辑框，联想到说NET调试至少Win8以上，想下一个新版WDK，里面肯定有新版WinDbg，上网一看，WDK都到Win10，艾玛，跟不上发展了。下了Win10的WDK，官网也找了以前Win8.1的连接，可是下不了了，不知为啥，就算了，装WDK10吧，从SDK中也可以获得WinDbg调试器，想想这会小哥还在西安出差。
在Win8以后的驱动开发，微软都集成到了Visual Studio中，不打算用IDE的，可是NET调试环境要求至少Win8，WDK7600不能编译Win8上的驱动，符号表什么也没下Win8的，那我现在配置这个环境用处不大啊，还不如直接用VS开发，配置过程微软都自动化了。
果然，新版WinDbg的NET调试选项里有key，搞上了，开始连接目标计算机，此处我有两处很恶心的小细节，第一个就是调试计算机连接目标计算机的时候，目标计算机要重启，第二个就是微软官网上说会出来以下画面：
![这里写图片描述](https://img-blog.csdn.net/20180528104225955?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
而我的是
![这里写图片描述](https://img-blog.csdn.net/20180528104247430?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
然后就卡住了，然后我把直连网线，换成了交换机，还是一样，后来发现只要点击工具栏上的break按钮就好了，真是醉哭了，囧。
2015/10/22 09:05:50 今天早上把交换机撤了，网线直连测了一下，可以调试，这样看来就确实很方便了，比串口还方便，速度也快，不是所有机器都有串口的，而且串口线的通信质量不能让人放心啊。


