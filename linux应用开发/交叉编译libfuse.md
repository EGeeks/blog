# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [github libfuse](https://github.com/libfuse/libfuse)
> [Meson官网关于Cross compilation](http://mesonbuild.com/Cross-compilation.html)
> [linux 编译jsoncpp](https://blog.csdn.net/u012459903/article/details/80987012)
> [meson安装](https://blog.csdn.net/syx3239/article/details/83038132)
> [meson+ninja编译libfuse-3.2.3](https://blog.csdn.net/liny000/article/details/80934202)
> [使用 meson 编译代码](https://blog.csdn.net/caspiansea/article/details/78848021)
> [fuse-2.9.0编译 安装到 板子上](https://blog.csdn.net/ypist/article/details/7644060)
> [嵌入式 linux 基于fuse 的 exfat 文件系统实现](https://blog.csdn.net/ternence_hsu/article/details/54343775)
> [交叉编译中的build、host和target](https://www.cnblogs.com/electron/p/3548039.html)

# 方法
libfuse到3.0.0版本以后就是用meson编译了，需要研究meson的交叉编译方法，我很烦这些自己造轮子的人，我也很看不起这样的人，唯技术论，做成一件事光靠技术绚丽是不够的，总之，垃圾，轮子学了一堆，程序写的一坨shit，有p用。
用python来编译c语言，干，开发c语言我还得装个python？等等python2还是python3，干，没事做天天瞎jb造轮子。
来来来，我们来看看官网上的meson的交叉编译方法，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190503173347617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
这个图只是一部分，我交叉编译一下得搞这么多东西，干，什么事情都要扯到python吗？我倒想看看10年后python能不能一直笑道最后，还是c语言能笑到最后。
我就下载一个2.x的最后版本2.9.7，通用的`./configure`，舒服，抽空我把2.x的编译系统移植到3.x上，除非超过50%的开源项目都用meson，否则这辈子都不会去学这个轮子。
修改`include/fuse_kernel.h`line91，否则编译不过，
```c
#ifdef __linux__
#include <linux/types.h>
#else
#include <stdint.h>
#define __u64 uint64_t
#define __s64 int64_t
#define __u32 uint32_t
#define __s32 int32_t
#define __u16 uint16_t
#endif
```
记住这个时间，2019-05-03 17:29:20，看我下次遇到meson是什么时候。
