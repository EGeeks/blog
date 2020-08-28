# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Qt编写输入法V2018超级终结版](https://blog.csdn.net/feiyangqingyun/article/details/82812426)
> [Qt5软键盘实现中文拼音输入法](https://blog.csdn.net/onlyshi/article/details/78408000)
> [QT5的软键盘输入法实现](https://blog.csdn.net/tracing/article/details/50617571)
> [QT5自定义中英文虚拟键盘](https://blog.csdn.net/ZuoYueXian/article/details/84142684)
> [Qt5.7以上调用虚拟键盘(支持中文)，以及源码修改(可拖动，水平缩放)](https://blog.csdn.net/i7891090/article/details/76040368)
> [QT5.7 调用虚拟键盘并且添加中文（mingw）](https://blog.csdn.net/zonghengzhongguo/article/details/52956149)
> [QT虚拟键盘中拼音输入法的使用](https://blog.csdn.net/cqltbe131421/article/details/69951924)
> [如何控制qt自带的虚拟键盘？](http://www.qtcn.org/bbs/read-htm-tid-63420.html)
> [No.02 简易软键盘 - 支持中文输入](https://blog.csdn.net/wu9797/article/details/79018689)
> [QT之全平台虚拟软键盘](https://blog.csdn.net/wzs250969969/article/details/78418725)
> [QT5的软键盘输入法实现](https://blog.csdn.net/tracing/article/details/50617571)
> [Qt libqevdevtouchplugin.so插件的改写](https://www.it610.com/article/5674344.htm)
> [module "QtQuick" is not installed](https://www.cnblogs.com/huiz/p/7017040.html)
> [Qt 5.9 qml 使用自带虚拟键盘](https://blog.csdn.net/a844651990/article/details/79032650)
> [linux下qt虚拟键盘](https://blog.csdn.net/gaibian_one/article/details/78807072)
> [arm开发板上使用qt5.8虚拟键盘（支持中文）](https://blog.csdn.net/u012391388/article/details/78485365)

# Visual keyboard
```shell
qe@ubuntu:~/program/qt-everywhere-opensource-src-5.9.6/t2080-2.0/plugins/platforminputcontexts$ powerpc64-fsl-linux-readelf -d libqtvirtualkeyboardplugin.so 

Dynamic section at offset 0xbdc88 contains 30 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Quick.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Gui.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Qml.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Network.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Core.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libpthread.so.0]
 0x0000000000000001 (NEEDED)             Shared library: [libstdc++.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [libm.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [libgcc_s.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000c (INIT)               0xcdeb8
 0x000000000000000d (FINI)               0xcded0
 0x000000006ffffef5 (GNU_HASH)           0x148
 0x0000000000000005 (STRTAB)             0x28d0
 0x0000000000000006 (SYMTAB)             0x188
 0x000000000000000a (STRSZ)              15374 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000003 (PLTGOT)             0xd2808
 0x0000000000000002 (PLTRELSZ)           6888 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x14a00
 0x0000000070000000 (PPC64_GLINK)        0x4da38
 0x0000000000000007 (RELA)               0x6918
 0x0000000000000008 (RELASZ)             57576 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000006ffffffe (VERNEED)            0x6828
 0x000000006fffffff (VERNEEDNUM)         6
 0x000000006ffffff0 (VERSYM)             0x64de
 0x000000006ffffff9 (RELACOUNT)          1979
 0x0000000000000000 (NULL)               0x0
```
部署，
```shell
export QT_IM_MODULE=qtvirtualkeyboard
export QML2_IMPORT_PATH=$QTDIR/qml # qrc:///QtQuick/VirtualKeyboard/content/InputPanel.qml:30:1: module "QtQuick" is not installed
root@t2080rdb:~# ls qt/ 
lib/     plugins/ qml/     
root@t2080rdb:~# ls qt/plugins/
generic/               imageformats/          platforminputcontexts/ platforms/                     
root@t2080rdb:~# ls qt/plugins/imageformats
libqsvg.so # qrc:/QtQuick/VirtualKeyboard/content/styles/default/style.qml:953:22: QML Image: Error decoding: qrc:/QtQuick/VirtualKeyboard/content/styles/default/images/selectionhandle-bottom.svg: Unsupported image format
root@t2080rdb:~# ls qt/plugins/platforminputcontexts/
libqtvirtualkeyboardplugin.so
root@t2080rdb:~# ls qt/qml                           
Qt                 QtCharts           QtLocation         QtPositioning      QtQml              QtQuick.2          QtSensors          QtWebChannel       builtins.qmltypes
QtBluetooth        QtGamepad          QtNfc              QtPurchasing       QtQuick            QtScxml            QtTest             QtWebSockets
root@t2080rdb:~# ls qt/lib
fonts                    libQt5Gui.so.5           libQt5PrintSupport.so.5  libQt5Quick.so.5         libQt5Sql.so.5           libQt5Widgets.so.5       libiconv.so
libQt5Core.so.5          libQt5Network.so.5       libQt5Qml.so.5           libQt5SerialPort.so.5    libQt5Svg.so.5 # libQt5Svg.so.5: cannot open shared object file: No such file or directory          libcharset.so
```
# 问题
- 没有GPU，偶发卡死（问题很大，需要一个心跳，卡死时自动重启）
- 键盘第一次加载很慢
- 键盘按一次，重复输入多个数
- 无法切割窗口，键盘全屏覆盖，不知道输入了多少数据。

```shell
# 启动时
JIT is disabled for QML. Property bindings and animations will be very slow. Visit https://wiki.qt.io/V4 to learn about possible solutions for your platform.
# 运行时
This plugin does not support setting window masks
```

