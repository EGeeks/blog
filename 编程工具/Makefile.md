# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 条件判断
通过条件判断ifeq/ifneq实现跨平台通用，
```shell
ifeq ($(ARCH),powerpc)
	LIBCMEM_INC_DIRS := include
	LIBCMEM_INC_FILES := cmem.h cmemapi.h
	LIBCMEM_INC := $(addprefix $(LIBCMEM_INC_DIRS)/,$(LIBCMEM_INC_FILES))
	LIBCMEM_INC_FLAGS := $(addprefix -I ,$(LIBCMEM_INC_DIRS)) -I../usdpaa/include
	LIBCMEM_LIB_FLAGS := -L../usdpaa/lib_powerpc64 -lusdpaa_dma_mem
else
	LIBCMEM_INC_DIRS := include
	LIBCMEM_INC_FILES := cmem.h cmemapi.h
	LIBCMEM_INC := $(addprefix $(LIBCMEM_INC_DIRS)/,$(LIBCMEM_INC_FILES))
	LIBCMEM_INC_FLAGS := $(addprefix -I ,$(LIBCMEM_INC_DIRS))
	LIBCMEM_LIB_FLAGS := 
endif
```

# ignoring invalid character `#' in expression
参考[ERROR : arm-linux-ld:u-boot.lds:1: ignoring invalid character `#' in expression](https://blog.csdn.net/penglijiang/article/details/73466561)
```shell
powerpc-fsl-linux-ld.bfd:u-boot.lds:1: ignoring invalid character `#' in expression
powerpc-fsl-linux-ld.bfd:u-boot.lds:1: syntax error
Makefile:1200: recipe for target 'u-boot' failed
make: *** [u-boot] Error 1
info: work <build_workspace> build <ubootenvtool>
```
修改，
```c
//#define CONFIG_ID_EEPROM
/*#define CONFIG_ID_EEPROM*/
```

# \$@ \$^ \$<
Makefile的三个非常有用的变量。\$@为目标文件，\$^为所有的依赖文件，\$<为第一个依赖文件。Makefile有一个缺省规则，所有的 .o文件都是依赖与相应的.c文件。

# 调试打印
> [Makefile中的几个调试方法](https://blog.csdn.net/wlqingwei/article/details/44459139)

文中有错误，中间无逗号，info，warning和error都是静态的，无法动态输出。echo只能在target：后面的语句中使用，且前面是个TAB。
```shell
$(info "here add the debug info")
$(warning "here add the debug info")
$(error "here add the debug info")
$(info $(TARGET_DEVICE) )
@echo "start the compilexxxxxxxxxxxxxxxxxxxxxxx"
```

# Makefile 中:= ?= += =的区别
> [Makefile 中:= ?= += =的区别](https://www.cnblogs.com/wanqieddy/archive/2011/09/21/2184257.html)

 1. = 是最基本的赋值，make会将整个makefile展开后，再决定变量的值。
 2.  := 是覆盖之前的值，变量的值取决于它在makefile中的位置。 
 3. ?= 是如果没有被赋值过就赋予等号后面的值。
 4. += 是添加等号后面的值+= 是添加等号后面的值。

# Makefile指定输出目录
> [makefile 指定文件的生成目录](https://blog.csdn.net/javababy3/article/details/77918228) 

# Makefile内建函数
> [Makefile所有内嵌函数](https://www.cnblogs.com/lidabo/p/6185582.html)
> [make 内建函数](https://www.cnblogs.com/Jokeyyu/p/7805390.html)

# 通用Makefile
> [玩转Makefile | 一次编译多个目标](https://blog.csdn.net/yychuyu/article/details/79950414)
> [Linux下多文件夹编写Makefile详解](https://blog.csdn.net/qq_21792169/article/details/50448639)
> [多文件，多头文件时gcc与makefile的编写经验](https://blog.csdn.net/GMPY_Tiger/article/details/50903620)
> [编写包含多文件的Makefile以及Makefile的嵌套实验](https://blog.csdn.net/asure__cpp/article/details/44180157)
> [Makefile 自动搜索 c 和 cpp 文件, 并生成 .a 静态库文件](https://www.cnblogs.com/lzpong/p/9205736.html)
> [Makefile编译当前目录下所有c文件到共享库](https://blog.csdn.net/oujiangping/article/details/78667935)
> [一个简单的Makefile编译所有c代码文件为每个单独程序](https://blog.csdn.net/jinhangdev/article/details/80581408)

