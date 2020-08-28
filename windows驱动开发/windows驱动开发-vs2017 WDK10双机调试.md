# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> Getting started with Windows drivers [Provision a computer for driver deployment and testing (WDK 10)](https://docs.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/provision-a-target-computer-wdk-8-1)

# 配置目标计算机
调试机是win7 x64平台，建议先开启网络共享，公用网络上（也肯是专用网络，域，到网络与共享中心查看）。
![这里写图片描述](https://img-blog.csdn.net/20180623152417603?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在防火墙的高级设置中可以看到，网络发现和文件与打印机共享里关于公用网络的设置的远程地址为本地子网，这儿根据需求，如果不是一个网段的，需要设置成任意IP。
![这里写图片描述](https://img-blog.csdn.net/20180706192141405?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
安装WDK10测试工具，安装包在WDK10安装目录下，我的是，
```
C:\Program Files (x86)\Windows Kits\10\Remote\x64\WDK Test Target Setup x64-x64_en-us.msi
```
配置vs2017，点击Driver > Test > Configure Devices，点击Add new device。
![这里写图片描述](https://img-blog.csdn.net/20180706195136896?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
中间调试机可能会重启几次。
