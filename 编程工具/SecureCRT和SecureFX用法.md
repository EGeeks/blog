# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 安装
> [SecureCRT & SecureFX](https://blog.csdn.net/u012867916/article/details/77430393)
> [SecureCRT脚本(VBS)运行](https://blog.csdn.net/u011329967/article/details/80210983)
> [SecureCRT中python脚本编写学习指南](https://www.cnblogs.com/zhaoyujiao/p/4908627.html)

# SecureCRT脚本自动发送stop
菜单`Script > Run`或`Script > Cancel`，可添加到button快捷键。
```vb
#$language = "VBScript"
#$interface = "1.0"

'================================================================
Sub Main
'================================================================
'crt.Screen.Synchronous = True
'--------------------------------
'-------------------------------------------
Dim var1
var1=1000
Const delay1 = 1000
Do 
'--------------------------------------------
'crt.Screen.Send "stop" & chr(13)
crt.Screen.Send "stop"
crt.Sleep delay1 
'--------------------------------------------
if var1=1 then 
exit do
end if
var1=var1-1 
Loop
'crt.Screen.Synchronous = False
End Sub

```
# SecureCRT buttonbar
通过视图菜单打开Button Bar和Command Window，Button Bar可以指定快捷键，Command Window可以缓存输入。
![21](https://img-blog.csdn.net/20180625142243363?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# SecureCRT telnet显示颜色
![44](https://img-blog.csdnimg.cn/20181226171849570.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# SecureCRT显示乱码
设置Session Options，设置字符集UTF-8，显示汉字。
![43](https://img-blog.csdnimg.cn/20181226171825276.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# SecureCRT ubuntu telnet第一次自动登录失败
取消这个选项，
![46](https://img-blog.csdnimg.cn/20181228103501385.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
> [“回车”（Carriage Return）和“换行”（Line Feed）](https://blog.csdn.net/wanghuiqi2008/article/details/8069775)

0x0D（asc码是13） 指的是“回车”   \r是把光标置于本行行首
0x0A（asc码是10） 指的是“换行”    \n是把光标置于下一行的同一列
0x0D + 0x0A           回车换行               \r\n把光标置于下一行行首  
\n是换行，英文是line feed，ASCII码是0x0A。
\r是回车，英文是carriage return ,ASCII码是0x0D。
Unix系统里，每行结尾只有“<换行>”，即"\n"
Windows系统里面，每行结尾是“<回车><换行>”，即“\r\n”（此处顺序一定不能颠倒了！！！）
Mac系统里，每行结尾是“<换行>”，即"\n"

# SecureCRT X/Y/Zmodem
## 下载
u-boot下执行`loadx`，linux下执行`lrz -beyX <file name>`，点击菜单`Send Xmodem`，
![281](https://img-blog.csdnimg.cn/20200222164117902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
## 上传
linux执行`lsz -beyX <file name>`，点击菜单`Receive Xmodem`。

## Xmodem operation was canceled by remote peer
使用Xmodem命令没有加`<file name>`参数导致的，但是Ymodem执行`lrz -bey --ymodem`不加`<file name>`参数也可以用。

## 配置X/Y/Zmodem默认工作目录
可以设置Upload和Download目录，不用每次都费事点很多次文件夹。
![326](https://img-blog.csdnimg.cn/20200805191818503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# SecureCRT TFTP server
SecureCRT集成了TFTP服务端，不需要单独安装软件了，很方便，参考`配置X/Y/Zmodem默认工作目录`图。

# SecureFX FTP
连接FTP服务器，点击Quick Connect，不是新建连接，

# SecureFX显示乱码
设置Session Options不管用，到路径`C:\Users\zc\AppData\Roaming\VanDyke\Config`的文件夹`C:\Users\zc\AppData\Roaming\VanDyke\Config\Sessions`中，找到自己的会话，
![223](https://img-blog.csdnimg.cn/20191115171149781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
打开`ini`文件，`D:"Filenames Always Use UTF8"=00000000`修改为`D:"Filenames Always Use UTF8"=00000001`，断开重连即可。
