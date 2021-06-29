# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 设置IP DNS
DHCP模式不支持一个网卡配多个IP，配置固定IP，
![69](https://img-blog.csdnimg.cn/20210527235021400.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
![68](https://img-blog.csdnimg.cn/20210527235010567.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)


# 命令行杀掉进程
```bash
taskkill /im /f nginx.exe
```

# CAJ转PDF
speedpdf，大小限制也是2MB，干。。。

# XP Chrome 您的时钟快了
安装KB931125补丁，方法：双击rootsupd.exe，一闪而过就安装好了。

# pci内存控制器和sm总线控制器驱动安装
参考[Win7装机时PCI简易通讯控制器叹号处理](https://blog.csdn.net/salutlu/article/details/18142853),选择系统设备，Intel即可。

# 启用或关闭Windows功能
搜索打开控制面板，选择程序，选择启用或关闭Windows功能。

# DHCP服务器
安装DHCP服务器，
```cpp
设置 > 应用和功能 > 可选功能 > 添加功能 > RSAT: DHCP服务器工具
```
Windows管理工具打开DHCP程序，添加一个DHCP服务即可。

# 休眠按钮与快速启动
![20](https://img-blog.csdnimg.cn/20201011124416739.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
# 禁止QQ弹出腾讯网新闻
![6](https://img-blog.csdnimg.cn/20200920132553564.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70#pic_center)
# 禁用驱动程序强制签名
Win7开机按F8，可临时禁用驱动程序强制签名，Win10，
```cpp
设置 > 更新与安全 > 恢复 > 立即重启 > 疑难解答 > 高级选项 > 启动设置 > 重启 > 7或F7
```

# 触摸板双指滚动方向相反
> [Win10系统屏幕内容与鼠标/触摸板同向滚动的技巧](http://www.xitongtiandi.net/wenzhang/win10/31061.html)

我的电脑的Windows设置里无法设置滚动方向，不知道是不是因为我更新了Win10导致的，或者是因为我没注册？我一直在未注册状态，但并不影响我干什么事情。。。

# Be4
密钥被禁之后，删除`C:\Users\xx\AppData\Roaming\Scooter Software\Beyond Compare 4`下的所有内容

# Win10在系统安装
打算在Win10上给另一个硬盘装系统，选择在系统安装是因为调试机在系统安装界面不识别鼠标键盘（无语），结果默认装成了双系统，新系统的引导项装入老系统，老系统硬盘拔出后，新系统竟然无法启动，所以这种方法不行，卸载方法见下文。另外，在系统安装时要双机source文件下的setup，否则无法选择安装位置，导致原有系统盘被覆盖。

# 双系统删除方法
Win7用Win+R，Win10直接在任务栏搜索窗口输入msconfig，打开系统配置，在引导选项卡删除系统引导项，
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL3d3dy54aXRvbmd6aGlqaWEubmV0L3VwbG9hZHMvYWxsaW1nLzE2MTEwOS83My0xNjExMFo5M0o0LTUwLXdhdGVyLmpwZw?x-oss-process=image/format,png)
# NTFS硬盘在Linux挂载失败
挂载时报错`The disk contains an unclean file system (0, 0). Metadata kept in Windows ca`，在电源按钮功能设置界面，关掉快速启动。
![311](https://img-blog.csdnimg.cn/2020053015551449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# NTFS硬盘在Linux挂载成只读文件系统
在Linux下执行，
```bash
$ mount -t ntfs-3g -o remove_hiberfile /dev/sda4 ./win10
```

# ping
> [解除win10禁ping方法](https://blog.csdn.net/wudinaniya/article/details/80956158)

Win10：设置，更新和安全，Windows安全中心，防火墙和网络保护，关闭防火墙，或者点击高级设置，设置入站规则，
![136](https://img-blog.csdnimg.cn/20200323231933553.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# RAID
想把虚拟机文件从笔记本拷贝到台式机，提高编译速度，手头两个小SSD，每个120GB，但是虚拟机130GB，所以发挥余热（穷）以及不浪费快递的时间（穷），需要合二为一，打开windows磁盘管理，
![这里写图片描述](https://img-blog.csdn.net/20180701234855221?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
磁盘0是系统盘，120GB，买机器时一起买的SSD，1和2是我的两个SSD，嗯，我有3个120GB的SSD（穷），将1和2删除分区变成未分配状态，右键菜单出现4个新建选项，我选择跨区卷，将两个盘线性相连，带区卷是RAID-0，并行相连，可提高速度，镜像卷为RAID-1，RAID-5至少需要3块盘。一件下一步到底，即可创建出一个230GB的大盘。
