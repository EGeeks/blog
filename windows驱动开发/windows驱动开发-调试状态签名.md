# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

WDK8之后，微软为驱动开发提供了visual studio IDE开发环境，驱动签名也自动化了，但我暂时还没用过，下面使用WDK7600提供的工具对驱动进行签名，这个签名只能用于调试目的，Windows系统必须打开测试模式。

# 创建self根证书
```shell
C:\WinDDK\7600.16385.1\bin\amd64>makecert -r -pe -ss ZhuCeCertStore -n CN=zhuce.com ZhuCe.cer
```
![创建证书截图](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTgwNDIyMTYyMzUxNjk5?x-oss-process=image/format,png)

# 创建驱动catalog文件（对于inf文件安装的驱动）
stampinf –d的时间格式是月/日/年，月是2位，日是2位，年是4位，不足的补0。然后执行Inf2Cat。
Infcat工作目录下不相关的其他inf文件都要删除。HSAC项目中每次编译会由HSAC.inx会生成HSAC.inf，若是出错，就删掉这个无关的inf文件。
```shell
stampinf.exe -f E:\HSAC_WDF_1_2\HSAC_WDF\objfre_win7_amd64\HSAC.inf -d 10/31/2015 -v 1.0.0000.0
```
```shell
cd ..\selfsign
Inf2Cat.exe /driver:E:\HSAC_WDF_1_2\HSAC_WDF\objfre_win7_amd64 /os:Vista_x64
```
红线所标的一个斜线，WDK文档的例子是没有的，实测这个斜杠加不加都可以执行，WDK7600中的Inf2Cat需要.NET 2.0或者3.5，如果系统没有，自行下载dotNetFx35xxxx.exe安装即可。

# 签名驱动文件
```shell
C:\WinDDK\7600.16385.1\bin\selfsign>cd ..\amd64
C:\WinDDK\7600.16385.1\bin\amd64>SignTool.exe sign /v /s ZhuCeCertStore /n zhuce.com /t http://timestamp.verisign.com/scripts/timestamp.dll E:\HSAC_WDF_1_2\HSAC_WDF\objfre_win7_amd64\hsacx64.cat
```
```shell
cd ..\amd64
SignTool.exe sign /v /s ZhuCeCertStore /n zhuce.com /t http://timestamp.verisign.com/scripts/timestamp.dll E:\HSAC_WDF\HSAC_WDF\objfre_win7_amd64\hsacx64.cat
```

# 配置电脑为测试模式
以管理员方式启动cmd，用bcdedit工具。
```shell
C:\Windows\system32>bcdedit.exe -set TESTSIGNING ON
```
对于win10，参考[Win10驱动签名总结](https://blog.csdn.net/tansh4731/article/details/80017233)
```shell
bcdedit -set loadoptions DDISABLE_INTEGRITY_CHECKS
bcdedit -set TESTSIGNING ON
bcdedit -set loadoptions ENABLE_INTEGRITY_CHECKS
bcdedit -set TESTSIGNING OFF
```

# 安装self根证书
```shell
C:\Windows\system32>cd ..\..\WinDDK\7600.16385.1\bin\amd64
C:\WinDDK\7600.16385.1\bin\amd64>CertMgr.exe /add ZhuCe.cer /s /r localMachine root
```
至此，就完成了证书的安装，但仅仅是调试状态下的，换一台电脑必须从头操作一遍，一般签名需要花很多dollar和时间，在驱动调试时，完成签名是很方便的，尤其是在驱动双机调试的时候，必须有签名，否则驱动加载不了，是无法attach上去调试的。打开windows的证书管理器certmgr.msc，可以看到安装的证书。
![这里写图片描述](https://img-blog.csdn.net/20180711225933893?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 总结脚本
```shell
%WDKPATH%\bin\amd64\stampinf.exe -f .\HSAC_WDF\objfre_win7_amd64\HSAC.inf -d 10/31/2015 -v 1.0.0000.0
%WDKPATH%\bin\selfsign\Inf2Cat.exe /driver:.\HSAC_WDF\objfre_win7_amd64 /os:Vista_x64
%WDKPATH%\bin\amd64\SignTool.exe sign /v /s ZhuCeCertStore /n zhuce.com /t http://timestamp.verisign.com/scripts/timestamp.dll .\HSAC_WDF\objfre_win7_amd64\hsacx64.cat
```
签名时系统时间一定要正确，否则证书无效

# 他家的工具deso13b
xp是不需要驱动签名的，Win7 32bit打开调试模式后也不需要签名，但Win7x64需要强制签名，否则每次开机F8很麻烦，这里需要用到一个工具，deso13b.exe，软件支持
* Windows Vista 32-bit
* Windows Vista 64-bit
* Windows Server 2008 32-bit
* Windows Server 2008 64-bit
* Windows 7 32-bit
* Windows 7 64-bit
不知道现在有没有出新版的，软件打开如下，管理员权限
 1. 首先使能测试模式
 2. 选择Sign a System File
 3. 按提示输入驱动路径点击OK即可。典型的路径为C:\Windows\System32\drivers\HSAC.sys
 
# Remove Watemarks去水印
在deso13b.exe中选择Remove Watemarks，会提示一个网页，外国网页打不开，自己百度这个软件名，可以下载一个去水印工具，有x86和x64版本。
运行，输入y，重启之后，水印就会消失。方便快捷。
```shell
     Remove all Watermark on desktop, such as Test Mode/Evaluation Copy.
     Version:  0.4,  01/17/2009
     Support:  Windows Vista /Server 2008 /Windows 7,  32bit(x64)
               All Service Pack & all language of Windows.
     Author:   deepxw
     Blog:     http://deepxw.lingd.net
               http://deepxw.blogspot.com   (English)
Please right click the exe file, run as administrator, and dsiable UAC.
Do you really want to apply this patch?
(Y=Yes  /  N=No )
Y
```

# 禁用驱动程序强制签名
Win7开机按F8，可临时禁用驱动程序强制签名，Win10，
```cpp
设置 > 更新与安全 > 恢复 > 立即重启 > 疑难解答 > 高级选项 > 启动设置 > 重启 > 7或F7
```

