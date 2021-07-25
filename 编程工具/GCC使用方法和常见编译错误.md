# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [gcc警告选项汇总](https://blog.csdn.net/qq_17308321/article/details/79979514)
> [gcc -E 预处理阶段做了哪些事？](https://blog.csdn.net/luoyir1997/article/details/82455058)
> [GCC -E选项：生成预处理文件](http://c.biancheng.net/view/2375.html)
> [Linux direct io使用](http://www.lenky.info/archives/2012/05/1660)
> [解决 array subscript has type char 错误](https://blog.csdn.net/sjygqz/article/details/106583420)
> [configure: error: newly created file is older than distributed files!](https://blog.csdn.net/kangear/article/details/48422677)
> [Linux Dynamic System Table Usage](https://mp.weixin.qq.com/s/6fcwFtVS6C52m5-mYGyQGw)

# -E
在dlopen场景中，它是从dymamic symbol table中查找该符号是否存在，以便后续进行调用。默认情况下，如果主程序中的全局函数并未被其他调用，是不会导出到dynamic symbol table中的。如果在so文件使用了该symbol，那么在dlopen时会报错。一旦遇到报错，一个是查看下是否在LD Library path中，另外一个是查看dymamic symbol table，是否存在。
```bash
$ objdump -T  <file> | grep "symbol"
```
如果是so文件要调用主程序symbol，而该symbol没有被导出，那么就需要用-E来更改默认行为，把所有全局符号表导出到dynamic symbol table中。如果LD，那么直接-E放到ldflags中。如果是GCC，那么采用-Wl,-E方式增加到CFLAGS中。

# symbolmap: 00000001: invalid section
```bash
$ sudo apt install libelf-dev
```

# configure: error: newly created file is older than distributed files!
```bash
$ hwclock --set --date="07/24/2012 12:33:22"
$ find . -name "*" -exec touch '{}' \;
```

# array subscript has type 'char'
此错误警告应防止使用负数组索引。将char改为u8，或者添加编译选项` -Wno-char-subscripts`。

# undefined reference to symbol 'pthread_rwlock_rdlock@@GLIBC_2.4
添加`-lpthread`

# error: expected ',' or '...' before 'new'
移植了一个链表实现到Qt，编译报错，思考半天，是变量名`new`和c++的关键字冲突导致。
```cpp
static inline void __list_add(struct list_head *new,
                  struct list_head *prev,
                  struct list_head *next)
{
    next->prev = new;
    new->next = next;
    new->prev = prev;
    prev->next = new;
}

```

# WARNING: "__aeabi_uldivmod" [xx.ko] undefined!
使用的是32位定点数编译器，不支持除数为unsigned long long。

#  warning: 'xxx' defined but not used [-Wunused-function]
使用`__maybe_unused`，
```c
//linux/compiler-gcc.h
#define __maybe_unused  __attribute__((unused))
```
或利用`__unused`来消除警告，
```clike
#ifndef __unused
#    define __unused			__attribute__((unused))
#endif
__unused static void axienet_systim_to_hwtstamp(struct axienet_local *lp,
				struct skb_shared_hwtstamps *shhwtstamps,
				u64 regval) 
```

# *.c:63:1: error: expected ';', identifier or '(' before 'unsigned'
![313](https://img-blog.csdnimg.cn/20200612213733256.png)
这是因为包含的头文件里某行代码缺少一个`;`导致的，很隐晦。

# 变量没有赋初值
```c
void *meta, *uninitialized_var(meta_mem);
```

# ISO C90 forbids mixed declarations and code
C语言变量的声明必须放在函数最开始地方。

# make: 警告：检测到时钟错误。您的创建可能是不完整的
由于两个系统时间不一样，这里应用并没有重新生成，导致应用无法执行，`make clean`没有完全清除，手动删除上次的应用，再编译即可。
```bash
$ make clean
$ make
make: Warning: File 'phytool' has modification time 12081777 s in the future
  CC      phytool.o
  CC      print_phy.o
  CC      print_mv6.o
make: 警告：检测到时钟错误。您的创建可能是不完整的。
$ sudo make install
make: Warning: File 'phytool' has modification time 12081771 s in the future
make: 警告：检测到时钟错误。您的创建可能是不完整的。
$ ls -l
总用量 156
-rw-r----- 1 storage storage 18091 9月  12  2017 LICENSE
-rw-r----- 1 storage storage  1000 9月  12  2017 Makefile
-rw-r----- 1 storage storage 25859 2月  17  2020 phytool
-rw-r----- 1 storage storage 10556 9月  12  2017 phytool.c
-rw-r----- 1 storage storage  1507 9月  12  2017 phytool.h
-rw-rw-r-- 1 storage storage 14792 9月  30 17:44 phytool.o
-rw-r----- 1 storage storage  6147 9月  12  2017 print_mv6.c
-rw-rw-r-- 1 storage storage 13432 9月  30 17:44 print_mv6.o
-rw-r----- 1 storage storage  4290 9月  12  2017 print_phy.c
-rw-rw-r-- 1 storage storage  8728 9月  30 17:44 print_phy.o
-rw-r----- 1 storage storage  2634 9月  12  2017 README.md
$ chmod +x phytool 
$ ./phytool 
-bash: ./phytool: 没有那个文件或目录
```

# gcc -E
-E选项：生成预处理文件。

# error: ‘O_DIRECT’ undeclared
在源文件的最顶端加上_GNU_SOURCE宏定义，或在编译时加在命令行上也可以。

# warnings "_BSD_SOURCE and _SVID_SOURCE are deprecated, use _DEFAULT_SOURCE"
> [Source Code FAQ for The Linux Programming Interface](http://man7.org/tlpi/code/faq.html)
> [glibc 2.20 outputs warnings for _BSD_SOURCE (Stg.h) on unknown archs](https://gitlab.haskell.org/ghc/ghc/issues/9185)

You are compiling the code on a system that has glibc 2.20 or later installed. In glibc 2.20, the _BSD_SOURCE and _SVID_SOURCE feature test macros were deprecated. They continue to expose the definitions that they exposed in earlier glibc versions, but their use produces the warning noted above. Instead, the _DEFAULT_SOURCE macro should be used. There are a few possible workarounds to avoid these warnings:

Replace all instances of #define _BSD_SOURCE and #define _SVID_SOURCE in the source code with #define _DEFAULT_SOURCE
Modify the Makefile.inc file to add -D_DEFAULT_SOURCE to the definition of the IMPL_CFLAGS macro.
Download the latest code tarball, which contains the fix described in the previous point.

# error: this statement may fall through [-Werror=implicit-fallthrough=]
case没有加break

# ev.c:(.text+0x5204): undefined reference to `floor'
缺少静态链接库libm.a，引入-lm

# error: unused parameter 'cb' [-Werror=unused-parameter]
-Wno-unused-parameter

# warning: ignoring return value of ‘copy_from_user’, declared with attribute warn_unused_result [-Wunused-result]
```bash
ccflags-y += -Wno-unused-result
```

# 查看内置宏
linux下，
```shell
j2@j2-pc:~/fdk_develop$ arm-xilinx-linux-gnueabi-gcc -dM -E - < /dev/null
j2@j2-pc:~/fdk_develop$ gcc -dM -E - < /dev/null
```
windows下mingw，
```bash
gcc -posix -E -dM - < nul
```

# warning: the frame size of 1736 bytes is larger than 1024 bytes [-Wframe-larger-than=]
局部变量太大，超过栈大小，将静态分配改为动态分配。

# undefined reference to `vermanInit'
```shell
arm-xilinx-linux-gnueabi-gcc -L. -lhw -o verprint verprint.o
verprint.o: In function `main':
verprint.c:(.text+0x1c): undefined reference to `vermanInit'
verprint.c:(.text+0x40): undefined reference to `vermanShow'
collect2: error: ld returned 1 exit status
Makefile:37: recipe for target 'verprint' failed
make: *** [verprint] Error 1
```
Makefile写错，
```Makefile
ok:    $(CC) $^ -L. -lhw -o $@
error: $(CC) -L. -lhw -o $^ $@
```

