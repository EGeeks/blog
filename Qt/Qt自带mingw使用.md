# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [MinGW 编译libevent](https://blog.csdn.net/wo_Niu123/article/details/85719737)
> [MinGW怎么安装pthread库呢](https://bbs.csdn.net/topics/370025334)
> [Libev on Windows](https://stackoverflow.com/questions/8042796/libev-on-windows)

# 命令行中使用
安装Qt的时候，安装了mingw，所以就不单独安装了，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190305191231747.png)
将这个路径添加到Path，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190305191417948.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# msys
使用`mingw-get-setup.exe`安装msys，修改`C:\MinGW\msys\1.0\etc\fstab`，注意路径的斜杠，msys不需要配置环境变量
```shell
# Win32_Path				Mount_Point
#-------------------------------------	-----------
#C:/MinGW				/mingw
C:/Qt/Qt5.9.8/Tools/mingw530_32				/mingw
```
测试，ls命令并没有把mount目录显示出来，但是gcc已经可以使用了，
```shell
zc@DESKTOP-KVKC06A ~$ ls /
bin  etc  home  m.ico  msys.bat  msys.ico  postinstall  sbin  share
zc@DESKTOP-KVKC06A ~$ mount
C:\Qt\Qt5.9.8\Tools\mingw530_32 on /mingw type user (binmode)
C:\Users\zc\AppData\Local\Temp on /tmp type user (binmode,noumount)
C:\MinGW\msys\1.0 on /usr type user (binmode,noumount)
C:\MinGW\msys\1.0 on / type user (binmode,noumount)
c: on /c type user (binmode,noumount)
d: on /d type user (binmode,noumount)
e: on /e type user (binmode,noumount)
z: on /z type user (binmode,noumount)
zc@DESKTOP-KVKC06A ~$ cd /mingw/
bin/              etc/              include/          libexec/          opt/
build-info.txt    i686-w64-mingw32/ lib/              licenses/         share/
zc@DESKTOP-KVKC06A ~$ gcc -v
Using built-in specs.
COLLECT_GCC=C:\Qt\Qt5.9.8\Tools\mingw530_32\bin\gcc.exe
COLLECT_LTO_WRAPPER=C:/Qt/Qt5.9.8/Tools/mingw530_32/bin/../libexec/gcc/i686-w64-mingw32/5.3.0/lto-wrapper.exe
Target: i686-w64-mingw32
Configured with: ../../../src/gcc-5.3.0/configure --host=i686-w64-mingw32 --build=i686-w64-mingw32 --target=i686-w64-mingw32 --prefix=/mingw32 --with-sysroot=/c/mingw530/i686-530-posix-dwarf-rt_v4-rev0/mingw32 --with-gxx-include-dir=/mingw32/i686-w64-mingw32/include/c++ --enable-shared --enable-static --disable-multilib --enable-languages=c,c++,fortran,lto --enable-libstdcxx-time=yes --enable-threads=posix --enable-libgomp --enable-libatomic --enable-lto --enable-graphite --enable-checking=release --enable-fully-dynamic-string --enable-version-specific-runtime-libs --disable-sjlj-exceptions --with-dwarf2 --disable-isl-version-check --disable-libstdcxx-pch --disable-libstdcxx-debug --enable-bootstrap --disable-rpath --disable-win32-registry --disable-nls --disable-werror --disable-symvers --with-gnu-as --with-gnu-ld --with-arch=i686 --with-tune=generic --with-libiconv --with-system-zlib --with-gmp=/c/mingw530/prerequisites/i686-w64-mingw32-static --with-mpfr=/c/mingw530/prerequisites/i686-w64-mingw32-static --with-mpc=/c/mingw530/prerequisites/i686-w64-mingw32-static --with-isl=/c/mingw530/prerequisites/i686-w64-mingw32-static --with-pkgversion='i686-posix-dwarf-rev0, Built by MinGW-W64 project' --with-bugurl=http://sourceforge.net/projects/mingw-w64 CFLAGS='-O2 -pipe -I/c/mingw530/i686-530-posix-dwarf-rt_v4-rev0/mingw32/opt/include -I/c/mingw530/prerequisites/i686-zlib-static/include -I/c/mingw530/prerequisites/i686-w64-mingw32-static/include' CXXFLAGS='-O2 -pipe -I/c/mingw530/i686-530-posix-dwarf-rt_v4-rev0/mingw32/opt/include -I/c/mingw530/prerequisites/i686-zlib-static/include -I/c/mingw530/prerequisites/i686-w64-mingw32-static/include' CPPFLAGS= LDFLAGS='-pipe -L/c/mingw530/i686-530-posix-dwarf-rt_v4-rev0/mingw32/opt/lib -L/c/mingw530/prerequisites/i686-zlib-static/lib -L/c/mingw530/prerequisites/i686-w64-mingw32-static/lib -Wl,--large-address-aware'
Thread model: posix
gcc version 5.3.0 (i686-posix-dwarf-rev0, Built by MinGW-W64 project)
```

