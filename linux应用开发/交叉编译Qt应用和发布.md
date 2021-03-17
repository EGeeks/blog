# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Linux下打包发布Qt应用程序](https://blog.csdn.net/qq_27350133/article/details/83445258)
> [关于qt缺少xcb问题终极解决办法](https://loongson.cloud.csdn.net/p/droexo04)

# 交叉编译
```shell
$ source ~/program/QorIQ-SDK-V2.0-20160527-yocto/fsl-qoriq/2.0/ppc64/environment-setup-ppc64e6500-fsl-linux
$ export PATH=/home/qe/program/qt-everywhere-opensource-src-5.9.6/t2080-2.0/bin:$PATH
$ export LD_LIBRARY_PATH=/home/qe/program/qt-everywhere-opensource-src-5.9.6/t20802.0/lib:$LD_LIBRARY_PATH
$ qmake
$ make
$ make INSTALL_ROOT=. install
```
# 发布
```shell
qe@ubuntu:~/project/qt/recorder$ powerpc64-fsl-linux-readelf -d recorder

Dynamic section at offset 0xab2870 contains 31 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libQt5PrintSupport.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Widgets.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Gui.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5SerialPort.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Sql.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libQt5Core.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libpthread.so.0]
 0x0000000000000001 (NEEDED)             Shared library: [libstdc++.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [libm.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [libgcc_s.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000c (INIT)               0x10ac2b00
 0x000000000000000d (FINI)               0x10ac2b18
 0x000000006ffffef5 (GNU_HASH)           0x10000220
 0x0000000000000005 (STRTAB)             0x10004480
 0x0000000000000006 (SYMTAB)             0x10000250
 0x000000000000000a (STRSZ)              21756 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x10ace798
 0x0000000000000002 (PLTRELSZ)           14112 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x1000f370
 0x0000000070000000 (PPC64_GLINK)        0x100d5608
 0x0000000000000007 (RELA)               0x1000a060
 0x0000000000000008 (RELASZ)             21264 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000006ffffffe (VERNEED)            0x10009f00
 0x000000006fffffff (VERNEEDNUM)         9
 0x000000006ffffff0 (VERSYM)             0x1000997c
 0x0000000000000000 (NULL)               0x0
```

